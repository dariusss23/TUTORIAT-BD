# 📊 Funcții Grup și Clauzele GROUP BY, HAVING

---

## 1. Ce sunt Funcțiile Grup (Agregat)?

**Funcțiile grup** (agregat) operează asupra unui set de linii și returnează **un singur rezultat** pentru fiecare grup. Spre deosebire de funcțiile single-row (care returnează un rezultat per linie), funcțiile agregat colapsează mai multe linii într-o singură valoare.

### Funcțiile grup principale:

| Funcție | Descriere | Tipuri de date acceptate |
| :--- | :--- | :--- |
| `COUNT(*)` | Numărul total de linii (**inclusiv NULL**) | Orice |
| `COUNT(expr)` | Numărul de linii cu valori **non-NULL** | Orice |
| `COUNT(DISTINCT col)` | Numărul de valori **distincte** non-NULL | Orice |
| `SUM(expr)` | Suma valorilor | `NUMBER` |
| `AVG(expr)` | Media aritmetică a valorilor | `NUMBER` |
| `MAX(expr)` | Valoarea maximă | `NUMBER`, `CHAR`, `VARCHAR2`, `DATE` |
| `MIN(expr)` | Valoarea minimă | `NUMBER`, `CHAR`, `VARCHAR2`, `DATE` |

---

### Reguli generale:
- Funcțiile agregat pot apărea în clauzele: **`SELECT`**, **`ORDER BY`** și **`HAVING`**.
- `AVG`, `SUM`, `MIN`, `MAX`, `COUNT(coloana)` — **ignoră valorile `NULL`**.
- `COUNT(*)` — **nu ignoră** valorile `NULL`, numără toate liniile.
- `COUNT` returnează întotdeauna un număr **≥ 0** și **niciodată NULL**.
- `AVG` și `SUM` operează **doar** asupra valorilor numerice.
- `MAX` și `MIN` pot opera și asupra șirurilor de caractere sau datelor calendaristice.

---

## 2. Comportamentul funcțiilor față de NULL — Exemplu concret

```sql
-- Tabelul EMPLOYEES are 107 angajați, dintre care unul are DEPARTMENT_ID = NULL

SELECT COUNT(DEPARTMENT_ID) FROM EMPLOYEES;
-- rezultat: 106 (ignoră NULL-ul de pe coloana DEPARTMENT_ID)

SELECT COUNT(EMPLOYEE_ID) FROM EMPLOYEES;
-- rezultat: 107 (toate valorile de pe EMPLOYEE_ID sunt non-NULL)

SELECT COUNT(*) FROM EMPLOYEES;
-- rezultat: 107 (numără toate liniile, indiferent de NULL)
```

> 💡 **Rezumat comportament NULL:**
> ```
> COUNT(*)          → numără TOATE liniile (inclusiv NULL)
> COUNT(coloana)    → numără doar valorile NON-NULL
> COUNT(DISTINCT x) → numără valorile distincte NON-NULL
> AVG, SUM, MIN, MAX, COUNT(col) → IGNORĂ valorile NULL
> ```

---

### `COUNT(DISTINCT col)` — valori distincte

```sql
-- Câte departamente distincte există în tabelul EMPLOYEES (excluzând NULL)
SELECT COUNT(DISTINCT DEPARTMENT_ID)
FROM EMPLOYEES;
```

---

## 3. Clauza GROUP BY

**`GROUP BY`** divide liniile unui tabel în **grupuri** pe baza valorilor uneia sau mai multor coloane. Funcțiile agregat se aplică apoi fiecărui grup în parte, returnând **un singur rezultat per grup**.

### Sintaxă generală:

```sql
SELECT coloana_grupare, functie_agregat(coloana)
FROM tabel
[WHERE conditie]
GROUP BY coloana_grupare
[ORDER BY ...];
```

---

### 3.1. Fără GROUP BY — funcția se aplică întregului tabel

```sql
-- Suma salariilor tuturor angajaților (fără GROUP BY → se aplică pe tot tabelul)
SELECT SUM(SALARY)
FROM EMPLOYEES E;
```

> 💡 Absența lui `GROUP BY` face ca întregul tabel să fie tratat ca **un singur grup**.

---

### 3.2. Cu GROUP BY — un rezultat per grup

```sql
-- Suma salariilor pe fiecare departament
SELECT E.DEPARTMENT_ID, SUM(SALARY)
FROM EMPLOYEES E
GROUP BY E.DEPARTMENT_ID;
```

---

### 3.3. GROUP BY cu subcerere nesincronizată în SELECT

