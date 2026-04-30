# 🌳 Cereri Ierarhice, WITH și EXISTS

## 1. Cereri Ierarhice

### 💡 Problema din viața reală

Orice companie are o organigramă. Cineva e CEO, sub el sunt directori, sub directori sunt manageri, sub manageri sunt angajați obișnuiți.

Toate aceste relații sunt stocate într-un **singur tabel** — `EMPLOYEES`. Fiecare angajat știe cine e managerul lui prin coloana `MANAGER_ID`:

```
EMPLOYEES:
ID  | Nume      | Manager_ID
────┼───────────┼───────────
100 | Steven    | NULL    ← nu are manager, e CEO
101 | Neena     | 100     ← managerul ei e Steven
102 | Lex       | 100     ← managerul lui e Steven
103 | Alexander | 102     ← managerul lui e Lex
104 | Bruce     | 103     ← managerul lui e Alexander
```

Vizual, arată așa:

```
            Steven (100)
           /             \
      Neena (101)      Lex (102)
                           \
                        Alexander (103)
                               \
                             Bruce (104)
```

**Problema:** Cum afișezi toată această structură cu un singur query SQL? Nu poți folosi un SELECT obișnuit — nu știi din start câte niveluri are ierarhia.

**Soluția:** `START WITH` + `CONNECT BY`

---

### 1.1. Cum funcționează — pas cu pas

Gândește-te la Oracle ca la cineva care **merge din ușă în ușă**:

```
1. START WITH → "Bate la ușa lui Steven (CEO)"
2. CONNECT BY → "De la Steven, găsește pe toți cei al căror MANAGER_ID = Steven"
               → Găsește: Neena și Lex
3. Repetă:    → "De la Neena, găsește pe toți ai ei..." → nimeni
               → "De la Lex, găsește pe toți ai lui..." → Alexander
4. Repetă:    → "De la Alexander..." → Bruce
5. Stop       → Bruce nu mai are subalterni
```

**Sintaxa:**

```sql
SELECT LEVEL, EMPLOYEE_ID, FIRST_NAME, MANAGER_ID
FROM EMPLOYEES
START WITH EMPLOYEE_ID = 100          -- pornești de la Steven
CONNECT BY PRIOR EMPLOYEE_ID = MANAGER_ID;
--         ↑
--    PRIOR = "cel găsit în pasul anterior"
--    Adică: "al cărui EMPLOYEE_ID (din pasul anterior)
--            este egal cu MANAGER_ID-ul celui curent"
```

**Rezultat:**

```
LEVEL | ID  | Nume      | Manager_ID
──────┼─────┼───────────┼───────────
  1   | 100 | Steven    | NULL       ← punctul de start
  2   | 101 | Neena     | 100        ← subalternii lui Steven
  2   | 102 | Lex       | 100
  3   | 103 | Alexander | 102        ← subalternii lui Lex
  4   | 104 | Bruce     | 103        ← subalternii lui Alexander
```

`LEVEL` îți spune automat pe ce treaptă ești:
- 1 = CEO (de unde ai pornit)
- 2 = subalternii direcți ai CEO-ului
- 3 = subalternii subalternilor... și tot așa

---

### 1.2. PRIOR — Direcția de mers

`PRIOR` e cel mai confuz cuvânt. Iată cel mai simplu mod de a-l înțelege:

> **`PRIOR X = Y` înseamnă: "X-ul celui deja găsit = Y-ul celui pe care îl caut"**

Poți merge în **două direcții**:

**⬇️ De sus în jos** (de la șef la subalterni) — cel mai comun:
```sql
CONNECT BY PRIOR EMPLOYEE_ID = MANAGER_ID
-- "EMPLOYEE_ID-ul celui deja găsit = MANAGER_ID-ul celui pe care îl caut"
-- Adică: caut pe toți cei care au ca manager pe cel deja găsit
```

**⬆️ De jos în sus** (de la subaltern la șef):
```sql
CONNECT BY PRIOR MANAGER_ID = EMPLOYEE_ID
-- "MANAGER_ID-ul celui deja găsit = EMPLOYEE_ID-ul celui pe care îl caut"
-- Adică: caut managerul celui deja găsit
```

---

### 1.3. Exemplu: Cine sunt șefii lui Bruce?

Pornesc de la Bruce și urc până la CEO:

```sql
SELECT LEVEL, FIRST_NAME, MANAGER_ID
FROM EMPLOYEES
START WITH EMPLOYEE_ID = 104          -- pornesc de la Bruce
CONNECT BY PRIOR MANAGER_ID = EMPLOYEE_ID;  -- urc spre șef
```

**Rezultat:**

```
LEVEL | Nume      | Manager_ID
──────┼───────────┼───────────
  1   | Bruce     | 103        ← punctul de start
  2   | Alexander | 102        ← șeful lui Bruce
  3   | Lex       | 100        ← șeful lui Alexander
  4   | Steven    | NULL       ← CEO (nu mai are șef)
```

---

### 1.4. Truc Vizual: Indentare cu LPAD

Poți face ierarhia să arate ca o organigramă reală folosind `LPAD` (adaugă spații în față):

```sql
SELECT
    LEVEL,
    LPAD(' ', (LEVEL - 1) * 4) || FIRST_NAME AS ORGANIGARMA
FROM EMPLOYEES
START WITH MANAGER_ID IS NULL          -- pornesc de la CEO (cel fără șef)
CONNECT BY PRIOR EMPLOYEE_ID = MANAGER_ID;
```

**Rezultat:**

```
LEVEL | ORGANIGARMA
──────┼─────────────────────
  1   | Steven
  2   |     Neena
  2   |     Lex
  3   |         Alexander
  4   |             Bruce
```

`LPAD(' ', (LEVEL-1) * 4)` = pune spații proporțional cu nivelul → efectul de indentare.

---

### 1.5. Filtrare în Cereri Ierarhice: WHERE vs CONNECT BY

Există o diferență importantă între cele două:

**`WHERE`** — filtrează rânduri din rezultat, **după** ce ierarhia e construită complet:

```sql
-- Construiește toată ierarhia, dar afișează doar primele 2 niveluri
SELECT LEVEL, FIRST_NAME
FROM EMPLOYEES
WHERE LEVEL <= 2                        -- filtru aplicat la final
START WITH MANAGER_ID IS NULL
CONNECT BY PRIOR EMPLOYEE_ID = MANAGER_ID;
```

```
Rezultat: Steven, Neena, Lex
(Alexander și Bruce sunt excluși din afișare,
 dar ierarhia lor s-a construit în memorie)
```

**`AND` în CONNECT BY** — taie o ramură întreagă, **în timp ce** se construiește ierarhia:

```sql
-- Exclude Lex și TOȚI subalternii lui (Alexander, Bruce)
SELECT LEVEL, FIRST_NAME
FROM EMPLOYEES
START WITH MANAGER_ID IS NULL
CONNECT BY PRIOR EMPLOYEE_ID = MANAGER_ID
       AND EMPLOYEE_ID <> 102;          -- taie ramura lui Lex complet
```

```
Rezultat: Steven, Neena
(Lex e exclus, deci Oracle nu mai continuă pe ramura lui →
 Alexander și Bruce dispar și ei)
```

> 💡 **Regulă simplă:**
> - Vrei să limitezi ce *afișezi*? → `WHERE`
> - Vrei să tai o *ramură întreagă*? → `AND` în `CONNECT BY`

---

## 2. Clauza WITH

### 💡 Problema din viața reală

Imaginează-ți că ești contabil și trebuie să calculezi salariul total pe departament, apoi să compari fiecare departament cu media tuturor departamentelor.

Fără WITH, ajungi să scrii același calcul de 2-3 ori în același query:

```sql
-- ❌ Fără WITH: calculezi totalul de 3 ori
SELECT DEPARTMENT_ID
FROM EMPLOYEES
GROUP BY DEPARTMENT_ID
HAVING SUM(SALARY) > (
    SELECT AVG(SUM(SALARY))   -- calculezi suma din nou
    FROM EMPLOYEES
    GROUP BY DEPARTMENT_ID    -- și din nou
);
```

E greu de citit și dacă vrei să schimbi ceva, trebuie să modifici în 3 locuri.

**WITH** îți permite să dai un **nume** unui calcul și să îl folosești de câte ori vrei:

```sql
-- ✅ Cu WITH: calculezi o dată, folosești de oriunde
WITH TOTAL_PE_DEPT AS (
    SELECT DEPARTMENT_ID, SUM(SALARY) AS TOTAL
    FROM EMPLOYEES
    GROUP BY DEPARTMENT_ID
)
SELECT DEPARTMENT_ID
FROM TOTAL_PE_DEPT
WHERE TOTAL > (SELECT AVG(TOTAL) FROM TOTAL_PE_DEPT);
--                                        ↑
--                              folosit a doua oară, fără recalcul
```

