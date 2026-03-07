# 🔧 Funcții SQL Oracle — Referință Completă cu Exemple Practice

---

## 1. Introducere

**Funcțiile SQL** sunt predefinite în sistemul Oracle și pot fi utilizate direct în instrucțiuni SQL. Nu trebuie confundate cu funcțiile definite de utilizator (scrise în PL/SQL).

### Reguli generale:
- Dacă o funcție primește un argument de **alt tip** decât cel așteptat, sistemul încearcă o **conversie implicită**.
- Dacă o funcție primește un argument **NULL**, returnează automat `NULL`.
- ⚠️ **Excepții** de la regula NULL: `CONCAT`, `NVL` și `REPLACE`.

### Clasificare:

| Tip | Descriere |
| :--- | :--- |
| **Single-row** | Returnează un rezultat pentru **fiecare linie** din tabel |
| **Multiple-row (agregat)** | Returnează un rezultat pentru **un grup** de linii |

```sql
-- Exemplu single-row (câte un rezultat per linie)
SELECT LENGTH(FIRST_NAME) FROM EMPLOYEES;

-- Exemplu multiple-row (un singur rezultat pentru tot tabelul)
SELECT MAX(SALARY) FROM EMPLOYEES;
```

> 💡 **Testare rapidă:** `SELECT apel_functie FROM dual;`

---

## 2. Tipuri de Date: CHAR vs VARCHAR2

Diferența dintre cele două tipuri de șiruri de caractere este esențială:

| Tip | Stocare | Input `'asd'` într-o coloană de 10 | Lungime stocată |
| :--- | :--- | :--- | :--- |
| `VARCHAR2(10)` | Lungime **variabilă** | `'asd'` | 3 |
| `CHAR(10)` | Lungime **fixă** | `'asd       '` (+ 7 blank-uri) | 10 |

> ⚠️ `CHAR` completează automat cu spații până la lungimea declarată. Aceasta poate cauza probleme la comparații dacă nu ești atent.

---

## 3. Funcții de Conversie

### 3.1. Prezentare generală

| Funcție | Descriere | Exemplu |
| :--- | :--- | :--- |
| `TO_CHAR` | Convertește un număr sau dată în șir de caractere | `TO_CHAR(SYSDATE, 'DD/MM/YYYY')` → `'18/04/2007'` |
| `TO_DATE` | Convertește un șir în dată calendaristică | `TO_DATE('18-APR-2007', 'dd-mon-yyyy')` |
| `TO_NUMBER` | Convertește un șir în număr | `TO_NUMBER('+123.234', 'S999D999')` → `123.234` |

---

### 3.2. Conversii implicite vs. explicite

**Implicite** — realizate automat de Oracle:

```
VARCHAR2 / CHAR  →  NUMBER      (ex: '-1' + 2 → 1)
VARCHAR2 / CHAR  →  DATE
NUMBER           →  VARCHAR2    (ex: 2 || 'ASD' → '2ASD')
DATE             →  VARCHAR2    (ex: SYSDATE || 'ASD')
```

```sql
-- Conversie implicită șir → număr
SELECT '-1' + 2 FROM DUAL;  -- rezultat: 1

-- Conversie implicită număr → șir (prin concatenare)
SELECT 2 || 'ASD' FROM DUAL;  -- rezultat: '2ASD'

-- Conversie implicită dată → șir (prin concatenare)
SELECT SYSDATE || 'ASD' FROM DUAL;
```

---

### 3.3. Conversia datelor calendaristice și `NLS_DATE_FORMAT`

> ⚠️ La conversia implicită șir → dată, formatul șirului **trebuie să respecte** `NLS_DATE_FORMAT`.

```sql
-- EROARE: NLS_DATE_FORMAT implicit este 'DD-MON-RR', nu 'DD-MM-YYYY'
SELECT * FROM EMPLOYEES
WHERE HIRE_DATE BETWEEN '20-02-1990' AND '20-03-2000';

-- Vizualizează formatul curent al datelor
SELECT VALUE FROM V$NLS_PARAMETERS
WHERE PARAMETER = 'NLS_DATE_FORMAT';

-- Schimbă formatul la nivel de sesiune
ALTER SESSION SET NLS_DATE_FORMAT = 'DD-MM-YYYY';
```

