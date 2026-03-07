# 🔍 Subcereri (Subinterogări) în SQL Oracle

---

## 1. Ce este o Subcerere?

O **subcerere** (subinterogare) este o comandă `SELECT` integrată într-o clauză a altei instrucțiuni SQL, numită **instrucțiune părinte** sau **instrucțiune exterioară**.

Subcererile mai sunt numite și **instrucțiuni SELECT imbricate** sau **interioare**. Rezultatele subcererii sunt utilizate în cadrul cererii exterioare pentru a determina rezultatul final.

```sql
SELECT expresie1, expresie2
FROM nume_tabel1
WHERE expresie_conditie operator (
    SELECT expresie
    FROM nume_tabel2
);
```

---

## 2. Clasificare: Nesincronizate vs. Sincronizate

| Tip | Mod de evaluare | Direcție |
| :--- | :--- | :--- |
| **Nesincronizate (necorelate)** | Cererea internă se execută **prima**, independent | Interior → Exterior |
| **Sincronizate (corelate)** | Cererea externă furnizează valori cererii interne | Exterior → Interior → Exterior |

---

## 3. Subcereri Nesincronizate (Necorelate)

Cererea internă este evaluată **o singură dată**, independent de cererea externă. Rezultatul ei este apoi folosit de cererea externă.

### Pași de execuție:
1. Cererea **internă** se execută prima și returnează o valoare (sau o mulțime de valori).
2. Cererea **externă** se execută o singură dată, folosind valorile returnate.

---

### 3.1. Exemplu cu `IN` în clauza `WHERE`

```sql
-- Angajații care lucrează în departamente al căror nume conține litera 'A'
SELECT *
FROM EMPLOYEES E
WHERE E.DEPARTMENT_ID IN (
    SELECT D.DEPARTMENT_ID
    FROM DEPARTMENTS D
    WHERE UPPER(D.DEPARTMENT_NAME) LIKE '%A%'
);
```

> 💡 Subcererea returnează o **listă de ID-uri** de departamente, iar cererea externă filtrează angajații folosind acea listă.

---

### 3.2. Exemplu cu funcție agregat

```sql
-- Angajații care câștigă mai mult decât media salariilor
SELECT FIRST_NAME, SALARY
FROM EMPLOYEES
WHERE SALARY > (
    SELECT AVG(SALARY)
    FROM EMPLOYEES
);
```

---

### 3.3. Exemplu cu `ANY` în clauza `WHERE`

```sql
-- Angajații cu salariul mai mare decât cel puțin un angajat din departamentul 20
SELECT *
FROM EMPLOYEES E
WHERE E.SALARY > ANY (
    SELECT E2.SALARY
    FROM EMPLOYEES E2
    WHERE E2.DEPARTMENT_ID = 20
);
-- Echivalent cu: SALARY > MIN(salariile din departamentul 20)
```

---

## 4. Subcereri Sincronizate (Corelate)

Cererea externă furnizează o linie candidat cererii interne, care se execută **pentru fiecare linie** din cererea externă.

### Pași de execuție:
1. Cererea externă determină o **linie candidat**.
2. Cererea internă se execută folosind valoarea liniei candidat.
3. Rezultatul cererii interne este folosit pentru a **califica sau descalifica** linia candidat.
4. Pașii se repetă pentru **fiecare linie** din tabelul extern.

---

### 4.1. Exemplu subcerere corelată în `SELECT`

```sql
-- Pentru fiecare angajat, afișează și numele departamentului său
SELECT
    E.EMPLOYEE_ID,
    (
        SELECT D.DEPARTMENT_NAME
        FROM DEPARTMENTS D
        WHERE D.DEPARTMENT_ID = E.DEPARTMENT_ID
    )
FROM EMPLOYEES E;
```

> 💡 Subcererea din `SELECT` primește `E.DEPARTMENT_ID` din cererea externă pentru **fiecare linie** în parte. Este o subcerere **corelată** și **scalară** (returnează exact 1 valoare).

---

### 4.2. Exemplu subcerere corelată în `WHERE`

```sql
-- Angajații care câștigă mai mult decât media salariilor din departamentul lor
SELECT E.FIRST_NAME, E.SALARY, E.DEPARTMENT_ID
FROM EMPLOYEES E
WHERE E.SALARY > (
    SELECT AVG(E2.SALARY)
    FROM EMPLOYEES E2
    WHERE E2.DEPARTMENT_ID = E.DEPARTMENT_ID
);
```

---

## 5. Unde pot apărea subcererile?

