# ✏️ Limbajul de Manipulare a Datelor (LMD / DML)

---

## 1. Introducere

**LMD (Limbajul de Manipulare a Datelor)** permite operarea asupra datelor din tabele.

### Comenzile LMD:

| Comandă | Descriere |
| :--- | :--- |
| `SELECT` | Regăsirea datelor |
| `INSERT` | Adăugarea de noi înregistrări |
| `UPDATE` | Modificarea valorilor din înregistrările existente |
| `DELETE` | Ștergerea înregistrărilor |
| `MERGE` | Adăugarea sau modificarea condiționată de înregistrări |

> 💡 Spre deosebire de LDD, comenzile LMD **pot fi anulate** cu `ROLLBACK` dacă nu s-a executat un `COMMIT`.

---

## 2. Comanda INSERT

### 2.1. Sintaxă generală

```sql
INSERT INTO tabel [(col1, col2, ...)]
VALUES (val1, val2, ...);

-- sau cu subcerere
INSERT INTO tabel [(col1, col2, ...)]
subcerere;
```

---

### 2.2. INSERT cu listă de coloane (recomandat)

```sql
-- ✅ Recomandat: cu lista de coloane (mai clar și mai sigur)
INSERT INTO ECHIPE (ID_ECHIPA, NUME_ECHIPA, VENIT_ANUAL)
VALUES (1, 'Real Madrid', 750000000);

-- ⚠️ Fără lista de coloane: trebuie specificate valori pentru TOATE coloanele în ordinea definirii
INSERT INTO ECHIPE
VALUES (2, 'Barcelona', SYSDATE, 700000000, 'BAR');
```

> 💡 Se recomandă **întotdeauna** specificarea listei de coloane — protejează față de modificări viitoare ale structurii tabelului.

---

### 2.3. Tipuri de date în VALUES

```sql
-- Caractere și date → între APOSTROFURI
INSERT INTO JUCATORI (ID_JUCATOR, NUME_JUCATOR, DATA_NASTERII)
VALUES (1, 'Messi', '24-06-1987');

-- Numere → FĂRĂ apostrofuri (apostrofurile determină conversie implicită inutilă)
INSERT INTO JUCATORI (ID_JUCATOR, NUMAR_TRICOU, SALARIU)
VALUES (1, 10, 50000000);

-- Funcții în VALUES
INSERT INTO ECHIPE (ID_ECHIPA, NUME_ECHIPA, DATA_INFIINTARE)
VALUES (3, 'Juventus', SYSDATE);
```

> ⚠️ Valorile de tip **caracter** și **dată calendaristică** se includ între **apostrofuri** (`'`).  
> ⚠️ **Nu** include valorile numerice între apostrofuri — ar determina conversii implicite inutile.

---

### 2.4. Inserarea valorilor NULL

```sql
-- Implicit: omiterea coloanei din lista de coloane → NULL automat
INSERT INTO JUCATORI (ID_JUCATOR, NUME_JUCATOR)
VALUES (2, 'Ronaldo');
-- NUMAR_TRICOU și ECHIPA_ID vor fi NULL automat

-- Explicit: specificarea directă a NULL
INSERT INTO JUCATORI (ID_JUCATOR, NUME_JUCATOR, NUMAR_TRICOU)
VALUES (3, 'Neymar', NULL);

-- Explicit cu șir vid (pentru caractere sau date)
INSERT INTO JUCATORI (ID_JUCATOR, NUME_JUCATOR, OBSERVATII)
VALUES (4, 'Mbappe', '');
-- '' este echivalent cu NULL în Oracle
```

---

### 2.5. INSERT din subcerere

```sql
-- Copierea angajaților din departamentul 20 într-un alt tabel
INSERT INTO EMPLOYEES_BACKUP (EMPLOYEE_ID, FIRST_NAME, SALARY)
    SELECT EMPLOYEE_ID, FIRST_NAME, SALARY
    FROM EMPLOYEES
    WHERE DEPARTMENT_ID = 20;
```

> ⚠️ Coloanele din `SELECT` trebuie să corespundă ca **număr și tip** celor din lista `INTO`.  
> ⚠️ Dacă nu există lista de coloane în `INTO`, subcererea trebuie să furnizeze valori pentru **toate** coloanele, în ordinea definirii.

---

### 2.6. INSERT multi-tabel — `INSERT ALL`

