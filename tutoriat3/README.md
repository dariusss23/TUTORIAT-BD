# ✏️ Limbajul de Manipulare a Datelor (LMD / DML)

---

## 1. Introducere

**LMD (Limbajul de Manipulare a Datelor)** permite operarea asupra datelor din tabele. Spre deosebire de LDD (care modifică *structura*), LMD modifică *conținutul* — rândurile din tabele.

### Comenzile LMD:

| Comandă | Descriere |
| :--- | :--- |
| `SELECT` | Regăsirea datelor |
| `INSERT` | Adăugarea de noi înregistrări |
| `UPDATE` | Modificarea valorilor din înregistrările existente |
| `DELETE` | Ștergerea înregistrărilor |
| `MERGE` | Adăugarea sau modificarea condiționată de înregistrări |

> 💡 Spre deosebire de LDD, comenzile LMD **pot fi anulate** cu `ROLLBACK` dacă nu s-a executat un `COMMIT`.

### De ce contează distincția LMD vs. LDD?

Aceasta este una dintre cele mai importante distincții în SQL Oracle:

- Comenzile **LMD** (`INSERT`, `UPDATE`, `DELETE`) sunt înregistrate în **undo log** — Oracle păstrează o copie a datelor dinainte de modificare, ceea ce permite `ROLLBACK`.
- Comenzile **LDD** (`CREATE`, `ALTER`, `DROP`, `TRUNCATE`) fac **COMMIT implicit** — nu pot fi anulate odată executate.
- O consecință importantă: dacă execuți un `CREATE TABLE` în mijlocul unei serii de `INSERT`-uri, Oracle face automat `COMMIT` pentru toate inserările anterioare, chiar dacă nu ai vrut asta.

```
INSERT → INSERT → CREATE TABLE → ROLLBACK
                  ↑
                  COMMIT implicit! INSERT-urile nu mai pot fi anulate.
```

---

## 2. Comanda INSERT

`INSERT` adaugă rânduri noi într-un tabel. Există mai multe forme ale sintaxei, fiecare potrivită pentru scenarii diferite.

### 2.1. Sintaxă generală

```sql
-- Forma 1: valori explicite
INSERT INTO tabel [(col1, col2, ...)]
VALUES (val1, val2, ...);

-- Forma 2: date din altă cerere
INSERT INTO tabel [(col1, col2, ...)]
subcerere;
```

---

### 2.2. INSERT cu listă de coloane (recomandat)

Specificarea explicită a coloanelor este considerată **bună practică** din mai multe motive:

```sql
-- ✅ Recomandat: cu lista de coloane
INSERT INTO ECHIPE (ID_ECHIPA, NUME_ECHIPA, VENIT_ANUAL)
VALUES (1, 'Real Madrid', 750000000);

-- ⚠️ Fără lista de coloane: trebuie specificate valori pentru TOATE coloanele, în ordinea definirii
INSERT INTO ECHIPE
VALUES (2, 'Barcelona', SYSDATE, 700000000, 'BAR');
```

**De ce să specifici întotdeauna lista de coloane?**

Dacă cineva modifică ulterior structura tabelului (adaugă o coloană cu `ALTER TABLE`), forma fără listă de coloane **va eșua** sau va insera date în coloana greșită. Forma cu listă de coloane rămâne corectă indiferent de modificările structurale.

```sql
-- Tabelul inițial: (ID, NUME, SALARIU)
INSERT INTO ANGAJATI VALUES (1, 'Popescu', 5000);  -- ✅ OK acum

-- Cineva adaugă o coloană: ALTER TABLE ANGAJATI ADD DATA_ANGAJARE DATE;
-- Tabelul devine: (ID, NUME, SALARIU, DATA_ANGAJARE)
INSERT INTO ANGAJATI VALUES (2, 'Ionescu', 6000);  -- ❌ EROARE: lipsește a 4-a valoare!

-- Cu lista de coloane: nu e afectat de modificarea structurii
INSERT INTO ANGAJATI (ID, NUME, SALARIU) VALUES (2, 'Ionescu', 6000);  -- ✅ OK
```

---

### 2.3. Tipuri de date în VALUES

Regulile de formatare a valorilor în `INSERT` sunt stricte și generează erori frecvente:

```sql
-- Caractere → între APOSTROFURI
INSERT INTO JUCATORI (ID_JUCATOR, NUME_JUCATOR)
VALUES (1, 'Messi');

-- Date calendaristice → între APOSTROFURI (format implicit DD-MON-YY sau DD-MM-YYYY)
INSERT INTO JUCATORI (ID_JUCATOR, NUME_JUCATOR, DATA_NASTERII)
VALUES (1, 'Messi', '24-06-1987');

-- Date cu format explicit (recomandat pentru claritate)
INSERT INTO JUCATORI (ID_JUCATOR, NUME_JUCATOR, DATA_NASTERII)
VALUES (1, 'Messi', TO_DATE('24-06-1987', 'DD-MM-YYYY'));

-- Numere → FĂRĂ apostrofuri
INSERT INTO JUCATORI (ID_JUCATOR, NUMAR_TRICOU, SALARIU)
VALUES (1, 10, 50000000);

-- Funcții Oracle în VALUES
INSERT INTO ECHIPE (ID_ECHIPA, NUME_ECHIPA, DATA_INFIINTARE)
VALUES (3, 'Juventus', SYSDATE);  -- SYSDATE = data și ora curentă

-- Expresii aritmetice în VALUES
INSERT INTO CONTRACTE (ID_CONTRACT, SALARIU_LUNAR, SALARIU_ANUAL)
VALUES (1, 5000, 5000 * 12);
```

> ⚠️ Valorile de tip **caracter** și **dată calendaristică** se includ între **apostrofuri** (`'`).
> ⚠️ **Nu** include valorile numerice între apostrofuri — deși Oracle face conversie implicită, este o practică proastă care poate ascunde erori.

**Eroare frecventă cu datele calendaristice:**

```sql
-- ❌ Poate eșua dacă formatul sesiunii nu este DD-MM-YYYY
INSERT INTO JUCATORI (DATA_NASTERII) VALUES ('1987-06-24');

-- ✅ Sigur indiferent de configurația sesiunii
INSERT INTO JUCATORI (DATA_NASTERII) VALUES (TO_DATE('1987-06-24', 'YYYY-MM-DD'));
```

---

### 2.4. Inserarea valorilor NULL

Există trei moduri de a insera `NULL`, fiecare cu utilizarea sa specifică:

```sql
-- Metoda 1 — Implicit: omiterea coloanei din lista de coloane
-- Când coloana nu are DEFAULT, primește NULL automat
INSERT INTO JUCATORI (ID_JUCATOR, NUME_JUCATOR)
VALUES (2, 'Ronaldo');
-- NUMAR_TRICOU, ECHIPA_ID, DATA_NASTERII vor fi NULL automat

-- Metoda 2 — Explicit cu cuvântul cheie NULL
-- Utilă când vrei să fii explicit sau când nu folosești lista de coloane
INSERT INTO JUCATORI (ID_JUCATOR, NUME_JUCATOR, NUMAR_TRICOU, ECHIPA_ID)
VALUES (3, 'Neymar', NULL, NULL);

-- Metoda 3 — Șir vid '' (echivalent cu NULL în Oracle)
-- Atenție: comportament diferit față de alte SGBD-uri (ex: MySQL)
INSERT INTO JUCATORI (ID_JUCATOR, NUME_JUCATOR, OBSERVATII)
VALUES (4, 'Mbappe', '');
-- '' este tratat ca NULL în Oracle — coloana va conține NULL, nu un șir gol
```

