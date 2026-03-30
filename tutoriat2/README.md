# 🏗️ LCD & LDD — Controlul și Definirea Datelor în SQL Oracle

---

## 1. Introducere & Context Conceptual

### De ce există tranzacțiile?

Imaginează-ți un transfer bancar:

```
Cont A: 1000 RON  →  scade cu 500  →  Cont A: 500 RON
Cont B: 200 RON   →  crește cu 500 →  Cont B: 700 RON
```

Dacă sistemul pică **între** cele două operații, banii dispar. Tranzacțiile garantează că fie **ambele** operații au loc, fie **niciuna**.

### Proprietățile ACID

| Proprietate | Descriere | Exemplu |
| :--- | :--- | :--- |
| **A**tomicity | Totul sau nimic | Transfer bancar complet sau deloc |
| **C**onsistency | Datele rămân valide | Soldul nu poate deveni negativ |
| **I**solation | Tranzacțiile nu se interferează | Doi utilizatori nu văd modificările celuilalt până la COMMIT |
| **D**urability | COMMIT = permanent | Chiar și după o pană de curent |

> 💡 **Isolation** este cel mai ușor de demonstrat în SQL Developer — deschide două sesiuni și arată că INSERT-ul dintr-una nu apare în cealaltă până la COMMIT.

### LCD vs LDD — diferența fundamentală

| | LCD (TCL) | LDD (DDL) |
| :--- | :--- | :--- |
| **Ce controlează** | *Datele* (rânduri) | *Structura* (tabele, coloane) |
| **Reversibil?** | ✅ Da (cu ROLLBACK) | ❌ Nu (COMMIT implicit) |
| **Exemple** | `INSERT`, `UPDATE`, `DELETE` | `CREATE`, `ALTER`, `DROP` |

---

## 2. Limbajul de Control al Datelor (LCD / TCL)

**LCD** gestionează **tranzacțiile** — unități logice de lucru formate dintr-o secvență de comenzi `INSERT`, `UPDATE`, `DELETE` care trebuie să se execute **atomic** pentru a menține consistența bazei de date.

> 💡 **Prin „modificări" se înțelege aplicarea comenzilor:** `INSERT`, `UPDATE`, `DELETE`.

### Comenzile LCD

| Comandă | Descriere |
| :--- | :--- |
| `COMMIT` | Permanentizează **definitiv** modificările din tranzacția curentă |
| `ROLLBACK` | Anulează modificările până la **ultimul `COMMIT`** executat |
| `ROLLBACK TO savepoint` | Anulează modificările **doar până la punctul marcat** |
| `SAVEPOINT nume` | Marchează un punct intermediar în tranzacție |

> ⚠️ **Rularea unei comenzi LDD** (`CREATE`, `ALTER`, `DROP`, `TRUNCATE`, `RENAME`) **aduce implicit și rularea unui `COMMIT`** — permanentizează automat toate modificările anterioare!

---

## 3. Exemple pas cu pas — Comportamentul tranzacțiilor

Aceste exemple sunt esențiale pentru înțelegerea exactă a cum funcționează `COMMIT`, `ROLLBACK` și `SAVEPOINT`.

---

### Exemplul 1 — ROLLBACK după COMMIT nu mai are efect

```
TABEL_TEST ()
INSERT 1        → TABEL_TEST(1)
INSERT 2        → TABEL_TEST(1, 2)
COMMIT          → TABEL_TEST(1, 2)   ← modificările sunt permanentizate
ROLLBACK        → TABEL_TEST(1, 2)   ← ROLLBACK nu mai are ce anula (după COMMIT)
INSERT 3        → TABEL_TEST(1, 2, 3)
ROLLBACK        → TABEL_TEST(1, 2)   ← INSERT 3 este anulat
```

> 💡 `ROLLBACK` anulează modificările **doar până la ultimul `COMMIT`**. Un `ROLLBACK` imediat după `COMMIT` nu face nimic.

**De ce?** Gândește-te la COMMIT ca la salvarea unui fișier. Odată salvat, `Ctrl+Z` nu mai poate anula ceea ce a fost scris pe disc.

---

### Exemplul 2 — CREATE produce COMMIT implicit + SAVEPOINT cu ROLLBACK TO

```
TABEL_TEST ()
INSERT 1                    → TABEL_TEST(1)
INSERT 2                    → TABEL_TEST(1, 2)
CREATE (+ COMMIT implicit)  → TABEL_TEST(1, 2)   ← INSERT 1 și 2 permanentizate!
INSERT 3                    → TABEL_TEST(1, 2, 3)
SAVEPOINT P                 ← marchează punctul P
INSERT 4                    → TABEL_TEST(1, 2, 3, 4)
ROLLBACK TO P               → TABEL_TEST(1, 2, 3)  ← INSERT 4 anulat, INSERT 3 păstrat
```

> 💡 `ROLLBACK TO P` anulează **doar ce s-a întâmplat după** `SAVEPOINT P`. INSERT 3 rămâne deoarece este **înainte** de P.