Permite inserarea în **mai multe tabele simultan** pe baza unei singure subcereri.

#### Inserare necondiționată (în toate tabelele):

```sql
INSERT ALL
    INTO ANGAJATI_IT     (ID, NUME, SALARIU) VALUES (ID, NUME, SALARIU)
    INTO ANGAJATI_BACKUP (ID, NUME, SALARIU) VALUES (ID, NUME, SALARIU)
SELECT EMPLOYEE_ID ID, FIRST_NAME NUME, SALARY SALARIU
FROM EMPLOYEES
WHERE JOB_ID = 'IT_PROG';
```

#### Inserare condiționată cu `ALL` — evaluează **toate** condițiile:

```sql
-- ALL: fiecare WHEN este evaluat independent; o înregistrare poate intra în mai multe tabele
INSERT ALL
    WHEN SALARY < 5000  THEN INTO ANGAJATI_JUNIOR  VALUES (EMPLOYEE_ID, FIRST_NAME, SALARY)
    WHEN SALARY >= 5000 THEN INTO ANGAJATI_SENIOR  VALUES (EMPLOYEE_ID, FIRST_NAME, SALARY)
    WHEN DEPARTMENT_ID = 20 THEN INTO ANGAJATI_DEPT20 VALUES (EMPLOYEE_ID, FIRST_NAME, SALARY)
SELECT EMPLOYEE_ID, FIRST_NAME, SALARY, DEPARTMENT_ID
FROM EMPLOYEES;
```

#### Inserare condiționată cu `FIRST` — evaluează condițiile **în ordine, se oprește la prima adevărată**:

```sql
-- FIRST: o înregistrare intră DOAR în primul tabel a cărui condiție e adevărată
INSERT FIRST
    WHEN SALARY < 3000  THEN INTO ANGAJATI_JUNIOR  VALUES (EMPLOYEE_ID, FIRST_NAME, SALARY)
    WHEN SALARY < 6000  THEN INTO ANGAJATI_MID     VALUES (EMPLOYEE_ID, FIRST_NAME, SALARY)
    WHEN SALARY >= 6000 THEN INTO ANGAJATI_SENIOR  VALUES (EMPLOYEE_ID, FIRST_NAME, SALARY)
    ELSE INTO ANGAJATI_ALTII VALUES (EMPLOYEE_ID, FIRST_NAME, SALARY)
SELECT EMPLOYEE_ID, FIRST_NAME, SALARY
FROM EMPLOYEES;
```

> 💡 **`ALL`** — toate condițiile `WHEN` sunt evaluate; o înregistrare poate fi inserată în **mai multe tabele**.  
> 💡 **`FIRST`** — condițiile sunt evaluate **în ordine**; o înregistrare intră **doar** în primul tabel cu condiție adevărată; celelalte `WHEN` sunt ignorate.

---

### 2.7. Erori posibile la INSERT

| Situație | Eroare |
| :--- | :--- |
| Valoare `NULL` pentru o coloană `NOT NULL` | Eroare constrângere |
| Valori duplicate care încalcă `UNIQUE` sau `PRIMARY KEY` | Eroare constrângere |
| Valoare FK inexistentă în tabelul părinte | Eroare constrângere referențială |
| Condiție `CHECK` nerespectată | Eroare constrângere |
| Incompatibilitate tip de date | Eroare conversie |
| Valoare mai mare decât dimensiunea coloanei | Eroare dimensiune |

---

## 3. Comanda UPDATE

### 3.1. Sintaxă generală

```sql
-- O singură coloană
UPDATE tabel [alias]
SET coloana = expresie
[WHERE conditie];

-- Mai multe coloane
UPDATE tabel [alias]
SET col1 = expr1, col2 = expr2, ...
[WHERE conditie];

-- Cu subcerere
UPDATE tabel [alias]
SET (col1, col2, ...) = (subcerere)
[WHERE conditie];
```

---

### 3.2. Exemple practice