> ⚠️ **Particularitate Oracle:** În Oracle, șirul vid `''` este echivalent cu `NULL`. Aceasta diferă de standardul SQL și de alte SGBD-uri unde `''` și `NULL` sunt distincte.

**Interacțiunea NULL cu constrângerile:**

```sql
-- NOT NULL → nu permite inserarea de NULL prin niciuna din metode
CREATE TABLE TEST (
    ID   NUMBER PRIMARY KEY,
    NUME VARCHAR2(50) NOT NULL
);

INSERT INTO TEST (ID) VALUES (1);                -- ❌ EROARE: NUME este NOT NULL
INSERT INTO TEST (ID, NUME) VALUES (1, NULL);    -- ❌ EROARE: NUME este NOT NULL
INSERT INTO TEST (ID, NUME) VALUES (1, '');      -- ❌ EROARE: '' = NULL, tot NOT NULL
INSERT INTO TEST (ID, NUME) VALUES (1, 'Ion');   -- ✅ OK
```

---

### 2.5. INSERT din subcerere

Permite copierea datelor dintr-un tabel (sau mai multe) într-un alt tabel printr-un singur `INSERT`:

```sql
-- Copiere simplă dintr-un tabel în altul
INSERT INTO EMPLOYEES_BACKUP
    SELECT * FROM EMPLOYEES;

-- Copiere selectivă cu filtrare și transformare
INSERT INTO EMPLOYEES_BACKUP (EMPLOYEE_ID, FULL_NAME, ANNUAL_SALARY)
    SELECT EMPLOYEE_ID,
           FIRST_NAME || ' ' || LAST_NAME,
           SALARY * 12
    FROM EMPLOYEES
    WHERE DEPARTMENT_ID = 20;

-- Copiere din mai multe tabele prin JOIN
INSERT INTO RAPORT_ANGAJATI (ID_ANG, NUME, DEPT_NUME, SALARIU)
    SELECT e.EMPLOYEE_ID,
           e.FIRST_NAME || ' ' || e.LAST_NAME,
           d.DEPARTMENT_NAME,
           e.SALARY
    FROM EMPLOYEES e
    JOIN DEPARTMENTS d ON e.DEPARTMENT_ID = d.DEPARTMENT_ID
    WHERE e.SALARY > 5000;
```

> ⚠️ Coloanele din `SELECT` trebuie să corespundă ca **număr și tip** celor din lista `INTO`.
> ⚠️ Dacă nu există lista de coloane în `INTO`, subcererea trebuie să furnizeze valori pentru **toate** coloanele, în ordinea definirii.
> 💡 La `INSERT` din subcerere **nu se folosește** clauza `VALUES`.

**Comparație INSERT VALUES vs. INSERT subcerere:**

| Aspect | INSERT VALUES | INSERT subcerere |
| :--- | :--- | :--- |
| Rânduri inserate | 1 singur rând | Oricâte rânduri returnează SELECT |
| Sursă date | Valori literale / expresii | Alt tabel / cerere |
| Clauza VALUES | ✅ Prezentă | ❌ Absentă |
| Transformări | Limitate | Orice expresie SQL |

---

### 2.6. INSERT multi-tabel — `INSERT ALL`

`INSERT ALL` permite inserarea în **mai multe tabele simultan** pe baza unei singure subcereri sursă. Este util pentru distribuirea datelor în tabele specializate fără a parcurge sursa de mai multe ori.

#### Inserare necondiționată — toate rândurile merg în toate tabelele:

```sql
INSERT ALL
    INTO ANGAJATI_IT     (ID, NUME, SALARIU) VALUES (ID, NUME, SALARIU)
    INTO ANGAJATI_BACKUP (ID, NUME, SALARIU) VALUES (ID, NUME, SALARIU)
SELECT EMPLOYEE_ID ID, FIRST_NAME NUME, SALARY SALARIU
FROM EMPLOYEES
WHERE JOB_ID = 'IT_PROG';
-- Fiecare angajat IT va fi inserat în AMBELE tabele
```

#### Inserare condiționată cu `ALL` — fiecare condiție este evaluată independent:

```sql
-- ALL: condițiile sunt evaluate TOATE; o înregistrare poate intra în mai multe tabele
INSERT ALL
    WHEN SALARY < 5000  THEN INTO ANGAJATI_JUNIOR  VALUES (EMPLOYEE_ID, FIRST_NAME, SALARY)
    WHEN SALARY >= 5000 THEN INTO ANGAJATI_SENIOR  VALUES (EMPLOYEE_ID, FIRST_NAME, SALARY)
    WHEN DEPARTMENT_ID = 20 THEN INTO ANGAJATI_DEPT20 VALUES (EMPLOYEE_ID, FIRST_NAME, SALARY)
SELECT EMPLOYEE_ID, FIRST_NAME, SALARY, DEPARTMENT_ID
FROM EMPLOYEES;
-- Un angajat cu SALARY=6000 și DEPARTMENT_ID=20 va intra în ANGAJATI_SENIOR și ANGAJATI_DEPT20
```

#### Inserare condiționată cu `FIRST` — prima condiție adevărată câștigă:

```sql
-- FIRST: condițiile sunt evaluate ÎN ORDINE; o înregistrare intră DOAR în primul tabel potrivit
INSERT FIRST
    WHEN SALARY < 3000  THEN INTO ANGAJATI_JUNIOR  VALUES (EMPLOYEE_ID, FIRST_NAME, SALARY)
    WHEN SALARY < 6000  THEN INTO ANGAJATI_MID     VALUES (EMPLOYEE_ID, FIRST_NAME, SALARY)
    WHEN SALARY >= 6000 THEN INTO ANGAJATI_SENIOR  VALUES (EMPLOYEE_ID, FIRST_NAME, SALARY)
    ELSE INTO ANGAJATI_ALTII VALUES (EMPLOYEE_ID, FIRST_NAME, SALARY)
SELECT EMPLOYEE_ID, FIRST_NAME, SALARY
FROM EMPLOYEES;
-- Un angajat cu SALARY=2500: prima condiție (< 3000) e adevărată → merge în JUNIOR → stop
-- Un angajat cu SALARY=4000: prima condiție falsă, a doua (< 6000) adevărată → merge în MID → stop
```

> 💡 **`ALL`** — toate condițiile `WHEN` sunt evaluate; o înregistrare poate fi inserată în **mai multe tabele**.
> 💡 **`FIRST`** — condițiile sunt evaluate **în ordine**; o înregistrare intră **doar** în primul tabel cu condiție adevărată.

**Exemplu comparat ALL vs. FIRST cu același set de date:**

Presupunem un angajat cu `SALARY = 4500`:

```
Condiții:
  WHEN SALARY < 5000 THEN → TABEL_A
  WHEN SALARY < 8000 THEN → TABEL_B

INSERT ALL:   4500 < 5000 ✅ → TABEL_A; 4500 < 8000 ✅ → TABEL_B  → inserat în AMBELE
INSERT FIRST: 4500 < 5000 ✅ → TABEL_A; stop                      → inserat DOAR în TABEL_A
```

#### Clauza `ELSE` în `INSERT FIRST`:

```sql
-- ELSE prinde înregistrările care nu satisfac niciun WHEN
INSERT FIRST
    WHEN SALARY < 3000  THEN INTO JUNIOR  VALUES (EMPLOYEE_ID, SALARY)
    WHEN SALARY < 8000  THEN INTO MEDIU   VALUES (EMPLOYEE_ID, SALARY)
    ELSE INTO SENIOR VALUES (EMPLOYEE_ID, SALARY)  -- prinde SALARY >= 8000
SELECT EMPLOYEE_ID, SALARY FROM EMPLOYEES;
```