---

### 2.1. Cum să te gândești la WITH

WITH e ca și cum ai crea o **tabelă temporară** care există doar pe durata query-ului:

```
Fără WITH:                          Cu WITH:
─────────────────────────           ─────────────────────────
Calculezi în mers, repet.           PASUL 1: Calculezi o dată
                                    → dai un nume rezultatului
                                    PASUL 2: Folosești numele
                                    de câte ori vrei
```

Un alt mod de a gândi: e ca o **variabilă** din programare. Stochezi un rezultat, îi dai un nume, și îl reutilizezi.

---

### 2.2. Sintaxa

```sql
WITH nume_ales AS (
    -- orice query normal
    SELECT ...
    FROM ...
    WHERE ...
)
SELECT ...
FROM nume_ales          -- folosești numele ca pe un tabel
WHERE ...;
```

---

### 2.3. Exemplu Pas cu Pas

**Cerință:** Afișează departamentele care plătesc salarii mai mari decât media tuturor departamentelor.

```sql
-- PASUL 1: Definești blocul WITH
-- "Calculează totalul salariilor pe fiecare departament și numește-l TOTAL_PE_DEPT"
WITH TOTAL_PE_DEPT AS (
    SELECT DEPARTMENT_ID, SUM(SALARY) AS TOTAL
    FROM EMPLOYEES
    GROUP BY DEPARTMENT_ID
)

-- PASUL 2: Folosești blocul ca pe un tabel normal
SELECT D.DEPARTMENT_NAME, T.TOTAL
FROM DEPARTMENTS D
JOIN TOTAL_PE_DEPT T ON D.DEPARTMENT_ID = T.DEPARTMENT_ID
WHERE T.TOTAL > (SELECT AVG(TOTAL) FROM TOTAL_PE_DEPT)  -- folosit a doua oară!
ORDER BY T.TOTAL DESC;
```

`TOTAL_PE_DEPT` e calculat **o singură dată**, dar apare în query în **două locuri** — în JOIN și în WHERE.

---

### 2.4. Mai Multe Blocuri WITH

Poți defini mai multe blocuri, separate prin virgulă. Un bloc poate folosi blocurile definite înaintea lui:

```sql
WITH
    -- Bloc 1: total salarii pe departament
    TOTAL_PE_DEPT AS (
        SELECT DEPARTMENT_ID, SUM(SALARY) AS TOTAL
        FROM EMPLOYEES
        GROUP BY DEPARTMENT_ID
    ),
    -- Bloc 2: media totalurilor (folosește Bloc 1)
    MEDIA_TOTALE AS (
        SELECT AVG(TOTAL) AS MEDIE
        FROM TOTAL_PE_DEPT      -- referă Bloc 1!
    )

-- Query principal: departamentele peste medie
SELECT D.DEPARTMENT_NAME, T.TOTAL
FROM DEPARTMENTS D
JOIN TOTAL_PE_DEPT T ON D.DEPARTMENT_ID = T.DEPARTMENT_ID
JOIN MEDIA_TOTALE M  ON T.TOTAL > M.MEDIE
ORDER BY T.TOTAL DESC;
```

> ⚠️ Ordinea contează: `MEDIA_TOTALE` poate folosi `TOTAL_PE_DEPT` pentru că e definit **înaintea** lui. Invers nu merge.

---

### 2.5. Exemplu Practic: Angajatul cu Salariul Maxim din Fiecare Departament

```sql
-- Cine câștigă cel mai mult în fiecare departament?
WITH MAX_PE_DEPT AS (
    SELECT DEPARTMENT_ID, MAX(SALARY) AS SAL_MAX
    FROM EMPLOYEES
    GROUP BY DEPARTMENT_ID
)
SELECT E.FIRST_NAME, E.SALARY, E.DEPARTMENT_ID
FROM EMPLOYEES E
JOIN MAX_PE_DEPT M ON E.DEPARTMENT_ID = M.DEPARTMENT_ID
                  AND E.SALARY = M.SAL_MAX
ORDER BY E.DEPARTMENT_ID;
```

**Fără WITH**, ar fi trebuit să scrii subcererea cu `MAX` de două ori. Cu WITH, o scrii o dată și o folosești curat.