---

### 3.4. `TO_CHAR` — Formate pentru date calendaristice

| Element | Semnificație |
| :--- | :--- |
| `D` | Numărul zilei din săptămână (duminică=1, sâmbătă=7) |
| `DD` | Numărul zilei din lună |
| `DDD` | Numărul zilei din an |
| `DY` | Abrevierea zilei (MON, THU etc.) |
| `DAY` | Numele complet al zilei |
| `MM` | Numărul lunii |
| `MON` | Abrevierea lunii (JAN, FEB etc.) |
| `MONTH` | Numele complet al lunii |
| `YYYY` | Anul (4 cifre) |
| `HH12` / `HH24` | Orele (0–12 / 0–24) |
| `MI` | Minutele |
| `SS` | Secundele |

```sql
SELECT TO_CHAR(SYSDATE, 'DD-MM-YYYY HH24:MI:SS') FROM DUAL;
-- rezultat: '18-04-2007 14:35:22'

SELECT TO_CHAR(SYSDATE, 'DD.MONTH-YYYY HH:MI') FROM DUAL;
-- rezultat: '18.APRIL-2007 02:35'

SELECT TO_CHAR(SYSDATE) FROM DUAL;
-- conversie cu NLS_DATE_FORMAT implicit (fără format specificat)
```

---

### 3.5. `TO_CHAR` — Formate pentru numere

```sql
SELECT TO_CHAR(7.500, '9.999') FROM DUAL;
-- rezultat: '7.500'  (impune afișarea cu 3 zecimale)
```

---

### 3.6. `TO_NUMBER` — Reguli importante

```sql
-- ✅ Corect: cifrele din șir se încadrează în format
SELECT TO_NUMBER('+123.234', 'S999D999') FROM DUAL;

-- ❌ Eroare: formatul permite 1 zecimală, șirul are 3
SELECT TO_NUMBER('+123.234', 'S999D9') FROM DUAL;

-- ✅ Cu separator de mii
SELECT TO_NUMBER('123,123.234', '999G999D999') FROM DUAL;
SELECT TO_NUMBER('123,123.234', '999,999.999') FROM DUAL;

-- ❌ Eroare: nu poți combina ',' cu 'D' sau 'G' cu '.'
SELECT TO_NUMBER('123,123.234', '999G999.999') FROM DUAL;
```

> 💡 **Regulă:** În format, folosești fie perechea `G` și `D`, fie perechea `,` și `.` — **nu le combina**.

---

## 4. Funcții pentru Prelucrarea Caracterelor

| Funcție | Descriere | Exemplu |
| :--- | :--- | :--- |
| `LENGTH(string)` | Lungimea șirului | `LENGTH('Informatica')` → `11` |
| `SUBSTR(string, start [,n])` | Subșir de la `start`, `n` caractere | `SUBSTR('Informatica', 1, 4)` → `'Info'` |
| `LTRIM(string [,'chars'])` | Șterge caractere din **stânga** | `LTRIM('QQWQER', 'Q')` → `'WER'` |
| `RTRIM(string [,'chars'])` | Șterge caractere din **dreapta** | `RTRIM('infoXXXX', 'X')` → `'info'` |
| `TRIM(... FROM expr)` | Șterge din stânga, dreapta sau ambele | `TRIM(BOTH 'Q' FROM 'QQWQERQ')` → `'WQER'` |
| `LPAD(string, length [,'chars'])` | Completează la **stânga** | `LPAD('QQQ', 6, 'AS')` → `'ASAQQQQ'` |
| `RPAD(string, length [,'chars'])` | Completează la **dreapta** | `RPAD('info', 6, 'X')` → `'infoXX'` |
| `REPLACE(s1, s2 [,s3])` | Înlocuiește toate aparițiile lui `s2` cu `s3` | `REPLACE('QWQ12QW', 'QW', 'AB')` → `'ABQ12AB'` |
| `TRANSLATE(string, source, dest)` | Înlocuire caracter cu caracter | `TRANSLATE('QWQ12QW', 'QW', 'AB')` → `'ABA12AB'` |
| `UPPER(string)` | Transformă în **majuscule** | `UPPER('iNfO')` → `'INFO'` |
| `LOWER(string)` | Transformă în **minuscule** | `LOWER('InFo')` → `'info'` |
| `INITCAP(string)` | Prima literă majusculă | `INITCAP('iNfO')` → `'Info'` |
| `INSTR(string, chars [,start [,n]])` | Caută a `n`-a apariție, returnează poziția | `INSTR('ASDQWQWERT', 'QW', 4, 2)` → `6` |
| `ASCII(char)` | Codul ASCII al primului caracter | `ASCII('alfa')` → `97` |
| `CHR(num)` | Caracterul pentru codul ASCII | `CHR(97)` → `'a'` |
| `CONCAT(s1, s2)` | Concatenare două șiruri | `CONCAT('In', 'fo')` → `'Info'` |