---

### 2.7. Erori posibile la INSERT

| Situație | Cod eroare | Exemplu |
| :--- | :--- | :--- |
| Valoare `NULL` pentru coloană `NOT NULL` | ORA-01400 | `INSERT INTO T (ID, NUME) VALUES (1, NULL)` când NUME e NOT NULL |
| Duplicate pe coloană `UNIQUE` sau `PRIMARY KEY` | ORA-00001 | Inserare cu ID deja existent |
| Valoare FK inexistentă în tabelul părinte | ORA-02291 | ID departament inexistent |
| Condiție `CHECK` nerespectată | ORA-02290 | Salariu negativ când CHECK (SALARY > 0) |
| Incompatibilitate tip de date | ORA-01722 | `VALUES ('text')` pentru coloană NUMBER |
| Valoare mai lungă decât dimensiunea coloanei | ORA-12899 | `'Superlungtext'` în VARCHAR2(5) |
| Număr greșit de coloane/valori | ORA-00947 | Liste de lungimi diferite |

---

## 3. Comanda UPDATE

`UPDATE` modifică valorile din rândurile existente ale unui tabel. Este probabil cea mai periculoasă comandă LMD dacă este folosită fără clauza `WHERE`.

### 3.1. Sintaxă generală

```sql
-- O singură coloană
UPDATE tabel [alias]
SET coloana = expresie
[WHERE conditie];

-- Mai multe coloane simultan
UPDATE tabel [alias]
SET col1 = expr1,
    col2 = expr2,
    col3 = expr3
[WHERE conditie];

-- Cu subcerere pe o coloană
UPDATE tabel [alias]
SET coloana = (subcerere_scalara)
[WHERE conditie];

-- Cu subcerere pe mai multe coloane
UPDATE tabel [alias]
SET (col1, col2) = (subcerere_cu_doua_coloane)
[WHERE conditie];
```

---

### 3.2. Exemple practice

```sql
-- Modificarea unui singur câmp, un singur rând (prin PK)
UPDATE EMPLOYEES
SET SALARY = 9000
WHERE EMPLOYEE_ID = 100;

-- Modificarea mai multor câmpuri simultan pentru un singur rând
UPDATE EMPLOYEES
SET SALARY    = 9000,
    JOB_ID    = 'IT_PROG',
    MANAGER_ID = 101
WHERE EMPLOYEE_ID = 100;

-- Modificare pe baza unei expresii (mărire procentuală)
UPDATE EMPLOYEES
SET SALARY = SALARY * 1.10
WHERE DEPARTMENT_ID = 20;

-- Modificare cu condiție compusă
UPDATE EMPLOYEES
SET SALARY = SALARY * 1.15
WHERE DEPARTMENT_ID = 20
  AND JOB_ID = 'IT_PROG'
  AND SALARY < 7000;

-- Update cu subcerere scalară (returnează o singură valoare)
UPDATE EMPLOYEES
SET SALARY = (SELECT AVG(SALARY) FROM EMPLOYEES WHERE DEPARTMENT_ID = 20)
WHERE DEPARTMENT_ID = 30;

-- Update cu subcerere pe mai multe coloane
-- Copiază salariul și job-ul de la angajatul 100 la angajatul 101
UPDATE EMPLOYEES
SET (SALARY, JOB_ID) = (
    SELECT SALARY, JOB_ID
    FROM EMPLOYEES
    WHERE EMPLOYEE_ID = 100
)
WHERE EMPLOYEE_ID = 101;

-- Setarea explicită a NULL (anularea valorii)
UPDATE EMPLOYEES
SET COMMISSION_PCT = NULL
WHERE DEPARTMENT_ID = 50;

-- Update cu subcerere corelată (bazată pe alt tabel)
UPDATE EMPLOYEES e
SET e.SALARY = e.SALARY * 1.20
WHERE e.DEPARTMENT_ID IN (
    SELECT d.DEPARTMENT_ID
    FROM DEPARTMENTS d
    WHERE d.LOCATION_ID = 1700
);
```

---

### 3.3. Reguli și capcane importante UPDATE

> ⚠️ **Fără clauza `WHERE`** — sunt afectate **toate liniile** din tabel!

```sql
-- ⚠️ Aceasta modifică SALARIUL TUTUROR angajaților la 5000!
UPDATE EMPLOYEES
SET SALARY = 5000;
-- Dacă ai 200 de angajați, toate cele 200 de rânduri sunt modificate.
```

**Bune practici înainte de UPDATE:**

```sql
-- Pas 1: Testează clauza WHERE cu un SELECT înainte de a face UPDATE
SELECT * FROM EMPLOYEES
WHERE DEPARTMENT_ID = 20;
-- Verifică că sunt exact rândurile pe care vrei să le modifici

-- Pas 2: Abia apoi fă UPDATE-ul
UPDATE EMPLOYEES
SET SALARY = SALARY * 1.10
WHERE DEPARTMENT_ID = 20;

-- Pas 3: Verifică rezultatul
SELECT * FROM EMPLOYEES
WHERE DEPARTMENT_ID = 20;

-- Pas 4: COMMIT dacă ești mulțumit, ROLLBACK dacă nu
COMMIT;  -- sau ROLLBACK;
```

**Comportamentul subcererilor scalare care returnează NULL:**

```sql
-- Dacă subcererea nu returnează niciun rând, SET coloana = NULL!
UPDATE EMPLOYEES
SET SALARY = (
    SELECT SALARY FROM EMPLOYEES WHERE EMPLOYEE_ID = 9999  -- ID inexistent
)
WHERE EMPLOYEE_ID = 100;
-- Rezultat: SALARY devine NULL pentru angajatul 100!
```

> 💡 Verifică întotdeauna că subcererea din `SET` returnează exact un rând înainte de a executa `UPDATE`.

---

### 3.4. UPDATE cu alias

Aliasul poate fi util în `UPDATE` mai ales când există subcereri corelate:

```sql
-- Fără alias: mai puțin clar în subcereri corelate
UPDATE EMPLOYEES
SET SALARY = SALARY * 1.10
WHERE DEPARTMENT_ID = (
    SELECT DEPARTMENT_ID FROM DEPARTMENTS WHERE DEPARTMENT_NAME = 'IT'
);

-- Cu alias: mai clar, mai ușor de citit
UPDATE EMPLOYEES e
SET e.SALARY = e.SALARY * 1.10
WHERE e.DEPARTMENT_ID = (
    SELECT d.DEPARTMENT_ID
    FROM DEPARTMENTS d
    WHERE d.DEPARTMENT_NAME = 'IT'
);
```

---

## 4. Comanda DELETE

`DELETE` șterge rânduri dintr-un tabel. La fel ca `UPDATE`, este periculoasă fără clauza `WHERE`.

### 4.1. Sintaxă generală

```sql
DELETE FROM tabel [alias]
[WHERE conditie];
```

---

### 4.2. Exemple practice

