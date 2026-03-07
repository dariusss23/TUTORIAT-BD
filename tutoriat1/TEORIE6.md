# 🏗️ LCD & LDD — Controlul și Definirea Datelor în SQL Oracle

---

## 1. Limbajul de Control al Datelor (LCD / TCL)

**LCD** gestionează **tranzacțiile** — unități logice de lucru formate dintr-o secvență de comenzi `INSERT`, `UPDATE`, `DELETE` care trebuie să se execute **atomic** pentru a menține consistența bazei de date.

> 💡 **Prin „modificări" se înțelege aplicarea comenzilor:** `INSERT`, `UPDATE`, `DELETE`.

### Comenzile LCD:

| Comandă | Descriere |
| :--- | :--- |
| `COMMIT` | Permanentizează **definitiv** modificările din tranzacția curentă |
| `ROLLBACK` | Anulează modificările până la **ultimul `COMMIT`** executat |
| `ROLLBACK TO savepoint` | Anulează modificările **doar până la punctul marcat** |
| `SAVEPOINT nume` | Marchează un punct intermediar în tranzacție |

> ⚠️ **Rularea unei comenzi LDD** (`CREATE`, `ALTER`, `DROP`, `TRUNCATE`, `RENAME`) **aduce implicit și rularea unui `COMMIT`** — permanentizează automat toate modificările anterioare!

---

## 2. Exemple pas cu pas — Comportamentul tranzacțiilor

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

---

### Rezumat reguli tranzacții:

```
COMMIT           → permanentizează TOT + distruge toate SAVEPOINT-urile
ROLLBACK         → anulează TOT până la ultimul COMMIT
ROLLBACK TO P    → anulează doar ce e DUPĂ savepoint P
LDD (CREATE etc) → COMMIT implicit automat → distruge SAVEPOINT-urile anterioare
```

---

## 3. Limbajul de Definire a Datelor (LDD / DDL)

**LDD** este utilizat pentru definirea și modificarea structurii obiectelor bazei de date.

| Comandă | Descriere |
| :--- | :--- |
| `CREATE` | Creează un obiect nou |
| `ALTER` | Modifică structura unui obiect existent |
| `DROP` | Șterge fizic un obiect și conținutul său |
| `TRUNCATE` | Șterge datele, păstrează structura |
| `RENAME` | Redenumește un obiect |

> ⚠️ Instrucțiunile LDD au **efect imediat** și **ireversibil** (COMMIT implicit) — nu pot fi anulate cu `ROLLBACK`.

---

### Reguli de Numire a Obiectelor:

| Regulă | Detaliu |
| :--- | :--- |
| Începe cu **literă** | Obligatoriu |
| **Lungime maximă** | 30 caractere (8 pentru baza de date, 128 pentru link-uri) |
| **Caractere permise** | `A-Z`, `a-z`, `0-9`, `_`, `$`, `#` |
| **Unicitate** | Două obiecte ale aceluiași utilizator nu pot avea același nume |
| **Cuvinte rezervate** | Interzise ca identificatori |
| **Case-insensitive** | `EMPLOYEES` = `employees` = `Employees` |

---

## 4. Crearea Tabelelor — `CREATE TABLE`

### 4.1. Metoda 1 — Definiție explicită

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

---

### 4.2. Reguli importante despre constrângeri la creare

> ⚠️ **Constrângerile trebuie să aibă nume unic la nivelul serverului, nu doar al tabelului.**  
> Dacă în `ECHIPE` există o constrângere numită `CHEIE_PRIMARA`, nu mai poți folosi același nume în niciun alt tabel.

```
Constrângere cu CONSTRAINT <nume>  → salvată cu numele dat
Constrângere fără CONSTRAINT <nume> → salvată cu un nume generat automat (SYS_...)
```

**Niveluri de definire a constrângerilor:**