---

## 3. Operatorul EXISTS

### 💡 Problema din viața reală

Uneori nu vrei *date* dintr-un alt tabel — vrei doar să știi dacă *există* ceva acolo.

**Exemple de întrebări de tip "există?":**
- "Arată-mi departamentele care **au** cel puțin un angajat"
- "Arată-mi clienții care **au** cel puțin o comandă"
- "Arată-mi categoriile de produse care **au** cel puțin un produs în stoc"

Nu-ți pasă câți angajați are departamentul, sau ce comandă a făcut clientul. Vrei doar DA sau NU.

---

### 3.1. Cum funcționează EXISTS

Oracle parcurge rând cu rând tabelul principal (ex: `DEPARTMENTS`) și pentru fiecare rând se întreabă:

> "Dacă rulez subcererea pentru **acest** rând, se întoarce ceva?"
> - DA → include rândul în rezultat
> - NU → sare peste el

```sql
SELECT D.DEPARTMENT_NAME
FROM DEPARTMENTS D
WHERE EXISTS (
    SELECT 1                              -- ce returnezi nu contează
    FROM EMPLOYEES E
    WHERE E.DEPARTMENT_ID = D.DEPARTMENT_ID   -- legătura cu exteriorul
);
```

**De ce `SELECT 1`?** EXISTS nu se uită la *valorile* returnate — verifică doar dacă există *cel puțin un rând*. Prin convenție se scrie `SELECT 1` ca să fie clar că valorile nu contează.

---

### 3.2. Exemplu Vizual

```
DEPARTMENTS:           EMPLOYEES:
ID | Name              ID | Dept_ID
───┼────────           ───┼────────
10 | IT         →  →  →  | 10  ✅ există → afișez IT
20 | HR         →  →  →  | 20  ✅ există → afișez HR
30 | Sales      →  →  ✗  niciun angajat cu dept 30 → NU afișez Sales
```

```sql
-- Afișează doar departamentele care AU angajați
SELECT D.DEPARTMENT_NAME
FROM DEPARTMENTS D
WHERE EXISTS (
    SELECT 1 FROM EMPLOYEES E
    WHERE E.DEPARTMENT_ID = D.DEPARTMENT_ID
);

-- Rezultat:
-- IT
-- HR
-- (Sales nu apare — nu are angajați)
```

---

### 3.3. NOT EXISTS — Opusul

Dacă EXISTS înseamnă "există cel puțin unul", NOT EXISTS înseamnă "nu există niciunul":

```sql
-- Departamentele care NU au niciun angajat
SELECT D.DEPARTMENT_NAME
FROM DEPARTMENTS D
WHERE NOT EXISTS (
    SELECT 1 FROM EMPLOYEES E
    WHERE E.DEPARTMENT_ID = D.DEPARTMENT_ID
);

-- Rezultat:
-- Sales  ← singurul departament fără angajați
```

---

### 3.4. EXISTS vs IN — Când să folosești care?

La prima vedere par identice. De fapt, de cele mai multe ori chiar returnează același rezultat:

```sql
-- Varianta IN
SELECT DEPARTMENT_NAME FROM DEPARTMENTS
WHERE DEPARTMENT_ID IN (
    SELECT DEPARTMENT_ID FROM EMPLOYEES
);

-- Varianta EXISTS (echivalentă ca rezultat)
SELECT DEPARTMENT_NAME FROM DEPARTMENTS D
WHERE EXISTS (
    SELECT 1 FROM EMPLOYEES E
    WHERE E.DEPARTMENT_ID = D.DEPARTMENT_ID
);
```

**Diferența practică — capcana cu NULL:**

```sql
-- ❌ NOT IN cu NULL în subcerere → returnează 0 rânduri!
SELECT DEPARTMENT_NAME FROM DEPARTMENTS
WHERE DEPARTMENT_ID NOT IN (
    SELECT DEPARTMENT_ID FROM EMPLOYEES
    -- Dacă vreun angajat are DEPARTMENT_ID = NULL...
    -- NOT IN (10, 20, NULL) → condiție UNKNOWN → 0 rânduri!
);

-- ✅ NOT EXISTS funcționează corect chiar și cu NULL-uri
SELECT DEPARTMENT_NAME FROM DEPARTMENTS D
WHERE NOT EXISTS (
    SELECT 1 FROM EMPLOYEES E
    WHERE E.DEPARTMENT_ID = D.DEPARTMENT_ID
);
```