**Greșeala frecventă:** mulți studenți cred că `ROLLBACK TO P` anulează și INSERT 3. Vizualizează-l ca o linie de timp — P este un marker pe linie, ROLLBACK TO P taie tot ce e **la dreapta** lui P.

---

### Exemplul 3 — ROLLBACK simplu ignoră SAVEPOINT-ul

```
TABEL_TEST ()
INSERT 1                    → TABEL_TEST(1)
INSERT 2                    → TABEL_TEST(1, 2)
CREATE (+ COMMIT implicit)  → TABEL_TEST(1, 2)
INSERT 3                    → TABEL_TEST(1, 2, 3)
SAVEPOINT P                 ← marchează punctul P
INSERT 4                    → TABEL_TEST(1, 2, 3, 4)
ROLLBACK                    → TABEL_TEST(1, 2)   ← ROLLBACK complet, până la ultimul COMMIT
```

> ⚠️ `ROLLBACK` (fără `TO`) ignoră complet `SAVEPOINT`-ul și merge **până la ultimul `COMMIT`** — în acest caz până la `CREATE` care a făcut COMMIT implicit.

**Comparație directă cu Exemplul 2:**

| | Exemplul 2 | Exemplul 3 |
| :--- | :--- | :--- |
| Comanda | `ROLLBACK TO P` | `ROLLBACK` |
| Rezultat | `(1, 2, 3)` | `(1, 2)` |
| Motivul | Anulează doar după P | Anulează până la ultimul COMMIT |

---

### Exemplul 4 — ROLLBACK TO un SAVEPOINT distrus de LDD

```
TABEL_TEST ()
INSERT 1                    → TABEL_TEST(1)
INSERT 2                    → TABEL_TEST(1, 2)
SAVEPOINT P                 ← marchează punctul P
CREATE (+ COMMIT implicit)  → TABEL_TEST(1, 2)   ← COMMIT implicit DISTRUGE savepoint-ul P!
INSERT 3                    → TABEL_TEST(1, 2, 3)
INSERT 4                    → TABEL_TEST(1, 2, 3, 4)
ROLLBACK TO P               → ❌ EROARE — savepoint-ul P nu mai există!
```

> ⚠️ Un `COMMIT` (inclusiv cel implicit din LDD) **distruge toate `SAVEPOINT`-urile** definite anterior. Nu mai poți face `ROLLBACK TO P` după un `COMMIT`.

**Mesajul de eroare Oracle:**
```
ORA-01086: savepoint 'P' never established in this session or is invalid
```

---

### Rezumat reguli tranzacții

```
COMMIT           → permanentizează TOT + distruge toate SAVEPOINT-urile
ROLLBACK         → anulează TOT până la ultimul COMMIT
ROLLBACK TO P    → anulează doar ce e DUPĂ savepoint P
LDD (CREATE etc) → COMMIT implicit automat → distruge SAVEPOINT-urile anterioare
```

---

## 4. Demo SQL Developer

### Setup inițial

```sql
-- Creăm tabelul de test (LDD → COMMIT implicit)
CREATE TABLE TABEL_TEST (
    VAL NUMBER(10)
);

-- Verificăm că e gol
SELECT * FROM TABEL_TEST;
```

### Demo 1 — Două sesiuni simultane (Isolation)

**Sesiunea 1:**
```sql
INSERT INTO TABEL_TEST VALUES (1);
INSERT INTO TABEL_TEST VALUES (2);
-- NU facem COMMIT încă
SELECT * FROM TABEL_TEST;  -- vedem: 1, 2
```

**Sesiunea 2** (fereastră nouă în SQL Developer):
```sql
SELECT * FROM TABEL_TEST;  -- vedem: gol! Sesiunea 2 nu vede modificările necommise
```

**Sesiunea 1:**
```sql
COMMIT;
```

**Sesiunea 2:**
```sql
SELECT * FROM TABEL_TEST;  -- acum vedem: 1, 2
```

> 💡 Aceasta demonstrează proprietatea **Isolation** din ACID.

### Demo 2 — SAVEPOINT și ROLLBACK TO

```sql
DELETE FROM TABEL_TEST;
COMMIT;

INSERT INTO TABEL_TEST VALUES (10);
INSERT INTO TABEL_TEST VALUES (20);
SAVEPOINT sp1;
INSERT INTO TABEL_TEST VALUES (30);
INSERT INTO TABEL_TEST VALUES (40);
SELECT * FROM TABEL_TEST;  -- 10, 20, 30, 40

ROLLBACK TO sp1;
SELECT * FROM TABEL_TEST;  -- 10, 20  (30 și 40 anulate)

COMMIT;
```

### Demo 3 — COMMIT implicit din LDD distruge SAVEPOINT

```sql
DELETE FROM TABEL_TEST;
COMMIT;

INSERT INTO TABEL_TEST VALUES (100);
SAVEPOINT sp_test;
INSERT INTO TABEL_TEST VALUES (200);

-- Această comandă LDD face COMMIT implicit și distruge sp_test!
CREATE TABLE TEMP_TEST (x NUMBER);

INSERT INTO TABEL_TEST VALUES (300);

-- Încearcă rollback la savepoint-ul distrus:
ROLLBACK TO sp_test;
-- ORA-01086: savepoint 'SP_TEST' never established in this session or is invalid
```