---

### 4.1. Detalii `SUBSTR` — indexare de la 1, nu de la 0

```sql
SUBSTR('Informatica', 5, 2)  -- 'rm'      de la poziția 5, 2 caractere
SUBSTR('Informatica', 5)     -- 'rmatica' de la poziția 5 până la final
SUBSTR('Informatica', -5)    -- 'atica'   ultimele 5 caractere
SUBSTR('Informatica', -5, 3) -- 'ait'     primele 3 din ultimele 5
```

> ⚠️ Indexarea începe de la **1**, nu de la 0!

---

### 4.2. Detalii `LENGTH`

```sql
SELECT LENGTH('') FROM DUAL;  -- rezultat: NULL
```

> ⚠️ În Oracle, `''` (șir gol) este **echivalent cu NULL**. Nu există șir cu 0 caractere. Prin urmare `LENGTH('')` = `NULL`, nu `0`.

---

### 4.3. Detalii `INSTR`

```sql
-- Caută 'QW' în 'ASDQWQWERT' începând de la poziția 4, a 2-a apariție
SELECT INSTR('ASDQWQWERT', 'QW', 4, 2) FROM DUAL;  -- rezultat: 6

-- Dacă nu se găsește, returnează 0 (nu NULL!)
SELECT INSTR('ASDQWQWERT', 'QW', 4, 3) FROM DUAL;  -- rezultat: 0
```

---

### 4.4. `REPLACE` vs. `TRANSLATE`

```sql
-- REPLACE: înlocuiește subșirul 'QW' ca unitate
SELECT REPLACE('QWQ12QW', 'QW', 'AB') FROM DUAL;  -- 'ABQ12AB'

-- Dacă al 3-lea param lipsește, subșirul este șters
SELECT REPLACE('QWQ12QW', 'QW') FROM DUAL;  -- 'Q12'

-- TRANSLATE: înlocuiește caracter cu caracter (Q→A, W→B)
SELECT TRANSLATE('QWQ12QW', 'QW', 'AB') FROM DUAL;  -- 'ABA12AB'

-- Dacă 'W' nu are corespondent în destinație, este șters
SELECT TRANSLATE('QWQ12QW', 'QW', 'A') FROM DUAL;  -- 'AQ12A'
```

---

### 4.5. Exemple practice pe tabelul EMPLOYEES