Subcererile pot fi folosite în mai multe clauze ale unei instrucțiuni SQL:

---

### 5.1. Subcerere în `WHERE`

Cel mai frecvent caz — filtrarea rezultatelor pe baza valorilor returnate de subcerere.

```sql
SELECT *
FROM EMPLOYEES E
WHERE E.DEPARTMENT_ID IN (
    SELECT D.DEPARTMENT_ID
    FROM DEPARTMENTS D
    WHERE UPPER(D.DEPARTMENT_NAME) LIKE '%A%'
);
```

---

### 5.2. Subcerere în `SELECT`

> ⚠️ **Regulă importantă:** O subcerere în `SELECT` poate returna **o singură înregistrare și o singură coloană** (valoare scalară). Altfel se produce eroare.

```sql
SELECT
    E.EMPLOYEE_ID,
    (
        SELECT D.DEPARTMENT_NAME
        FROM DEPARTMENTS D
        WHERE D.DEPARTMENT_ID = E.DEPARTMENT_ID
    )
FROM EMPLOYEES E;
```

---

### 5.3. Subcerere în `FROM` (tabel derivat)

Subcererea înlocuiește un tabel, creând un **tabel derivat** temporar.

> ⚠️ **Reguli obligatorii pentru subcererea din `FROM`:**
> - Subcererea trebuie să primească un **alias** (ex: `T`).
> - Coloanele obținute prin operații, funcții sau alte subcereri trebuie să primească și ele **alias**.

```sql
-- Salariul anual al angajaților din departamentul 20
SELECT T.EMPLOYEE_ID, T.SALARIU
FROM (
    SELECT E.EMPLOYEE_ID, E.SALARY * 12 SALARIU
    FROM EMPLOYEES E
    WHERE E.DEPARTMENT_ID = 20
) T;
```

> 💡 `T` este alias-ul subcererii. `SALARIU` este alias-ul coloanei calculate `SALARY * 12` — **obligatoriu** deoarece este o expresie.

---

### 5.4. Subcerere în `HAVING`

```sql
-- Departamentele cu salariul mediu mai mare decât media generală
SELECT DEPARTMENT_ID, AVG(SALARY)
FROM EMPLOYEES
GROUP BY DEPARTMENT_ID
HAVING AVG(SALARY) > (
    SELECT AVG(SALARY)
    FROM EMPLOYEES
);
```

---

## 6. Operatori pentru Subcereri

### 6.1. Operatori Single-Row

Folosiți când subcererea returnează **o singură linie**:

| Operator | Semnificație |
| :---: | :--- |
| `=` | Egal cu valoarea returnată |
| `<>` sau `!=` | Diferit de valoarea returnată |
| `>` | Mai mare decât valoarea returnată |
| `>=` | Mai mare sau egal |
| `<` | Mai mic decât valoarea returnată |
| `<=` | Mai mic sau egal |

> ⚠️ Dacă o subcerere cu operator single-row returnează **mai mult de o linie**, se produce eroare ORA-01427.

---

### 6.2. Operatori Multi-Row

Folosiți când subcererea returnează **mai multe linii**:

| Operator | Semnificație |
| :---: | :--- |
| `IN` | Egal cu oricare valoare din listă |
| `NOT IN` | Diferit de toate valorile din listă |
| `ANY` / `SOME` | Condiție adevărată față de **cel puțin una** din valori |
| `ALL` | Condiție adevărată față de **toate** valorile |

> 💡 `NOT` poate fi combinat cu `IN`, `ANY` și `ALL`.  
> 💡 `SOME` este sinonim cu `ANY` (standard ISO).

---

## 7. Detalii despre `ANY` și `ALL`

### `ANY` — „cel puțin una din valori"

Condiția este adevărată dacă este satisfăcută de **oricare** valoare din subcerere.

```
3 > ANY (4, 2, 5)  →  TRUE   (3 > 2 ✅)
3 > ANY (4, 5, 5)  →  FALSE  (3 nu este > decât nicio valoare din listă)
```

```sql
-- Angajați cu salariu mai mare decât cel puțin un angajat din departamentul 20
WHERE SALARY > ANY (SELECT SALARY FROM EMPLOYEES WHERE DEPARTMENT_ID = 20)
-- Echivalent cu: SALARY > MIN(salariile din departamentul 20)
```

---

### `ALL` — „toate valorile"

Condiția este adevărată doar dacă este satisfăcută față de **toate** valorile din subcerere.