### Cleanup după demo

```sql
DROP TABLE TABEL_TEST;
DROP TABLE TEMP_TEST;
```

---

## 5. Limbajul de Definire a Datelor (LDD / DDL)

**LDD** este utilizat pentru definirea și modificarea structurii obiectelor bazei de date.

| Comandă | Descriere |
| :--- | :--- |
| `CREATE` | Creează un obiect nou |
| `ALTER` | Modifică structura unui obiect existent |
| `DROP` | Șterge fizic un obiect și conținutul său |
| `TRUNCATE` | Șterge datele, păstrează structura |
| `RENAME` | Redenumește un obiect |

> ⚠️ Instrucțiunile LDD au **efect imediat** și **ireversibil** (COMMIT implicit) — nu pot fi anulate cu `ROLLBACK`.

### Reguli de Numire a Obiectelor

| Regulă | Detaliu |
| :--- | :--- |
| Începe cu **literă** | Obligatoriu |
| **Lungime maximă** | 30 caractere (8 pentru baza de date, 128 pentru link-uri) |
| **Caractere permise** | `A-Z`, `a-z`, `0-9`, `_`, `$`, `#` |
| **Unicitate** | Două obiecte ale aceluiași utilizator nu pot avea același nume |
| **Cuvinte rezervate** | Interzise ca identificatori |
| **Case-insensitive** | `EMPLOYEES` = `employees` = `Employees` |

### Tipuri de Date Principale

| Tip | Descriere | Când să folosești |
| :--- | :--- | :--- |
| `VARCHAR2(n)` | Șir cu lungime **variabilă**, max `n` octeți (max 4000) | Texte de lungime variabilă (nume, descrieri) |
| `CHAR(n)` | Șir cu lungime **fixă** de `n` octeți (max 2000, implicit 1) | Coduri fixe (ex: cod țară `'RO'`) |
| `NUMBER(p, s)` | Număr cu `p` cifre totale, dintre care `s` sunt zecimale | Prețuri, cantități, ID-uri |
| `DATE` | Dată calendaristică (01-01-4712 î.Hr. — 31-12-9999 d.Hr.) | Date, timestamp-uri |
| `LONG` | Șir cu lungime variabilă, max 2GB | Texte foarte lungi (rar folosit azi) |

**Exemple `NUMBER(p, s)`:**

| Declarație | Valoare stocată | Observație |
| :--- | :--- | :--- |
| `NUMBER(5, 2)` | `123.45` | ✅ OK — 3 cifre întreg + 2 zecimale |
| `NUMBER(5, 2)` | `1234.5` | ❌ Eroare — 4 cifre întreg depășește p-s=3 |
| `NUMBER(10)` | `1234567890` | ✅ OK — număr întreg de 10 cifre |
| `NUMBER(10, 2)` | `99999999.99` | ✅ OK — max 8 cifre întreg + 2 zecimale |

---

## 6. Crearea Tabelelor — `CREATE TABLE`

### Metoda 1 — Definiție explicită

```sql
CREATE TABLE ECHIPE (
    ID_ECHIPA       NUMBER(10),
    NUME_ECHIPA     VARCHAR2(10)  NOT NULL,   -- constrângere la nivel de COLOANĂ
    DATA_INFIINTARE DATE          DEFAULT SYSDATE,
    VENIT_ANUAL     NUMBER(10, 2),
    ABREVIERE_NUME  VARCHAR2(3)   CONSTRAINT VERIF_LG CHECK (LENGTH(ABREVIERE_NUME) = 3),
    --                                        ↑ constrângere la nivel de COLOANĂ cu nume dat

    CONSTRAINT CHEIE_PRIMARA PRIMARY KEY (ID_ECHIPA),  -- constrângere la nivel de TABEL
    CHECK (VENIT_ANUAL > 100000)                       -- constrângere la nivel de TABEL (fără nume → SYS_...)
);
```

```sql
CREATE TABLE JUCATORI (
    ID_JUCATOR   NUMBER(10),
    NUME_JUCATOR VARCHAR2(10),
    NUMAR_TRICOU NUMBER(10),
    ECHIPA_ID    NUMBER(10),
    CONSTRAINT CHEIE_PRIMARA_2 PRIMARY KEY (ID_JUCATOR),
    CONSTRAINT NR_TRICOU_UNIC  UNIQUE (NUMAR_TRICOU),
    CONSTRAINT FK_JUC_ECH      FOREIGN KEY (ECHIPA_ID)
                               REFERENCES ECHIPE(ID_ECHIPA)
);
```

### Metoda 2 — Creare din subcerere (`CREATE TABLE AS`)

```sql
CREATE TABLE COPIE_EMPLOYEES AS
    SELECT EMPLOYEE_ID, SALARY * 12
    FROM EMPLOYEES;
```

> 💡 Se creează un tabel cu **structura și datele** determinate de `SELECT`.  
> ⚠️ Din tabelul/tabelele sursă se preiau **doar constrângerile de tip `NOT NULL`**. Celelalte constrângeri (PK, FK, UNIQUE, CHECK) **nu se copiază**.