```sql
-- Modificarea salariului unui singur angajat (identificat prin PK)
UPDATE EMPLOYEES
SET SALARY = 9000
WHERE EMPLOYEE_ID = 100;

-- Modificarea mai multor coloane simultan
UPDATE EMPLOYEES
SET SALARY = 9000,
    JOB_ID = 'IT_PROG'
WHERE EMPLOYEE_ID = 100;

-- Mărire salariu cu 10% pentru toți angajații dintr-un departament
UPDATE EMPLOYEES
SET SALARY = SALARY * 1.10
WHERE DEPARTMENT_ID = 20;

-- Update cu subcerere
UPDATE EMPLOYEES
SET SALARY = (SELECT AVG(SALARY) FROM EMPLOYEES)
WHERE DEPARTMENT_ID = 30;

-- Update cu subcerere pe mai multe coloane
UPDATE EMPLOYEES
SET (SALARY, JOB_ID) = (
    SELECT SALARY, JOB_ID
    FROM EMPLOYEES
    WHERE EMPLOYEE_ID = 100
)
WHERE EMPLOYEE_ID = 101;
```

---

### 3.3. Reguli importante UPDATE

> ⚠️ **Fără clauza `WHERE`** — sunt afectate **toate liniile** din tabel!

```sql
-- ⚠️ Aceasta modifică SALARIUL TUTUROR angajaților!
UPDATE EMPLOYEES
SET SALARY = 5000;
```

> 💡 Pentru identificarea unei linii se recomandă folosirea **cheii primare** în `WHERE`.  
> ⚠️ Erorile posibile sunt similare cu cele de la `INSERT` (NOT NULL, UNIQUE, FK, CHECK, tip de date).

---

## 4. Comanda DELETE

### 4.1. Sintaxă generală

```sql
DELETE FROM tabel
[WHERE conditie];
```

---

### 4.2. Exemple practice

```sql
-- Ștergerea unui singur angajat (identificat prin PK)
DELETE FROM EMPLOYEES
WHERE EMPLOYEE_ID = 107;

-- Ștergerea tuturor angajaților dintr-un departament
DELETE FROM EMPLOYEES
WHERE DEPARTMENT_ID = 20;

-- Ștergerea cu subcerere
DELETE FROM EMPLOYEES
WHERE DEPARTMENT_ID IN (
    SELECT DEPARTMENT_ID
    FROM DEPARTMENTS
    WHERE UPPER(DEPARTMENT_NAME) LIKE '%SALES%'
);
```

---

### 4.3. Reguli importante DELETE

> ⚠️ **Fără clauza `WHERE`** — sunt șterse **toate liniile** din tabel!

```sql
-- ⚠️ Aceasta șterge TOȚI angajații!
DELETE FROM EMPLOYEES;
```

> 💡 Spre deosebire de `TRUNCATE`, `DELETE` este o comandă **LMD** — poate fi anulată cu `ROLLBACK`.

---

### Comparație DELETE vs. TRUNCATE:

| | `DELETE FROM` | `TRUNCATE TABLE` |
| :--- | :--- | :--- |
| **Tip** | LMD | LDD |
| **ROLLBACK posibil** | ✅ Da | ❌ Nu |
| **Clauza WHERE** | ✅ Da (șterge selectiv) | ❌ Nu (șterge tot) |
| **Viteză** | Mai lentă | Rapidă |
| **Declanșează triggere** | ✅ Da | ❌ Nu |

---

## 5. Rezumat Rapid

```
INSERT:
  INSERT INTO tabel (col1, col2) VALUES (val1, val2)
  INSERT INTO tabel (col1, col2) subcerere
  INSERT ALL INTO ... INTO ... subcerere          → necondiționat
  INSERT ALL  WHEN cond THEN INTO ... subcerere   → ALL: evaluează toate condițiile
  INSERT FIRST WHEN cond THEN INTO ... subcerere  → FIRST: prima condiție adevărată

  Caractere și DATE → apostrofuri (' ')
  Numere → fără apostrofuri
  NULL implicit → omite coloana din lista INTO
  NULL explicit → VALUES (..., NULL, ...)

UPDATE:
  SET col = val [WHERE conditie]
  SET (col1, col2) = (subcerere) [WHERE conditie]
  ⚠️ Fără WHERE → modifică TOATE liniile!

DELETE:
  DELETE FROM tabel [WHERE conditie]
  ⚠️ Fără WHERE → șterge TOATE liniile!
  ✅ Poate fi anulat cu ROLLBACK (spre deosebire de TRUNCATE)
```

---

## 6. Capcane Frecvente ⚠️

