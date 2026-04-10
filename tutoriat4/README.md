# 📋 Analiza Sintaxei SELECT

Sintaxa generală a comenzii `SELECT`:

```sql
SELECT { [ {DISTINCT | UNIQUE} | ALL] lista_campuri | *} 
FROM [nume_schemă.]nume_obiect 
[, [nume_schemă.]nume_obiect ...] 
[WHERE condiție_clauza_where] 
[GROUP BY expresie [, expresie ...] 
[HAVING condiție_clauza_having] 
[ORDER BY {expresie | poziţie} [, {expresie | poziţie} ...] ];
```

---

## Ordinea de Execuție a Clauzelor

Deși scriem clauzele într-o anumită ordine, Oracle le **execută** în altă ordine:

```
Ordinea de scriere:    SELECT → FROM → WHERE → GROUP BY → HAVING → ORDER BY
Ordinea de execuție:   FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY
```

> 💡 Aceasta explică de ce **nu poți folosi alias-uri din `SELECT` în clauza `WHERE`** — `WHERE` se execută înainte ca alias-urile să fie definite. Poți folosi alias-uri doar în `ORDER BY`, care se execută ultimul.

```sql
-- ❌ Eroare: WHERE nu știe ce este SALARIU_ANUAL (alias definit în SELECT)
SELECT FIRST_NAME, SALARY * 12 AS SALARIU_ANUAL
FROM EMPLOYEES
WHERE SALARIU_ANUAL > 60000;

-- ✅ Corect: folosești expresia originală în WHERE
SELECT FIRST_NAME, SALARY * 12 AS SALARIU_ANUAL
FROM EMPLOYEES
WHERE SALARY * 12 > 60000;

-- ✅ Corect: alias-ul poate fi folosit în ORDER BY
SELECT FIRST_NAME, SALARY * 12 AS SALARIU_ANUAL
FROM EMPLOYEES
ORDER BY SALARIU_ANUAL DESC;
```

---

## 1. Instrucțiunea SELECT: Reguli și Alias-uri

`SELECT` este o clauză **obligatorie** într-o interogare SQL. Câmpurile din `SELECT` se separă prin **virgulă** și pot fi:

### 1.1. Coloane simple

```sql
SELECT FIRST_NAME, SALARY
FROM EMPLOYEES;
```

Selectează direct coloanele `FIRST_NAME` și `SALARY` din tabel.

---

### 1.2. Operații matematice

```sql
SELECT FIRST_NAME, SALARY * 12 AS SALARIU_ANUAL
FROM EMPLOYEES;
```

Se realizează o **operație matematică** pentru a calcula salariul anual.

**Operatorii aritmetici disponibili:**

| Operator | Descriere | Exemplu |
| :--- | :--- | :--- |
| `+` | Adunare | `SALARY + COMMISSION_PCT` |
| `-` | Scădere | `SALARY - 500` |
| `*` | Înmulțire | `SALARY * 12` |
| `/` | Împărțire | `SALARY / 30` |

> ⚠️ **Orice operație aritmetică cu `NULL` returnează `NULL`.**
> Dacă `COMMISSION_PCT` este `NULL`, atunci `SALARY * COMMISSION_PCT` este `NULL`.

```sql
-- Dacă COMMISSION_PCT este NULL, bonusul va fi NULL (nu 0!)
SELECT FIRST_NAME, SALARY * COMMISSION_PCT AS BONUS
FROM EMPLOYEES;

-- Soluție: NVL pentru a înlocui NULL cu 0
SELECT FIRST_NAME, SALARY * NVL(COMMISSION_PCT, 0) AS BONUS
FROM EMPLOYEES;
```

---

### 1.3. Funcții aplicate pe coloane

```sql
SELECT UPPER(FIRST_NAME) AS NUME_MAJUSCULE
FROM EMPLOYEES;
```

Funcția `UPPER()` transformă valorile din coloană în **majuscule**.

---

### 1.4. Concatenarea șirurilor

Oracle folosește operatorul `||` (două bare verticale) pentru concatenarea șirurilor:

```sql
-- Concatenare simplă
SELECT FIRST_NAME || ' ' || LAST_NAME AS NUME_COMPLET
FROM EMPLOYEES;

-- Concatenare cu text literal
SELECT FIRST_NAME || ' are salariul ' || SALARY AS DESCRIERE
FROM EMPLOYEES;

-- Alternativ: funcția CONCAT (acceptă doar 2 argumente)
SELECT CONCAT(FIRST_NAME, LAST_NAME) AS NUME
FROM EMPLOYEES;
-- Pentru mai mult de 2 șiruri, CONCAT nu e practic — folosește ||
```

---

### 1.5. Subcereri (Subqueries)

Subcererile sunt comenzi `SELECT` incluse în alte interogări.

```sql
SELECT FIRST_NAME, SALARY
FROM EMPLOYEES
WHERE SALARY > (
    SELECT AVG(SALARY)
    FROM EMPLOYEES
);
```

Această interogare returnează angajații care au **salariul mai mare decât media salariilor**.

---

### 1.6. Alias-uri — reguli detaliate

Un **alias** este un nume temporar dat unei coloane sau expresii în rezultat. Se poate folosi cu sau fără `AS`:

```sql
-- Cu AS (recomandat, mai clar)
SELECT SALARY * 12 AS SALARIU_ANUAL
FROM EMPLOYEES;

-- Fără AS (funcționează, dar mai puțin lizibil)
SELECT SALARY * 12 SALARIU_ANUAL
FROM EMPLOYEES;
```

**Reguli pentru alias-uri:**

| Situație | Sintaxă | Exemplu |
| :--- | :--- | :--- |
| Alias simplu, fără spații | Cu sau fără `AS` | `SALARY * 12 AS SAL_AN` |
| Alias cu spații | Ghilimele duble obligatorii | `SALARY * 12 AS "Salariu Anual"` |
| Alias cu caractere speciale | Ghilimele duble obligatorii | `SALARY AS "Sal. ($)"` |
| Alias cu litere mici | Ghilimele duble (altfel Oracle convertește la majuscule) | `SALARY AS "salariu"` |

```sql
-- ✅ Alias simplu
SELECT SALARY * 12 AS SAL_AN FROM EMPLOYEES;

-- ✅ Alias cu spații — ghilimele duble obligatorii
SELECT SALARY * 12 AS "Salariu Anual" FROM EMPLOYEES;

-- ⚠️ Fără ghilimele — Oracle convertește la majuscule (SALARIU, nu salariu)
SELECT SALARY AS salariu FROM EMPLOYEES;
-- Coloana va apărea ca SALARIU în rezultat

-- ✅ Litere mici garantate
SELECT SALARY AS "salariu" FROM EMPLOYEES;
```

> ⚠️ **Atenție la confuzie:**
> - **Apostrofuri** (`' '`) → pentru valori de tip șir de caractere: `WHERE LAST_NAME = 'King'`
> - **Ghilimele duble** (`" "`) → pentru alias-uri și identificatori cu spații/caractere speciale

---

### 1.7. Coloane calculate cu DUAL

`DUAL` este un tabel special Oracle cu un singur rând și o singură coloană. Este folosit pentru a evalua expresii care nu necesită un tabel real:

```sql
-- Calculează o expresie fără tabel
SELECT 2 + 3 FROM DUAL;             -- rezultat: 5
SELECT SYSDATE FROM DUAL;           -- data și ora curentă
SELECT UPPER('hello') FROM DUAL;    -- rezultat: HELLO
SELECT 100 * 12 FROM DUAL;          -- rezultat: 1200
```

---

### 1.8. DISTINCT și ALL

`DISTINCT` (sau `UNIQUE`) elimină rândurile duplicate din rezultat:

```sql
-- Fără DISTINCT: afișează toate departamentele (cu repetări)
SELECT DEPARTMENT_ID FROM EMPLOYEES;

-- Cu DISTINCT: afișează fiecare departament o singură dată
SELECT DISTINCT DEPARTMENT_ID FROM EMPLOYEES;

-- DISTINCT pe mai multe coloane: elimină combinațiile duplicate
SELECT DISTINCT DEPARTMENT_ID, JOB_ID FROM EMPLOYEES;
-- Un rând este duplicat doar dacă AMBELE coloane au aceleași valori
```

> ⚠️ `DISTINCT` se aplică la **combinația tuturor coloanelor** din `SELECT`, nu doar la prima coloană.

```sql
-- Exemplu:
-- DEPARTMENT_ID = 10, JOB_ID = 'IT_PROG'
-- DEPARTMENT_ID = 10, JOB_ID = 'HR_REP'
-- Acestea sunt considerate DIFERITE de DISTINCT — nicio eliminare

SELECT DISTINCT DEPARTMENT_ID, JOB_ID FROM EMPLOYEES;
-- Ambele rânduri de mai sus vor apărea în rezultat
```

---

### 1.9. Wildcard (`*`) — selectarea tuturor coloanelor

```sql
-- Selectează toate coloanele din tabel
SELECT * FROM EMPLOYEES;

-- Selectează toate coloanele cu prefix de tabel
SELECT E.* FROM EMPLOYEES E;

-- ❌ EROARE: nu poți combina * cu coloane individuale fără prefix
SELECT *, SALARY * 12 FROM EMPLOYEES;

-- ✅ Corect: prefix tabel pentru *
SELECT E.*, SALARY * 12 AS SAL_AN FROM EMPLOYEES E;
```

> ⚠️ Evită `SELECT *` în aplicații de producție — dacă structura tabelului se schimbă, rezultatele se pot schimba neașteptat.

---

## 2. Referențierea Tabelelor și Ambiguitatea

Clauza **`FROM`** este obligatorie. Există mai multe moduri de a referenția coloanele:

### 2.1. Simplu

```sql
SELECT EMPLOYEE_ID, FIRST_NAME
FROM EMPLOYEES;
```

Se folosesc direct numele coloanelor — funcționează atâta timp cât nu există ambiguitate (un singur tabel sau coloane cu nume unice).

---

### 2.2. Prefixat cu numele tabelului

```sql
SELECT EMPLOYEES.EMPLOYEE_ID, EMPLOYEES.FIRST_NAME
FROM EMPLOYEES;
```

Coloanele sunt prefixate cu numele tabelului — verbose, dar elimină ambiguitatea.

---

### 2.3. Prefixat cu alias de tabel (recomandat)

```sql
SELECT E.EMPLOYEE_ID, E.FIRST_NAME
FROM EMPLOYEES E;
```

`E` este un **alias** pentru tabelul `EMPLOYEES` — mai scurt și mai ușor de citit, mai ales în query-uri cu JOIN-uri.

---

### 2.4. Alias de tabel cu spații

```sql
SELECT "TABEL ANGAJATI".EMPLOYEE_ID, "TABEL ANGAJATI".FIRST_NAME
FROM EMPLOYEES "TABEL ANGAJATI";
```

Dacă aliasul conține **spații**, trebuie pus între ghilimele duble `" "`.

---

### 2.5. De ce prefixăm coloanele?

Prefixarea devine **obligatorie** când există ambiguitate:

```sql
-- ❌ Ambiguitate: DEPARTMENT_ID există în ambele tabele
SELECT EMPLOYEE_ID, FIRST_NAME, DEPARTMENT_ID, DEPARTMENT_NAME
FROM EMPLOYEES, DEPARTMENTS
WHERE EMPLOYEES.DEPARTMENT_ID = DEPARTMENTS.DEPARTMENT_ID;
-- ORA-00918: column ambiguously defined

-- ✅ Corect: prefixăm coloana ambiguă
SELECT E.EMPLOYEE_ID, E.FIRST_NAME, E.DEPARTMENT_ID, D.DEPARTMENT_NAME
FROM EMPLOYEES E, DEPARTMENTS D
WHERE E.DEPARTMENT_ID = D.DEPARTMENT_ID;
```

> 💡 **Bună practică:** Prefixează întotdeauna coloanele când lucrezi cu mai mult de un tabel, chiar dacă nu există ambiguitate — face query-ul mai clar și mai ușor de întreținut.

---

## 3. Filtrarea Datelor (WHERE) și Operatori

Clauza `WHERE` filtrează rândurile care vor fi incluse în rezultat. Dacă un query are mai multe condiții, se folosește **o singură clauză `WHERE`**, iar condițiile sunt legate prin operatori logici.

### 3.1. Operatori de comparație

| Operator | Descriere | Exemplu |
| :--- | :--- | :--- |
| `=` | Egal | `WHERE SALARY = 5000` |
| `<>` sau `!=` | Diferit | `WHERE DEPARTMENT_ID <> 10` |
| `<` | Mai mic | `WHERE SALARY < 3000` |
| `<=` | Mai mic sau egal | `WHERE SALARY <= 3000` |
| `>` | Mai mare | `WHERE SALARY > 5000` |
| `>=` | Mai mare sau egal | `WHERE SALARY >= 5000` |

```sql
SELECT FIRST_NAME, SALARY
FROM EMPLOYEES
WHERE SALARY >= 3000;
```

---

### 3.2. Apartenența la interval (`BETWEEN`)

`BETWEEN val1 AND val2` verifică dacă valoarea se află într-un **interval închis** (inclusiv capetele).

```sql
SELECT FIRST_NAME, SALARY
FROM EMPLOYEES
WHERE SALARY BETWEEN 1500 AND 3000;
-- Echivalent cu: WHERE SALARY >= 1500 AND SALARY <= 3000
```

> ⚠️ `val1` trebuie să fie **mai mic sau egal** cu `val2`. `BETWEEN 3000 AND 1500` nu returnează niciun rezultat.

```sql
-- BETWEEN funcționează și cu date calendaristice
SELECT FIRST_NAME, HIRE_DATE
FROM EMPLOYEES
WHERE HIRE_DATE BETWEEN TO_DATE('01-01-2005', 'DD-MM-YYYY')
                    AND TO_DATE('31-12-2010', 'DD-MM-YYYY');

-- NOT BETWEEN — în afara intervalului
SELECT FIRST_NAME, SALARY
FROM EMPLOYEES
WHERE SALARY NOT BETWEEN 5000 AND 7000;
```

---

### 3.3. Apartenența la mulțime (`IN`)

`IN` verifică dacă o valoare aparține unei **liste de valori**. Este echivalentul mai multor condiții `OR`:

```sql
SELECT FIRST_NAME, DEPARTMENT_ID
FROM EMPLOYEES
WHERE DEPARTMENT_ID IN (10, 30);
-- Echivalent cu: WHERE DEPARTMENT_ID = 10 OR DEPARTMENT_ID = 30

-- IN cu șiruri de caractere
SELECT FIRST_NAME, JOB_ID
FROM EMPLOYEES
WHERE JOB_ID IN ('IT_PROG', 'SA_REP', 'AD_PRES');

-- NOT IN — exclude valorile din listă
SELECT FIRST_NAME, DEPARTMENT_ID
FROM EMPLOYEES
WHERE DEPARTMENT_ID NOT IN (10, 20, 30);
```

> ⚠️ **Capcana `NOT IN` cu NULL:** Dacă lista din `NOT IN` conține un `NULL`, rezultatul întregii condiții devine `UNKNOWN` și **niciun rând nu este returnat**.

```sql
-- ❌ Problemă: dacă oricare valoare din IN este NULL, NOT IN returnează 0 rânduri
SELECT * FROM EMPLOYEES
WHERE DEPARTMENT_ID NOT IN (10, NULL, 30);
-- Niciun rezultat! NULL în lista NOT IN face totul UNKNOWN

-- ✅ Soluție: filtrează NULL-urile explicit
SELECT * FROM EMPLOYEES
WHERE DEPARTMENT_ID NOT IN (10, 30)
  AND DEPARTMENT_ID IS NOT NULL;
```

---

### 3.4. Operatori logici (`AND`, `OR`, `NOT`)

Permite combinarea mai multor condiții:

```sql
-- AND: ambele condiții trebuie să fie adevărate
SELECT FIRST_NAME, SALARY, DEPARTMENT_ID
FROM EMPLOYEES
WHERE SALARY >= 2000 AND DEPARTMENT_ID = 50;

-- OR: cel puțin una din condiții trebuie să fie adevărată
SELECT FIRST_NAME, SALARY, DEPARTMENT_ID
FROM EMPLOYEES
WHERE SALARY >= 10000 OR DEPARTMENT_ID = 10;

-- NOT: neagă condiția
SELECT FIRST_NAME, JOB_ID
FROM EMPLOYEES
WHERE NOT JOB_ID = 'IT_PROG';
-- Echivalent cu: WHERE JOB_ID <> 'IT_PROG'
```

**Prioritatea operatorilor logici** (de la cel mai prioritar la cel mai puțin):

```
1. NOT        (cel mai prioritar)
2. AND
3. OR         (cel mai puțin prioritar)
```

> ⚠️ **Prioritatea operatorilor poate da rezultate neașteptate fără paranteze!**

```sql
-- Fără paranteze: AND se evaluează înainte de OR
SELECT * FROM EMPLOYEES
WHERE DEPARTMENT_ID = 10 OR DEPARTMENT_ID = 20 AND SALARY > 5000;
-- Interpretat ca: DEPARTMENT_ID = 10 OR (DEPARTMENT_ID = 20 AND SALARY > 5000)
-- → toți din dept 10 + doar cei din dept 20 cu salary > 5000

-- Cu paranteze: controlezi ordinea evaluării
SELECT * FROM EMPLOYEES
WHERE (DEPARTMENT_ID = 10 OR DEPARTMENT_ID = 20) AND SALARY > 5000;
-- → din dept 10 sau 20, dar doar cei cu salary > 5000
```

---

### 3.5. Pattern Matching (LIKE)

Folosit pentru a căuta șabloane în șiruri de caractere:

| Metacaracter | Semnificație |
| :--- | :--- |
| `_` (underscore) | Exact **un singur** caracter (orice) |
| `%` (procent) | **Zero sau mai multe** caractere (orice) |

> 💡 `LIKE` este **case-sensitive** în Oracle pentru comparații cu coloane de tip CHAR/VARCHAR2. `'j%'` nu va găsi `'John'`.

---

### 3.6. Exemple fundamentale

```sql
-- Nume care încep cu 'J'
SELECT FIRST_NAME FROM EMPLOYEES
WHERE FIRST_NAME LIKE 'J%';
-- Rezultate: Jack, Jill, Jason, John, Jennifer...

-- Al doilea caracter este 'a'
SELECT FIRST_NAME FROM EMPLOYEES
WHERE FIRST_NAME LIKE '_a%';
-- Rezultate: Jack, Marlon, Jason (orice nume cu 'a' pe poziția 2)

-- Numele are exact 4 caractere
SELECT FIRST_NAME FROM EMPLOYEES
WHERE FIRST_NAME LIKE '____';
-- Rezultate: Jack, Jill, John (exact 4 underscore-uri = exact 4 caractere)

-- Numele se termină cu 'n'
SELECT FIRST_NAME FROM EMPLOYEES
WHERE FIRST_NAME LIKE '%n';
-- Rezultate: Jason, Brian, John...

-- Numele începe cu 'J' și se termină cu 'k'
SELECT FIRST_NAME FROM EMPLOYEES
WHERE FIRST_NAME LIKE 'J%k';
-- Rezultate: Jack, Jungk

-- Al doilea caracter este 'a' și numele se termină cu 'n'
SELECT FIRST_NAME FROM EMPLOYEES
WHERE FIRST_NAME LIKE '_a%n';
-- Rezultate: Jason, Marlon
```

---

### 3.7. NOT LIKE

```sql
-- Excluderea unui șablon
SELECT FIRST_NAME FROM EMPLOYEES
WHERE FIRST_NAME NOT LIKE 'J%';
-- Toate numele care nu încep cu 'J'
```

---

### 3.8. Caractere speciale în LIKE — ESCAPE

Dacă vrei să cauți literalmente caracterele `%` sau `_` (nu ca metacaractere), folosești clauza `ESCAPE`:

```sql
-- Caută angajații al căror JOB_ID conține caracterul '_' (nu ca wildcard)
-- Fără ESCAPE, _ este tratat ca wildcard → caută orice caracter
SELECT JOB_ID FROM EMPLOYEES
WHERE JOB_ID LIKE 'IT\_PROG' ESCAPE '\';
-- \ este caracterul de escape; \_  înseamnă literal underscore

-- Alt exemplu: caută valori care conțin literal '%'
SELECT NOTE FROM NOTES
WHERE NOTE LIKE '%50\%%' ESCAPE '\';
-- Caută șiruri care conțin '50%' undeva
```

---

### 3.9. LIKE cu funcții pentru case-insensitive

```sql
-- Căutare case-insensitive (LIKE este case-sensitive în Oracle)
SELECT FIRST_NAME FROM EMPLOYEES
WHERE UPPER(FIRST_NAME) LIKE 'J%';
-- Găsește: John, john, JOHN, jOHN...

-- Sau cu LOWER
SELECT FIRST_NAME FROM EMPLOYEES
WHERE LOWER(FIRST_NAME) LIKE '%son';
```

---

## 4. GROUP BY — Metafora "Cutiilor"

`GROUP BY` se execută **după WHERE** și grupează rândurile după coloanele specificate. 

### 🎯 Conceptul: De ce avem nevoie de GROUP BY?

**GROUP BY este momentul în care SQL trece de la o simplă listă de date la un instrument de analiză.**

Fără `GROUP BY`, baza de date vede fiecare rând ca pe o entitate separată. Cu `GROUP BY`, îi spui bazei de date:

> **"Pune toate rândurile care au aceeași valoare într-o anumită coloană în aceeași cutie, și apoi calculează ceva pentru fiecare cutie."**

#### Exemplul cu Fructele

Imaginează-ți că ai pe masă o grămadă de fructe amestecate: mere, banane și portocale.

```
1. FĂRĂ GROUP BY:
   Ai o listă lungă: "Măr, Banană, Măr, Portocală, Banană...".
   → SQL vede 10 fructe individuale

2. CU GROUP BY tip_fruct:
   Creezi trei cutii etichetate:
   ┌─────────────┐  ┌─────────────┐  ┌──────────────┐
   │    MERE     │  │   BANANE    │  │  PORTOCALE   │
   │ Măr, Măr    │  │ Banană,     │  │ Portocală,   │
   │ Măr, Măr    │  │ Banană      │  │ Portocală    │
   └─────────────┘  └─────────────┘  └──────────────┘
   
   → Ai 4 mere, 3 banane, 2 portocale
```

**De ce facem asta?**

Odată ce ai fructele în cutii, nu mai poți vedea fiecare fruct individual (SQL "uită" detaliile despre fiecare măr în parte), dar poți folosi funcții de agregare pentru a afla detalii despre fiecare cutie:

- **COUNT**: Câte fructe sunt în cutia de mere?
- **SUM**: Cât cântăresc toate merele la un loc?
- **AVG**: Care este greutatea medie a unui măr?
- **MAX**: Care este cel mai greu măr?
- **MIN**: Care este cel mai ușor măr?

#### Exemplul Practic: Vânzări pe Categorii

Presupun că ai acest tabel numit `Vanzari`:

```
Produs   | Categorie | Pret
─────────┼───────────┼──────
Laptop   | IT        | 4000
Mouse    | IT        | 100
Tricou   | Haine     | 50
Monitor  | IT        | 1200
Blugi    | Haine     | 150
Cizme    | Haine     | 200
```

Dacă vrei să știi cât ai vândut pe fiecare categorie, scrii:

```sql
SELECT Categorie, SUM(Pret)
FROM Vanzari
GROUP BY Categorie;
```

Rezultatul va fi:

```
Categorie | SUM(Pret)
──────────┼──────────
IT        | 5300
Haine     | 400
```

**Ce s-a întâmplat?**

1. SQL a pus toate produsele IT într-o cutie (Laptop, Mouse, Monitor)
2. SQL a pus toate produsele Haine într-alta (Tricou, Blugi, Cizme)
3. SQL a calculat SUM(Pret) pentru fiecare cutie
4. Acum SQL "uită" despre produsele individuale — știe doar totalurile pe categorie

---

### ⭐ REGULA DE AUR A GROUP BY

```
Cea mai importantă regulă este: 
Orice coloană care apare în SELECT și nu este înfășurată 
într-o funcție (ca SUM, COUNT, etc.) TREBUIE să apară în GROUP BY
```

#### De ce această regulă?

```sql
-- ❌ GREȘIT: SQL nu știe ce produs să afișeze pentru grupul "IT"
--    (Laptop, Mouse sau Monitor?)
SELECT Categorie, Produs, SUM(Pret) 
FROM Vanzari 
GROUP BY Categorie;
-- ORA-00937: not a single-group group function

-- ✅ CORECT: selectezi coloane care sunt în GROUP BY
SELECT Categorie, SUM(Pret) AS Total
FROM Vanzari 
GROUP BY Categorie;

-- ✅ CORECT: grupezi și după Produs (fiecare produs e propria cutie)
SELECT Categorie, Produs, SUM(Pret) AS Total
FROM Vanzari 
GROUP BY Categorie, Produs;
```

**De ce se întâmplă eroarea?**

Odată ce ai "pus fructele în cutii", SQL nu mai poate accesa informații despre fructele individuale. Dacă ceri `Produs` (un detaliu individual) dar grupezi doar după `Categorie`, SQL nu știe care produs să afișeze din cutie.

---

### 4.1 GROUP BY Simplu

```sql
-- Grupează după departament și calculează salariu mediu
SELECT DEPARTMENT_ID, AVG(SALARY) AS SALARIU_MEDIU, COUNT(*) AS NR_ANGAJATI
FROM EMPLOYEES
GROUP BY DEPARTMENT_ID
ORDER BY DEPARTMENT_ID;
```

Rezultat exemplu:
```
DEPARTMENT_ID | SALARIU_MEDIU | NR_ANGAJATI
10            | 4400.00       | 1
20            | 9500.00       | 2
30            | 4150.00       | 6
50            | 3475.50       | 45
...
```

---

### 4.2 GROUP BY cu Expresii

```sql
-- Grupează după funcția aplicată pe coloană
SELECT UPPER(FIRST_NAME) AS NUME_UPPER, COUNT(*) NR
FROM EMPLOYEES
GROUP BY UPPER(FIRST_NAME);

-- Grupează după an de angajare
SELECT TO_CHAR(HIRE_DATE, 'YYYY') AS AN_ANGAJARE, COUNT(*) NR_ANGAJATI
FROM EMPLOYEES
GROUP BY TO_CHAR(HIRE_DATE, 'YYYY');

-- Grupează după salariu împărțit la 1000 (doar ordinul de mărime)
SELECT FLOOR(SALARY / 1000) * 1000 AS INTERVAL_SAL, COUNT(*) NR
FROM EMPLOYEES
GROUP BY FLOOR(SALARY / 1000) * 1000;
```

---

### 4.3 GROUP BY cu Mai Multe Coloane

```sql
-- Grupează după departament ȘI job
SELECT DEPARTMENT_ID, JOB_ID, COUNT(*) AS NR_ANGAJATI, AVG(SALARY) AS SALARIU_MEDIU
FROM EMPLOYEES
GROUP BY DEPARTMENT_ID, JOB_ID
ORDER BY DEPARTMENT_ID, JOB_ID;
```

Rezultat exemplu:
```
DEPARTMENT_ID | JOB_ID   | NR_ANGAJATI | SALARIU_MEDIU
10            | AD_ASST  | 1           | 4400.00
20            | MK_MAN   | 1           | 13000.00
20            | MK_REP   | 1           | 6000.00
30            | PU_CLERK | 5           | 2780.00
30            | PU_MAN   | 1           | 11000.00
...
```

**Puncte importante:**

```
✅ Rândurile din aceleași grupuri (dept 20, job MK) sunt îmbinate
✅ Se calculează COUNT, AVG etc. pe fiecare grup separat
❌ Nu poți selecta JOB_ID dacă e doar DEPARTMENT_ID în GROUP BY
```

---

### 4.4 Combinații Goale — NVL în GROUP BY

```sql
-- ❌ Problemă: NULL-urile din COMMISSION_PCT sunt tratate ca o grupă separată
SELECT COMMISSION_PCT, COUNT(*) NR
FROM EMPLOYEES
GROUP BY COMMISSION_PCT;

-- Rezultat:
-- NULL | 35 (toți angajații fără comision sunt într-un singur grup)
-- 0.1  | 3
-- 0.15 | 10
-- ...

-- ✅ Soluție: înlocuiește NULL cu o valoare
SELECT NVL(COMMISSION_PCT, 0) AS COMMISSION, COUNT(*) NR
FROM EMPLOYEES
GROUP BY NVL(COMMISSION_PCT, 0);

-- Rezultat:
-- 0    | 35
-- 0.1  | 3
-- 0.15 | 10
-- ...
```