```sql
-- Ștergerea unui singur rând (prin PK — cea mai sigură formă)
DELETE FROM EMPLOYEES
WHERE EMPLOYEE_ID = 107;

-- Ștergerea mai multor rânduri cu condiție simplă
DELETE FROM EMPLOYEES
WHERE DEPARTMENT_ID = 20;

-- Ștergerea cu condiție compusă
DELETE FROM EMPLOYEES
WHERE DEPARTMENT_ID = 20
  AND SALARY < 3000;

-- Ștergerea cu subcerere
DELETE FROM EMPLOYEES
WHERE DEPARTMENT_ID IN (
    SELECT DEPARTMENT_ID
    FROM DEPARTMENTS
    WHERE UPPER(DEPARTMENT_NAME) LIKE '%SALES%'
);

-- Ștergerea cu subcerere corelată
DELETE FROM EMPLOYEES e
WHERE NOT EXISTS (
    SELECT 1
    FROM DEPARTMENTS d
    WHERE d.DEPARTMENT_ID = e.DEPARTMENT_ID
);
-- Șterge angajații care nu au un departament valid (orfani)

-- Ștergerea rândurilor duplicate (păstrează unul singur)
DELETE FROM ANGAJATI
WHERE ROWID NOT IN (
    SELECT MIN(ROWID)
    FROM ANGAJATI
    GROUP BY EMPLOYEE_ID
);
-- Păstrează primul rând inserat pentru fiecare EMPLOYEE_ID duplicat
```

---

### 4.3. Reguli și capcane importante DELETE

> ⚠️ **Fără clauza `WHERE`** — sunt șterse **toate liniile** din tabel!

```sql
-- ⚠️ Aceasta șterge TOȚI angajații!
DELETE FROM EMPLOYEES;
-- Tabelul rămâne gol, dar structura există, și ROLLBACK e posibil
```

**Erori frecvente la DELETE — constrângeri FK:**

```sql
-- ❌ Eroare: nu poți șterge un departament dacă există angajați în el
DELETE FROM DEPARTMENTS
WHERE DEPARTMENT_ID = 20;
-- ORA-02292: integrity constraint violated - child record found

-- ✅ Soluția 1: șterge mai întâi angajații din departament
DELETE FROM EMPLOYEES WHERE DEPARTMENT_ID = 20;
DELETE FROM DEPARTMENTS WHERE DEPARTMENT_ID = 20;

-- ✅ Soluția 2: dacă FK are ON DELETE CASCADE, ștergerea departamentului
--              șterge automat și angajații asociați
DELETE FROM DEPARTMENTS WHERE DEPARTMENT_ID = 20;  -- cascade dacă e definit
```

---

### 4.4. Comparație DELETE vs. TRUNCATE vs. DROP

| | `DELETE FROM` | `TRUNCATE TABLE` | `DROP TABLE` |
| :--- | :--- | :--- | :--- |
| **Tip** | LMD | LDD | LDD |
| **ROLLBACK posibil** | ✅ Da | ❌ Nu | ❌ Nu |
| **Clauza WHERE** | ✅ Da (selectiv) | ❌ Nu (tot) | ❌ Nu |
| **Păstrează structura** | ✅ Da | ✅ Da | ❌ Nu |
| **Viteză** | Lentă (rând cu rând) | Instant | Instant |
| **Declanșează triggere** | ✅ Da | ❌ Nu | ❌ Nu |
| **Spațiu eliberat** | ❌ Nu imediat | ✅ Da | ✅ Da |
| **Resetează secvențe** | ❌ Nu | ❌ Nu | ❌ Nu |

**Când să folosești ce:**

- `DELETE` — când vrei să ștergi selectiv sau ai nevoie de posibilitatea `ROLLBACK`
- `TRUNCATE` — când vrei să golești complet un tabel rapid și definitiv
- `DROP` — când vrei să elimini complet tabelul din baza de date

---

## 5. Comanda MERGE

`MERGE` combină `INSERT` și `UPDATE` într-o singură comandă. Este utilă când vrei să sincronizezi două tabele: actualizează rândurile care există deja și inserează rândurile noi.

### 5.1. Sintaxă generală

```sql
MERGE INTO tabel_destinatie [alias_dest]
USING sursa [alias_sursa]
ON (conditie_join)
WHEN MATCHED THEN
    UPDATE SET col1 = val1, col2 = val2
    [WHERE conditie_update]
    [DELETE WHERE conditie_delete]
WHEN NOT MATCHED THEN
    INSERT (col1, col2, ...)
    VALUES (val1, val2, ...)
    [WHERE conditie_insert];
```

### 5.2. Exemplu practic

```sql
-- Sincronizare ANGAJATI_BACKUP cu EMPLOYEES
-- Dacă angajatul există în backup → actualizează salariul
-- Dacă nu există → inserează-l

MERGE INTO ANGAJATI_BACKUP b
USING EMPLOYEES e
ON (b.EMPLOYEE_ID = e.EMPLOYEE_ID)
WHEN MATCHED THEN
    UPDATE SET b.SALARY    = e.SALARY,
               b.JOB_ID    = e.JOB_ID
    WHERE b.SALARY <> e.SALARY  -- actualizează doar dacă salariul s-a schimbat
WHEN NOT MATCHED THEN
    INSERT (b.EMPLOYEE_ID, b.FIRST_NAME, b.SALARY)
    VALUES (e.EMPLOYEE_ID, e.FIRST_NAME, e.SALARY)
    WHERE e.SALARY > 3000;  -- inserează doar angajații cu salariu > 3000
```

> 💡 `MERGE` este echivalentul operației `UPSERT` din alte SGBD-uri.
> 💡 Clauza `DELETE WHERE` din `WHEN MATCHED` permite și ștergerea rândurilor care îndeplinesc o condiție suplimentară după actualizare.

---

## 6. Rezumat Rapid

```
INSERT:
  INSERT INTO tabel (col1, col2) VALUES (val1, val2)     → 1 rând explicit
  INSERT INTO tabel (col1, col2) subcerere               → N rânduri din SELECT
  INSERT ALL INTO ... INTO ... subcerere                  → necondiționat (toate tabelele)
  INSERT ALL  WHEN cond THEN INTO ... subcerere           → ALL: evaluează toate WHEN
  INSERT FIRST WHEN cond THEN INTO ... subcerere          → FIRST: prima condiție adevărată

  Caractere și DATE → apostrofuri (' ')
  Numere → fără apostrofuri
  Date → TO_DATE('val', 'format') pentru siguranță
  NULL implicit → omite coloana din lista INTO
  NULL explicit → VALUES (..., NULL, ...)
  '' = NULL în Oracle

UPDATE:
  SET col = val [WHERE conditie]
  SET (col1, col2) = (subcerere) [WHERE conditie]
  ⚠️ Fără WHERE → modifică TOATE rândurile!
  ⚠️ Subcerere scalară fără rezultat → SET coloana = NULL!

DELETE:
  DELETE FROM tabel [WHERE conditie]
  ⚠️ Fără WHERE → șterge TOATE rândurile!
  ✅ ROLLBACK posibil (spre deosebire de TRUNCATE)
  ⚠️ FK fără ON DELETE CASCADE → ORA-02292 la ștergerea părintelui

MERGE:
  USING sursa ON (conditie_join)
  WHEN MATCHED THEN UPDATE ...
  WHEN NOT MATCHED THEN INSERT ...
```

---

## 7. Capcane Frecvente ⚠️