> 💡 **Regulă simplă:** Folosește `NOT EXISTS` în loc de `NOT IN` dacă subcererea poate conține valori NULL. `NOT EXISTS` nu are această problemă niciodată.

---

### 3.5. EXISTS cu Condiții Suplimentare

Poți adăuga orice condiție suplimentară în subcerere:

```sql
-- Departamentele care au cel puțin un angajat cu salariul > 10.000
SELECT D.DEPARTMENT_NAME
FROM DEPARTMENTS D
WHERE EXISTS (
    SELECT 1
    FROM EMPLOYEES E
    WHERE E.DEPARTMENT_ID = D.DEPARTMENT_ID
      AND E.SALARY > 10000
);
```

---

## 4. Erori Frecvente

### 4.1. Cereri Ierarhice

```sql
-- ❌ EROARE: lipsește PRIOR
CONNECT BY EMPLOYEE_ID = MANAGER_ID;
-- ORA-01788: CONNECT BY clause required in this query block

-- ✅ CORECT:
CONNECT BY PRIOR EMPLOYEE_ID = MANAGER_ID;

-- ❌ EROARE: subcerere în CONNECT BY (nu este permis)
CONNECT BY PRIOR EMPLOYEE_ID = (SELECT MANAGER_ID FROM ...);

-- ❌ Confuzie clasică: WHERE vs AND în CONNECT BY
-- WHERE — exclude rândul din afișare, dar ramura lui continuă
WHERE EMPLOYEE_ID <> 102
-- AND în CONNECT BY — exclude rândul ȘI tăie toată ramura lui
CONNECT BY ... AND EMPLOYEE_ID <> 102
```

### 4.2. Clauza WITH

```sql
-- ❌ EROARE: folosești un bloc înainte să îl definești
WITH B AS (SELECT * FROM A_BLOCK ...)   -- A_BLOCK nu e definit încă!
SELECT ...

-- ✅ CORECT: definești în ordine
WITH
    A_BLOCK AS (SELECT ...),            -- definit primul
    B AS (SELECT * FROM A_BLOCK ...)    -- definit al doilea
SELECT ...

-- ❌ EROARE: punct și virgulă în interiorul WITH
WITH BLOC AS (
    SELECT ...;     -- ← greșit, ; e doar la finalul întregului query
)
```

### 4.3. EXISTS

```sql
-- ❌ Greșeală clasică: subcererea nu este legată de tabelul exterior
WHERE EXISTS (
    SELECT 1 FROM EMPLOYEES    -- nu are WHERE cu legătură la DEPARTMENTS!
);
-- Returnează MEREU TRUE dacă EMPLOYEES are măcar un rând
-- Deci afișezi TOATE departamentele, indiferent dacă au angajați sau nu

-- ✅ CORECT: subcererea TREBUIE să refere tabelul din exterior
WHERE EXISTS (
    SELECT 1 FROM EMPLOYEES E
    WHERE E.DEPARTMENT_ID = D.DEPARTMENT_ID   -- ← legătura cu D
);

-- ❌ NOT IN cu NULL (capcană!)
WHERE DEPARTMENT_ID NOT IN (SELECT DEPARTMENT_ID FROM EMPLOYEES);
-- Dacă vreun angajat are DEPARTMENT_ID NULL → 0 rezultate

-- ✅ Mai sigur cu NOT EXISTS
WHERE NOT EXISTS (
    SELECT 1 FROM EMPLOYEES E
    WHERE E.DEPARTMENT_ID = D.DEPARTMENT_ID
);
```

---

## 5. Exerciții Practice

### Ex 1: Organigramă completă

**Cerință:** Afișează toți angajații în formă de organigramă (cu indentare), pornind de la CEO în jos.

<details>
<summary>🔍 Soluție</summary>

```sql
SELECT
    LEVEL,
    LPAD(' ', (LEVEL-1) * 4) || FIRST_NAME AS ORGANIGARMA,
    EMPLOYEE_ID,
    MANAGER_ID
FROM EMPLOYEES
START WITH MANAGER_ID IS NULL
CONNECT BY PRIOR EMPLOYEE_ID = MANAGER_ID;
```

</details>

---

### Ex 2: Calea până la CEO

**Cerință:** Bruce (EMPLOYEE_ID = 104) vrea să știe prin câți șefi trece până la CEO. Afișează lanțul complet de la el în sus.