---

### 4.5 Ordinea Conținutului în GROUP BY

Ordinea coloanelor în `GROUP BY` nu trebuie să coincidă cu ordinea din `SELECT`:

```sql
-- Valabil:
SELECT DEPARTMENT_ID, JOB_ID, COUNT(*) NR
FROM EMPLOYEES
GROUP BY JOB_ID, DEPARTMENT_ID;  -- ordine inversă

-- Oracle le grupează corect indiferent de ordine
```

---

### 4.6 WHERE vs GROUP BY — Diferența Critică

```
WHERE:     Se execută ÎNAINTE de GROUP BY
           Filtrează RÂNDURI individuale
           Nu poate folosi funcții aggregate (COUNT, SUM, AVG, etc.)
           Se aplică la datele inițiale

GROUP BY:  Se execută DUPĂ WHERE
           Grupează rândurile care au trecut de WHERE
           TREBUIE să conțină coloanele care nu sunt în funcții aggregate
           Funcțiile aggregate se calculează pe fiecare grup
```

**Exemplu care ilustrează diferența:**

```sql
-- Calculează salariu mediu pe departament, PENTRU TOȚI ANGAJAȚII
SELECT DEPARTMENT_ID, AVG(SALARY) AS SAL_MEDIU
FROM EMPLOYEES
GROUP BY DEPARTMENT_ID;

-- Calculează salariu mediu pe departament, dar DOAR pentru angajații cu SAL > 2000
SELECT DEPARTMENT_ID, AVG(SALARY) AS SAL_MEDIU
FROM EMPLOYEES
WHERE SALARY > 2000
GROUP BY DEPARTMENT_ID;
-- WHERE filtrează mai întâi (reduce setul de date), apoi GROUP BY grupează restul
```

---

## 5. HAVING — Filtrarea Grupurilor

`HAVING` se execută **după GROUP BY** și filtrează **grupurile** (nu rândurile individuale). **Poate folosi funcții aggregate.**

### 5.1 Conceptul: HAVING filtrează "Cutiile"

```
WHERE:   Filtrează RÂNDURI înainte de a crea "cutiile"
         ❌ Nu poate folosi COUNT/SUM/AVG

HAVING:  Filtrează "CUTIILE" după ce sunt create
         ✅ TREBUIE să folosească COUNT/SUM/AVG
```

### 5.2 HAVING Simplu

```sql
-- ❌ GREȘIT: COUNT() nu poate fi în WHERE
SELECT DEPARTMENT_ID, COUNT(*) NR_ANGAJATI
FROM EMPLOYEES
WHERE COUNT(*) > 5
GROUP BY DEPARTMENT_ID;
-- ORA-00934

-- ✅ CORECT: COUNT() merge în HAVING
SELECT DEPARTMENT_ID, COUNT(*) NR_ANGAJATI
FROM EMPLOYEES
GROUP BY DEPARTMENT_ID
HAVING COUNT(*) > 5;
-- Returnează doar departamentele cu mai mult de 5 angajați
```

**Exemplu:** Afișează departamentele care au salariu mediu mai mare de 5000:

```sql
SELECT DEPARTMENT_ID, AVG(SALARY) SAL_MEDIU
FROM EMPLOYEES
GROUP BY DEPARTMENT_ID
HAVING AVG(SALARY) > 5000
ORDER BY SAL_MEDIU DESC;
```

---

### 5.3 HAVING cu Mai Multe Condiții

```sql
-- Departamentele cu 5+ angajați ȘI salariu mediu > 3000
SELECT DEPARTMENT_ID, COUNT(*) NR_ANGAJATI, AVG(SALARY) SAL_MEDIU, MAX(SALARY) SAL_MAX
FROM EMPLOYEES
GROUP BY DEPARTMENT_ID
HAVING COUNT(*) >= 5 AND AVG(SALARY) > 3000
ORDER BY DEPARTMENT_ID;
```

---

### 5.4 Combinare WHERE + GROUP BY + HAVING

```sql
-- Selectează doar angajații angajați după 2005
-- Grupează după departament
-- Filtrează grupurile care au salariu mediu > 4000
SELECT DEPARTMENT_ID, COUNT(*) NR_ANGAJATI, AVG(SALARY) SAL_MEDIU
FROM EMPLOYEES
WHERE HIRE_DATE > TO_DATE('01-01-2005', 'DD-MM-YYYY')
GROUP BY DEPARTMENT_ID
HAVING AVG(SALARY) > 4000
ORDER BY SAL_MEDIU DESC;
```

**Fluxul de execuție:**

```
1. WHERE: Filtrează angajații cu HIRE_DATE > 01-01-2005
2. GROUP BY: Grupează rândurile rămase după DEPARTMENT_ID
3. HAVING: Filtrează grupurile cu AVG(SALARY) > 4000
4. SELECT: Afișează coloanele
5. ORDER BY: Sortează după SAL_MEDIU DESC
```

---

### 5.5 Operatori în HAVING

Orice operator valabil în WHERE funcționează și în HAVING:

```sql
-- BETWEEN în HAVING
SELECT DEPARTMENT_ID, AVG(SALARY) SAL_MEDIU
FROM EMPLOYEES
GROUP BY DEPARTMENT_ID
HAVING AVG(SALARY) BETWEEN 3000 AND 6000;

-- IN în HAVING
SELECT JOB_ID, COUNT(*) NR
FROM EMPLOYEES
GROUP BY JOB_ID
HAVING COUNT(*) IN (1, 2, 3);
-- Doar job-urile care au 1, 2 sau 3 angajați

-- AND / OR în HAVING
SELECT DEPARTMENT_ID, COUNT(*) NR, AVG(SALARY) SAL_MEDIU
FROM EMPLOYEES
GROUP BY DEPARTMENT_ID
HAVING COUNT(*) > 5 OR AVG(SALARY) > 8000;
```

---

### 5.6 HAVING cu Alias — Atenție!

⚠️ **Atenție:** Alias-ul din SELECT nu funcționează întotdeauna în HAVING:

```sql
-- ❌ RISCANT: nu e garantat că merge (depinde de versiune Oracle)
SELECT DEPARTMENT_ID, COUNT(*) NR_ANGAJATI
FROM EMPLOYEES
GROUP BY DEPARTMENT_ID
HAVING NR_ANGAJATI > 5;
-- Poate funcționa, dar nu e reliable

-- ✅ RECOMANDAT: folosește expresia din GROUP BY
SELECT DEPARTMENT_ID, COUNT(*) NR_ANGAJATI
FROM EMPLOYEES
GROUP BY DEPARTMENT_ID
HAVING COUNT(*) > 5;
```

---

## 6. Sortarea Datelor (ORDER BY)

Clauza `ORDER BY` sortează rezultatul interogării. Se execută **ultima** dintre toate clauzele.

### 6.1 Sintaxă și opțiuni

```sql
ORDER BY {expresie | alias | poziţie} [ASC | DESC] [NULLS FIRST | NULLS LAST]
         [, ...]
```

- **Default**: Sortarea este `ASC` (crescătoare).
- Se poate sorta după: coloană, alias, expresie, sau **poziția coloanei** în lista `SELECT`.