```
3 > ALL (4, 2, 5)  →  FALSE  (3 nu este > 4 și > 5)
3 > ALL (1, 2, 0)  →  TRUE   (3 > 1 ✅, 3 > 2 ✅, 3 > 0 ✅)
```

```sql
-- Angajați cu salariu mai mare decât toți angajații din departamentul 20
WHERE SALARY > ALL (SELECT SALARY FROM EMPLOYEES WHERE DEPARTMENT_ID = 20)
-- Echivalent cu: SALARY > MAX(salariile din departamentul 20)
```

---

### Echivalențe importante:

```
> ANY   ≡  > MIN(...)      cel puțin una
> ALL   ≡  > MAX(...)      toate
= ANY   ≡  IN
≠ ALL   ≡  NOT IN
```

---

### Comportament cu mulțimea vidă:

| Operator | Subcerere returnează `∅` | Rezultat |
| :--- | :--- | :--- |
| `ALL` | Nicio valoare de comparat | `TRUE` (condiție satisfăcută trivial) |
| `ANY` | Nicio valoare de comparat | `FALSE` (nicio valoare nu satisface) |

> ⚠️ `NOT IN` cu o subcerere ce conține `NULL` nu va returna **nicio linie** — un comportament foarte frecvent greșit înțeles.

---

## 8. Reguli și Restricții

- O subcerere trebuie inclusă între **paranteze**.
- Subcererea se plasează de obicei în **partea dreaptă** a operatorului de comparație.
- Subcererile nu pot conține clauza **`ORDER BY`** (cu excepția subcererilor din `FROM`).
- Operatorii **single-row** se folosesc doar cu subcereri ce returnează **o singură linie**.
- Operatorii **multi-row** (`IN`, `ANY`, `ALL`) se folosesc cu subcereri ce returnează **mai multe linii**.
- `ANY` și `ALL` funcționează doar cu subcereri ce returnează **o singură coloană**.
- Subcererile din **`SELECT`** pot returna **doar o singură valoare** (scalare).
- Subcererile din **`FROM`** trebuie să primească obligatoriu un **alias**, iar coloanele calculate din ele trebuie și ele să aibă **alias**.

---

## 9. Rezumat Rapid

```
Nesincronizate (necorelate):
  → cererea internă se execută PRIMA, o singură dată
  → evaluare: interior → exterior

Sincronizate (corelate):
  → cererea internă se execută pentru FIECARE linie candidat
  → evaluare: exterior → interior → exterior

Unde pot apărea:
  WHERE   → filtrare (cel mai frecvent)
  SELECT  → valoare scalară (1 linie, 1 coloană OBLIGATORIU)
  FROM    → tabel derivat (alias OBLIGATORIU pe subcerere și coloane calculate)
  HAVING  → filtrare grupuri

Operatori single-row:  =  <>  >  >=  <  <=     (subcerere → 1 linie)
Operatori multi-row:   IN  ANY  ALL  NOT IN     (subcerere → n linii)

ANY  →  cel puțin una   (> ANY ≡ > MIN)
ALL  →  toate valorile  (> ALL ≡ > MAX)
=ANY ≡  IN
ANY cu mulțime vidă → FALSE
ALL cu mulțime vidă → TRUE
```

---

## 10. Capcane Frecvente ⚠️

| Situație | Problemă | Soluție |
| :--- | :--- | :--- |
| Operator single-row, subcerere returnează mai multe linii | Eroare ORA-01427 | Folosește `IN`, `ANY` sau `ALL` |
| Subcerere în `SELECT` returnează mai mult de o linie | Eroare Oracle | Subcererea din `SELECT` trebuie să fie **scalară** (1 linie, 1 coloană) |
| Subcerere în `FROM` fără alias | Eroare Oracle | Alias-ul este **obligatoriu** pentru tabelul derivat |
| Coloană calculată în `FROM` fără alias | Nu poate fi referențiată din exterior | Adaugă alias coloanelor calculate din subcererea în `FROM` |
| `NOT IN` când subcererea conține `NULL` | Nu returnează nicio linie | Filtrează `NULL` cu `WHERE col IS NOT NULL` sau folosește `NOT EXISTS` |
| `ALL` cu subcerere vidă | Returnează `TRUE` — poate fi neașteptat | Verifică dacă subcererea poate returna mulțime vidă |
| `ANY` cu subcerere vidă | Returnează `FALSE` — niciun rezultat | Idem — tratează cazul subcererii vide |
| `ORDER BY` în subcerere (în `WHERE`) | Eroare Oracle | `ORDER BY` nu este permis decât în subcereri din `FROM` |