<details>
<summary>🔍 Soluție</summary>

```sql
SELECT LEVEL, FIRST_NAME, EMPLOYEE_ID, MANAGER_ID
FROM EMPLOYEES
START WITH EMPLOYEE_ID = 104
CONNECT BY PRIOR MANAGER_ID = EMPLOYEE_ID;
```

</details>

---

### Ex 3: Doar primele 2 niveluri din organigramă

**Cerință:** Afișează ierarhia, dar doar CEO-ul și subalternii lui direcți (maxim 2 niveluri).

<details>
<summary>🔍 Soluție</summary>

```sql
SELECT LEVEL, LPAD(' ', (LEVEL-1)*4) || FIRST_NAME AS ORGANIGARMA
FROM EMPLOYEES
WHERE LEVEL <= 2
START WITH MANAGER_ID IS NULL
CONNECT BY PRIOR EMPLOYEE_ID = MANAGER_ID;
```

</details>

---

### Ex 4: WITH — Angajatul cu salariul maxim pe departament

**Cerință:** Afișează pentru fiecare departament cine are salariul cel mai mare.

<details>
<summary>🔍 Soluție</summary>

```sql
WITH MAX_PE_DEPT AS (
    SELECT DEPARTMENT_ID, MAX(SALARY) AS SAL_MAX
    FROM EMPLOYEES
    GROUP BY DEPARTMENT_ID
)
SELECT E.FIRST_NAME, E.SALARY, D.DEPARTMENT_NAME
FROM EMPLOYEES E
JOIN DEPARTMENTS D ON E.DEPARTMENT_ID = D.DEPARTMENT_ID
JOIN MAX_PE_DEPT M ON E.DEPARTMENT_ID = M.DEPARTMENT_ID
                  AND E.SALARY = M.SAL_MAX
ORDER BY E.DEPARTMENT_ID;
```

</details>

---

### Ex 5: WITH — Departamente cu salariu mediu peste media globală

**Cerință:** Afișează departamentele al căror salariu mediu e mai mare decât media salariilor din toată compania.

<details>
<summary>🔍 Soluție</summary>

```sql
WITH MEDIE_DEPT AS (
    SELECT DEPARTMENT_ID, AVG(SALARY) AS SAL_MEDIU
    FROM EMPLOYEES
    GROUP BY DEPARTMENT_ID
)
SELECT D.DEPARTMENT_NAME, M.SAL_MEDIU
FROM DEPARTMENTS D
JOIN MEDIE_DEPT M ON D.DEPARTMENT_ID = M.DEPARTMENT_ID
WHERE M.SAL_MEDIU > (SELECT AVG(SALARY) FROM EMPLOYEES)
ORDER BY M.SAL_MEDIU DESC;
```

</details>

---

### Ex 6: WITH cu două blocuri

**Cerință:** Afișează departamentele al căror total de salarii e mai mare decât media totalurilor pe departament.

<details>
<summary>🔍 Soluție</summary>

```sql
WITH
    TOTAL_DEPT AS (
        SELECT DEPARTMENT_ID, SUM(SALARY) AS TOTAL
        FROM EMPLOYEES
        GROUP BY DEPARTMENT_ID
    ),
    MEDIE_TOTALE AS (
        SELECT AVG(TOTAL) AS MEDIE
        FROM TOTAL_DEPT
    )
SELECT D.DEPARTMENT_NAME, T.TOTAL
FROM DEPARTMENTS D
JOIN TOTAL_DEPT T   ON D.DEPARTMENT_ID = T.DEPARTMENT_ID
JOIN MEDIE_TOTALE M ON T.TOTAL > M.MEDIE
ORDER BY T.TOTAL DESC;
```

</details>

---

### Ex 7: EXISTS — Departamente cu angajați

**Cerință:** Afișează doar departamentele care au cel puțin un angajat.

<details>
<summary>🔍 Soluție</summary>

```sql
SELECT D.DEPARTMENT_ID, D.DEPARTMENT_NAME
FROM DEPARTMENTS D
WHERE EXISTS (
    SELECT 1
    FROM EMPLOYEES E
    WHERE E.DEPARTMENT_ID = D.DEPARTMENT_ID
);
```

</details>

---

### Ex 8: NOT EXISTS — Departamente fără angajați

**Cerință:** Afișează departamentele care nu au niciun angajat (departamente goale).