**Comparație metode:**

| Aspect | Metoda 1 (explicită) | Metoda 2 (AS SELECT) |
| :--- | :--- | :--- |
| Structura | Definită manual | Derivată din SELECT |
| Date inițiale | Tabel gol | Copiază datele din sursă |
| Constrângeri | Toate definite manual | Doar NOT NULL preluat |
| Când e utilă | Tabele noi | Copii rapide, tabele de lucru |

### Comportamentul valorii DEFAULT

Când se face un `INSERT` fără a specifica o valoare pentru o coloană:
- Dacă **nu există `DEFAULT`** → coloana primește `NULL`
- Dacă **există `DEFAULT`** → coloana primește valoarea default, **nu** `NULL`

```sql
-- DATA_INFIINTARE are DEFAULT SYSDATE
INSERT INTO ECHIPE (ID_ECHIPA, NUME_ECHIPA) VALUES (1, 'Real');
-- DATA_INFIINTARE va fi automat completată cu data curentă

-- Forțarea NULL chiar dacă există DEFAULT:
INSERT INTO ECHIPE (ID_ECHIPA, NUME_ECHIPA, DATA_INFIINTARE) VALUES (2, 'Barca', NULL);
-- DATA_INFIINTARE va fi NULL (DEFAULT este ignorat când specificăm explicit NULL)
```

---

## 7. Constrângeri — Analiză Detaliată

### Niveluri de definire

| Constrângere | Nivel coloană | Nivel tabel | Observație |
| :--- | :---: | :---: | :--- |
| `NOT NULL` | ✅ Da | ❌ Nu | **Doar** la nivel de coloană |
| `UNIQUE` | ✅ Da | ✅ Da | La nivel de tabel dacă e compusă |
| `PRIMARY KEY` | ✅ Da | ✅ Da | La nivel de tabel dacă e compusă |
| `FOREIGN KEY` | ❌ Nu | ✅ Da | **Doar** la nivel de tabel |
| `CHECK` | ✅ Da | ✅ Da | La nivel de coloană **nu poate referi altă coloană** |

> 💡 **Constrângerile la nivel de coloană** pot lucra **doar** cu coloana în dreptul căreia sunt definite.  
> 💡 **Constrângerile la nivel de tabel** pot lucra cu **mai multe coloane** (ex: PK compuse, CHECK pe mai multe coloane).

### Naming și unicitate

```
Constrângere cu CONSTRAINT <nume>   → salvată cu numele dat
Constrângere fără CONSTRAINT <nume> → salvată cu un nume generat automat (SYS_...)
```

> ⚠️ **Constrângerile trebuie să aibă nume unic la nivelul serverului, nu doar al tabelului.**  
> Dacă în `ECHIPE` există o constrângere numită `CHEIE_PRIMARA`, nu mai poți folosi același nume în niciun alt tabel.

### PRIMARY KEY compusă — când și de ce

O cheie primară **compusă** folosește **mai multe coloane** împreună pentru a identifica unic un rând.

```sql
-- Exemplu: un jucător poate juca pentru mai multe echipe în perioade diferite
CREATE TABLE CONTRACTE (
    ID_JUCATOR  NUMBER(10),
    ID_ECHIPA   NUMBER(10),
    DATA_START  DATE,
    DATA_SFARSIT DATE,
    SALARIU     NUMBER(10, 2),
    -- PK compusă — combinația jucator + echipa + data_start este unică
    CONSTRAINT PK_CONTRACT PRIMARY KEY (ID_JUCATOR, ID_ECHIPA, DATA_START)
);
```

> ⚠️ PK compusă **nu poate fi definită la nivel de coloană** — trebuie la nivel de tabel.

### FOREIGN KEY — detalii și opțiuni ON DELETE

```sql
CONSTRAINT FK_JUC_ECH FOREIGN KEY (ECHIPA_ID)
    REFERENCES ECHIPE(ID_ECHIPA)
    [ON DELETE CASCADE | ON DELETE SET NULL]
```

| Opțiune | Efect la ștergerea din tabelul părinte | Când să folosești |
| :--- | :--- | :--- |
| *(fără opțiune)* | ❌ Eroare — nu se poate șterge | Date critice, relații stricte |
| `ON DELETE CASCADE` | Șterge automat și înregistrările din tabelul copil | Relații de dependență (ex: comenzi → detalii comenzi) |
| `ON DELETE SET NULL` | Setează FK-ul la `NULL` în tabelul copil | Relații opționale (ex: angajat → departament) |

> ⚠️ **FOREIGN KEY** poate face referință **doar** la coloane care în tabelul referențiat au constrângere de tip `PRIMARY KEY` sau `UNIQUE`.

**Demo — erori intenționate în SQL Developer:**

