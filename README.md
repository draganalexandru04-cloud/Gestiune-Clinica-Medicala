# Sistem de Gestiune a Bazei de Date pentru o Clinică Medicală (ClinicaMed)

## 1. Modelul Entităţi – Asociaţii (DEA)
Modelul Entităţi – Asociaţii permite reprezentarea informaţiilor despre structura bazelor de date folosind trei elemente de construcţie: entităţi, atribute ale entităţilor ṣi asocieri între entităţi.

* **Entităţile** modelează clase de obiecte concrete sau abstracte despre care se colectează informaţii, au existenţă independentă ṣi pot fi identificate în mod unic. Exemplu de entităţi în cadrul clinicii: `PACIENTI`, `MEDICI`, `PROGRAMARI`, `SPECIALIZARI`.
* **Atributele** modelează proprietăţi distincte ale entităţilor. De exemplu, entitatea `MEDICI` are ca atribute `id_medic` (cheie primară), `nume_medic`, `specializare` ṣi `telefon_medic`. 
* **Asociaţiile** reprezintă o conexiune între entităţi. Pentru entităţile A ṣi B, putem avea una din următoarele tipuri de asocieri:
    * **One-to-one (1-la-1):** O entitate din A este asociată cel mult unei entităţi din B şi invers.
    * **One-to-many (1-la-n):** O entitate din A este asociată cu oricâte entităţi din B şi o entitate din B este asociată cel mult unei entităţi din A (ex: între `SPECIALIZARI` ṣi `MEDICI`).
    * **Many-to-many (n-la-n):** O entitate din A este asociată cu oricâte entităţi din B şi invers (ex: între `PACIENTI` ṣi `MEDICI`, prin intermediul programărilor).

## 2. Descrierea Aplicației (ClinicaMed)
Se consideră subuniversul unei aplicații de gestiune pentru o clinică medicală, numită **ClinicaMed**. Pentru a realiza această aplicație este nevoie de o bază de date pentru a stoca toate informațiile necesare.
* **Programări:** Trebuie să cunoaștem pacientul care a solicitat serviciul, medicul care efectuează consultația, serviciul medical prestat, data programării și ora la care are loc aceasta.
* **Pacienți:** Ne interesează numele acestora, codul numeric personal (CNP) și numărul de telefon.
* **Medici:** Trebuie să cunoaștem numele complet, numărul de telefon și specializarea medicală din care fac parte.
* **Specializări:** Reținem denumirea acesteia pentru a putea clasifica personalul medical.
* **Servicii Medicale:** Trebuie să știm denumirea investigației și prețul aferent acestuia.

## 3. Modelul Relaţional al Datelor (MRD)
Pentru proiectarea logică a bazei de date vom folosi Modelul Relaţional al Datelor (MRD). O relație poate fi vazută ca un tabel bidimensional cu toate valorile atomice. Pentru a face trecerea de la DEA la MRD ținem cont de următoarele reguli:
* Entitățile devin tabele (relații);
* Fiecărei relații i se adaugă cheia surogat de tip autonumber;
* Atributele entităților devin atribute ale relațiilor;
* Asociațiile devin relații, cu câte o cheie străină către relațiile suport.

### Structura Bazei de Date:
**1. SPECIALIZARI** 
* `id_specializare`: cheie primară, AutoNumber (Long Integer).
* `nume_specializare`: Text (255), Not Null.

**2. MEDICI**
* `id_medic`: cheie primară, AutoNumber.
* `nume_medic`: Text (255), Not Null.
* `specializare`: cheie străină (Foreign Key), dependență către `id_specializare`.
* `telefon_med`: Text (50).

**3. PACIENTI** 
* `id_pacient`: cheie primară, AutoNumber.
* `nume_pacient`: Text (255), Not Null.
* `CNP`: Text (13), Not Null.
* `telefon_paci`: Text (50).

**4. SERVICII_MEDICALE**
* `id_serviciu`: cheie primară, AutoNumber.
* `denumire_serviciu`: Text (255), Not Null.
* `pret`: Currency / Number (Double), Not Null.

**5. PROGRAMARI** (Tabel de legătură pentru asocierea many-to-many)
* `id_programare`: cheie primară, AutoNumber.
* `pacient`: cheie străină către `PACIENTI`.
* `medic`: cheie străină către `MEDICI`.
* `serviciu`: cheie străină către `SERVICII_MEDICALE`.
* `data_programare`: Date/Time, Not Null.
* `ora_programare`: Text (50).

