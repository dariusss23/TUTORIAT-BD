# 🗄️ Tabele, Relații, Diagrame, Normalizare

---

## 1. Ce este un ERD și de ce îl folosim?

**Diagrama Entitate-Relație (ERD)** este un instrument de modelare conceptuală care descrie structura unui sistem de date înainte de implementare. Scopul ei este să răspundă la întrebările: *Ce entități există în sistem? Cum se leagă între ele? Ce informații stocăm despre fiecare?*

ERD-ul este independent de tehnologie — nu contează dacă vei folosi Oracle, PostgreSQL sau MySQL. Este un limbaj comun între analiști, programatori și clienți.

Odată finalizat, ERD-ul se transformă relativ mecanic în **model relațional** (tabele, coloane, constrângeri), care la rândul lui devine cod SQL.

---

## 2. Entități

O **entitate** reprezintă orice concept bine definit despre care sistemul trebuie să stocheze informații: persoane, locuri, obiecte, evenimente, concepte abstracte. Se scrie cu majuscule în dreptunghi.

Există trei tipuri principale:

---

### Entitate Strong (tare)

Există independent în sistem. Are o cheie primară proprie care o identifică unic fără ajutorul altor entități.

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│    STUDENT   │     │     CURS     │     │   PROFESOR   │
└──────────────┘     └──────────────┘     └──────────────┘
```

---

### Entitate Dependentă (slabă / weak entity)

Nu poate exista fără o altă entitate. Cheia ei primară este **compusă** — include în mod obligatoriu cheia primară a entității de care depinde, plus cel puțin un atribut propriu.

Reprezentată cu dreptunghi cu linii duble `╔══╗`.

```
┌──────────┐              ╔══════════════╗
│  PROIECT │──────────────║     TASK     ║
└──────────┘              ╚══════════════╝
  #id_proiect     PK: (#id_proiect, #nr_task)