| Constrângere | Nivel coloană | Nivel tabel | Observație |
| :--- | :---: | :---: | :--- |
| `NOT NULL` | ✅ Da | ❌ Nu | **Doar** la nivel de coloană |
| `UNIQUE` | ✅ Da | ✅ Da | La nivel de tabel dacă e compusă |
| `PRIMARY KEY` | ✅ Da | ✅ Da | La nivel de tabel dacă e compusă |
| `FOREIGN KEY` | ❌ Nu | ✅ Da | **Doar** la nivel de tabel |
| `CHECK` | ✅ Da | ✅ Da | La nivel de coloană **nu poate referi altă coloană** |

> 💡 **Constrângerile la nivel de coloană** pot lucra **doar** cu coloana în dreptul căreia sunt definite.  
> 💡 **Constrângerile la nivel de tabel** pot lucra cu **mai multe coloane** (ex: PK compuse, CHECK pe mai multe coloane).

> ⚠️ **FOREIGN KEY** poate face referință **doar** la coloane care în tabelul referențiat au constrângere de tip `PRIMARY KEY` sau `UNIQUE`.

---

### 4.3. Comportamentul valorii DEFAULT

Când se face un `INSERT` fără a specifica o valoare pentru o coloană:
- Dacă **nu există `DEFAULT`** → coloana primește `NULL`
- Dacă **există `DEFAULT`** → coloana primește valoarea default, **nu** `NULL`

```sql
-- DATA_INFIINTARE are DEFAULT SYSDATE
INSERT INTO ECHIPE (ID_ECHIPA, NUME_ECHIPA) VALUES (1, 'Real');
-- DATA_INFIINTARE va fi automat completată cu data curentă
```

---

### 4.4. Metoda 2 — Creare din subcerere (`CREATE TABLE AS`)

```sql
CREATE TABLE COPIE_EMPLOYEES AS
    SELECT EMPLOYEE_ID, SALARY * 12
    FROM EMPLOYEES;
```

> 💡 Se creează un tabel cu **structura și datele** determinate de `SELECT`.  
> ⚠️ Din tabelul/tabelele sursă se preiau **doar constrângerile de tip `NOT NULL`**. Celelalte constrângeri (PK, FK, UNIQUE, CHECK) **nu se copiază**.

---

### 4.5. Tipuri de Date Principale

| Tip | Descriere |
| :--- | :--- |
| `VARCHAR2(n)` | Șir cu lungime **variabilă**, max `n` octeți (max 4000) |
| `CHAR(n)` | Șir cu lungime **fixă** de `n` octeți (max 2000, implicit 1) |
| `NUMBER(p, s)` | Număr cu `p` cifre totale, dintre care `s` sunt zecimale |
| `DATE` | Dată calendaristică (01-01-4712 î.Hr. — 31-12-9999 d.Hr.) |
| `LONG` | Șir cu lungime variabilă, max 2GB |

---

### 4.6. Detalii FOREIGN KEY — ON DELETE

```sql
CONSTRAINT FK_JUC_ECH FOREIGN KEY (ECHIPA_ID)
    REFERENCES ECHIPE(ID_ECHIPA)
    [ON DELETE CASCADE | ON DELETE SET NULL]
```

| Opțiune | Efect la ștergerea din tabelul părinte |
| :--- | :--- |
| *(fără opțiune)* | ❌ Eroare — nu se poate șterge dacă există înregistrări copil |
| `ON DELETE CASCADE` | Șterge automat și înregistrările din tabelul copil |
| `ON DELETE SET NULL` | Setează FK-ul la `NULL` în tabelul copil |

---

## 5. Modificarea Tabelelor — `ALTER TABLE`

### 5.1. Adăugare coloană

```sql
ALTER TABLE JUCATORI
ADD NR_GOLURI NUMBER(10) DEFAULT 0;
```

> ⚠️ Nu se poate specifica poziția — coloana nouă devine automat **ultima** în structură.

---

### 5.2. Modificare coloană (`MODIFY`)