```sql
-- Se pot adăuga și subcereri nesincronizate în SELECT alături de GROUP BY
SELECT
    E.DEPARTMENT_ID,
    SUM(SALARY),
    (SELECT 1 FROM DUAL)   -- subcerere nesincronizată (constantă)
FROM EMPLOYEES E
GROUP BY E.DEPARTMENT_ID;
```

> 💡 Subcererile **nesincronizate** (necorelate) sunt permise în `SELECT` alături de `GROUP BY`, deoarece returnează o valoare **constantă** — aceeași pentru fiecare grup.

---

### 3.4. GROUP BY pe mai multe coloane

```sql
-- Numărul de angajați per departament și per job
SELECT DEPARTMENT_ID, JOB_ID, COUNT(*)
FROM EMPLOYEES
GROUP BY DEPARTMENT_ID, JOB_ID;
```

---

### 3.5. Statistici complete pe grupuri

```sql
SELECT DEPARTMENT_ID,
       COUNT(*)        NR_ANGAJATI,
       AVG(SALARY)     MEDIE,
       MAX(SALARY)     MAXIM,
       MIN(SALARY)     MINIM,
       SUM(SALARY)     TOTAL
FROM EMPLOYEES
GROUP BY DEPARTMENT_ID
ORDER BY DEPARTMENT_ID;
```

---

## 4. Reguli importante pentru GROUP BY

---

### 4.1. ⚠️ Regula principală: ce apare în SELECT trebuie să apară în GROUP BY

> **Toate coloanele care apar independent în clauza `SELECT` trebuie să apară și în clauza `GROUP BY`.**  
> Excepție: funcțiile agregat și subcererile nesincronizate.

```sql
-- ✅ Corect: DEPARTMENT_ID apare și în GROUP BY
SELECT E.DEPARTMENT_ID, SUM(SALARY)
FROM EMPLOYEES E
GROUP BY E.DEPARTMENT_ID;

-- ✅ Corect: subcerere nesincronizată nu trebuie pusă în GROUP BY
SELECT E.DEPARTMENT_ID, SUM(SALARY), (SELECT 1 FROM DUAL)
FROM EMPLOYEES E
GROUP BY E.DEPARTMENT_ID;

-- ❌ Eroare ORA-00979: FIRST_NAME nu e funcție agregat și nu e în GROUP BY
SELECT DEPARTMENT_ID, FIRST_NAME, AVG(SALARY)
FROM EMPLOYEES
GROUP BY DEPARTMENT_ID;
```

Expresiile valide în `SELECT` cu `GROUP BY`:

| Tip expresie | Trebuie în GROUP BY? |
| :--- | :---: |
| Coloană simplă | ✅ Da |
| Funcție agregat (`AVG`, `SUM` etc.) | ❌ Nu |
| Subcerere nesincronizată | ❌ Nu |
| Expresie bazată pe coloana de grupare | ✅ Da |

---

### 4.2. Sortare implicită

Când este utilizată clauza `GROUP BY`, Oracle sortează implicit rezultatul în **ordine crescătoare** după coloanele de grupare.

---

### 4.3. NULL formează un grup separat

Valorile `NULL` dintr-o coloană de grupare formează un **grup propriu**.

```sql
-- Angajații fără departament (NULL) formează propriul grup
SELECT DEPARTMENT_ID, COUNT(*)
FROM EMPLOYEES
GROUP BY DEPARTMENT_ID;
-- Un grup va afișa: NULL | 1
```

---

## 5. Clauza HAVING

**`HAVING`** permite restricționarea grupurilor returnate la cele care îndeplinesc o condiție. Este echivalentul lui `WHERE`, dar aplicat **grupurilor** — după ce acestea au fost formate.

### Sintaxă generală:

```sql
SELECT coloana_grupare, functie_agregat(coloana)
FROM tabel
[WHERE conditie_filtrare_linii]
GROUP BY coloana_grupare
HAVING conditie_filtrare_grupuri
[ORDER BY ...];
```

---

### 5.1. Exemplu de bază cu HAVING

```sql
-- Departamentele cu salariul mediu mai mare de 8000
SELECT DEPARTMENT_ID, AVG(SALARY)
FROM EMPLOYEES
GROUP BY DEPARTMENT_ID
HAVING AVG(SALARY) > 8000;
```

---

### 5.2. HAVING cu mai multe condiții

```sql
-- Departamentele cu mai mult de 5 angajați și salariul mediu > 6000
SELECT DEPARTMENT_ID, COUNT(*), AVG(SALARY)
FROM EMPLOYEES
GROUP BY DEPARTMENT_ID
HAVING COUNT(*) > 5
   AND AVG(SALARY) > 6000;
```