| Situație | Problemă | Soluție |
| :--- | :--- | :--- |
| `UPDATE` sau `DELETE` fără `WHERE` | Afectează **toate** rândurile din tabel | Testează întâi cu `SELECT` cu aceeași condiție |
| `INSERT` valoare numerică între apostrofuri | Conversie implicită inutilă, risc de eroare | Nu pune numerele între apostrofuri |
| `INSERT` dată fără `TO_DATE` | Poate eșua dacă formatul sesiunii diferă | Folosește `TO_DATE('val', 'format')` |
| `INSERT ALL` vs. `INSERT FIRST` confuzie | `ALL` poate insera același rând în mai multe tabele | Folosește `FIRST` dacă vrei inserare exclusivă |
| `INSERT` din subcerere cu `VALUES` | Sintaxă incorectă — `VALUES` nu se folosește cu subcerere | Elimină `VALUES`; subcererea furnizează direct valorile |
| `DELETE` cu FK fără CASCADE | ORA-02292 — există rânduri copil | Șterge mai întâi rândurile din tabelele copil |
| `UPDATE SET col = (subcerere fără rezultat)` | Coloana devine `NULL` | Verifică că subcererea returnează cel puțin un rând |
| Șiruri goale `''` tratate diferit | `''` = `NULL` în Oracle (nu șir gol!) | Conștientizează diferența față de MySQL/PostgreSQL |
| `MERGE` fără ambele clauze | `WHEN MATCHED` și `WHEN NOT MATCHED` sunt opționale individual, dar cel puțin una trebuie să fie prezentă | Verifică că ai cel puțin o clauză `WHEN` |
| `INSERT` fără lista de coloane după ALTER TABLE | Eroare la număr de valori sau valori în coloana greșită | Specifică întotdeauna lista de coloane |

---

## 8. Secvențe (SEQUENCE)

O **secvență** este un obiect al bazei de date care generează **întregi unici** în mod automat. Este utilizată frecvent pentru generarea valorilor de cheie primară sau alte coloane numerice unice.

> 💡 Secvențele sunt **independente de tabele** — aceeași secvență poate fi folosită pentru mai multe tabele.
> 💡 Valorile generate de o secvență sunt **unice în sesiune**, dar pot exista **goluri** (gaps) dacă se face `ROLLBACK` sau dacă există `NOCACHE`.

### 8.1. Crearea unei secvențe — `CREATE SEQUENCE`

```sql
CREATE SEQUENCE nume_secventa
    [INCREMENT BY n]
    [START WITH n]
    [{MAXVALUE n | NOMAXVALUE}]
    [{MINVALUE n | NOMINVALUE}]
    [{CYCLE | NOCYCLE}]
    [{CACHE n | NOCACHE}];
```

### Parametrii disponibili:

| Parametru | Descriere | Valoare implicită |
| :--- | :--- | :--- |
| `INCREMENT BY n` | Diferența dintre două numere generate succesiv (negativ → descrescător) | `1` |
| `START WITH n` | Numărul inițial al secvenței | `1` |
| `MAXVALUE n` | Valoarea maximă | `10^27` (asc.) / `-1` (desc.) |
| `MINVALUE n` | Valoarea minimă | `1` (asc.) / `-10^27` (desc.) |
| `CYCLE` / `NOCYCLE` | Reîncepe de la `MINVALUE` după atingerea `MAXVALUE` | `NOCYCLE` |
| `CACHE n` / `NOCACHE` | Câte numere să prealoce în memorie pentru performanță | `CACHE 20` |

**Despre `CACHE`:**

Oracle prealoca valorile secvenței în memorie (`CACHE 20` = 20 valori preîncărcate). Dacă serverul se oprește neașteptat, valorile din cache care nu au fost folosite se **pierd** — apar goluri în secvență. `NOCACHE` elimină golurile cauzate de restart, dar e mai lent (acces la disc la fiecare `NEXTVAL`).

---

### 8.2. Exemple de creare

```sql
-- Secvență simplă pentru ID-uri (cel mai comun caz)
CREATE SEQUENCE SEQ_JUCATORI
    START WITH 1
    INCREMENT BY 1
    NOCACHE
    NOCYCLE;

-- Secvență cu pas de 10 (pentru coduri de departamente 10, 20, 30...)
CREATE SEQUENCE SEQ_DEPT
    START WITH 10
    INCREMENT BY 10
    MAXVALUE 9000
    NOCYCLE
    CACHE 20;

-- Secvență descrescătoare
CREATE SEQUENCE SEQ_DESC
    START WITH 100
    INCREMENT BY -1
    MINVALUE 1
    NOCYCLE;

-- Secvență circulară (se reia după atingerea maximului)
CREATE SEQUENCE SEQ_CICLIC
    START WITH 1
    INCREMENT BY 1
    MAXVALUE 999
    CYCLE
    CACHE 10;
-- La atingerea valorii 999, următoarea valoare va fi 1 din nou
```

---

### 8.3. Utilizarea secvențelor — `NEXTVAL` și `CURRVAL`

Secvențele se folosesc prin două **pseudocoloane**:

| Pseudocoloană | Descriere |
| :--- | :--- |
| `nume_secv.NEXTVAL` | Returnează **următoarea valoare** a secvenței și o incrementează |
| `nume_secv.CURRVAL` | Returnează **valoarea curentă** (ultima generată cu `NEXTVAL`) |

> ⚠️ `NEXTVAL` trebuie apelat **cel puțin o dată** înainte de a putea folosi `CURRVAL` în sesiunea curentă.
> ⚠️ Fiecare apel la `NEXTVAL` **incrementează ireversibil** secvența, chiar și dacă tranzacția face `ROLLBACK` ulterior.

```sql
-- Demonstrație NEXTVAL și CURRVAL
SELECT SEQ_JUCATORI.NEXTVAL FROM DUAL;  -- rezultat: 1 (prima apelare)
SELECT SEQ_JUCATORI.CURRVAL FROM DUAL;  -- rezultat: 1 (valoarea curentă)
SELECT SEQ_JUCATORI.NEXTVAL FROM DUAL;  -- rezultat: 2
SELECT SEQ_JUCATORI.NEXTVAL FROM DUAL;  -- rezultat: 3
SELECT SEQ_JUCATORI.CURRVAL FROM DUAL;  -- rezultat: 3

-- Utilizare tipică în INSERT
INSERT INTO JUCATORI (ID_JUCATOR, NUME_JUCATOR, NUMAR_TRICOU)
VALUES (SEQ_JUCATORI.NEXTVAL, 'Messi', 10);
-- ID_JUCATOR = 4 (continuă de unde a rămas)

INSERT INTO JUCATORI (ID_JUCATOR, NUME_JUCATOR, NUMAR_TRICOU)
VALUES (SEQ_JUCATORI.NEXTVAL, 'Ronaldo', 7);
-- ID_JUCATOR = 5

-- Referențierea valorii generate (pentru a o folosi în al doilea INSERT)
INSERT INTO CONTRACTE (ID_CONTRACT, ID_JUCATOR, DATA_START)
VALUES (SEQ_CONTRACTE.NEXTVAL, SEQ_JUCATORI.CURRVAL, SYSDATE);
-- SEQ_JUCATORI.CURRVAL = 5 (ultimul ID generat pentru Ronaldo)

-- Utilizare în UPDATE
UPDATE JUCATORI
SET ID_JUCATOR = SEQ_JUCATORI.NEXTVAL
WHERE NUME_JUCATOR = 'Neymar';
```

---

### 8.4. Unde pot fi folosite pseudocoloanele

✅ **Permis:**

| Locație | Exemplu |
| :--- | :--- |
| Lista `SELECT` a cererilor **simple** (fără subcereri) | `SELECT SEQ.NEXTVAL FROM DUAL` |
| Lista `SELECT` a unei cereri sursă pentru `INSERT` | `INSERT INTO t SELECT SEQ.NEXTVAL, col FROM ...` |
| Clauza `VALUES` a comenzii `INSERT` | `VALUES (SEQ.NEXTVAL, 'text')` |
| Clauza `SET` a comenzii `UPDATE` | `SET col = SEQ.NEXTVAL` |

❌ **Interzis:**