```sql
-- ❌ EROARE: FK spre coloană fără PK/UNIQUE
CREATE TABLE TEST_FK (
    ID NUMBER(10),
    ECHIPA_NUME VARCHAR2(10),
    CONSTRAINT FK_GRESIT FOREIGN KEY (ECHIPA_NUME)
        REFERENCES ECHIPE(NUME_ECHIPA)  -- NUME_ECHIPA nu e PK/UNIQUE!
);
-- ORA-02270: no matching unique or primary key for this column-list

-- ❌ EROARE: CHECK la nivel de coloană referențiază altă coloană
CREATE TABLE TEST_CHECK (
    PRET_MIN NUMBER(10),
    PRET_MAX NUMBER(10) CHECK (PRET_MAX > PRET_MIN)  -- nu poți referi PRET_MIN aici!
);
-- ORA-02438: Column check constraint cannot reference other columns

-- ✅ CORECT: CHECK la nivel de tabel
CREATE TABLE TEST_CHECK (
    PRET_MIN NUMBER(10),
    PRET_MAX NUMBER(10),
    CHECK (PRET_MAX > PRET_MIN)  -- la nivel de tabel: poți referi orice coloană
);
```

---

## 8. Modificarea Tabelelor — `ALTER TABLE`

### Adăugare coloană

```sql
ALTER TABLE JUCATORI
ADD NR_GOLURI NUMBER(10) DEFAULT 0;
```

> ⚠️ Nu se poate specifica poziția — coloana nouă devine automat **ultima** în structură.

### Modificare coloană (`MODIFY`)

```sql
-- ❌ Modificare tip de date → posibilă DOAR dacă nu există valori non-NULL sau tabelul e gol
ALTER TABLE JUCATORI
MODIFY NR_GOLURI VARCHAR2(10);

-- ❌ Micșorare dimensiune → posibilă DOAR dacă nu există valori non-NULL sau tabelul e gol
ALTER TABLE JUCATORI
MODIFY NR_GOLURI NUMBER(5);

-- ✅ Creșterea dimensiunii → posibilă ORICÂND
ALTER TABLE JUCATORI
MODIFY NR_GOLURI VARCHAR2(15);
```

**Rezumat reguli `MODIFY`:**

| Modificare | Condiție |
| :--- | :--- |
| Mărire dimensiune | ✅ Oricând |
| Micșorare dimensiune | ⚠️ Doar dacă coloana are doar `NULL` sau tabelul e gol |
| Modificare tip de date | ⚠️ Doar dacă valorile coloanei sunt `NULL` |
| `CHAR` ↔ `VARCHAR2` | ⚠️ Doar dacă valorile sunt `NULL` sau dimensiunea nu se modifică |
| Modificare `DEFAULT` | ✅ Oricând (afectează doar inserările viitoare) |

**De ce aceste restricții?** Oracle trebuie să poată converti datele existente la noul tip/dimensiune. Dacă există `'Hello World'` în coloană și vrei să o transformi în `NUMBER`, conversia e imposibilă → eroare.

### Ștergere coloană

```sql
ALTER TABLE JUCATORI
DROP COLUMN NR_GOLURI;
```

### Gestionarea constrângerilor

```sql
-- Adăugare constrângeri (mai multe simultan)
ALTER TABLE JUCATORI
ADD (
    CONSTRAINT VERIF_NR_TRICOU CHECK (NUMAR_TRICOU >= 1 AND NUMAR_TRICOU <= 99),
    CONSTRAINT LG_NUME         CHECK (LENGTH(NUME_JUCATOR) >= 5)
);

-- Ștergere constrângere
ALTER TABLE JUCATORI
DROP CONSTRAINT LG_NUME;

-- Dezactivare constrângere
ALTER TABLE JUCATORI
DISABLE CONSTRAINT VERIF_NR_TRICOU;

-- Reactivare constrângere
ALTER TABLE JUCATORI
ENABLE CONSTRAINT VERIF_NR_TRICOU;
```

> ⚠️ **Atenție la reactivare!** Când dezactivezi o constrângere și modifici datele, la reactivare **toate datele din tabel trebuie să respecte condiția constrângerii**. Dacă există date care nu o respectă, reactivarea va **eșua**.

**Scenariu tipic de problemă:**

```sql
-- 1. Dezactivăm constrângerea
ALTER TABLE JUCATORI DISABLE CONSTRAINT VERIF_NR_TRICOU;

-- 2. Inserăm date invalide (nr. tricou 150 — depășește limita 99)
INSERT INTO JUCATORI VALUES (1, 'Messi', 150, NULL);
COMMIT;

-- 3. Încercăm să reactivăm
ALTER TABLE JUCATORI ENABLE CONSTRAINT VERIF_NR_TRICOU;
-- ORA-02293: cannot validate (USER.VERIF_NR_TRICOU) - check constraint violated
```

### Demo complet ALTER TABLE în SQL Developer