---

### 6.2 Exemple practice

#### Sortare simplă crescătoare (ASC — default)

```sql
SELECT FIRST_NAME, SALARY
FROM EMPLOYEES
ORDER BY SALARY;

-- Echivalent explicit
SELECT FIRST_NAME, SALARY
FROM EMPLOYEES
ORDER BY SALARY ASC;
```

---

#### Sortare descrescătoare (DESC)

```sql
SELECT FIRST_NAME, SALARY
FROM EMPLOYEES
ORDER BY SALARY DESC;
```

---

#### Sortare după alias

```sql
SELECT FIRST_NAME, SALARY * 12 AS SAL_ANUAL
FROM EMPLOYEES
ORDER BY SAL_ANUAL DESC;
-- ✅ Aliasul poate fi folosit în ORDER BY (se execută după SELECT)
```

---

#### Sortare după poziție

```sql
SELECT FIRST_NAME, SALARY, DEPARTMENT_ID
FROM EMPLOYEES
ORDER BY 2 DESC, 3 ASC;
-- Echivalent cu: ORDER BY SALARY DESC, DEPARTMENT_ID ASC
```

> ⚠️ Sortarea după poziție este fragilă — dacă schimbi ordinea coloanelor din `SELECT`, sortarea se schimbă și ea. Preferă sortarea după nume de coloană sau alias.

---

#### Sortare după mai multe coloane

```sql
SELECT FIRST_NAME, SALARY, COMMISSION_PCT
FROM EMPLOYEES
ORDER BY SALARY DESC, COMMISSION_PCT DESC;
```

Sortarea se face mai întâi după **SALARY descrescător**, apoi pentru rândurile cu același SALARY, după **COMMISSION_PCT descrescător**.

---

#### Sortare după expresie

```sql
-- Sortează după salariul anual, chiar dacă nu e în SELECT
SELECT FIRST_NAME, SALARY
FROM EMPLOYEES
ORDER BY SALARY * 12 DESC;

-- Sortare după funcție
SELECT FIRST_NAME
FROM EMPLOYEES
ORDER BY LENGTH(FIRST_NAME) ASC;
-- Sortează după lungimea numelui (cei cu cele mai scurte nume primii)
```

---

### 6.8 NULL în ORDER BY

```sql
-- Sortare ASC: NULL apare ULTIMUL (implicit în Oracle)
SELECT FIRST_NAME, COMMISSION_PCT
FROM EMPLOYEES
ORDER BY COMMISSION_PCT ASC;

-- Sortare DESC: NULL apare PRIMUL (implicit în Oracle)
SELECT FIRST_NAME, COMMISSION_PCT
FROM EMPLOYEES
ORDER BY COMMISSION_PCT DESC;

-- Control explicit al poziției NULL
SELECT FIRST_NAME, COMMISSION_PCT
FROM EMPLOYEES
ORDER BY COMMISSION_PCT ASC NULLS FIRST;  -- NULL primul, indiferent de direcție

SELECT FIRST_NAME, COMMISSION_PCT
FROM EMPLOYEES
ORDER BY COMMISSION_PCT DESC NULLS LAST;  -- NULL ultimul, indiferent de direcție
```

---

## 7. Funcții Aggregate

| Funcție | Descriere | Exemplu | Returnează NULL dacă |
| :--- | :--- | :--- | :--- |
| `COUNT(*)` | Contează rândurile | `COUNT(*) AS NR_RÂNDURI` | Niciodată (e 0 pentru grup gol) |
| `COUNT(coloană)` | Contează non-NULL | `COUNT(COMMISSION_PCT)` | Nu sunt rânduri cu date |
| `SUM(coloană)` | Suma valorilor | `SUM(SALARY)` | Toate rândurile sunt NULL |
| `AVG(coloană)` | Medie aritmetică | `AVG(SALARY)` | Toate rândurile sunt NULL |
| `MAX(coloană)` | Valoarea maximă | `MAX(SALARY)` | Toate rândurile sunt NULL |
| `MIN(coloană)` | Valoarea minimă | `MIN(SALARY)` | Toate rândurile sunt NULL |

### 7.1 COUNT — cu Atenție la NULL

```sql
-- COUNT(*): contează TOATE rândurile, inclusiv cu NULL
SELECT COUNT(*) NR_TOTAL FROM EMPLOYEES;
-- Rezultat: 107 (toți angajații)

-- COUNT(coloană): contează doar non-NULL
SELECT COUNT(COMMISSION_PCT) NR_CU_COMISION FROM EMPLOYEES;
-- Rezultat: 35 (doar cei care au comision, NULL-urile se ignoră)

-- Diferența:
SELECT 
  COUNT(*) AS TOTAL,
  COUNT(COMMISSION_PCT) AS CU_COMISION,
  COUNT(*) - COUNT(COMMISSION_PCT) AS FARA_COMISION
FROM EMPLOYEES;
-- TOTAL: 107
-- CU_COMISION: 35
-- FARA_COMISION: 72
```

---

### 7.2 SUM — Ignoră NULL

```sql
-- SUM: adună doar non-NULL (ignoră NULL-urile)
SELECT SUM(COMMISSION_PCT) TOTAL_COMISION FROM EMPLOYEES;
-- NULL-urile nu afectează suma

-- ⚠️ Dacă TOATE rândurile sunt NULL → returnează NULL (nu 0!)
SELECT SUM(COMMISSION_PCT) FROM EMPLOYEES
WHERE COMMISSION_PCT = 'XYZ';  -- niciun rând nu se potrivește
-- Rezultat: NULL

-- Soluție: foloseşte NVL sau COALESCE
SELECT NVL(SUM(COMMISSION_PCT), 0) FROM EMPLOYEES;
-- Returnează 0 în loc de NULL
```

---

### 7.3 AVG — Ignoră NULL

```sql
-- AVG: medie pe valorile non-NULL
SELECT AVG(SALARY) SALARIU_MEDIU FROM EMPLOYEES;
-- NULL-urile sunt ignorate (nu sunt considerate 0)

-- ⚠️ AVG(SALARY) ≠ SUM(SALARY) / COUNT(*)
-- De ce? Pentru că COUNT(*) include rândurile cu NULL, dar AVG nu

SELECT 
  COUNT(*) AS TOTAL_RÂNDURI,
  COUNT(SALARY) AS RÂNDURI_CU_SALARY,
  SUM(SALARY) / COUNT(*) AS MEDIE_INCORECTĂ,
  AVG(SALARY) AS MEDIE_CORECTĂ
FROM EMPLOYEES;
-- MEDIE_INCORECTĂ vs MEDIE_CORECTĂ ar putea fi diferite dacă sunt NULL-uri
```

---

### 7.4 MAX și MIN

```sql
-- MAX/MIN funcționează cu orice tip de date
SELECT 
  MAX(SALARY) SAL_MAX,
  MIN(SALARY) SAL_MIN,
  MAX(HIRE_DATE) ULTIMA_ANGAJARE,
  MIN(HIRE_DATE) PRIMA_ANGAJARE
FROM EMPLOYEES;

-- Cu GROUP BY:
SELECT 
  DEPARTMENT_ID,
  MAX(SALARY) SAL_MAX,
  MIN(SALARY) SAL_MIN
FROM EMPLOYEES
GROUP BY DEPARTMENT_ID;
```