| Locație | Motiv |
| :--- | :--- |
| Lista `SELECT` a unei **vizualizări** | Vizualizările nu pot folosi secvențe |
| `SELECT` cu `DISTINCT` | Ar putea genera valori nedeterminist |
| `SELECT` cu `GROUP BY` sau `HAVING` | Idem |
| `SELECT` cu `ORDER BY` | Idem |
| **Subcereri** în `SELECT`, `WHERE`, `UPDATE`, `DELETE` | Nu este permis în contexte imbricate |
| Clauza `DEFAULT` din `CREATE TABLE` sau `ALTER TABLE` | Nu este suportat (în versiuni < 12c) |

```sql
-- ❌ Eroare: NEXTVAL într-o subcerere din WHERE
SELECT * FROM EMPLOYEES
WHERE EMPLOYEE_ID = (SELECT SEQ_JUCATORI.NEXTVAL FROM DUAL);
-- ORA-02287: sequence number not allowed here

-- ❌ Eroare: NEXTVAL cu ORDER BY
SELECT SEQ_JUCATORI.NEXTVAL FROM EMPLOYEES ORDER BY EMPLOYEE_ID;
-- ORA-02287: sequence number not allowed here

-- ❌ Eroare: NEXTVAL în DEFAULT (Oracle < 12c)
CREATE TABLE TEST (
    ID NUMBER DEFAULT SEQ_JUCATORI.NEXTVAL
);
-- ORA-00984: column not allowed here

-- ✅ Corect: NEXTVAL în SELECT simplu
SELECT SEQ_JUCATORI.NEXTVAL FROM DUAL;

-- ✅ Corect: NEXTVAL în VALUES
INSERT INTO TEST (ID) VALUES (SEQ_JUCATORI.NEXTVAL);

-- ✅ Corect: NEXTVAL în SELECT sursă pentru INSERT
INSERT INTO TEST (ID, NUME)
SELECT SEQ_JUCATORI.NEXTVAL, FIRST_NAME FROM EMPLOYEES;
```

---

### 8.5. De ce apar goluri în secvență?

Golurile (gaps) în valorile generate sunt normale și așteptate. Apar din mai multe motive:

```
Cauza 1 — ROLLBACK:
  INSERT cu NEXTVAL (valoare = 5) → ROLLBACK
  → Secvența a avansat la 5, dar rândul nu există
  → Următorul NEXTVAL va fi 6, nu 5

Cauza 2 — CACHE și restart server:
  CACHE 20 → Oracle a prealoca valorile 1-20 în memorie
  → Serverul se oprește după ce s-au folosit 1-5
  → La restart, Oracle începe din nou de la 21 (valorile 6-20 se pierd)

Cauza 3 — Sesiuni multiple:
  Sesiunea A: NEXTVAL = 1
  Sesiunea B: NEXTVAL = 2
  Sesiunea A: NEXTVAL = 3
  → Valorile sunt atribuite în ordinea apelurilor, nu a COMMIT-urilor
```

> 💡 **Concluzie:** Secvențele garantează **unicitatea**, nu **consecutivitatea**. Dacă aplicația ta necesită numere consecutive fără goluri, secvențele nu sunt soluția potrivită (și nici o soluție simplă nu există în medii concurente).

---

### 8.6. Modificarea secvențelor — `ALTER SEQUENCE`

```sql
-- Modificarea pasului
ALTER SEQUENCE SEQ_JUCATORI INCREMENT BY 5;

-- Modificarea valorii maxime
ALTER SEQUENCE SEQ_JUCATORI MAXVALUE 100000;

-- Activarea opțiunii CYCLE
ALTER SEQUENCE SEQ_JUCATORI CYCLE;

-- Modificarea cache-ului
ALTER SEQUENCE SEQ_JUCATORI CACHE 50;

-- Dezactivarea cache-ului
ALTER SEQUENCE SEQ_JUCATORI NOCACHE;
```

> ⚠️ **Nu se poate modifica `START WITH`** cu `ALTER SEQUENCE` — valoarea de start este fixată la creare. Dacă vrei să „resetezi" o secvență, trebuie să o ștergi și să o recreezi.

---

### 8.7. Consultarea dicționarului datelor pentru secvențe

```sql
-- Secvențele utilizatorului curent (cel mai des folosit)
SELECT SEQUENCE_NAME, MIN_VALUE, MAX_VALUE, INCREMENT_BY,
       CYCLE_FLAG, CACHE_SIZE, LAST_NUMBER
FROM USER_SEQUENCES;

-- Toate secvențele accesibile utilizatorului
SELECT * FROM ALL_SEQUENCES;

-- Toate secvențele din baza de date (necesită privilegii DBA)
SELECT * FROM DBA_SEQUENCES;
```

**Coloana `LAST_NUMBER`** arată următoarea valoare care va fi generată (sau prima valoare din cache-ul curent dacă `CACHE` este activ).

---

### 8.8. Ștergerea secvențelor — `DROP SEQUENCE`

```sql
DROP SEQUENCE SEQ_JUCATORI;
```

> ⚠️ Ștergerea unei secvențe este **ireversibilă** (LDD → COMMIT implicit).
> ⚠️ Dacă secvența era folosită în INSERT-uri automate, acestea vor eșua după ștergere.

---

### 8.9. Rezumat Secvențe

```
CREATE SEQUENCE seq_name
    START WITH n        → valoare de start (implicit 1)
    INCREMENT BY n      → pasul (negativ → descrescător)
    MAXVALUE / MINVALUE → limite
    CYCLE / NOCYCLE     → reîncepe sau nu după limită
    CACHE n / NOCACHE   → numere preîncărcate (implicit 20)

Utilizare:
  seq.NEXTVAL  → următoarea valoare unică (incrementează secvența)
  seq.CURRVAL  → valoarea curentă (necesită NEXTVAL anterior în sesiune)

✅ Permis în: VALUES, SET, SELECT simplu, SELECT sursă pentru INSERT
❌ Interzis în: vizualizări, SELECT cu DISTINCT/GROUP BY/HAVING/ORDER BY,
               subcereri, DEFAULT din CREATE/ALTER TABLE (< Oracle 12c)

ALTER SEQUENCE → modifică parametrii (nu poate modifica START WITH)
DROP SEQUENCE  → ștergere permanentă

Goluri în secvență sunt normale (ROLLBACK, CACHE + restart, concurență)
```

---

### 8.10. Capcane frecvente cu secvențe ⚠️

| Situație | Problemă | Soluție |
| :--- | :--- | :--- |
| `CURRVAL` înainte de `NEXTVAL` | ORA-08002: secvența nu a fost încă utilizată în sesiune | Apelează `NEXTVAL` cel puțin o dată mai întâi |
| `NEXTVAL` în clauza `DEFAULT` (< 12c) | ORA-00984: coloana nu este permisă | Generează ID-ul în `INSERT` explicit |
| `NEXTVAL` în subcerere | ORA-02287: sequence number not allowed here | Mută `NEXTVAL` în `VALUES` sau în `SELECT`-ul principal |
| `NEXTVAL` cu `ORDER BY` | ORA-02287 | Nu combina `NEXTVAL` cu `ORDER BY` |
| Goluri după `ROLLBACK` | Valoarea este consumată chiar și la `ROLLBACK` | Secvențele nu revin — proiectate pentru unicitate, nu consecutivitate |
| Goluri după restart cu `CACHE` | Valorile din cache se pierd la oprirea serverului | Folosește `NOCACHE` dacă consecutivitatea e critică (cu impact de performanță) |
| Resetarea unei secvențe | `ALTER SEQUENCE` nu poate modifica `START WITH` | `DROP SEQUENCE` + `CREATE SEQUENCE` cu noile valori |