```sql
-- Setup
CREATE TABLE PRODUSE (
    ID_PRODUS  NUMBER(10) CONSTRAINT PK_PROD PRIMARY KEY,
    NUME       VARCHAR2(20) NOT NULL,
    PRET       NUMBER(8, 2)
);

INSERT INTO PRODUSE VALUES (1, 'Laptop', 3500.00);
INSERT INTO PRODUSE VALUES (2, 'Mouse', 75.50);
COMMIT;

-- Adăugăm o coloană
ALTER TABLE PRODUSE ADD STOC NUMBER(10) DEFAULT 0;

-- Verificăm că DEFAULT a fost aplicat
SELECT * FROM PRODUSE;  -- STOC este 0 pentru rândurile existente? 
-- NU! DEFAULT se aplică doar la INSERT-uri viitoare. Rândurile existente primesc NULL la ADD COLUMN.
-- Excepție: Oracle 11g+ aplică DEFAULT la ADD COLUMN cu NOT NULL.

-- Mărim dimensiunea coloanei NUME
ALTER TABLE PRODUSE MODIFY NUME VARCHAR2(50);

-- Încercăm să micșorăm (va eșua dacă există date)
ALTER TABLE PRODUSE MODIFY NUME VARCHAR2(5);
-- ORA-01441: cannot decrease column length because some value in the column is too long

-- Adăugăm constrângere CHECK
ALTER TABLE PRODUSE ADD CONSTRAINT CHK_PRET CHECK (PRET > 0);

-- Cleanup
DROP TABLE PRODUSE;
```

---

## 9. Ștergerea și Redenumirea Tabelelor

### `DROP TABLE`

```sql
DROP TABLE ECHIPE;
```

> ⚠️ **Atenție la dependențe FK!** Înainte de a șterge un tabel, trebuie să fie **eliminate mai întâi constrângerile de tip `FOREIGN KEY`** din alte tabele care referențiază tabelul respectiv.

```sql
-- Varianta 1: șterge mai întâi FK-ul din tabelul copil
ALTER TABLE JUCATORI DROP CONSTRAINT FK_JUC_ECH;
DROP TABLE ECHIPE;

-- Varianta 2: șterge direct cu CASCADE CONSTRAINTS
DROP TABLE ECHIPE CASCADE CONSTRAINTS;
```

> 💡 `CASCADE CONSTRAINTS` șterge automat **toate constrângerile FK** din alte tabele care referențiază `ECHIPE`. Tabelele copil (ex: `JUCATORI`) **rămân**, doar constrângerea FK este eliminată.

### `TRUNCATE TABLE`

```sql
TRUNCATE TABLE JUCATORI;
```

> ⚠️ `TRUNCATE` este LDD — efect **definitiv**, **nu poate fi anulat** cu `ROLLBACK`.

### Comparație DROP vs. TRUNCATE vs. DELETE

| | `DROP TABLE` | `TRUNCATE TABLE` | `DELETE FROM` |
| :--- | :--- | :--- | :--- |
| **Șterge structura** | ✅ Da | ❌ Nu | ❌ Nu |
| **Șterge datele** | ✅ Da | ✅ Da | ✅ Da |
| **ROLLBACK posibil** | ❌ Nu (LDD) | ❌ Nu (LDD) | ✅ Da (LMD) |
| **Viteză** | Instant | Instant | Lent (rând cu rând) |
| **Trigger-e activate** | ❌ Nu | ❌ Nu | ✅ Da |
| **Spațiu eliberat** | ✅ Da | ✅ Da | ❌ Nu (necesită SHRINK) |

> 💡 `DELETE` e mai lent deoarece Oracle înregistrează fiecare ștergere în undo log (pentru ROLLBACK). `TRUNCATE` nu înregistrează rândurile individual.

### `RENAME`

```sql
RENAME ECHIPE TO CLUBURI;
```

> 💡 La redenumire sunt transferate automat constrângerile, indecșii și privilegiile.  
> ⚠️ Sunt **invalidate** toate obiectele dependente (vizualizări, sinonime, proceduri stocate).

---

## 10. Consultarea Dicționarului Datelor

| Vizualizare | Conținut |
| :--- | :--- |
| `USER_TABLES` | Informații complete despre tabelele utilizatorului curent |
| `TAB` | Informații de bază despre tabelele din schema curentă |
| `USER_CONSTRAINTS` | Informații despre constrângerile definite |
| `USER_CONS_COLUMNS` | Coloanele implicate în constrângeri |

```sql
-- Listă tabele
SELECT * FROM USER_TABLES;
SELECT * FROM TAB;

-- Constrângerile unui tabel
SELECT CONSTRAINT_NAME, CONSTRAINT_TYPE, TABLE_NAME, STATUS
FROM USER_CONSTRAINTS
WHERE TABLE_NAME = 'JUCATORI';

-- Coloanele implicate în constrângeri
SELECT CONSTRAINT_NAME, COLUMN_NAME, POSITION
FROM USER_CONS_COLUMNS
WHERE TABLE_NAME = 'JUCATORI'
ORDER BY CONSTRAINT_NAME, POSITION;

-- Join util: constrângeri + coloane
SELECT c.CONSTRAINT_NAME, c.CONSTRAINT_TYPE, cc.COLUMN_NAME, c.STATUS
FROM USER_CONSTRAINTS c
JOIN USER_CONS_COLUMNS cc ON c.CONSTRAINT_NAME = cc.CONSTRAINT_NAME
WHERE c.TABLE_NAME = 'JUCATORI'
ORDER BY c.CONSTRAINT_TYPE;
```

**Tipuri de constrângeri în `USER_CONSTRAINTS`:**