<details>
<summary>🔍 Soluție</summary>

```sql
SELECT D.DEPARTMENT_ID, D.DEPARTMENT_NAME
FROM DEPARTMENTS D
WHERE NOT EXISTS (
    SELECT 1
    FROM EMPLOYEES E
    WHERE E.DEPARTMENT_ID = D.DEPARTMENT_ID
);
```

</details>

---

### Ex 9: EXISTS cu condiție

**Cerință:** Afișează departamentele care au cel puțin un angajat cu salariul peste 10.000.

<details>
<summary>🔍 Soluție</summary>

```sql
SELECT D.DEPARTMENT_NAME
FROM DEPARTMENTS D
WHERE EXISTS (
    SELECT 1
    FROM EMPLOYEES E
    WHERE E.DEPARTMENT_ID = D.DEPARTMENT_ID
      AND E.SALARY > 10000
);
```

</details>

---

## 6. Rezumat Rapid

```
╔═══════════════════════════════════════════════════════════════╗
║             CERERI IERARHICE                                  ║
╠═══════════════════════════════════════════════════════════════╣
║ START WITH conditie   → de unde pornești (ex: CEO)           ║
║ CONNECT BY PRIOR ...  → cum navighezi (spre subalterni/șefi) ║
║ LEVEL                 → treapta curentă (1 = punct de start) ║
║                                                               ║
║ PRIOR pe stânga → cobori (șef → subaltern)                   ║
║ PRIOR pe dreapta → urci (subaltern → șef)                    ║
║                                                               ║
║ WHERE LEVEL <= n  → limitezi afișarea (ramura rămâne intactă) ║
║ AND în CONNECT BY → tai ramura întreagă                       ║
║ ❌ Nu poți pune subcereri în CONNECT BY                       ║
╚═══════════════════════════════════════════════════════════════╝

╔═══════════════════════════════════════════════════════════════╗
║             CLAUZA WITH                                       ║
╠═══════════════════════════════════════════════════════════════╣
║ WITH NUME AS (SELECT ...)                                     ║
║ SELECT ... FROM NUME                                          ║
║                                                               ║
║ ✅ Calculezi o dată, folosești de câte ori vrei               ║
║ ✅ Poți defini mai multe blocuri (separate prin virgulă)      ║
║ ✅ Un bloc poate folosi blocurile definite înaintea lui        ║
║ ⚠️ Nu ; după blocul WITH — doar la finalul query-ului        ║
╚═══════════════════════════════════════════════════════════════╝

╔═══════════════════════════════════════════════════════════════╗
║             OPERATORUL EXISTS                                 ║
╠═══════════════════════════════════════════════════════════════╣
║ WHERE EXISTS (SELECT 1 FROM ... WHERE legatura)              ║
║                                                               ║
║ ✅ = "există cel puțin un rând care îndeplinește condiția"    ║
║ ❌ NOT EXISTS = "nu există niciun rând"                       ║
║                                                               ║
║ ⚠️ Subcererea TREBUIE să fie legată de tabelul exterior!     ║
║ ⚠️ NOT IN cu NULL → 0 rânduri. Folosește NOT EXISTS în loc   ║
║                                                               ║
║ IN    → "valoarea mea e în lista asta?"                       ║
║ EXISTS → "există ceva acolo legat de mine?"                   ║
╚═══════════════════════════════════════════════════════════════╝
```

---

## 🎯 Reguli de Aur

```
CERERI IERARHICE:
  1. PRIOR e obligatoriu în CONNECT BY — fără el, eroare
  2. Cobori (șef → subaltern): PRIOR EMPLOYEE_ID = MANAGER_ID
  3. Urci (subaltern → șef):   PRIOR MANAGER_ID = EMPLOYEE_ID
  4. WHERE = filtrează ce afișezi, AND în CONNECT BY = taie ramura

WITH:
  5. Definești blocurile ÎNAINTE de SELECT-ul principal
  6. Ordinea blocurilor contează — poți referi doar ce e definit înainte
  7. Blocul se comportă exact ca un tabel — îl poți pune în FROM, JOIN, WHERE

EXISTS:
  8. Subcererea din EXISTS TREBUIE să aibă o condiție care leagă cele două tabele
  9. Ce selectezi în subcerere nu contează (SELECT 1, SELECT *, orice)
 10. Dacă folosești NOT IN și subcererea poate avea NULL → folosește NOT EXISTS
```