```sql
-- ❌ Modificare tip de date → posibilă DOAR dacă nu există valori non-NULL sau tabelul e gol
ALTER TABLE JUCATORI
MODIFY NR_GOLURI VARCHAR2(10);

-- ❌ Micșorare dimensiune → posibilă DOAR dacă nu există valori non-NULL sau tabelul e gol
ALTER TABLE JUCATORI
MODIFY NR_GOLURI NUMBER(5);

-- ⚠️ Modificare VARCHAR2 → VARCHAR2 de dimensiune mai mică → posibilă DOAR dacă valorile
--    existente respectă noua dimensiune sau nu există valori non-NULL
ALTER TABLE JUCATORI
MODIFY NR_GOLURI VARCHAR2(5);

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

---

### 5.3. Ștergere coloană

```sql
ALTER TABLE JUCATORI
DROP COLUMN NR_GOLURI;
```

---

### 5.4. Adăugare constrângeri

```sql
-- Se pot adăuga mai multe constrângeri simultan
ALTER TABLE JUCATORI
ADD (
    CONSTRAINT VERIF_NR_TRICOU CHECK (NUMAR_TRICOU >= 1 AND NUMAR_TRICOU <= 99),
    CONSTRAINT LG_NUME         CHECK (LENGTH(NUME_JUCATOR) >= 5)
);
```

---

### 5.5. Ștergere constrângeri

```sql
ALTER TABLE JUCATORI
DROP CONSTRAINT LG_NUME;
```

---

### 5.6. Dezactivare / Activare constrângeri

```sql
ALTER TABLE JUCATORI
DISABLE CONSTRAINT VERIF_NR_TRICOU;

ALTER TABLE JUCATORI
ENABLE CONSTRAINT VERIF_NR_TRICOU;
```

> ⚠️ **Atenție la reactivare!** Când dezactivezi o constrângere și modifici datele, la reactivare **toate datele din tabel trebuie să respecte condiția constrângerii**. Dacă există date care nu o respectă, reactivarea va **eșua**.

---

## 6. Ștergerea Tabelelor

### 6.1. `DROP TABLE`

```sql
DROP TABLE ECHIPE;
```

> ⚠️ **Atenție la dependențe FK!** Înainte de a șterge un tabel, trebuie să fie **eliminate mai întâi constrângerile de tip `FOREIGN KEY`** din alte tabele care referențiază tabelul respectiv.  
> În acest caz, constrângerea `FK_JUC_ECH` din `JUCATORI` referențiază `ECHIPE` — trebuie eliminată înainte de `DROP TABLE ECHIPE`.

```sql
-- Varianta 1: șterge mai întâi FK-ul din tabelul copil
ALTER TABLE JUCATORI DROP CONSTRAINT FK_JUC_ECH;
DROP TABLE ECHIPE;

-- Varianta 2: șterge direct cu CASCADE CONSTRAINTS
DROP TABLE ECHIPE CASCADE CONSTRAINTS;
```

---

### 6.2. `TRUNCATE TABLE`

```sql
TRUNCATE TABLE JUCATORI;
```

> ⚠️ `TRUNCATE` este LDD — efect **definitiv**, **nu poate fi anulat** cu `ROLLBACK`.

---

### Comparație DROP vs. TRUNCATE vs. DELETE:

| | `DROP TABLE` | `TRUNCATE TABLE` | `DELETE FROM` |
| :--- | :--- | :--- | :--- |
| **Șterge structura** | ✅ Da | ❌ Nu | ❌ Nu |
| **Șterge datele** | ✅ Da | ✅ Da | ✅ Da |
| **ROLLBACK posibil** | ❌ Nu (LDD) | ❌ Nu (LDD) | ✅ Da (LMD) |

---

## 7. Redenumirea Obiectelor — `RENAME`

```sql
RENAME ECHIPE TO CLUBURI;
```

> 💡 La redenumire sunt transferate automat constrângerile, indecșii și privilegiile.  
> ⚠️ Sunt **invalidate** toate obiectele dependente (vizualizări, sinonime, proceduri stocate).

---

## 8. Consultarea Dicționarului Datelor

| Vizualizare | Conținut |
| :--- | :--- |
| `USER_TABLES` | Informații complete despre tabelele utilizatorului curent |
| `TAB` | Informații de bază despre tabelele din schema curentă |
| `USER_CONSTRAINTS` | Informații despre constrângerile definite |
| `USER_CONS_COLUMNS` | Coloanele implicate în constrângeri |

```sql
SELECT * FROM USER_TABLES;
SELECT * FROM TAB;