---

### 7.5 Funcții Aggregate fără GROUP BY

```sql
-- Funcții aggregate pe ÎNTREAGA tabel
SELECT 
  COUNT(*) AS TOTAL_ANGAJATI,
  COUNT(COMMISSION_PCT) AS CU_COMISION,
  SUM(SALARY) AS TOTAL_SALARII,
  AVG(SALARY) AS SAL_MEDIU,
  MAX(SALARY) AS SAL_MAX,
  MIN(SALARY) AS SAL_MIN
FROM EMPLOYEES;

-- Rezultat: o singură linie cu agregări pe toți angajații
```

---

### 7.6 Aggregate Functions cu DISTINCT

```sql
-- COUNT(DISTINCT coloană): contează valori unice
SELECT COUNT(DISTINCT DEPARTMENT_ID) NR_DEPARTAMENTE FROM EMPLOYEES;
-- Rezultat: 11 (11 departamente diferite)

-- SUM(DISTINCT coloană): suma valorilor unice
SELECT SUM(DISTINCT SALARY) FROM EMPLOYEES;
-- Dacă 2 angajați au aceleași salariu, se adună doar o dată

-- AVG(DISTINCT coloană): medie a valorilor unice
SELECT AVG(DISTINCT SALARY) FROM EMPLOYEES;
```

---

## 8. Logica Valorilor NULL și UNKNOWN

`NULL` reprezintă absența unei valori — nu este zero, nu este șir gol, nu este `false`. Este **necunoscut**.

### 8.1 Reguli fundamentale cu NULL

```sql
-- ✅ Verificare corectă
WHERE COMMISSION_PCT IS NULL       -- angajații fără comision
WHERE COMMISSION_PCT IS NOT NULL   -- angajații cu comision

-- ❌ GREȘIT: comparație directă cu NULL → returnează întotdeauna UNKNOWN (niciun rând)
WHERE COMMISSION_PCT = NULL        -- nu funcționează
WHERE COMMISSION_PCT <> NULL       -- nu funcționează
```

**De ce `= NULL` nu funcționează?**

În logica SQL, compararea oricărei valori cu `NULL` produce `UNKNOWN` (nu `TRUE`, nu `FALSE`). Clauza `WHERE` include un rând doar dacă condiția este `TRUE` — `UNKNOWN` este tratat ca `FALSE`.

```
5 = NULL       → UNKNOWN
NULL = NULL    → UNKNOWN (chiar și NULL nu este egal cu NULL!)
NULL IS NULL   → TRUE ✅
```

---

### 8.2 Tabel de Adevăr — Logica cu UNKNOWN

| Operație | Rezultat | Explicație |
| :--- | :--- | :--- |
| `TRUE AND TRUE` | TRUE | — |
| `TRUE AND FALSE` | FALSE | — |
| `TRUE AND UNKNOWN` | UNKNOWN | Nu știm dacă ambele sunt true |
| `FALSE AND UNKNOWN` | **FALSE** | Chiar dacă al doilea e necunoscut, primul e false → tot false |
| `FALSE OR FALSE` | FALSE | — |
| `TRUE OR UNKNOWN` | **TRUE** | Chiar dacă al doilea e necunoscut, primul e true → tot true |
| `FALSE OR UNKNOWN` | UNKNOWN | Nu știm dacă cel puțin unul e true |
| `NOT UNKNOWN` | UNKNOWN | Negarea unui necunoscut rămâne necunoscută |

> 💡 **Regula de reținut:** `FALSE AND orice = FALSE`, `TRUE OR orice = TRUE`. În rest, UNKNOWN se „propagă".

---

### 8.3 NULL în expresii aritmetice și funcții

```sql
-- Orice operație aritmetică cu NULL → NULL
SELECT 5 + NULL FROM DUAL;      -- rezultat: NULL
SELECT NULL * 100 FROM DUAL;    -- rezultat: NULL
SELECT SALARY + NULL FROM DUAL; -- rezultat: NULL (nu SALARY!)

-- Funcția NVL: înlocuiește NULL cu o valoare implicită
SELECT FIRST_NAME, NVL(COMMISSION_PCT, 0) AS COMMISSION
FROM EMPLOYEES;
-- Dacă COMMISSION_PCT este NULL, afișează 0

-- Funcția NVL2: dacă NOT NULL → val1, dacă NULL → val2
SELECT FIRST_NAME, NVL2(COMMISSION_PCT, 'Are comision', 'Fără comision') AS STATUS
FROM EMPLOYEES;

-- Funcția COALESCE: returnează primul non-NULL din listă
SELECT COALESCE(COMMISSION_PCT, SALARY * 0.05, 0) AS BONUS
FROM EMPLOYEES;
```

---

### 8.4 NULL în ORDER BY

```sql
-- Sortare ASC: NULL apare ULTIMUL (implicit în Oracle)
SELECT FIRST_NAME, COMMISSION_PCT
FROM EMPLOYEES
ORDER BY COMMISSION_PCT ASC;

-- Sortare DESC: NULL apare PRIMUL (implicit în Oracle)
SELECT FIRST_NAME, COMMISSION_PCT
FROM EMPLOYEES
ORDER BY COMMISSION_PCT DESC;

-- Control explicit al poziției NULL
SELECT FIRST_NAME, COMMISSION_PCT
FROM EMPLOYEES
ORDER BY COMMISSION_PCT ASC NULLS FIRST;  -- NULL primul, indiferent de direcție

SELECT FIRST_NAME, COMMISSION_PCT
FROM EMPLOYEES
ORDER BY COMMISSION_PCT DESC NULLS LAST;  -- NULL ultimul, indiferent de direcție
```

---

## 9. Operatorul ROWNUM și Limitarea rezultatelor

Oracle nu are `LIMIT` ca MySQL — folosește `ROWNUM` pentru a limita numărul de rânduri returnate:

```sql
-- Primele 10 rânduri din tabel
SELECT * FROM EMPLOYEES
WHERE ROWNUM <= 10;

-- ⚠️ CAPCANA cu ROWNUM și ORDER BY:
-- Aceasta returnează 10 rânduri ALEATORII, nu primii 10 după salariu!
SELECT FIRST_NAME, SALARY FROM EMPLOYEES
WHERE ROWNUM <= 10
ORDER BY SALARY DESC;
-- ROWNUM se aplică ÎNAINTE de ORDER BY

-- ✅ Corect: folosește subcerere pentru a ordona mai întâi, apoi limitezi
SELECT * FROM (
    SELECT FIRST_NAME, SALARY
    FROM EMPLOYEES
    ORDER BY SALARY DESC
)
WHERE ROWNUM <= 10;
-- Acum returnezi primii 10 angajați după salariu
```

---

## 10. Depanarea Query-urilor — Erori Frecvente

### 10.1 Exemplu de query cu erori (exercițiu din curs)

```sql
-- Query-ul original cu erori:
SELECT employee_id, last_name 
salary * 12 ANNUAL SALARY 
FROM employees;
```

**Identifică toate erorile:**

<details>
<summary>🔍 Răspuns</summary>