| `CONSTRAINT_TYPE` | Tip constrângere |
| :--- | :--- |
| `P` | PRIMARY KEY |
| `U` | UNIQUE |
| `R` | FOREIGN KEY (Referential) |
| `C` | CHECK (inclusiv NOT NULL) |

---

## 11. Exerciții Practice

> 🎯 Rezolvă exercițiile în ordine. Fiecare exercițiu se bazează pe cel anterior.

### Exercițiul 1 — Tranzacții (LCD)

Dată secvența de comenzi, precizați starea tabelului `T` la fiecare pas:

```sql
CREATE TABLE T (X NUMBER);   -- T: ()
INSERT INTO T VALUES (1);    -- T: ?
INSERT INTO T VALUES (2);    -- T: ?
SAVEPOINT A;                 -- T: ?
INSERT INTO T VALUES (3);    -- T: ?
CREATE TABLE T2 (Y NUMBER);  -- T: ?  ← atenție!
INSERT INTO T VALUES (4);    -- T: ?
ROLLBACK TO A;               -- T: ?  ← ce se întâmplă?
```

<details>
<summary>🔍 Răspuns</summary>

```
CREATE TABLE T    → T: ()          — LDD, COMMIT implicit (dar tabelul era gol)
INSERT 1          → T: (1)
INSERT 2          → T: (1, 2)
SAVEPOINT A       → T: (1, 2)      — marchează punctul A
INSERT 3          → T: (1, 2, 3)
CREATE TABLE T2   → T: (1, 2, 3)   — LDD → COMMIT implicit → DISTRUGE savepoint A!
INSERT 4          → T: (1, 2, 3, 4)
ROLLBACK TO A     → ❌ EROARE: ORA-01086 — savepoint A nu mai există!
```

</details>

---

### Exercițiul 2 — Crearea tabelelor

Creează tabelele `DEPARTAMENTE` și `ANGAJATI` cu următoarele cerințe:

**DEPARTAMENTE:**
- `ID_DEP` — cheie primară, număr de 5 cifre
- `NUME_DEP` — șir variabil de max 30 caractere, obligatoriu
- `BUGET` — număr cu max 12 cifre, 2 zecimale; trebuie să fie > 0
- `DATA_CREARE` — dată, implicit data curentă

**ANGAJATI:**
- `ID_ANG` — cheie primară
- `NUME` — max 50 caractere, obligatoriu, minim 3 caractere
- `SALARIU` — număr cu 2 zecimale; dacă există, trebuie să fie între 1000 și 50000
- `EMAIL` — max 100 caractere, unic
- `ID_DEP` — FK spre DEPARTAMENTE; la ștergerea departamentului, setează NULL

<details>
<summary>🔍 Soluție</summary>

```sql
CREATE TABLE DEPARTAMENTE (
    ID_DEP      NUMBER(5),
    NUME_DEP    VARCHAR2(30) NOT NULL,
    BUGET       NUMBER(12, 2),
    DATA_CREARE DATE DEFAULT SYSDATE,
    CONSTRAINT PK_DEP    PRIMARY KEY (ID_DEP),
    CONSTRAINT CHK_BUGET CHECK (BUGET > 0)
);

CREATE TABLE ANGAJATI (
    ID_ANG  NUMBER(10),
    NUME    VARCHAR2(50) NOT NULL,
    SALARIU NUMBER(10, 2),
    EMAIL   VARCHAR2(100),
    ID_DEP  NUMBER(5),
    CONSTRAINT PK_ANG      PRIMARY KEY (ID_ANG),
    CONSTRAINT CHK_NUME    CHECK (LENGTH(NUME) >= 3),
    CONSTRAINT CHK_SAL     CHECK (SALARIU BETWEEN 1000 AND 50000),
    CONSTRAINT UQ_EMAIL    UNIQUE (EMAIL),
    CONSTRAINT FK_ANG_DEP  FOREIGN KEY (ID_DEP)
                           REFERENCES DEPARTAMENTE(ID_DEP)
                           ON DELETE SET NULL
);
```

</details>

---

### Exercițiul 3 — ALTER TABLE

Pornind de la tabelele din Exercițiul 2, efectuează modificările:

1. Adaugă coloana `TELEFON VARCHAR2(15)` în `ANGAJATI`
2. Mărește dimensiunea coloanei `TELEFON` la `VARCHAR2(20)`
3. Adaugă constrângerea că `EMAIL` trebuie să conțină `@` (folosind `LIKE`)
4. Dezactivează constrângerea de salariu, inserează un angajat cu salariul 500, reactivează constrângerea — ce se întâmplă?

<details>
<summary>🔍 Soluție</summary>