```sql
-- Concatenare cu || (poate lega oricâte șiruri, alternativă la CONCAT)
SELECT FIRST_NAME || ' CASTIGA ' || SALARY FROM EMPLOYEES;
-- Echivalent cu:
SELECT CONCAT(FIRST_NAME, CONCAT(' CASTIGA ', SALARY)) FROM EMPLOYEES;

-- Angajați al căror LAST_NAME începe cu J, M sau are 'A' pe poziția 3
SELECT INITCAP(FIRST_NAME), UPPER(LAST_NAME), LENGTH(LAST_NAME) LG
FROM EMPLOYEES
WHERE UPPER(LAST_NAME) LIKE 'J%'
   OR UPPER(LAST_NAME) LIKE 'M%'
   OR UPPER(LAST_NAME) LIKE '__A%'
ORDER BY LG DESC;

-- Variantă cu SUBSTR și IN
SELECT INITCAP(FIRST_NAME), UPPER(LAST_NAME), LENGTH(LAST_NAME) LG
FROM EMPLOYEES
WHERE SUBSTR(UPPER(LAST_NAME), 1, 1) IN ('J', 'M')
   OR SUBSTR(UPPER(LAST_NAME), 3, 1) = 'A'
ORDER BY LG DESC;

-- Căutare tolerantă la spații și majuscule/minuscule
SELECT EMPLOYEE_ID, FIRST_NAME, DEPARTMENT_ID
FROM EMPLOYEES
WHERE UPPER(TRIM(FIRST_NAME)) = 'STEVEN';

-- Angajați al căror FIRST_NAME se termină cu 'E'
SELECT EMPLOYEE_ID, FIRST_NAME, LENGTH(FIRST_NAME),
       INSTR(UPPER(FIRST_NAME), 'A')
FROM EMPLOYEES
WHERE SUBSTR(UPPER(FIRST_NAME), -1) = 'E';
```

---

## 5. Funcții Aritmetice

### 5.1. Pe o singură valoare

| Funcție | Descriere |
| :--- | :--- |
| `ABS(n)` | Valoarea absolută |
| `CEIL(n)` | Partea întreagă superioară (rotunjire în sus) |
| `FLOOR(n)` | Partea întreagă inferioară (rotunjire în jos) |
| `ROUND(n [,d])` | Rotunjire la `d` zecimale (default: 0) |
| `TRUNC(n [,d])` | Trunchiere la `d` zecimale (default: 0) |
| `MOD(m, n)` | Restul împărțirii `m / n` |
| `POWER(m, n)` | `m` ridicat la puterea `n` |
| `SQRT(n)` | Rădăcina pătrată |
| `SIGN(n)` | Semnul: `-1`, `0` sau `1` |

### 5.2. Pe o listă de valori

| Funcție | Descriere |
| :--- | :--- |
| `LEAST(e1, e2, ..., en)` | Cea mai mică valoare din listă |
| `GREATEST(e1, e2, ..., en)` | Cea mai mare valoare din listă |

---

### 5.3. Detalii `ROUND` și `TRUNC`

```sql
ROUND(123.65123412, 4)   -- 123.6512   rotunjire la 4 zecimale
TRUNC(123.65123412, 4)   -- 123.6512   trunchiere la 4 zecimale

ROUND(123.65123412)      -- 124        echivalent cu ROUND(..., 0)
TRUNC(123.65123412)      -- 123        echivalent cu TRUNC(..., 0)

ROUND(123.65123412, -1)  -- 120        rotunjire la zeci
TRUNC(123.65123412, -1)  -- 120        trunchiere la zeci
```

> 💡 Al doilea parametru poate fi **negativ** pentru rotunjire/trunchiere la zeci, sute etc.

---

### 5.4. Exemple practice cu `MOD` și `TRUNC`

```sql
-- Angajați al căror salariu NU este multiplu de 1000
SELECT EMPLOYEE_ID, FIRST_NAME, SALARY,
       TO_CHAR(SALARY + SALARY * 15/100, '999999.99') "SALARIU NOU",
       ROUND(SALARY + SALARY * 15/100, 2) "ROTUNJIT"
FROM EMPLOYEES
WHERE MOD(SALARY, 1000) <> 0;

-- Generare bară vizuală din '$' proporțională cu salariul
SELECT EMPLOYEE_ID, SALARY,
       LPAD('$', TRUNC(SALARY/1000), '$')
FROM EMPLOYEES;
```

---

## 6. Funcții pentru Date Calendaristice