---

## 9. Exerciții Practice

> 🎯 Rezolvă exercițiile în ordine. Fiecare exercițiu se bazează pe structura tabelelor din exercițiile anterioare.

### Setup — Crearea tabelelor de lucru

```sql
CREATE TABLE DEPARTAMENTE_EX (
    ID_DEP   NUMBER(5) CONSTRAINT PK_DEP_EX PRIMARY KEY,
    NUME_DEP VARCHAR2(50) NOT NULL,
    BUGET    NUMBER(12, 2) CONSTRAINT CHK_BUGET_EX CHECK (BUGET > 0)
);

CREATE TABLE ANGAJATI_EX (
    ID_ANG  NUMBER(10) CONSTRAINT PK_ANG_EX PRIMARY KEY,
    NUME    VARCHAR2(50) NOT NULL,
    SALARIU NUMBER(10, 2) CONSTRAINT CHK_SAL_EX CHECK (SALARIU > 0),
    ID_DEP  NUMBER(5) CONSTRAINT FK_ANG_DEP_EX
                      REFERENCES DEPARTAMENTE_EX(ID_DEP)
                      ON DELETE SET NULL
);

CREATE SEQUENCE SEQ_ANG_EX
    START WITH 1
    INCREMENT BY 1
    NOCACHE
    NOCYCLE;
```

---

### Exercițiul 1 — INSERT de bază

Inserează datele de mai jos în tabelele create:

**DEPARTAMENTE_EX:**
| ID_DEP | NUME_DEP | BUGET |
| :--- | :--- | :--- |
| 10 | IT | 500000 |
| 20 | HR | 200000 |
| 30 | Sales | 350000 |

**ANGAJATI_EX** (folosind SEQ_ANG_EX pentru ID-uri):
| NUME | SALARIU | ID_DEP |
| :--- | :--- | :--- |
| Popescu Ion | 5000 | 10 |
| Ionescu Maria | 4500 | 20 |
| Georgescu Dan | 6000 | 10 |
| Stanescu Ana | 3500 | 30 |

<details>
<summary>🔍 Soluție</summary>

```sql
-- Departamente
INSERT INTO DEPARTAMENTE_EX (ID_DEP, NUME_DEP, BUGET) VALUES (10, 'IT', 500000);
INSERT INTO DEPARTAMENTE_EX (ID_DEP, NUME_DEP, BUGET) VALUES (20, 'HR', 200000);
INSERT INTO DEPARTAMENTE_EX (ID_DEP, NUME_DEP, BUGET) VALUES (30, 'Sales', 350000);
COMMIT;

-- Angajați (ID generat de secvență)
INSERT INTO ANGAJATI_EX (ID_ANG, NUME, SALARIU, ID_DEP)
VALUES (SEQ_ANG_EX.NEXTVAL, 'Popescu Ion', 5000, 10);

INSERT INTO ANGAJATI_EX (ID_ANG, NUME, SALARIU, ID_DEP)
VALUES (SEQ_ANG_EX.NEXTVAL, 'Ionescu Maria', 4500, 20);

INSERT INTO ANGAJATI_EX (ID_ANG, NUME, SALARIU, ID_DEP)
VALUES (SEQ_ANG_EX.NEXTVAL, 'Georgescu Dan', 6000, 10);

INSERT INTO ANGAJATI_EX (ID_ANG, NUME, SALARIU, ID_DEP)
VALUES (SEQ_ANG_EX.NEXTVAL, 'Stanescu Ana', 3500, 30);
COMMIT;
```

</details>

---

### Exercițiul 2 — INSERT cu erori intenționate

Încearcă fiecare comandă și explică de ce eșuează sau reușește:

```sql
-- A)
INSERT INTO ANGAJATI_EX (ID_ANG, NUME, SALARIU, ID_DEP)
VALUES (SEQ_ANG_EX.NEXTVAL, NULL, 5000, 10);

-- B)
INSERT INTO ANGAJATI_EX (ID_ANG, NUME, SALARIU, ID_DEP)
VALUES (SEQ_ANG_EX.NEXTVAL, 'Test', -100, 10);

-- C)
INSERT INTO ANGAJATI_EX (ID_ANG, NUME, SALARIU, ID_DEP)
VALUES (SEQ_ANG_EX.NEXTVAL, 'Test', 5000, 99);

-- D)
INSERT INTO ANGAJATI_EX (ID_ANG, NUME, SALARIU)
VALUES (SEQ_ANG_EX.NEXTVAL, 'Test', 5000);
```

<details>
<summary>🔍 Soluție</summary>

```
A) ❌ ORA-01400: NUME are constrângere NOT NULL → NULL nu este permis
B) ❌ ORA-02290: CHECK constraint violat → SALARIU trebuie să fie > 0, iar -100 nu satisface condiția
C) ❌ ORA-02291: FK constraint violat → ID_DEP = 99 nu există în DEPARTAMENTE_EX
D) ✅ OK → ID_DEP este omis din lista de coloane, deci primește NULL implicit (coloana permite NULL)
```

</details>

---

### Exercițiul 3 — UPDATE

1. Mărește salariul tuturor angajaților din departamentul IT cu 15%.
2. Transferă toți angajații din departamentul Sales (ID 30) în HR (ID 20).
3. Setează salariul angajatului `Ionescu Maria` la media salariilor din departamentul IT.
4. Încearcă să updatezi ID-ul unui departament la o valoare care nu există în `DEPARTAMENTE_EX` — ce se întâmplă?

<details>
<summary>🔍 Soluție</summary>

```sql
-- 1. Mărire salariu IT cu 15%
UPDATE ANGAJATI_EX
SET SALARIU = SALARIU * 1.15
WHERE ID_DEP = 10;

-- Verificare
SELECT * FROM ANGAJATI_EX WHERE ID_DEP = 10;

-- 2. Transfer Sales → HR
UPDATE ANGAJATI_EX
SET ID_DEP = 20
WHERE ID_DEP = 30;

-- 3. Salariu = media din IT
UPDATE ANGAJATI_EX
SET SALARIU = (SELECT AVG(SALARIU) FROM ANGAJATI_EX WHERE ID_DEP = 10)
WHERE NUME = 'Ionescu Maria';

-- 4. Update cu FK inexistentă
UPDATE ANGAJATI_EX
SET ID_DEP = 999
WHERE ID_ANG = 1;
-- ❌ ORA-02291: integrity constraint violated - parent key not found
-- Nu poți seta un ID_DEP care nu există în DEPARTAMENTE_EX

COMMIT;
```

</details>

---

### Exercițiul 4 — DELETE și constrângeri FK

1. Încearcă să ștergi departamentul IT (ID 10) — ce se întâmplă?
2. Explică de ce comportamentul este diferit față de ștergerea departamentului Sales (după ce l-ai mutat în Exercițiul 3).
3. Șterge toți angajații cu salariul sub 5000, folosind un `SELECT` preliminar pentru a verifica ce vei șterge.

<details>
<summary>🔍 Soluție</summary>

```sql
-- 1. Ștergere departament IT
DELETE FROM DEPARTAMENTE_EX WHERE ID_DEP = 10;
-- Rezultat depinde de FK:
-- Dacă există angajați în IT: comportament depinde de ON DELETE opțiune
-- FK-ul nostru are ON DELETE SET NULL → ștergerea reușește, ID_DEP din angajați devine NULL

-- 2. Explicație:
-- La definirea FK_ANG_DEP_EX am folosit ON DELETE SET NULL
-- Deci ștergerea unui departament NU generează eroare — ID_DEP al angajaților afectați devine NULL
-- Dacă FK-ul ar fi fost definit fără opțiune (sau ON DELETE CASCADE), comportamentul ar fi diferit:
--   Fără opțiune → ORA-02292: child record found → ștergerea eșuează
--   ON DELETE CASCADE → angajații din acel departament ar fi șterși automat

-- 3. Verificare înainte de DELETE
SELECT * FROM ANGAJATI_EX WHERE SALARIU < 5000;
-- Verifică că sunt exact rândurile dorite

-- Abia după verificare, execută DELETE
DELETE FROM ANGAJATI_EX WHERE SALARIU < 5000;
COMMIT;
```