```sql
-- 1. Adăugare coloană
ALTER TABLE ANGAJATI ADD TELEFON VARCHAR2(15);

-- 2. Mărire dimensiune
ALTER TABLE ANGAJATI MODIFY TELEFON VARCHAR2(20);

-- 3. CHECK pentru email
ALTER TABLE ANGAJATI
ADD CONSTRAINT CHK_EMAIL CHECK (EMAIL LIKE '%@%');

-- 4. Dezactivare + inserare date invalide + reactivare
ALTER TABLE ANGAJATI DISABLE CONSTRAINT CHK_SAL;

INSERT INTO ANGAJATI VALUES (1, 'Ion Popescu', 500, 'ion@test.com', NULL, NULL);
COMMIT;

ALTER TABLE ANGAJATI ENABLE CONSTRAINT CHK_SAL;
-- ORA-02293: cannot validate - check constraint violated
-- Trebuie să ștergem/modificăm rândul invalid înainte de reactivare!
```

</details>

---

### Exercițiul 4 — Dicționarul datelor

```sql
-- Verifică toate constrângerile din tabelul ANGAJATI
-- și afișează: numele constrângerii, tipul și coloana asociată

-- Scrie query-ul tău aici:
```

<details>
<summary>🔍 Soluție</summary>

```sql
SELECT 
    c.CONSTRAINT_NAME,
    CASE c.CONSTRAINT_TYPE
        WHEN 'P' THEN 'PRIMARY KEY'
        WHEN 'U' THEN 'UNIQUE'
        WHEN 'R' THEN 'FOREIGN KEY'
        WHEN 'C' THEN 'CHECK / NOT NULL'
    END AS TIP_CONSTR,
    cc.COLUMN_NAME,
    c.STATUS
FROM USER_CONSTRAINTS c
JOIN USER_CONS_COLUMNS cc ON c.CONSTRAINT_NAME = cc.CONSTRAINT_NAME
WHERE c.TABLE_NAME = 'ANGAJATI'
ORDER BY c.CONSTRAINT_TYPE, c.CONSTRAINT_NAME;
```

</details>

---

## 12. Rezumat Rapid & Capcane Frecvente

### Cheat Sheet

```
╔══════════════════════════════════════════════════════════════╗
║                         LCD (TCL)                            ║
╠══════════════════════════════════════════════════════════════╣
║  COMMIT           → permanentizează TOT + distruge SAVEPOINT ║
║  ROLLBACK         → anulează TOT până la ultimul COMMIT      ║
║  ROLLBACK TO P    → anulează doar ce e după savepoint P      ║
║  SAVEPOINT P      → marchează punct intermediar              ║
║  ⚠️  LDD → COMMIT implicit → distruge SAVEPOINT-urile!       ║
╠══════════════════════════════════════════════════════════════╣
║                         LDD (DDL)                            ║
╠══════════════════════════════════════════════════════════════╣
║  CREATE TABLE     → Metoda 1: explicit                       ║
║                     Metoda 2: AS SELECT (preia doar NOT NULL)║
║  ALTER TABLE      → ADD / MODIFY / DROP COLUMN               ║
║                  → ADD / DROP / ENABLE / DISABLE CONSTRAINT  ║
║  DROP TABLE       → șterge tot (atenție FK!)                 ║
║  TRUNCATE         → șterge datele, păstrează structura       ║
║  RENAME           → invalidează obiecte dependente           ║
╠══════════════════════════════════════════════════════════════╣
║                      CONSTRÂNGERI                            ║
╠══════════════════════════════════════════════════════════════╣
║  NOT NULL         → DOAR la nivel de coloană                 ║
║  FOREIGN KEY      → DOAR la nivel de tabel                   ║
║  FK referință     → DOAR la PK sau UNIQUE                    ║
║  CHECK la coloană → NU poate referi altă coloană             ║
║  Nume constrângeri→ UNIC la nivelul serverului!              ║
╚══════════════════════════════════════════════════════════════╝
```

### Capcane Frecvente

| ⚠️ Situație | ❌ Problema | ✅ Soluția |
| :--- | :--- | :--- |
| `ROLLBACK` după `COMMIT` | Nu are efect | `ROLLBACK` anulează doar ce e **după** ultimul `COMMIT` |
| `ROLLBACK TO P` după LDD | Eroare ORA-01086 | `COMMIT` (inclusiv implicit) distruge toate savepoint-urile |
| `CREATE TABLE AS SELECT` | PK, FK, UNIQUE, CHECK nu se copiază | Adaugă constrângerile manual cu `ALTER TABLE` |
| Două constrângeri cu același nume | Eroare Oracle | Numele sunt unice **la nivel de server**, nu tabel |
| `DISABLE` → modificare date → `ENABLE` | Eroare dacă datele nu respectă constrângerea | Verifică/corectează datele înainte de `ENABLE` |
| `DROP TABLE` pe tabel cu FK spre el | Eroare Oracle | Șterge FK-ul din copil sau folosește `CASCADE CONSTRAINTS` |
| `MODIFY` tip de date cu date existente | Eroare Oracle | Posibil doar dacă valorile sunt `NULL` |
| `CHECK` la coloană referențiază alta | Eroare ORA-02438 | Mută constrângerea la nivel de tabel |
| Micșorare dimensiune cu date | Eroare Oracle | Posibil doar cu coloana goală sau tabelul gol |
| `ADD COLUMN` cu `DEFAULT` | Rândurile existente primesc NULL | În Oracle 11g+, DEFAULT + NOT NULL aplică valoarea |

---