| Funcție | Descriere | Exemplu |
| :--- | :--- | :--- |
| `SYSDATE` | Data și ora curentă | `SELECT SYSDATE FROM dual;` |
| `ADD_MONTHS(data, n)` | Adaugă `n` luni la dată | `ADD_MONTHS('02-APR-2007', 3)` → `'02-JUL-2007'` |
| `NEXT_DAY(data, zi)` | Următoarea zi a săptămânii după dată | `NEXT_DAY('18-APR-2007', 'Monday')` → `'23-APR-2007'` |
| `LAST_DAY(data)` | Ultima zi a lunii | `LAST_DAY('02-DEC-2007')` → `'31-DEC-2007'` |
| `MONTHS_BETWEEN(d2, d1)` | Numărul de luni dintre două date | `MONTHS_BETWEEN('02-DEC-2005', '10-OCT-2002')` → `37.74` |
| `TRUNC(data [,fmt])` | Data cu ora/componenta resetată | `TRUNC(SYSDATE, 'DD')` → azi la 00:00 |
| `ROUND(data [,fmt])` | Rotunjire la cea mai apropiată unitate | `ROUND(SYSDATE, 'MI')` → rotunjit la minut |
| `EXTRACT(unit FROM data)` | Extrage o componentă din dată | `EXTRACT(YEAR FROM HIRE_DATE)` → `1987` |

---

### 6.1. Operații aritmetice cu date

| Operație | Tip rezultat | Descriere |
| :--- | :--- | :--- |
| `data ± număr_zile` | `DATE` | Adaugă/scade zile |
| `data1 - data2` | `NUMBER` | Numărul de zile dintre două date |

```sql
SELECT SYSDATE - 1       FROM DUAL;  -- ieri
SELECT SYSDATE - 0.5     FROM DUAL;  -- acum 12 ore
SELECT SYSDATE + 5/24/60 FROM DUAL;  -- peste 5 minute (conversie manuală)
SELECT SYSDATE + INTERVAL '5' MINUTE FROM DUAL;  -- identic, mai lizibil

-- Numărul de zile dintre două date (rezultat real, include ore/minute)
SELECT SYSDATE - (SYSDATE - 1) FROM DUAL;  -- rezultat: 1
```

> ⚠️ Rezultatul scăderii a două date este un număr **real** (include fracțiuni de zi). Dacă prima dată este mai mică, rezultatul este **negativ**.

---

### 6.2. `TRUNC` și `ROUND` pe date cu format

```sql
-- Formate disponibile: YYYY / MM / DD / HH / MI
SELECT ROUND(SYSDATE, 'MI') FROM DUAL;  -- rotunjit la minut
SELECT TRUNC(SYSDATE, 'DD') FROM DUAL;  -- azi la 00:00:00
```

---

### 6.3. Exemple practice