```
Eroare 1: Lipsește virgula după last_name
          last_name    ← lipsește virgula aici
          salary * 12

Eroare 2: Aliasul "ANNUAL SALARY" conține un spațiu
          ANNUAL SALARY → trebuie "ANNUAL SALARY" (cu ghilimele duble)

Query corect:
SELECT employee_id, last_name,
       salary * 12 AS "ANNUAL SALARY"
FROM employees;
```

</details>

---

### 10.2 Erori frecvente și soluții

| Eroare | Cod Oracle | Cauză | Soluție |
| :--- | :--- | :--- | :--- |
| Alias fără ghilimele cu spații | ORA-00923 | `AS Salariu Anual` | `AS "Salariu Anual"` |
| Coloană în SELECT fără GROUP BY | ORA-00937 | Regula de aur încălcată | Adaugă coloana în GROUP BY sau folosește aggregate |
| COUNT în WHERE | ORA-00934 | WHERE se execută înainte de GROUP BY | Mută în HAVING |
| Alias din SELECT în WHERE | ORA-00904 | WHERE se execută înainte de SELECT | Folosește expresia originală |
| `SELECT *, col` fără prefix | ORA-00923 | `*` și coloane individuale incompatibile | `SELECT t.*, t.col` cu alias tabel |
| Lipsă virgulă între coloane | ORA-00923 | Sintaxă incorectă | Adaugă virgulă între coloane |
| String cu ghilimele duble | ORA-00904 | `WHERE name = "King"` | `WHERE name = 'King'` (apostrofuri) |
| Coloană ambiguă | ORA-00918 | Aceeași coloană în mai multe tabele | Prefixează: `E.SALARY` |
| NOT IN cu NULL | 0 rânduri | NULL în listă inversează condiția | Filtrează NULL explicit |

---

## 11. Exerciții Practice cu Soluții

### Exercițiul 1 — SELECT de bază și alias-uri

Scrie query-uri pentru:

1. Afișează numele complet (FIRST_NAME + ' ' + LAST_NAME) ca `Nume Complet`, salariul lunar și salariul anual pentru toți angajații.
2. Afișează job-urile distincte din companie.
3. Afișează angajații care au comision (COMMISSION_PCT nu este NULL), cu numele complet și comisionul total anual calculat.

<details>
<summary>🔍 Soluție</summary>

```sql
-- 1. Nume complet și salarii
SELECT FIRST_NAME || ' ' || LAST_NAME AS "Nume Complet",
       SALARY AS "Salariu Lunar",
       SALARY * 12 AS "Salariu Anual"
FROM EMPLOYEES;

-- 2. Job-uri distincte
SELECT DISTINCT JOB_ID
FROM EMPLOYEES
ORDER BY JOB_ID;

-- 3. Angajați cu comision
SELECT FIRST_NAME || ' ' || LAST_NAME AS "Nume Complet",
       COMMISSION_PCT,
       SALARY * COMMISSION_PCT * 12 AS "Comision Anual"
FROM EMPLOYEES
WHERE COMMISSION_PCT IS NOT NULL;
```

</details>

---

### Exercițiul 2 — Filtrare cu WHERE

1. Afișează angajații cu salariul între 4000 și 8000, din departamentele 20, 50 sau 80, ordonați descrescător după salariu.
2. Afișează angajații care nu au manager (MANAGER_ID este NULL).
3. Afișează angajații angajați după 1 ianuarie 2005, cu salariul peste 6000.

<details>
<summary>🔍 Soluție</summary>

```sql
-- 1. Filtrare combinată
SELECT FIRST_NAME, LAST_NAME, SALARY, DEPARTMENT_ID
FROM EMPLOYEES
WHERE SALARY BETWEEN 4000 AND 8000
  AND DEPARTMENT_ID IN (20, 50, 80)
ORDER BY SALARY DESC;

-- 2. Angajați fără manager
SELECT FIRST_NAME, LAST_NAME, MANAGER_ID
FROM EMPLOYEES
WHERE MANAGER_ID IS NULL;

-- 3. Angajați angajați după 2005 cu salariu > 6000
SELECT FIRST_NAME, LAST_NAME, HIRE_DATE, SALARY
FROM EMPLOYEES
WHERE HIRE_DATE > TO_DATE('01-01-2005', 'DD-MM-YYYY')
  AND SALARY > 6000
ORDER BY HIRE_DATE;
```

</details>

---

### Exercițiul 3 — GROUP BY și HAVING

1. Afișează pe fiecare departament: ID-ul, numărul de angajați și salariul mediu.
2. Afișează departamentele care au mai mult de 10 angajați.
3. Afișează job-urile care au salariu mediu mai mare de 5000.

<details>
<summary>🔍 Soluție</summary>

```sql
-- 1. Departament - nr angajați - salariu mediu
SELECT DEPARTMENT_ID, COUNT(*) NR_ANGAJATI, AVG(SALARY) SAL_MEDIU
FROM EMPLOYEES
GROUP BY DEPARTMENT_ID
ORDER BY DEPARTMENT_ID;

-- 2. Departamente cu 10+ angajați
SELECT DEPARTMENT_ID, COUNT(*) NR_ANGAJATI
FROM EMPLOYEES
GROUP BY DEPARTMENT_ID
HAVING COUNT(*) > 10;

-- 3. Job-uri cu salariu mediu > 5000
SELECT JOB_ID, AVG(SALARY) SAL_MEDIU, COUNT(*) NR_ANGAJATI
FROM EMPLOYEES
GROUP BY JOB_ID
HAVING AVG(SALARY) > 5000
ORDER BY SAL_MEDIU DESC;
```

</details>

---

### Exercițiul 4 — WHERE + GROUP BY + HAVING

1. Angajații angajați după 2005, cu salariu > 3000, din dept care au 5+ angajați cu aceste criterii.

<details>
<summary>🔍 Soluție</summary>

```sql
SELECT DEPARTMENT_ID, COUNT(*) NR, AVG(SALARY) MEDIE, MAX(SALARY) MAX_SAL
FROM EMPLOYEES
WHERE HIRE_DATE > TO_DATE('01-01-2005', 'DD-MM-YYYY')
  AND SALARY > 3000
GROUP BY DEPARTMENT_ID
HAVING COUNT(*) >= 5
ORDER BY MEDIE DESC;
```

</details>

---

## 📝 Rezumat Rapid

```
╔═══════════════════════════════════════════════════════════════╗
║           ORDINEA DE EXECUȚIE — CEA MAI IMPORTANTĂ           ║
╠═══════════════════════════════════════════════════════════════╣
║ FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY         ║
╚═══════════════════════════════════════════════════════════════╝

╔═══════════════════════════════════════════════════════════════╗
║              REGULA FUNDAMENTALĂ GROUP BY                     ║
╠═══════════════════════════════════════════════════════════════╣
║ Orice coloană din SELECT (non-aggregate) TREBUIE în GROUP BY  ║
╚═══════════════════════════════════════════════════════════════╝

╔═══════════════════════════════════════════════════════════════╗
║                    WHERE vs HAVING                            ║
╠═══════════════════════════════════════════════════════════════╣
║ WHERE:  Filtrează RÂNDURI (înainte GROUP BY)                  ║
║         ❌ Nu merge COUNT/SUM/AVG                             ║
║ HAVING: Filtrează GRUPURI (după GROUP BY)                    ║
║         ✅ Trebuie COUNT/SUM/AVG                              ║
╚═══════════════════════════════════════════════════════════════╝

```