---

## 4. Interogări SQL (Queries) ce confirmă funcționalitatea

### 1. Programări pe specializare și dată
Interogarea selectează informaţiile necesare pentru afişarea programărilor efectuate la specializarea 'Cardiologie' pentru data de 10/06/2026.

    SELECT 
        p.nume_pacient,
        m.nume_medic,
        s.nume_specializare,
        sm.denumire_serviciu,
        sm.pret,
        pr.ora_programare
    FROM (((PROGRAMARI AS pr 
        INNER JOIN PACIENTI AS p ON pr.pacient = p.id_pacient) 
        INNER JOIN MEDICI AS m ON pr.medic = m.id_medic) 
        INNER JOIN SPECIALIZARI AS s ON m.specializare = s.id_specializare) 
        INNER JOIN SERVICII_MEDICALE AS sm ON pr.serviciu = sm.id_serviciu
    WHERE pr.data_programare = #10/06/2026# 
      AND s.nume_specializare = 'Cardiologie';

### 2. Istoricul consultațiilor per medic
Extrage istoricul complet al consultațiilor și intervențiilor efectuate de medicul 'Aron Alin', utilă pentru monitorizarea activității individuale.

    SELECT 
        m.nume_medic,
        p.nume_pacient,
        pr.data_programare,
        sm.denumire_serviciu, 
        sm.pret
    FROM ((PROGRAMARI AS pr
        INNER JOIN MEDICI AS m ON pr.medic = m.id_medic)
        INNER JOIN PACIENTI AS p ON pr.pacient = p.id_pacient)
        INNER JOIN SERVICII_MEDICALE AS sm ON pr.serviciu = sm.id_serviciu
    WHERE m.nume_medic = 'Aron Alin';

### 3. Statistică financiară: Venituri totale per medic
Calculează veniturile totale generate de fiecare medic în parte utilizând funcția SUM și clauza GROUP BY.

    SELECT 
        m.nume_medic,
        s.nume_specializare,
        Count(pr.id_programare) AS nr_programari,
        Sum(sm.pret) AS venit_total
    FROM ((PROGRAMARI AS pr
        INNER JOIN MEDICI AS m ON pr.medic = m.id_medic)
        INNER JOIN SPECIALIZARI AS s ON m.specializare = s.id_specializare)
        INNER JOIN SERVICII_MEDICALE AS sm ON pr.serviciu = sm.id_serviciu
    GROUP BY 
        m.nume_medic,
        s.nume_specializare;

### 4. Pacientul cu valoarea maximă a cheltuielilor
Deoarece Access nu permite imbricarea directă MAX(SUM()), problema a fost descompusă logic.

**Pasul 1 (Subquery4_1): Suma totală per pacient**

    SELECT 
        p.nume_pacient,
        Sum(sm.pret) AS total_plata
    FROM (PROGRAMARI AS pr
        INNER JOIN PACIENTI AS p ON pr.pacient = p.id_pacient)
        INNER JOIN SERVICII_MEDICALE AS sm ON pr.serviciu = sm.id_serviciu
    GROUP BY p.nume_pacient;

**Pasul 2 (Subquery4_2): Extragerea valorii maxime**

    SELECT 
        Max(total_plata) AS max_plata
    FROM Subquery4_1;

**Pasul 3 (Query4): Interogarea finală**

    SELECT 
        sq1.nume_pacient,
        sq1.total_plata
    FROM Subquery4_1 AS sq1, Subquery4_2 AS sq2
    WHERE sq1.total_plata = sq2.max_plata;

### 5. Filtrarea serviciilor premium
Selectează exclusiv acele servicii medicale care au un tarif strict mai mare de 150 RON.

    SELECT 
        p.nume_pacient,
        pr.data_programare,
        sm.denumire_serviciu,
        sm.pret
    FROM (PROGRAMARI AS pr
        INNER JOIN PACIENTI AS p ON pr.pacient = p.id_pacient)
        INNER JOIN SERVICII_MEDICALE AS sm ON pr.serviciu = sm.id_serviciu
    WHERE sm.pret > 150;