```

Dacă ștergi proiectul, toate task-urile lui dispar. Asta e esența entității dependente.

Alte exemple tipice:
- `EPISOD` depinde de `SERIAL`
- `BILET` depinde de `CALATORIE` sau `TREN`
- `ATESTARE` depinde de `PROGRAM`

---

### Subentitate și Superentitate (IS-A / Generalizare)

Când mai multe entități au atribute comune, le putem grupa sub o **superentitate**. Fiecare **subentitate** moștenește toate atributele superentității și adaugă atribute proprii specifice.

Se reprezintă prin dreptunghiuri incluse în dreptunghiul superentității.

```
┌──────────────────────────────────────────────┐
│                    CONT                      │
│                                              │
│ 1               1(0)  ┌─────────────────┐    │
│─────────ISA───────────│      CURENT     │    │
│                       └─────────────────┘    │
│ 1               1(0)  ┌─────────────────┐    │
│─────────ISA───────────│     ECONOMII    │    │ 
│                       └─────────────────┘    │
│                                              │
└──────────────────────────────────────────────┘
```

**Cardinalitățile relației IS-A:**
- `1` pe partea superentității → orice cont de economii/curent are caracteristicile generale ale unui cont
- `1(0)` pe partea subentității → un cont poate fi cont curent, dar nu e obligatoiu sa fie curent

---

## 3. Atribute

Atributele descriu proprietățile unei entități sau ale unei relații.

---

### Atribut simplu

Valoare atomică, indivizibilă. Stocată direct.

```
STUDENT: #id_student, nume, prenume, data_nasterii, email
```

---

### Atribut compus

Format din mai multe sub-atribute. Nu se stochează ca întreg — se stochează fiecare component.

```
ADRESA → strada + număr + oraș + județ + cod_poștal
NUME   → prenume + nume_familie
```

---

### Atribut derivat

Nu se stochează fizic. Se calculează din alte atribute la momentul interogării.

```
varsta        → calculată din data_nasterii
vechime       → calculată din data_angajarii
total_comanda → calculată din suma(pret * cantitate)
```

În Oracle, atributele derivate se implementează de regulă prin **view-uri** sau expresii în `SELECT`, nu ca coloane stocate.

---

### Atribut multivaluat

Poate avea mai multe valori pentru aceeași entitate. **Nu se poate stoca direct** — necesită tabel separat.

```
ANGAJAT: telefoane → [0721111111, 0722222222, 0733333333]
PRODUS:  culori_disponibile → [roșu, albastru, verde]
```

Implementare:
```
TELEFOANE_ANGAJAT(#id_angajat, #telefon, tip)
  FK: id_angajat → ANGAJAT(id_angajat)
```

---

### Atribut cheie

Identifică unic instanțele entității. Notat cu `PK` sau `#` în modelele românești sau subliniat în notația Chen.

---

## 4. Chei Primare

---

### Proprietăți obligatorii

| Proprietate | Descriere |
|-------------|-----------|
|  **Unică** | Nicio două rânduri nu au aceeași valoare |
|  **Stabilă** | Nu se schimbă în timp (email, nume, adresa se pot schimba!) |
|  **Minimală** | Fără atribute inutile |
|  **Non-nulă** | Mereu are o valoare |

---

### Cheie naturală vs. Cheie surogat

**Cheia naturală** provine din datele reale ale domeniului:

| Exemplu | Context |
| :--- | :--- |
| ISBN | Cărți |
| CNP | Persoane fizice în România |
| CUI | Companii înregistrate în România |
| IBAN | Conturi bancare |
| Cod tren (`IR 1581`) | Transport feroviar |

Avantaj — nu adaugă coloană extra, are semnificație în lumea reală.

Dezavantaj — poate deveni instabilă (coduri de produs se schimbă), poate să nu fie garantat unică în toate contextele.

**Cheia surogat** este generată artificial de sistem:

```sql
-- Oracle: secvență
CREATE SEQUENCE seq_studenti START WITH 1 INCREMENT BY 1;
INSERT INTO studenti VALUES (seq_studenti.NEXTVAL, 'Popescu', 'Ion');

-- MySQL/PostgreSQL: auto-increment
id INT AUTO_INCREMENT PRIMARY KEY

-- UUID
'550e8400-e29b-41d4-a716-446655440000'
```

Avantaj — garantat unică, stabilă, simplă numeric.

Dezavantaj — nu are semnificație de business; UUID aleator degradează performanța indexului.

---

### UUID și indexul B-tree

Cheile primare sunt stocate intern ca **B-tree** (arbore de căutare echilibrat). Înregistrările sunt sortate pe disc după valoarea cheii.

Dacă cheile sunt **secvențiale** (1, 2, 3...), înregistrările noi se adaugă la final — o singură pagină de disc accesată per inserare.

Dacă cheile sunt **aleatoare** (UUID v4), fiecare inserare ajunge într-o poziție aleatoare în arbore — o altă pagină de disc de fiecare dată. La volum mare, asta înseamnă sute de operații I/O inutile per secundă.

**Soluție:** UUID secvențial (ULID, UUID v7) — parte secvențială la început + parte aleatoare la final. Păstrează unicitatea globală fără penalitatea de performanță.

---

### Cheie simplă vs. Cheie compusă

**Cheie simplă** — un singur atribut:
```
DEPARTAMENT(#id_dept, nume, buget)
```

**Cheie compusă** — mai multe atribute împreună. Apare în două situații clasice:

*La entități dependente:*
```
TRANZACTIE(#id_cont, #nr_tranzactie, suma, tip, data_ora)
  ↑ tranzacția nr. 547 din contul RO49AAAA... — fără cont, numărul e inutil
```

*La tabele asociative (M:N):*
```
INSCRIS(#id_student, #id_curs, data_inscriere, nota_finala)
```

---

## 5. Relații

O **relație** exprimă o asociere între două sau mai multe entități. Este de obicei un verb (lucrează_în, conduce, participă_la, aparține_de).

**Cum se implementează în SQL:**
- Relație 1:M → coloană FK în tabelul de pe partea `M`
- Relație 1:1 → coloană FK + UNIQUE constraint
- Relație M:N → tabel asociativ nou cu PK compusă

---

### Aritatea relației

Numărul entităților implicate:

**Unară (reflexivă)** — o entitate cu ea însăși:
```
             supervizează
ANGAJAT ────────────────────── ANGAJAT
(manager_id referă employee_id din același tabel)
```

**Binară** — două entități; cel mai comun caz:
```
             participă
STUDENT ─────────────────── CURS
```

**Ternară** — trei entități simultan, reprezentată cu **romb**:
```
        ┌──────────┐
        │ STUDENT  │── M(0)
        └──────────┘      \
                           ◇ SPONSORIZARE ── M(0) ── ┌──────────┐
                          /                           │ COMPANIE │
        ┌──────────┐── M(0)                           └──────────┘
        │ PROGRAM  │
        └──────────┘
```

O relație ternară **nu este echivalentă** cu trei relații binare — pierd informația despre combinația celor trei entități simultan.

---

## 6. Cardinalitățile Relațiilor

Cardinalitatea se notează pe **fiecare capăt** al arcului. Răspunde la două întrebări distincte pentru fiecare capăt.

---

### Cele două tipuri de cardinalitate

**Cardinalitate opțională (maximă)** — răspunde la „**poate**?":
> Câți angajați *pot* lucra într-un departament? → mai mulți → `M`

**Cardinalitate obligatorie (minimă)** — răspunde la „**trebuie**?":
> Câți angajați *trebuie* să lucreze într-un departament? → cel puțin 1 → `(1)`

Combinând: `M(1)` — unul sau mai mulți angajați per departament.

---

### Cele 4 simboluri

| Simbol | Citire | Participare | Multiplicitate |
| :---: | :--- | :--- | :--- |
| `1(1)` | exact una, obligatoriu | obligatorie | singular |
| `1(0)` | zero sau una, opțional | opțională | singular |
| `M(1)` | una sau mai multe, obligatoriu | obligatorie | multiplu |
| `M(0)` | zero sau mai multe, opțional | opțională | multiplu |

Când max = min, se poate scrie un singur simbol. Exemplu: dacă un episod aparține **exact** unui serial (cel puțin 1, cel mult 1), se scrie `1` în loc de `1(1)`.

---

### Metoda de determinare

Fixezi **o** entitate și te uiți spre cealaltă:

```
MEDIC ──── consultă ──── PACIENT

Capătul PACIENT (câți pacienți per medic fixat?):
  Poate?   → mai mulți → M
  Trebuie? → un medic poate să nu aibă niciun pacient → (0)
  → M(0)

Capătul MEDIC (câți medici per pacient fixat?):
  Poate?   → mai mulți (un pacient e consultat de mai mulți) → M
  Trebuie? → cel puțin unul → (1)
  → M(1)

 ┌───────┐ M(0)      consultă      M(1) ┌─────────┐
 │ MEDIC │──────────────────────────────│ PACIENT │
 └───────┘                              └─────────┘
```

Tipul relației (1:1, 1:N, M:N) = cardinalitățile **maxime** (literele, nu parantezele): `M` și `M` → **M:N**.

---

## 7. Normalizarea Bazelor de Date
 
**Normalizarea** elimină redundanța și anomaliile de actualizare dintr-un model relațional. Se aplică după ce ai un model brut și vrei să ajungi la tabele curate.
 
**Cele 3 anomalii pe care le rezolvă:**
- **Inserare** — nu poți adăuga un departament fără niciun angajat în el
- **Ștergere** — ștergând ultimul angajat dintr-un dept., pierzi și datele despre dept.
- **Actualizare** — schimbi un nume în 50 de rânduri în loc de unul; o greșeală lasă BD inconsistentă
 
---
 
### Dependența Funcțională
 
`A → B` — valoarea lui A determină **unic** valoarea lui B.
 
```
id_student → nume, email, an_studiu     ✅
id_curs    → denumire, credite          ✅
email      → id_student                 ✅ (email e unic)
an_studiu  → id_student                 ❌ mai mulți studenți în același an
```
 
**Dependență parțială** — un atribut depinde de **parte** din PK compusă:
```
INSCRIS(#id_student, #id_curs, nota, denumire_curs)
                                     ↑
                     depinde doar de id_curs → încălcare 2NF
```
 
**Dependență tranzitivă** — un atribut non-cheie depinde de alt atribut non-cheie:
```
STUDENT(#id_student, nume, id_facultate, nume_facultate)
                           └───────────────────────────┘
        id_student → id_facultate → nume_facultate   (tranzitiv) → încălcare 3NF
```
 
---
 
### 1NF — Forma Normală 1
 
**Condiție:** fiecare celulă conține o singură valoare atomică, fără liste sau coloane repetitive.
 
❌ **Atribute multivaluate:**
```
| id_student | nume    | telefoane                  |
|------------|---------|----------------------------|
| 1          | Popescu | 0721111111, 0722222222     |
| 2          | Ionescu | 0733333333                 |
```
 
✅ **Soluție:**
```
STUDENT(#id_student, nume, email)
TELEFON(#id_student, #telefon, tip)
  FK: id_student → STUDENT
```
 
❌ **Coloane repetitive:**
```
| id_student | nota_1 | materie_1 | nota_2 | materie_2 | nota_3 | materie_3 |
```
 
✅ **Soluție:**
```
NOTA(#id_student, #id_curs, nota, data_examen)
```
 
---
 
### 2NF — Forma Normală 2
 
**Condiție:** 1NF + fiecare atribut non-cheie depinde de **întreaga** PK compusă, nu de o parte.
 
> Se aplică **doar** când PK e compusă. PK simplă → 2NF automat satisfăcută.
 
❌ **Problemă:**
```
INSCRIS(#id_student, #id_curs, nota, data_inscriere,
        nume_student, email_student,
        denumire_curs, credite_curs)
 
  (id_student, id_curs) → nota, data_inscriere    ✅ completă
  id_student → nume_student, email_student         ❌ parțială
  id_curs    → denumire_curs, credite_curs         ❌ parțială
```
 
✅ **Soluție:**
```
STUDENT(#id_student, nume_student, email_student)
CURS(#id_curs, denumire_curs, credite_curs)
INSCRIS(#id_student, #id_curs, nota, data_inscriere)
  FK: id_student → STUDENT
  FK: id_curs    → CURS
```
 
---
 
### 3NF — Forma Normală 3
 
**Condiție:** 2NF + niciun atribut non-cheie nu depinde de alt atribut non-cheie (`cheie → A → B`).
 
❌ **Problemă:**
```
ANGAJAT(#id_angajat, nume, id_dept, nume_dept, oras_dept)
 
  id_angajat → id_dept                  ✅ directă
  id_dept    → nume_dept, oras_dept     ❌ tranzitivă (id_dept nu e PK)
```
 
Consecință: dacă dept. IT se mută din București în Cluj, trebuie actualizate rândurile **tuturor** angajaților săi — risc de inconsistență.
 
✅ **Soluție:**
```
ANGAJAT(#id_angajat, nume, salariu, id_dept)
  FK: id_dept → DEPARTAMENT
 
DEPARTAMENT(#id_dept, nume_dept, oras_dept, buget)
```
 
---

### BCNF — Boyce-Codd Normal Form

**Condiție:** 3NF + orice determinant funcțional este o **supercheie** (`A → B` → A trebuie să identifice unic tuplul).

Diferă de 3NF doar când există **mai multe chei candidate suprapuse**. O singură cheie candidată → BCNF = 3NF.

❌ **Problemă:**
```
REZERVARE(#student, #interval_orar, sala, building)

  (student, interval_orar) → sala, building    ✅ Cheie Candidată 1
  (sala, interval_orar)    → student           ✅ Cheie Candidată 2
  sala → building                              ❌ sala nu e supercheie → încălcare BCNF
```

Consecință: sala `S1` e în building `B1` → scriem `B1` de **100 de ori** (câte o rezervare). Dacă `S1` se mută în `B2`, trebuie modificate **100 de rânduri**.

✅ **Soluție:**
```
SALA(#sala, building, capacitate)

REZERVARE(#student, #interval_orar, sala)
  FK: sala → SALA
```

### 4NF — Forma Normală 4

**Condiție:** BCNF + nicio **dependență multivaluată non-trivială** (`A ↠ B` — A determină o mulțime de valori B, independent de restul coloanelor).

Apare când două atribute multivaluate **independente** coexistă în același tabel față de același determinant.

❌ **Problemă:**
```
PROFESOR_INFO(#profesor, #curs, #telefon)

  profesor ↠ curs     ❌ Ionescu predă {BD, Algoritmi, Rețele}
  profesor ↠ telefon  ❌ Ionescu are {0721..., 0733...}
  cursurile și telefoanele sunt complet INDEPENDENTE
```

Consecință: Ionescu adaugă un curs nou → trebuie adăugate **câte rânduri noi câte telefoane are**. Invers pentru un telefon nou.

```
| profesor | curs      | telefon    |
|----------|-----------|------------|
| Ionescu  | BD        | 0721111111 |
| Ionescu  | BD        | 0733333333 |  ← rând forțat
| Ionescu  | Algoritmi | 0721111111 |  ← rând forțat
| Ionescu  | Algoritmi | 0733333333 |  ← rând forțat
```

✅ **Soluție:**
```
PROFESOR_CURS(#profesor, #curs)

PROFESOR_TEL(#profesor, #telefon)
```

Acum adăugarea unui curs nou = 1 rând în `PROFESOR_CURS`, indiferent de câte telefoane are profesorul.

---

### 5NF — Forma Normală 5 (PJNF)

**Condiție:** 4NF + tabelul nu poate fi descompus în proiecții mai mici **fără a pierde informații** (*spurious tuples*).

Apare când există o **constrângere ciclică** între 3 entități — combinația (A, B, C) e validă doar dacă perechile (A,B), (A,C) și (B,C) sunt toate valide simultan.

❌ **Problemă:**
```
LIVRARE(#furnizor, #produs, #client)

  Regula: F livrează P lui C NUMAI DACĂ:
    F furnizează P  ȘI  F livrează lui C  ȘI  C cumpără P

| furnizor | produs | client  |
|----------|--------|---------|
| Alfa     | Laptop | Popescu |
| Alfa     | Mouse  | Ionescu |
| Beta     | Laptop | Ionescu |
```

Consecință: dacă descompui în **două** proiecții și faci JOIN, obții combinații invalide — ex. `(Alfa, Laptop, Ionescu)` deși Ionescu cumpără Mouse de la Alfa, nu Laptop.

✅ **Soluție:**
```
FURNIZOR_PRODUS(#furnizor, #produs)

FURNIZOR_CLIENT(#furnizor, #client)

PRODUS_CLIENT(#produs, #client)
```

Combinația validă se reconstituie prin JOIN pe toate trei:
```sql
SELECT fp.furnizor, fp.produs, fc.client
FROM FURNIZOR_PRODUS fp
JOIN FURNIZOR_CLIENT fc ON fp.furnizor = fc.furnizor
JOIN PRODUS_CLIENT   pc ON fp.produs   = pc.produs
                       AND fc.client   = pc.client
```

> ⚠️ Dacă ai nevoie de 5NF, de obicei e semn că ERD-ul nu a capturat corect constrângerile de business — merită revizuit mai întâi.

---

## 8. Denormalizarea

**Normalizarea maximă nu este întotdeauna optimă în practică.** Denormalizarea controlată înseamnă a introduce deliberat redundanță pentru a îmbunătăți performanța la citire.

> ⚠️ Denormalizarea se face **după** normalizare, ca optimizare conștientă — nu ca scurtătură de proiectare.

---

**Cazuri uzuale:**

**1. Rapoarte și analytics** — JOIN-uri pe mai multe tabele sunt lente; se creează tabele agregate:
```sql
-- Schema normalizată: 3 JOIN-uri la fiecare apel
SELECT s.nume, c.denumire, n.nota, p.nume AS profesor
FROM   NOTA n
JOIN   STUDENT  s ON n.id_student  = s.id_student
JOIN   CURS     c ON n.id_curs     = c.id_curs
JOIN   PROFESOR p ON c.id_profesor = p.id_profesor;

-- Denormalizat: tabel pregătit pentru raportare, fără JOIN-uri
SELECT nume_student, denumire_curs, nota, nume_profesor
FROM   RAPORT_NOTE;
```

**2. Data Warehousing** — modelul stea (*star schema*) denormalizează intenționat dimensiunile în jurul unui tabel central de fapte:
```
            TIMP
             │
PRODUS ── VANZARI ── CLIENT
             │
          LOCATIE
```

**3. Coloane calculate stocate** — `total_comanda` stocat fizic deși e derivat, pentru a evita recalcularea la fiecare query:
```sql
-- Normalizat: calculat la fiecare SELECT
SELECT SUM(cantitate * pret_unitar) FROM LINIE_COMANDA WHERE id_comanda = 101;

-- Denormalizat: stocat direct în tabel, actualizat prin trigger
ALTER TABLE COMANDA ADD total_comanda NUMBER(10,2);
```

---

| Context | Abordare |
| :--- | :---: |
| Aplicații OLTP (scrieri frecvente) | Normalizat |
| Raportare / Dashboard / Analytics | Tabel agregat denormalizat |
| Data Warehouse | Schemă stea |