```sql
-- Data de peste 30 de zile
SELECT TO_CHAR(SYSDATE + 30, 'DD-MON-YYYY HH24:MI:SS') FROM DUAL;

-- Câte zile au trecut de la 1 ianuarie al anului curent
SELECT SYSDATE - TO_DATE('01-01-' || TO_CHAR(SYSDATE, 'YYYY'), 'DD-MM-YYYY')
FROM DUAL;

-- Prima luni după 6 luni de la angajare (dată negociere salariu)
SELECT FIRST_NAME || ' ' || LAST_NAME, HIRE_DATE,
       NEXT_DAY(ADD_MONTHS(HIRE_DATE, 6), 'MONDAY') "NEGOCIERE"
FROM EMPLOYEES;

-- Numărul de luni de la angajare
-- Alias-ul poate fi folosit DOAR în ORDER BY, nu în WHERE sau HAVING
SELECT FIRST_NAME,
       MONTHS_BETWEEN(SYSDATE, HIRE_DATE) AS "NUMAR LUNI"
FROM EMPLOYEES
ORDER BY "NUMAR LUNI";

-- Ziua săptămânii la angajare (ordonare după numărul zilei cu 'D')
SELECT FIRST_NAME, HIRE_DATE,
       TO_CHAR(HIRE_DATE, 'DAY') AS ZI
FROM EMPLOYEES
ORDER BY TO_CHAR(HIRE_DATE, 'D');

-- Angajați angajați în 1987 — variante corecte
SELECT FIRST_NAME, HIRE_DATE FROM EMPLOYEES
WHERE EXTRACT(YEAR FROM HIRE_DATE) = 1987;

SELECT FIRST_NAME, HIRE_DATE FROM EMPLOYEES
WHERE TO_CHAR(HIRE_DATE, 'YYYY') = 1987;

-- ❌ Nu afișează nimic — conversia implicită folosește NLS_DATE_FORMAT ('DD-MON-RR')
SELECT FIRST_NAME, HIRE_DATE FROM EMPLOYEES
WHERE TO_CHAR(HIRE_DATE) LIKE '%1987%';

-- ✅ Variantă corectă cu LIKE pe DATE
SELECT FIRST_NAME, HIRE_DATE FROM EMPLOYEES
WHERE HIRE_DATE LIKE '%87%';

-- Angajații angajați exact în aceeași zi a săptămânii ca azi
SELECT * FROM EMPLOYEES
WHERE MOD(TRUNC(SYSDATE) - TRUNC(HIRE_DATE), 7) = 0;

-- Alternativă
SELECT * FROM EMPLOYEES
WHERE TO_CHAR(SYSDATE, 'D') = TO_CHAR(HIRE_DATE, 'D');
```

---

## 7. Funcții Diverse

### 7.1. `NVL`, `NVL2`, `NULLIF`, `COALESCE`

| Funcție | Descriere | Exemplu |
| :--- | :--- | :--- |
| `NVL(expr1, expr2)` | Dacă `expr1` e NULL → returnează `expr2` | `NVL(NULL, 1)` → `1` |
| `NVL2(expr1, expr2, expr3)` | Dacă `expr1` NOT NULL → `expr2`, altfel → `expr3` | `NVL2(1, 2, 3)` → `2` |
| `NULLIF(expr1, expr2)` | Dacă `expr1 = expr2` → NULL, altfel → `expr1` | `NULLIF(1, 1)` → `NULL` |
| `COALESCE(e1, e2, ..., en)` | Prima valoare NOT NULL din listă | `COALESCE(NULL, NULL, 1, 2)` → `1` |

```sql
SELECT NVL('A', 1) FROM DUAL;   -- ✅ 1 se convertește implicit la șir → 'A'
SELECT NVL(1, 'A') FROM DUAL;   -- ❌ Eroare: 'A' nu se convertește la NUMBER
SELECT NVL(1, '2') FROM DUAL;   -- ✅ '2' se convertește implicit la 2 → 1

-- Afișare comision formatat sau text dacă e NULL
SELECT LAST_NAME, COMMISSION_PCT,
       NVL(TO_CHAR(COMMISSION_PCT, '0.999'), 'FARA COMISION')
FROM EMPLOYEES;

-- Angajați cu salariul total > 10000 (inclusiv comision)
SELECT FIRST_NAME, SALARY, COMMISSION_PCT
FROM EMPLOYEES
WHERE SALARY + NVL(COMMISSION_PCT, 0) * SALARY > 10000;
```

---

### 7.2. `DECODE`

Funcționează ca un `SWITCH` / `IF-ELSE` în SQL:

```sql
DECODE(value, if1, then1, if2, then2, ..., ifN, thenN, [else])
```

```sql
-- Salariu negociat în funcție de job
SELECT FIRST_NAME, JOB_ID,
       DECODE(UPPER(JOB_ID),
              'IT_PROG', SALARY * 1.2,
              'SA_REP',  SALARY * 1.25,
              'SA_MAN',  SALARY * 1.35,
              SALARY)    "SALARIU NEGOCIAT"
FROM EMPLOYEES;
```

> ⚠️ Dacă nicio condiție nu este adevărată și nu există `ELSE`, rezultatul este `NULL`.

---

### 7.3. `CASE`

Echivalent cu `DECODE`, dar mai flexibil (suportă orice operator de comparație):