SELECT CONSTRAINT_NAME, CONSTRAINT_TYPE, TABLE_NAME
FROM USER_CONSTRAINTS
WHERE TABLE_NAME = 'JUCATORI';
```

---

## 9. Rezumat Rapid

```
LCD (TCL):
  COMMIT           → permanentizează TOT + distruge SAVEPOINT-urile
  ROLLBACK         → anulează TOT până la ultimul COMMIT
  ROLLBACK TO P    → anulează doar ce e după savepoint P
  SAVEPOINT P      → marchează punct intermediar
  ⚠️ LDD (CREATE, ALTER, DROP, TRUNCATE, RENAME) → COMMIT implicit!
  ⚠️ COMMIT (implicit sau explicit) → distruge toate SAVEPOINT-urile!

LDD (DDL):
  CREATE TABLE    → Metoda 1: definiție explicită
                    Metoda 2: AS SELECT (preia doar NOT NULL din sursă)
  ALTER TABLE     → ADD coloana / MODIFY coloana / DROP COLUMN
                  → ADD / DROP / ENABLE / DISABLE CONSTRAINT
  DROP TABLE      → șterge tot (atenție la FK din alte tabele!)
  TRUNCATE        → șterge datele, păstrează structura (ireversibil)
  RENAME          → redenumire (invalidează obiecte dependente)

Constrângeri:
  NOT NULL     → DOAR la nivel de coloană
  FOREIGN KEY  → DOAR la nivel de tabel; referință DOAR la PK sau UNIQUE
  UNIQUE/PK compusă → DOAR la nivel de tabel
  CHECK la coloană → NU poate referi altă coloană din tabel
  Nume constrângeri → UNIC la nivelul serverului (nu doar al tabelului)!
  Fără CONSTRAINT <nume> → nume generat automat (SYS_...)

DEFAULT:
  INSERT fără valoare specificată → NULL (sau DEFAULT dacă există)
```

---

## 10. Capcane Frecvente ⚠️

| Situație | Problemă | Soluție |
| :--- | :--- | :--- |
| `ROLLBACK` după `COMMIT` | Nu are efect | `ROLLBACK` anulează doar ce e **după** ultimul `COMMIT` |
| `ROLLBACK TO P` după un `COMMIT` (inclusiv implicit din LDD) | Eroare — savepoint distrus | `COMMIT` distruge toate savepoint-urile |
| `CREATE TABLE AS SELECT` | Nu se copiază PK, FK, UNIQUE, CHECK | Adaugă constrângerile manual cu `ALTER TABLE` |
| Două constrângeri cu același nume în tabele diferite | Eroare Oracle | Numele constrângerilor sunt unice **la nivel de server** |
| `DISABLE CONSTRAINT` + modificare date + `ENABLE` | Eroare dacă datele noi nu respectă constrângerea | Verifică datele înainte de reactivare |
| `DROP TABLE` pe tabel referențiat de FK | Eroare Oracle | Șterge FK-ul din tabelul copil sau folosește `CASCADE CONSTRAINTS` |
| `MODIFY` tip de date pe coloană cu date | Eroare Oracle | Posibil doar dacă valorile sunt `NULL` |
| `CHECK` la nivel de coloană referențiază altă coloană | Eroare Oracle | Mută constrângerea la nivel de tabel |
| Micșorare dimensiune coloană cu date | Eroare Oracle | Posibil doar cu coloana goală sau tabelul gol |