</details>

---

### Exercițiul 5 — INSERT ALL

Ai tabelele `ANGAJATI_JUNIOR` (salariu < 4000) și `ANGAJATI_SENIOR` (salariu >= 4000). Distribuie angajații din `ANGAJATI_EX` în cele două tabele folosind `INSERT ALL` cu condiții, astfel încât fiecare angajat să apară într-un singur tabel.

```sql
-- Setup tabele destinație
CREATE TABLE ANGAJATI_JUNIOR (ID NUMBER, NUME VARCHAR2(50), SALARIU NUMBER(10,2));
CREATE TABLE ANGAJATI_SENIOR (ID NUMBER, NUME VARCHAR2(50), SALARIU NUMBER(10,2));
```

<details>
<summary>🔍 Soluție</summary>

```sql
-- Cu INSERT FIRST: fiecare angajat merge în exact un singur tabel
INSERT FIRST
    WHEN SALARIU < 4000 THEN
        INTO ANGAJATI_JUNIOR (ID, NUME, SALARIU) VALUES (ID_ANG, NUME, SALARIU)
    WHEN SALARIU >= 4000 THEN
        INTO ANGAJATI_SENIOR (ID, NUME, SALARIU) VALUES (ID_ANG, NUME, SALARIU)
SELECT ID_ANG, NUME, SALARIU FROM ANGAJATI_EX;

-- Verificare
SELECT 'JUNIOR' TIP, COUNT(*) NR FROM ANGAJATI_JUNIOR
UNION ALL
SELECT 'SENIOR' TIP, COUNT(*) NR FROM ANGAJATI_SENIOR;

COMMIT;
```

> 💡 Am folosit `INSERT FIRST` (nu `ALL`) pentru a garanta că fiecare angajat apare într-un singur tabel. Cu `INSERT ALL`, un angajat cu salariu exact 4000 ar satisface ambele condiții (`< 4000` = fals, `>= 4000` = adevărat), deci ar merge doar în SENIOR — dar în general `FIRST` este mai sigur când vrem partiționare exclusivă.

</details>

---

### Exercițiul 6 — Secvențe

1. Creează o secvență `SEQ_DEP_EX` care începe de la 100, crește cu 10, are maxim 990 și este ciclică.
2. Folosind secvența, inserează 3 departamente noi.
3. Afișează valoarea curentă a secvenței fără să o incrementezi.
4. Ce se întâmplă dacă faci `ROLLBACK` după un `INSERT` care a folosit `NEXTVAL`?

<details>
<summary>🔍 Soluție</summary>

```sql
-- 1. Creare secvență
CREATE SEQUENCE SEQ_DEP_EX
    START WITH 100
    INCREMENT BY 10
    MAXVALUE 990
    CYCLE
    NOCACHE;

-- 2. Inserare departamente
INSERT INTO DEPARTAMENTE_EX (ID_DEP, NUME_DEP, BUGET)
VALUES (SEQ_DEP_EX.NEXTVAL, 'Marketing', 400000);  -- ID = 100

INSERT INTO DEPARTAMENTE_EX (ID_DEP, NUME_DEP, BUGET)
VALUES (SEQ_DEP_EX.NEXTVAL, 'Legal', 300000);       -- ID = 110

INSERT INTO DEPARTAMENTE_EX (ID_DEP, NUME_DEP, BUGET)
VALUES (SEQ_DEP_EX.NEXTVAL, 'Finance', 600000);     -- ID = 120

-- 3. Valoarea curentă (fără incrementare)
SELECT SEQ_DEP_EX.CURRVAL FROM DUAL;  -- rezultat: 120

-- 4. ROLLBACK după NEXTVAL:
INSERT INTO DEPARTAMENTE_EX (ID_DEP, NUME_DEP, BUGET)
VALUES (SEQ_DEP_EX.NEXTVAL, 'Test', 100000);  -- NEXTVAL = 130
ROLLBACK;
-- Rândul este anulat, DAR secvența a avansat la 130 și NU revine!
-- Următorul NEXTVAL va fi 140, nu 130.
-- Concluzie: golul 130 rămâne permanent în secvență.

COMMIT;
```

</details>

---

### Exercițiul 7 — Tranzacție completă

Scrie o secvență de comenzi care:
1. Inserează un departament nou „Contabilitate" cu buget 250000 și ID generat de secvență.
2. Inserează 2 angajați în acest departament cu IDs generați de `SEQ_ANG_EX`.
3. Marchează un savepoint după inserarea angajaților.
4. Actualizează salariul ambilor angajați cu 10%.
5. Decide să anulezi actualizarea salariilor (ROLLBACK la savepoint).
6. Confirmă definitiv inserările.

<details>
<summary>🔍 Soluție</summary>

```sql
-- 1. Inserare departament
INSERT INTO DEPARTAMENTE_EX (ID_DEP, NUME_DEP, BUGET)
VALUES (SEQ_DEP_EX.NEXTVAL, 'Contabilitate', 250000);

-- Salvăm ID-ul generat pentru utilizare ulterioară
-- (CURRVAL reține ultimul NEXTVAL din sesiunea curentă)
-- SEQ_DEP_EX.CURRVAL va fi ID-ul departamentului nou creat

-- 2. Inserare angajați
INSERT INTO ANGAJATI_EX (ID_ANG, NUME, SALARIU, ID_DEP)
VALUES (SEQ_ANG_EX.NEXTVAL, 'Dumitrescu Alina', 4800, SEQ_DEP_EX.CURRVAL);

INSERT INTO ANGAJATI_EX (ID_ANG, NUME, SALARIU, ID_DEP)
VALUES (SEQ_ANG_EX.NEXTVAL, 'Marinescu Bogdan', 5200, SEQ_DEP_EX.CURRVAL);

-- 3. Savepoint
SAVEPOINT dupa_insert;

-- 4. Actualizare salarii
UPDATE ANGAJATI_EX
SET SALARIU = SALARIU * 1.10
WHERE NUME IN ('Dumitrescu Alina', 'Marinescu Bogdan');

-- Verificare (optional)
SELECT NUME, SALARIU FROM ANGAJATI_EX
WHERE NUME IN ('Dumitrescu Alina', 'Marinescu Bogdan');

-- 5. Anulare actualizare (revenire la savepoint)
ROLLBACK TO dupa_insert;

-- Verificare după rollback (salariile revin la valorile inițiale)
SELECT NUME, SALARIU FROM ANGAJATI_EX
WHERE NUME IN ('Dumitrescu Alina', 'Marinescu Bogdan');

-- 6. Confirmare finală (departament + angajați inserați, fără actualizarea de salariu)
COMMIT;
```

</details>

---

### Cleanup

```sql
DROP TABLE ANGAJATI_JUNIOR;
DROP TABLE ANGAJATI_SENIOR;
DROP TABLE ANGAJATI_EX;
DROP TABLE DEPARTAMENTE_EX;
DROP SEQUENCE SEQ_ANG_EX;
DROP SEQUENCE SEQ_DEP_EX;
```