| Situație | Problemă | Soluție |
| :--- | :--- | :--- |
| `UPDATE` sau `DELETE` fără `WHERE` | Afectează **toate** liniile din tabel | Verifică întotdeauna că ai condiție în `WHERE` |
| `INSERT` valoare numerică între apostrofuri | Conversie implicită inutilă | Nu pune numerele între apostrofuri |
| `INSERT ALL` vs. `INSERT FIRST` confuzie | `ALL` poate duplica înregistrări în mai multe tabele | Folosește `FIRST` dacă vrei ca o înregistrare să intre **o singură dată** |
| `INSERT` din subcerere fără lista de coloane | Subcererea trebuie să acopere **toate** coloanele în ordine | Specifică întotdeauna lista de coloane în `INTO` |
| `DELETE` în loc de `TRUNCATE` pentru golire completă | Mai lent, dar reversibil | Folosește `TRUNCATE` dacă nu ai nevoie de `ROLLBACK` și vrei viteză |
| Încălcarea constrângerii FK la `DELETE` | Nu poți șterge o linie referențiată de o FK | Șterge mai întâi înregistrările din tabelul copil |

---

## 7. Secvențe (SEQUENCE)

O **secvență** este un obiect al bazei de date care generează **întregi unici** în mod automat. Este utilizată frecvent pentru generarea valorilor de cheie primară sau alte coloane numerice unice.

> 💡 Secvențele sunt **independente de tabele** — aceeași secvență poate fi folosită pentru mai multe tabele.

---

### 7.1. Crearea unei secvențe — `CREATE SEQUENCE`

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
| `INCREMENT BY n` | Diferența dintre două numere generate succesiv | `1` |
| `START WITH n` | Numărul inițial al secvenței | `1` |
| `MAXVALUE n` | Valoarea maximă | `10^27` (asc.) / `-1` (desc.) |
| `MINVALUE n` | Valoarea minimă | `1` (asc.) / `-10^27` (desc.) |
| `CYCLE` / `NOCYCLE` | Reîncepe de la capăt după atingerea limitei | `NOCYCLE` |
| `CACHE n` / `NOCACHE` | Câte numere să încarce în cache | `20` |

---

### 7.2. Exemple de creare

```sql
-- Secvență simplă, începe de la 1, crește cu 1
CREATE SEQUENCE SEQ_JUCATORI
    START WITH 1
    INCREMENT BY 1
    NOCACHE
    NOCYCLE;

-- Secvență pentru ID-uri de departamente, cu cache
CREATE SEQUENCE SEQ_DEPT
    START WITH 10
    INCREMENT BY 10
    MAXVALUE 9000
    NOCYCLE
    CACHE 20;
```

---

### 7.3. Utilizarea secvențelor — `NEXTVAL` și `CURRVAL`

Secvențele se folosesc prin două **pseudocoloane**:

| Pseudocoloană | Descriere |
| :--- | :--- |
| `nume_secv.NEXTVAL` | Returnează **următoarea valoare** a secvenței (unică la fiecare referire) |
| `nume_secv.CURRVAL` | Returnează **valoarea curentă** a secvenței |

> ⚠️ `NEXTVAL` trebuie apelat **cel puțin o dată** înainte de a putea folosi `CURRVAL`.

```sql
-- Prima utilizare: obținerea valorii curente (după cel puțin un NEXTVAL)
SELECT SEQ_JUCATORI.NEXTVAL FROM DUAL;  -- rezultat: 1
SELECT SEQ_JUCATORI.CURRVAL FROM DUAL;  -- rezultat: 1 (valoarea curentă)
SELECT SEQ_JUCATORI.NEXTVAL FROM DUAL;  -- rezultat: 2
SELECT SEQ_JUCATORI.NEXTVAL FROM DUAL;  -- rezultat: 3

-- Utilizare în INSERT (cel mai frecvent caz)
INSERT INTO JUCATORI (ID_JUCATOR, NUME_JUCATOR, NUMAR_TRICOU)
VALUES (SEQ_JUCATORI.NEXTVAL, 'Messi', 10);

INSERT INTO JUCATORI (ID_JUCATOR, NUME_JUCATOR, NUMAR_TRICOU)
VALUES (SEQ_JUCATORI.NEXTVAL, 'Ronaldo', 7);
-- ID-urile vor fi generate automat: 1, 2, 3...

-- Utilizare în UPDATE
UPDATE JUCATORI
SET ID_JUCATOR = SEQ_JUCATORI.NEXTVAL
WHERE NUME_JUCATOR = 'Neymar';
```