```sql
SELECT FIRST_NAME, JOB_ID,
       CASE
           WHEN UPPER(JOB_ID) = 'IT_PROG' THEN SALARY * 1.2
           WHEN UPPER(JOB_ID) = 'SA_REP'  THEN SALARY * 1.25
           WHEN UPPER(JOB_ID) = 'SA_MAN'  THEN SALARY * 1.35
           ELSE SALARY
       END "SALARIU NEGOCIAT"
FROM EMPLOYEES;
```

---

### 7.4. Reguli importante pentru `DECODE` și `CASE`

```sql
-- Dacă mai multe WHEN sunt adevărate, se folosește PRIMUL
SELECT CASE WHEN 1=1 THEN 1
            WHEN 1=1 THEN 2
       END CASE FROM DUAL;
-- rezultat: 1

-- ❌ Eroare: tipurile de date returnate trebuie să fie COMPATIBILE
SELECT CASE WHEN 1=1 THEN 1
            WHEN 1=1 THEN 'A'
       END CASE FROM DUAL;
-- Oracle încearcă să convertească 'A' la NUMBER → EROARE

-- Nicio condiție adevărată și fără ELSE → NULL
SELECT CASE WHEN 1=2 THEN 1
            WHEN 1=3 THEN 2
       END CASE FROM DUAL;
-- rezultat: NULL

SELECT DECODE(1, 2, 1, 3, 1) FROM DUAL;
-- rezultat: NULL
```

---

### 7.5. `CASE` cu `NVL` — combinare utilă

```sql
SELECT LAST_NAME, COMMISSION_PCT,
       CASE
           WHEN COMMISSION_PCT IS NOT NULL THEN TO_CHAR(COMMISSION_PCT, '0.999')
           ELSE 'FARA COMISION'
       END
FROM EMPLOYEES;
```

---

## 8. Rezumat Rapid

```
Funcții caractere:
  LENGTH, SUBSTR, LTRIM, RTRIM, TRIM, LPAD, RPAD
  REPLACE, TRANSLATE, UPPER, LOWER, INITCAP
  INSTR, ASCII, CHR, CONCAT

Funcții numerice:
  ABS, CEIL, FLOOR, ROUND, TRUNC, MOD, POWER, SQRT
  EXP, LN, LOG, SIGN, SIN, COS, TAN, LEAST, GREATEST

Funcții dată:
  SYSDATE, ADD_MONTHS, NEXT_DAY, LAST_DAY
  MONTHS_BETWEEN, TRUNC, ROUND, EXTRACT, LEAST, GREATEST

Funcții conversie:
  TO_CHAR, TO_DATE, TO_NUMBER

Funcții diverse:
  DECODE, CASE, NVL, NVL2, NULLIF, COALESCE
  UID, USER, VSIZE
```

---

## 9. Capcane Frecvente ⚠️

| Situație | Problemă | Soluție |
| :--- | :--- | :--- |
| `LENGTH('')` | Returnează NULL, nu 0 | `''` = NULL în Oracle |
| `INSTR(...)` negăsit | Returnează `0`, nu NULL | Verifică `= 0`, nu `IS NULL` |
| Conversie implicită dată | Depinde de `NLS_DATE_FORMAT` | Folosește `TO_DATE` explicit |
| `TO_CHAR(HIRE_DATE) LIKE '%1987%'` | Nu găsește nimic | Formatul implicit nu conține `YYYY` complet; folosește `TO_CHAR(HIRE_DATE, 'YYYY')` |
| `NVL(1, 'A')` | Eroare de conversie | Tipurile trebuie să fie compatibile |
| `CASE` cu tipuri mixte | Eroare dacă `1` și `'A'` apar împreună | Standardizează tipul returnat |
| `TRANSLATE('QW', 'QW', 'A')` | `W` este șters, nu înlocuit | Fiecare caracter fără corespondent în dest. e eliminat |
| Alias în `WHERE` | `WHERE "NUMAR LUNI" > 10` dă eroare | Alias-ul din SELECT poate fi folosit **doar** în `ORDER BY` |