---

### 5.3. HAVING fără GROUP BY

Dacă `HAVING` este folosit **fără** `GROUP BY`, întregul tabel este tratat ca un singur grup, iar rezultatul conține o singură linie — **reținută doar dacă** condiția din `HAVING` este îndeplinită.

```sql
-- Returnează o linie dacă salariul mediu general este mai mare de 6000
SELECT AVG(SALARY)
FROM EMPLOYEES
HAVING AVG(SALARY) > 6000;
```

---

## 6. WHERE vs. HAVING

| | `WHERE` | `HAVING` |
| :--- | :--- | :--- |
| **Filtrează** | Linii individuale | Grupuri de linii |
| **Se execută** | Înainte de grupare | După grupare |
| **Poate folosi funcții agregat** | ❌ Nu | ✅ Da |
| **Folosit împreună cu** | `FROM` | `GROUP BY` |

```sql
-- WHERE filtrează liniile ÎNAINTE de grupare
-- HAVING filtrează grupurile DUPĂ grupare
SELECT DEPARTMENT_ID, AVG(SALARY)
FROM EMPLOYEES
WHERE JOB_ID <> 'SA_REP'        -- filtrare linii (înainte de grupare)
GROUP BY DEPARTMENT_ID
HAVING AVG(SALARY) > 5000       -- filtrare grupuri (după grupare)
ORDER BY AVG(SALARY) DESC;
```

---

## 7. Ordinea de execuție a clauzelor

```
1. FROM        → identifică tabelele
2. WHERE       → filtrează liniile individuale
3. GROUP BY    → grupează liniile rămase
4. HAVING      → filtrează grupurile
5. SELECT      → calculează expresiile și funcțiile agregat
6. ORDER BY    → sortează rezultatul final
```

> 💡 Aceasta explică de ce:
> - **Nu poți folosi funcții agregat în `WHERE`** — gruparea nu a avut loc încă.
> - **Alias-urile din `SELECT` nu pot fi folosite în `HAVING`** — `SELECT` se execută după `HAVING`.
> - **Alias-urile din `SELECT` pot fi folosite în `ORDER BY`** — acesta se execută ultimul.

---

## 8. Rezumat Rapid

```
Funcții agregat:
  COUNT(*)           → numără TOATE liniile (inclusiv NULL)
  COUNT(coloana)     → numără valorile NON-NULL
  COUNT(DISTINCT x)  → numără valorile DISTINCTE non-NULL
  SUM, AVG           → doar valori numerice; ignoră NULL
  MAX, MIN           → numerice, caractere, date; ignoră NULL

GROUP BY:
  → divide tabelul în grupuri
  → toate coloanele din SELECT (non-agregat, non-subcerere) → obligatoriu în GROUP BY
  → subcererile nesincronizate din SELECT → NU trebuie în GROUP BY
  → sortare implicită crescătoare după coloanele de grupare
  → fără GROUP BY → întregul tabel = un singur grup
  → NULL formează un grup separat

HAVING:
  → filtrează grupuri (după GROUP BY)
  → poate folosi funcții agregat (WHERE nu poate)
  → fără GROUP BY → se aplică pe întregul tabel ca un grup

WHERE  → filtrează linii (înainte de grupare)
HAVING → filtrează grupuri (după grupare)

Ordine execuție: FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY
```

---

## 9. Capcane Frecvente ⚠️

| Situație | Problemă | Soluție |
| :--- | :--- | :--- |
| Coloană în `SELECT` care nu e în `GROUP BY` | Eroare ORA-00979 | Adaugă coloana în `GROUP BY` sau aplică funcție agregat |
| `COUNT(coloana)` pe coloană cu NULL | Returnează mai puțin decât `COUNT(*)` | Înțelege că `COUNT(col)` ignoră NULL — folosește `COUNT(*)` dacă vrei toate liniile |
| `AVG` pe coloane cu `NULL` | Media calculată fără NULL (poate induce în eroare) | Folosește `AVG(NVL(col, 0))` dacă vrei să incluzi NULL ca 0 |
| Funcție agregat în `WHERE` | Eroare Oracle | Mută condiția în `HAVING` |
| Alias din `SELECT` folosit în `HAVING` | Eroare Oracle | `HAVING` se execută înainte de `SELECT` — rescrie condiția complet |
| Alias din `SELECT` folosit în `WHERE` | Eroare Oracle | Idem — `WHERE` se execută înainte de `SELECT` |