---

### 7.4. Unde pot fi folosite pseudocoloanele

✅ **Permis:**

| Locație | Exemplu |
| :--- | :--- |
| Lista `SELECT` a comenzilor care **nu** fac parte din subcereri | `SELECT SEQ.NEXTVAL FROM DUAL` |
| Lista `SELECT` a unei cereri din `INSERT` | `INSERT INTO t SELECT SEQ.NEXTVAL, col FROM ...` |
| Clauza `VALUES` a comenzii `INSERT` | `VALUES (SEQ.NEXTVAL, 'text')` |
| Clauza `SET` a comenzii `UPDATE` | `SET col = SEQ.NEXTVAL` |

---

❌ **Interzis:**

| Locație | Motiv |
| :--- | :--- |
| Lista `SELECT` a unei **vizualizări** | Vizualizările nu pot folosi secvențe |
| `SELECT` cu `DISTINCT`, `GROUP BY`, `HAVING`, `ORDER BY` | Ar putea genera valori nedeterminist |
| **Subcereri** în `SELECT`, `UPDATE`, `DELETE` | |
| Clauza `DEFAULT` din `CREATE TABLE` sau `ALTER TABLE` | |

```sql
-- ❌ Eroare: NEXTVAL într-o subcerere
SELECT * FROM EMPLOYEES
WHERE EMPLOYEE_ID = (SELECT SEQ_JUCATORI.NEXTVAL FROM DUAL);

-- ❌ Eroare: NEXTVAL cu ORDER BY
SELECT SEQ_JUCATORI.NEXTVAL FROM EMPLOYEES ORDER BY EMPLOYEE_ID;

-- ❌ Eroare: NEXTVAL în DEFAULT
CREATE TABLE TEST (
    ID NUMBER DEFAULT SEQ_JUCATORI.NEXTVAL  -- nu este permis
);
```

---

### 7.5. Consultarea dicționarului datelor pentru secvențe

```sql
-- Secvențele utilizatorului curent
SELECT * FROM USER_SEQUENCES;

-- Toate secvențele accesibile utilizatorului
SELECT * FROM ALL_SEQUENCES;

-- Toate secvențele din baza de date (necesită privilegii DBA)
SELECT * FROM DBA_SEQUENCES;
```

---

### 7.6. Ștergerea secvențelor — `DROP SEQUENCE`

```sql
DROP SEQUENCE SEQ_JUCATORI;
```

---

### 7.7. Rezumat Secvențe

```
CREATE SEQUENCE seq_name
    START WITH n       → valoare de start (implicit 1)
    INCREMENT BY n     → pasul (implicit 1; negativ → descrescător)
    MAXVALUE / MINVALUE
    CYCLE / NOCYCLE    → reîncepe sau nu după atingerea limitei
    CACHE n / NOCACHE  → numere preîncărcate (implicit 20)

Utilizare:
  seq.NEXTVAL  → următoarea valoare unică (incrementează secvența)
  seq.CURRVAL  → valoarea curentă (necesită NEXTVAL anterior)

✅ Permis în: VALUES, SET, SELECT simplu, SELECT din INSERT
❌ Interzis în: vizualizări, SELECT cu DISTINCT/GROUP BY/HAVING/ORDER BY,
               subcereri, DEFAULT din CREATE/ALTER TABLE

DROP SEQUENCE nume_secventa;  → ștergere permanentă
```

---

### 7.8. Capcane frecvente cu secvențe ⚠️

| Situație | Problemă | Soluție |
| :--- | :--- | :--- |
| `CURRVAL` înainte de `NEXTVAL` | Eroare Oracle | Apelează `NEXTVAL` cel puțin o dată mai întâi |
| `NEXTVAL` în clauza `DEFAULT` | Eroare Oracle | Nu este permis — generează ID în `INSERT` explicit |
| `NEXTVAL` în subcerere | Eroare Oracle | Mută `NEXTVAL` în lista principală `SELECT` sau în `VALUES` |
| `NEXTVAL` cu `ORDER BY` | Eroare Oracle | Nu combina `NEXTVAL` cu `ORDER BY` în același `SELECT` |
| Goluri în secvență după `ROLLBACK` | Valoarea este consumată chiar și la `ROLLBACK` | Secvențele nu se „întorc" — sunt proiectate să genereze valori unice, nu consecutive |
