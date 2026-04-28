# PARTEA 1: Fundamentele JOIN

## 1.1. Ce este un JOIN?

### 🎯 Definiție simplă
**JOIN** = operația de a combina date din **două sau mai multe tabele** pe baza unor coloane comune.

### 🧠 Analogie de viață reală

Imaginează-ți două fișe de evidență:

```
📋 Fișa 1 - ANGAJAȚI              📋 Fișa 2 - DEPARTAMENTE
┌────┬──────────┬─────────┐      ┌─────────┬──────────────────┐
│ ID │   Nume   │  Dept   │      │  Dept   │   Nume Dept      │
├────┼──────────┼─────────┤      ├─────────┼──────────────────┤
│  1 │  Maria   │   10    │      │   10    │  IT              │
│  2 │  Ion     │   20    │      │   20    │  Vânzări         │
│  3 │  Ana     │   10    │      │   30    │  Marketing       │
└────┴──────────┴─────────┘      └─────────┴──────────────────┘
```

**JOIN-ul** conectează cele două fișe:

```
✨ REZULTAT după JOIN
┌────┬──────────┬─────────┬──────────────────┐
│ ID │   Nume   │  Dept   │   Nume Dept      │
├────┼──────────┼─────────┼──────────────────┤
│  1 │  Maria   │   10    │  IT              │
│  2 │  Ion     │   20    │  Vânzări         │
│  3 │  Ana     │   10    │  IT              │
└────┴──────────┴─────────┴──────────────────┘
```

---

## Tipuri de JOIN — Harta Decizională

```
                    ┌──────────────────┐
                    │   Ai nevoie de   │
                    │   toate liniile? │
                    └────────┬─────────┘
                             │
              ┌──────────────┴──────────────┐
              ▼                             ▼
        ┌────────────┐                 ┌───────────┐
        │    NU      │                 │    DA     │
        │ (doar      │                 │ (și cele  │
        │  potriviri)│                 │  fără     │
        │            │                 │  match)   │
        └─────┬──────┘                 └────┬──────┘
              │                             │
              ▼                             ▼
        ┌────────────┐                 ┌───────────┐
        │ INNER JOIN │                 │OUTER JOIN?│
        └────────────┘                 └────┬──────┘
                                            │
              ┌─────────────────────────────┼─────────────────────────────┐
              ▼                             ▼                             ▼
        ┌───────────┐               ┌───────────┐                 ┌───────────┐
        │  LEFT     │               │  RIGHT    │                 │   FULL    │
        │(toate din │               │(toate din │                 │(toate din │
        │ stânga)   │               │ dreapta)  │                 │ ambele)   │
        └───────────┘               └───────────┘                 └───────────┘
```

### Tabel comparativ:

| Tip | Descriere | Când îl folosim? |
| :--- | :--- | :--- |
| **INNER JOIN** (equijoin) | Returnează doar liniile cu valori egale pe coloanele de join | Vrem doar datele care au corespondență în ambele tabele |
| **NONEQUIJOIN** | Condiția de join folosește alți operatori decât `=` | Comparări de intervale, date, salarii |
| **LEFT OUTER JOIN** | Toate liniile din tabelul stâng + potrivirile din dreapta | Vrem toate înregistrările din tabela principală, chiar și fără match |
| **RIGHT OUTER JOIN** | Toate liniile din tabelul drept + potrivirile din stânga | Similar cu LEFT, dar din perspectiva celeilalte tabele |
| **FULL OUTER JOIN** | LEFT OUTER JOIN + RIGHT OUTER JOIN | Vrem TOATE înregistrările din ambele tabele |
| **SELF JOIN** | Un tabel unit cu el însuși | Ierarhii: angajat-manager, categorie-subcategorie |
| **CROSS JOIN** | Produsul cartezian (toate combinațiile posibile) | Generare de combinații, teste, matrici |
| **NATURAL JOIN** | Equijoin automat pe toate coloanele cu același nume | Când coloanele de join au IDENTIC același nume în ambele tabele |

---

## 1.2. Reguli de bază

### ✅ Reguli esențiale

| Regulă | Explicație | Exemplu |
|--------|-----------|---------|
| **Număr de condiții** | Pentru `n` tabele → minim `n-1` condiții JOIN | 3 tabele = 2 JOIN-uri |
| **Alias-uri** | Prescurtări pentru nume de tabele | `EMPLOYEES E` |
| **Prefixare** | Pune alias-ul înaintea coloanei | `E.FIRST_NAME` |
| **Maxim 30 caractere** | Alias-urile pot avea max 30 chars | `EMP` (bun) vs `E` (mai bun) |

### 🎨 Exemplu comparativ

```sql
-- ❌ GREU DE CITIT (fără alias-uri)
SELECT FIRST_NAME, DEPARTMENT_NAME
FROM EMPLOYEES, DEPARTMENTS
WHERE EMPLOYEES.DEPARTMENT_ID = DEPARTMENTS.DEPARTMENT_ID;

-- ✅ CLAR ȘI CONCIS (cu alias-uri)
SELECT E.FIRST_NAME, D.DEPARTMENT_NAME
FROM EMPLOYEES E, DEPARTMENTS D
WHERE E.DEPARTMENT_ID = D.DEPARTMENT_ID;
```

---

## 1.3. INNER JOIN - Primul și cel mai important

### 🎯 Ce face INNER JOIN?

Returnează **DOAR** liniile care au potriviri în **AMBELE** tabele.

```
Tabel STÂNGA     ∩     Tabel DREAPTA
    (A)                    (B)
    
    Rezultat = A ∩ B (intersecția)
```

### 📊 Exemplu vizual

```
ANGAJAȚI                    DEPARTAMENTE
┌────┬────────┬──────┐     ┌──────┬─────────┐
│ ID │  Nume  │ Dept │     │ Dept │  Nume   │
├────┼────────┼──────┤     ├──────┼─────────┤
│ 1  │ Maria  │  10  │ ──→ │  10  │   IT    │
│ 2  │ Ion    │  20  │ ──→ │  20  │ Vânzări │
│ 3  │ Ana    │  10  │ ──→ │  30  │ HR      │
│ 4  │ Mihai  │ NULL │     └──────┴─────────┘
└────┴────────┴──────┘

INNER JOIN returnează DOAR 1, 2, 3 (au match)
Mihai (linia 4) NU apare pentru că Dept = NULL
```

---

### 🔧 Trei Sintaxe Echivalente

#### **Sintaxa 1: WHERE (veche, Oracle)**

```sql
SELECT E.FIRST_NAME, D.DEPARTMENT_NAME
FROM EMPLOYEES E, DEPARTMENTS D
WHERE E.DEPARTMENT_ID = D.DEPARTMENT_ID;
```

**Caracteristici:**
- ✅ Simplă pentru cazuri simple
- ❌ JOIN-ul și filtrarea sunt amestecate
- ❌ Specifică Oracle (nu portabilă)

---

#### **Sintaxa 2: JOIN ... ON (modernă, ANSI) ⭐ RECOMANDATĂ**

```sql
SELECT E.FIRST_NAME, D.DEPARTMENT_NAME
FROM EMPLOYEES E
JOIN DEPARTMENTS D ON E.DEPARTMENT_ID = D.DEPARTMENT_ID;
```

**Caracteristici:**
- ✅ Clară și lizibilă
- ✅ Portabilă între baze de date
- ✅ Separă JOIN de filtrare
- ⚠️ Alias-urile sunt **OBLIGATORII**

**Exemplu cu filtrare suplimentară:**

```sql
SELECT E.FIRST_NAME, D.DEPARTMENT_NAME
FROM EMPLOYEES E
JOIN DEPARTMENTS D ON E.DEPARTMENT_ID = D.DEPARTMENT_ID
WHERE E.SALARY > 5000;  -- Filtrare separată de JOIN
```

---

#### **Sintaxa 3: JOIN ... USING (concisă)**

```sql
SELECT FIRST_NAME, DEPARTMENT_NAME
FROM EMPLOYEES
JOIN DEPARTMENTS USING (DEPARTMENT_ID);
```

**Caracteristici:**
- ✅ Foarte concisă
- ✅ Funcționează când coloanele au **ACELAȘI NUME**
- ⚠️ **NU** folosim alias-uri
- ⚠️ **NU** prefixăm coloana din USING

---

### 🎓 Exercițiu Ghidat 1

**Sarcină**: Afișați numele angajatului și titlul job-ului său.

**Tabele disponibile:**
- `EMPLOYEES` (EMPLOYEE_ID, FIRST_NAME, JOB_ID)
- `JOBS` (JOB_ID, JOB_TITLE)

<details>
<summary>💡 Indiciu 1</summary>

Trebuie să conectezi tabelele prin `JOB_ID`.
</details>

<details>
<summary>💡 Indiciu 2</summary>

Folosește sintaxa `JOIN ... ON` și alias-uri `E` și `J`.
</details>

<details>
<summary>✅ Soluție completă</summary>

```sql
SELECT E.FIRST_NAME, J.JOB_TITLE
FROM EMPLOYEES E
JOIN JOBS J ON E.JOB_ID = J.JOB_ID;
```

**Explicație:**
1. `FROM EMPLOYEES E` - tabelul principal cu alias E
2. `JOIN JOBS J` - tabelul secundar cu alias J
3. `ON E.JOB_ID = J.JOB_ID` - condiția de conectare
</details>

---

### 🎓 Exercițiu Ghidat 2

**Sarcină**: Câți angajați are fiecare job? Afișați titlul job-ului și numărul de angajați.

<details>
<summary>💡 Indiciu</summary>

Combină JOIN cu GROUP BY și COUNT.
</details>

<details>
<summary>✅ Soluție cu WHERE</summary>

```sql
SELECT J.JOB_TITLE, COUNT(E.EMPLOYEE_ID) AS NR_ANGAJATI
FROM EMPLOYEES E, JOBS J
WHERE E.JOB_ID = J.JOB_ID
GROUP BY J.JOB_TITLE;
```
</details>

<details>
<summary>✅ Soluție cu ON (recomandată)</summary>

```sql
SELECT J.JOB_TITLE, COUNT(E.EMPLOYEE_ID) AS NR_ANGAJATI
FROM EMPLOYEES E
JOIN JOBS J ON E.JOB_ID = J.JOB_ID
GROUP BY J.JOB_TITLE;
```
</details>

<details>
<summary>✅ Soluție cu USING</summary>

```sql
SELECT JOB_TITLE, COUNT(EMPLOYEE_ID) AS NR_ANGAJATI
FROM EMPLOYEES
JOIN JOBS USING (JOB_ID)
GROUP BY JOB_TITLE;
```

**Observație**: Fără alias-uri, fără prefix pe `JOB_ID`.
</details>

---

### 📊 Tabel Comparativ: Cele 3 Sintaxe

| Aspect | WHERE | ON | USING |
|--------|-------|-------|-------|
| **Alias obligatorii** | Nu | ✅ DA | ❌ NU |
| **Prefix pe coloana JOIN** | Da | Da | ❌ NU |
| **Portabilitate** | Oracle | ANSI (toate DB) | ANSI (toate DB) |
| **Separare JOIN/filtrare** | ❌ | ✅ | ✅ |
| **Coloane cu nume diferit** | ✅ | ✅ | ❌ |
| **Când să folosești** | Cod vechi | **RECOMANDATĂ** | Cazuri simple |

---

# PARTEA 2: JOIN-uri Avansate

## 2.1. OUTER JOIN - Când vrei și liniile fără match

### 🎯 Conceptul de bază

**OUTER JOIN** = include și liniile care **NU** au corespondent în cealaltă tabelă.

```
🧩 Diferența cheie:
INNER JOIN → doar intersecția ∩
OUTER JOIN → și elementele unice din una/ambele tabele
```

---

### 📊 LEFT JOIN

**Regula**: Toate liniile din tabelul **STÂNG** + potrivirile din dreapta.

```
ANGAJAȚI (STÂNGA)           DEPARTAMENTE (DREAPTA)
┌────┬────────┬──────┐      ┌──────┬─────────┐
│ ID │  Nume  │ Dept │      │ Dept │  Nume   │
├────┼────────┼──────┤      ├──────┼─────────┤
│ 1  │ Maria  │  10  │  ──→ │  10  │   IT    │
│ 2  │ Ion    │  20  │  ──→ │  20  │ Vânzări │
│ 3  │ Ana    │ NULL │  ✗   │  30  │ HR      │
└────┴────────┴──────┘      └──────┴─────────┘

LEFT JOIN returnează:
┌────┬────────┬──────────────┐
│ ID │  Nume  │  Departament │
├────┼────────┼──────────────┤
│ 1  │ Maria  │  IT          │
│ 2  │ Ion    │  Vânzări     │
│ 3  │ Ana    │  NULL        │ ← Ana apare, chiar dacă Dept = NULL
└────┴────────┴──────────────┘
```

**Cod:**

```sql
SELECT E.FIRST_NAME, D.DEPARTMENT_NAME
FROM EMPLOYEES E
LEFT JOIN DEPARTMENTS D ON E.DEPARTMENT_ID = D.DEPARTMENT_ID;
```

**Când îl folosim?**
- ✅ Rapoarte cu toți angajații (inclusiv cei fără departament)
- ✅ Analiză date incomplete
- ✅ Găsirea înregistrărilor orfane

---

### 📊 RIGHT JOIN

**Regula**: Toate liniile din tabelul **DREPT** + potrivirile din stânga.

```sql
SELECT E.FIRST_NAME, D.DEPARTMENT_NAME
FROM EMPLOYEES E
RIGHT JOIN DEPARTMENTS D ON E.DEPARTMENT_ID = D.DEPARTMENT_ID;
```

**Rezultat**: Toate departamentele, inclusiv HR (care nu are angajați).

```
┌───────────┬──────────────┐
│  Angajat  │ Departament  │
├───────────┼──────────────┤
│  Maria    │  IT          │
│  Ion      │  Vânzări     │
│  NULL     │  HR          │ ← HR apare, chiar fără angajați
└───────────┴──────────────┘
```

---

### 🔄 Echivalența LEFT ↔ RIGHT

**Aceste două query-uri sunt IDENTICE:**

```sql
-- Varianta 1: LEFT JOIN
SELECT E.FIRST_NAME, D.DEPARTMENT_NAME
FROM EMPLOYEES E
LEFT JOIN DEPARTMENTS D ON E.DEPARTMENT_ID = D.DEPARTMENT_ID;

-- Varianta 2: RIGHT JOIN (inversând tabelele)
SELECT E.FIRST_NAME, D.DEPARTMENT_NAME
FROM DEPARTMENTS D
RIGHT JOIN EMPLOYEES E ON E.DEPARTMENT_ID = D.DEPARTMENT_ID;
```

**💡 Regulă de aur**: `LEFT JOIN(A, B)` = `RIGHT JOIN(B, A)`

---

### 📊 FULL OUTER JOIN

**Regula**: Toate liniile din **AMBELE** tabele.

```sql
SELECT E.FIRST_NAME, D.DEPARTMENT_NAME
FROM EMPLOYEES E
FULL OUTER JOIN DEPARTMENTS D ON E.DEPARTMENT_ID = D.DEPARTMENT_ID;
```

**Rezultat**: Ana (fără dept) + HR (fără angajați).

```
┌───────────┬──────────────┐
│  Angajat  │ Departament  │
├───────────┼──────────────┤
│  Maria    │  IT          │
│  Ion      │  Vânzări     │
│  Ana      │  NULL        │ ← Angajat fără dept
│  NULL     │  HR          │ ← Dept fără angajați
└───────────┴──────────────┘
```

**Când îl folosim?**
- ✅ Reconciliere de date
- ✅ Comparare completă între două surse
- ✅ Identificare discrepanțe

---

### 🎓 Exercițiu Ghidat 3

**Sarcină**: Afișați toți angajații cu departamentul lor. Pentru cei fără departament, afișați "Fără departament".

<details>
<summary>💡 Indiciu</summary>

Folosește LEFT JOIN și funcția NVL sau COALESCE.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT E.FIRST_NAME, 
       E.LAST_NAME,
       NVL(D.DEPARTMENT_NAME, 'Fără departament') AS DEPARTAMENT
FROM EMPLOYEES E
LEFT JOIN DEPARTMENTS D ON E.DEPARTMENT_ID = D.DEPARTMENT_ID;

-- Alternativ cu COALESCE (ANSI standard):
SELECT E.FIRST_NAME, 
       E.LAST_NAME,
       COALESCE(D.DEPARTMENT_NAME, 'Fără departament') AS DEPARTAMENT
FROM EMPLOYEES E
LEFT JOIN DEPARTMENTS D ON E.DEPARTMENT_ID = D.DEPARTMENT_ID;
```
</details>

---

## 2.2. SELF JOIN - Tabelul unit cu el însuși

### 🎯 Conceptul

**SELF JOIN** = un tabel se conectează cu **o copie a lui însuși**.

**Cazuri de utilizare:**
- 👥 Ierarhii organizaționale (angajat → manager)
- 📂 Categorii și subcategorii
- 🔗 Relații de precedență (task → task precedent)

---

### 📊 Exemplu: Ierarhie Angajat-Manager

```
EMPLOYEES
┌────┬────────┬────────────┐
│ ID │  Nume  │ Manager_ID │
├────┼────────┼────────────┤
│ 1  │ Maria  │    NULL    │ ← CEO (fără manager)
│ 2  │ Ion    │      1     │ ← Managerul lui Ion este Maria
│ 3  │ Ana    │      1     │
│ 4  │ Mihai  │      2     │ ← Managerul lui Mihai este Ion
└────┴────────┴────────────┘
```

**Cod:**

```sql
SELECT E.FIRST_NAME AS Angajat,
       M.FIRST_NAME AS Manager
FROM EMPLOYEES E
JOIN EMPLOYEES M ON E.MANAGER_ID = M.EMPLOYEE_ID;
```

**Rezultat:**

```
┌──────────┬──────────┐
│ Angajat  │  Manager │
├──────────┼──────────┤
│  Ion     │  Maria   │
│  Ana     │  Maria   │
│  Mihai   │  Ion     │
└──────────┴──────────┘
```

**⚠️ Observație**: Maria (CEO) nu apare pentru că MANAGER_ID = NULL.

**Pentru a include și CEO-ul:**

```sql
SELECT E.FIRST_NAME AS Angajat,
       NVL(M.FIRST_NAME, 'CEO') AS Manager
FROM EMPLOYEES E
LEFT JOIN EMPLOYEES M ON E.MANAGER_ID = M.EMPLOYEE_ID;
```

---

## Greșeala de la tutoriat

Vom analizăm modul corect de a interoga o ierarhie de angajați și manageri dintr-un tabel unic (`EMPLOYEES`), evidențiind importanța utilizării cheilor corecte în operațiunile de `JOIN`.

### Diferența de Logică

Diferența majoră constă în **criteriul de unire (JOIN)**. Într-o bază de date relațională, pentru a asocia un angajat cu managerul său, trebuie să potrivim un identificator unic (cheia primară cu cheia străină), nu o valoare subiectivă precum salariul.

| Tip Interogare | Criteriu JOIN | Rezultat |
| :--- | :--- | :--- |
| **CORECT** | `E.MANAGER_ID = M.EMPLOYEE_ID` | Relație ierarhică validă (cine pe cine conduce). |
| **GRESIT** | `E.SALARY > M.SALARY` | Produs cartezian logic invalid; asociază angajații cu orice persoană care câștigă mai puțin. |

### Varianta Corectă (Self Join pe ID)

Această interogare utilizează corect `MANAGER_ID` pentru a stabili legătura directă între un subordonat și superiorul său, filtrând apoi rezultatele după salariu.

```sql
SELECT E.FIRST_NAME || ' ' || E.LAST_NAME AS Angajat,
       E.SALARY AS Salariu,
       M.FIRST_NAME AS Manager,
       M.SALARY AS Salariu_Manager
FROM EMPLOYEES E
JOIN EMPLOYEES M ON E.MANAGER_ID = M.EMPLOYEE_ID
WHERE E.SALARY > M.SALARY;
```

### Varianta Eronată (Join pe Salariu)

Această interogare eșuează deoarece `ON E.SALARY > M.SALARY` nu stabilește o relație de subordonare. Rezultatul va returna combinații arbitrare de angajați care nu au neapărat o legătură de management între ei.

```sql
-- GRESIT: Aceasta interogare nu identifică managerul, 
-- ci creează o listă de combinații aleatorii bazată pe salariu.
SELECT E.FIRST_NAME || ' ' || E.LAST_NAME AS Angajat,
       E.SALARY AS Salariu,
       M.FIRST_NAME AS Manager,
       M.SALARY AS Salariu_Manager
FROM EMPLOYEES E
JOIN EMPLOYEES M ON E.SALARY > M.SALARY;
```

> **Notă:** Folosirea salariului ca și condiție de `JOIN` în loc de `WHERE` (ca filtru) duce la pierderea integrității datelor și la rezultate irelevante pentru ierarhia organizațională.

---

## 2.3. NONEQUIJOIN - Alti operatori decât =

### 🎯 Conceptul

**NONEQUIJOIN** = JOIN cu operatori **diferiți** de `=`.

**Operatori posibili:**
- `>`, `<`, `>=`, `<=`, `<>`
- `BETWEEN ... AND ...`

---

### 📊 Exemplu: Grila de Salarii

```
EMPLOYEES                    JOB_GRADES
┌────┬─────────┐            ┌───────┬─────┬──────┐
│ ID │ Salary  │            │ Grade │ Min │ Max  │
├────┼─────────┤            ├───────┼─────┼──────┤
│ 1  │  3000   │            │   A   │ 1000│ 2999 │
│ 2  │  5000   │            │   B   │ 3000│ 5999 │
│ 3  │  8000   │            │   C   │ 6000│ 9999 │
└────┴─────────┘            └───────┴─────┴──────┘
```

**Cod:**

```sql
SELECT E.FIRST_NAME, 
       E.SALARY, 
       JG.GRADE_LEVEL
FROM EMPLOYEES E
JOIN JOB_GRADES JG 
  ON E.SALARY BETWEEN JG.LOWEST_SALARY AND JG.HIGHEST_SALARY;
```

**Rezultat:**

```
┌────────────┬─────────┬───────┐
│    Nume    │ Salary  │ Grade │
├────────────┼─────────┼───────┤
│   Maria    │  3000   │   B   │
│   Ion      │  5000   │   B   │
│   Ana      │  8000   │   C   │
└────────────┴─────────┴───────┘
```

---

## 2.4. JOIN pe Mai Multe Tabele

### 🎯 Regula de aur

Pentru **n tabele** → minim **n-1 condiții de JOIN**.

---

### 📊 Exemplu: 4 Tabele

**Sarcină**: Afișați angajatul, job-ul, departamentul și orașul.

```
EMPLOYEES → JOBS
    ↓
DEPARTMENTS → LOCATIONS
```

**Cod:**

```sql
SELECT E.FIRST_NAME, 
       J.JOB_TITLE, 
       D.DEPARTMENT_NAME, 
       L.CITY
FROM EMPLOYEES E
JOIN JOBS J        ON E.JOB_ID = J.JOB_ID                    -- JOIN 1
JOIN DEPARTMENTS D ON E.DEPARTMENT_ID = D.DEPARTMENT_ID      -- JOIN 2
JOIN LOCATIONS L   ON D.LOCATION_ID = L.LOCATION_ID;         -- JOIN 3

-- 4 tabele → 3 JOIN-uri ✅
```

---

### 🎓 Exercițiu Ghidat 5

**Sarcină**: Afișați numele complet al angajatului și țara în care lucrează (5 tabele).

**Lanț**: EMPLOYEES → DEPARTMENTS → LOCATIONS → COUNTRIES

<details>
<summary>💡 Indiciu</summary>

5 tabele → 4 JOIN-uri
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT E.FIRST_NAME || ' ' || E.LAST_NAME AS Angajat,
       C.COUNTRY_NAME AS Tara
FROM EMPLOYEES E
JOIN DEPARTMENTS D ON E.DEPARTMENT_ID = D.DEPARTMENT_ID
JOIN LOCATIONS L   ON D.LOCATION_ID = L.LOCATION_ID
JOIN COUNTRIES C   ON L.COUNTRY_ID = C.COUNTRY_ID;
```
</details>

---

# PARTEA 3: Cazuri Practice și Exerciții

## 3.1. Tipuri Speciale de JOIN

### 🎯 NATURAL JOIN

**Automat** face equijoin pe **toate** coloanele cu același nume.

```sql
SELECT EMPLOYEE_ID, FIRST_NAME, DEPARTMENT_NAME
FROM EMPLOYEES
NATURAL JOIN DEPARTMENTS;
```

**⚠️ PERICOL**: Dacă există mai multe coloane cu același nume, JOIN-ul se face pe TOATE!

**Reguli:**
- ❌ Fără alias-uri
- ❌ Fără prefix pe coloanele comune
- ❌ NU poate coexista cu USING

**Când să îl folosești?**
- ✅ Doar când ești SIGUR că vrei JOIN pe TOATE coloanele comune
- ❌ Evită-l în producție (risc mare)

---

### 🎯 CROSS JOIN (Produs Cartezian)

**Toate** combinațiile posibile între cele două tabele.

```sql
SELECT E.FIRST_NAME, D.DEPARTMENT_NAME
FROM EMPLOYEES E
CROSS JOIN DEPARTMENTS D;
```

**Rezultat**: Dacă ai 107 angajați și 27 departamente → **2,889 linii**!

**⚠️ PERICOL**: Produs cartezian accidental

```sql
-- GREȘIT! Lipsă condiție de JOIN
SELECT E.FIRST_NAME, D.DEPARTMENT_NAME
FROM EMPLOYEES E, DEPARTMENTS D;
-- Rezultat: 107 × 27 = 2,889 linii (fără sens!)
```

**Când să îl folosești?**
- ✅ Generare combinații de teste
- ✅ Matrici pentru analiză
- ❌ Aproape niciodată din greșeală!

---

## 3.2. Sintaxa Veche Oracle cu (+) - De evitat!

### ⚠️ Operatorul `(+)` - Specific Oracle

```sql
-- Echivalent LEFT JOIN
SELECT E.FIRST_NAME, D.DEPARTMENT_NAME
FROM EMPLOYEES E, DEPARTMENTS D
WHERE E.DEPARTMENT_ID = D.DEPARTMENT_ID(+);

-- Echivalent RIGHT JOIN
SELECT E.FIRST_NAME, D.DEPARTMENT_NAME
FROM EMPLOYEES E, DEPARTMENTS D
WHERE E.DEPARTMENT_ID(+) = D.DEPARTMENT_ID;
```

**Restricții critice:**
- ❌ NU poate fi pe ambele părți simultan
- ❌ NU poate fi combinat cu `IN`
- ❌ NU poate fi legat cu `OR`
- ❌ NU există echivalent pentru FULL OUTER JOIN

**💡 Recomandare**: Folosește **întotdeauna** sintaxa modernă (LEFT/RIGHT/FULL OUTER JOIN).

---

## 3.3. Capcane Frecvente

### ⚠️ Lista de Verificare

| Problemă | Cauză | Soluție |
|----------|-------|---------|
| **Eroare: "column ambiguously defined"** | USING cu prefix pe coloană | Nu prefixa coloana din USING |
| **Produs cartezian** | Lipsă condiție JOIN | n tabele → n-1 JOIN-uri |
| **NULL-uri neașteptate** | INNER JOIN în loc de LEFT | Folosește OUTER JOIN |
| **Eroare: "outer-join operator (+) not allowed"** | `(+)` pe ambele părți | Folosește FULL OUTER JOIN |
| **Rezultate duplicate** | Relații many-to-many | Verifică relațiile între tabele |

---

### 🎓 Exercițiu de Debugging

**Găsiți și corectați erorile:**

```sql
-- Query 1: GREȘIT
SELECT FIRST_NAME, DEPARTMENT_NAME
FROM EMPLOYEES E
JOIN DEPARTMENTS D USING (E.DEPARTMENT_ID);
```

<details>
<summary>✅ Soluție</summary>

**Eroare**: Cu USING nu folosești prefix `E.`

```sql
-- CORECT
SELECT FIRST_NAME, DEPARTMENT_NAME
FROM EMPLOYEES
JOIN DEPARTMENTS USING (DEPARTMENT_ID);
```
</details>

---

```sql
-- Query 2: GREȘIT
SELECT E.FIRST_NAME, D.DEPARTMENT_NAME, L.CITY
FROM EMPLOYEES E, DEPARTMENTS D, LOCATIONS L
WHERE E.DEPARTMENT_ID = D.DEPARTMENT_ID;
```

<details>
<summary>✅ Soluție</summary>

**Eroare**: Lipsă JOIN pentru LOCATIONS → produs cartezian!

```sql
-- CORECT
SELECT E.FIRST_NAME, D.DEPARTMENT_NAME, L.CITY
FROM EMPLOYEES E, DEPARTMENTS D, LOCATIONS L
WHERE E.DEPARTMENT_ID = D.DEPARTMENT_ID
  AND D.LOCATION_ID = L.LOCATION_ID;

-- SAU (mai bine):
SELECT E.FIRST_NAME, D.DEPARTMENT_NAME, L.CITY
FROM EMPLOYEES E
JOIN DEPARTMENTS D ON E.DEPARTMENT_ID = D.DEPARTMENT_ID
JOIN LOCATIONS L   ON D.LOCATION_ID = L.LOCATION_ID;
```
</details>

---

## 3.4. Exerciții Practice Finale

### 🏋️ Exercițiu 1: Managerii cu cei mai mulți subordonați

**Sarcină**: Afișați top 5 manageri după numărul de subordonați.

<details>
<summary>✅ Soluție</summary>

```sql
SELECT M.FIRST_NAME || ' ' || M.LAST_NAME AS Manager,
       COUNT(E.EMPLOYEE_ID) AS Nr_Subordonati
FROM EMPLOYEES M
JOIN EMPLOYEES E ON M.EMPLOYEE_ID = E.MANAGER_ID
GROUP BY M.EMPLOYEE_ID, M.FIRST_NAME, M.LAST_NAME
ORDER BY Nr_Subordonati DESC
FETCH FIRST 5 ROWS ONLY;

-- Alternativ (Oracle vechi):
-- WHERE ROWNUM <= 5
```
</details>

---

### 🏋️ Exercițiu 2: Departamente fără angajați

**Sarcină**: Găsiți departamentele care nu au niciun angajat.

<details>
<summary>✅ Soluție</summary>

```sql
SELECT D.DEPARTMENT_NAME
FROM DEPARTMENTS D
LEFT JOIN EMPLOYEES E ON D.DEPARTMENT_ID = E.DEPARTMENT_ID
WHERE E.EMPLOYEE_ID IS NULL;

-- Alternativ cu NOT EXISTS:
SELECT D.DEPARTMENT_NAME
FROM DEPARTMENTS D
WHERE NOT EXISTS (
    SELECT 1 
    FROM EMPLOYEES E 
    WHERE E.DEPARTMENT_ID = D.DEPARTMENT_ID
);
```
</details>

---

### 🏋️ Exercițiu 3: Raport Complet Multi-Nivel

**Sarcină**: Afișați angajatul, job-ul, managerul, departamentul, orașul și țara.

<details>
<summary>✅ Soluție</summary>

```sql
SELECT E.FIRST_NAME || ' ' || E.LAST_NAME AS Angajat,
       J.JOB_TITLE,
       M.FIRST_NAME || ' ' || M.LAST_NAME AS Manager,
       D.DEPARTMENT_NAME,
       L.CITY,
       C.COUNTRY_NAME
FROM EMPLOYEES E
LEFT JOIN EMPLOYEES M  ON E.MANAGER_ID = M.EMPLOYEE_ID
JOIN JOBS J            ON E.JOB_ID = J.JOB_ID
JOIN DEPARTMENTS D     ON E.DEPARTMENT_ID = D.DEPARTMENT_ID
JOIN LOCATIONS L       ON D.LOCATION_ID = L.LOCATION_ID
JOIN COUNTRIES C       ON L.COUNTRY_ID = C.COUNTRY_ID;
```

**Observații:**
- LEFT JOIN pentru manageri (CEO nu are manager)
- INNER JOIN pentru restul (date obligatorii)
</details>

---

## 📊 Cheat Sheet Final - Tipărește și Afișează!

```
╔═══════════════════════════════════════════════════════════════╗
║                    SQL JOIN CHEAT SHEET                       ║
╠═══════════════════════════════════════════════════════════════╣
║                                                               ║
║  INNER JOIN     → doar potrivirile din ambele tabele          ║
║  LEFT JOIN      → toate din stânga + potrivirile              ║
║  RIGHT JOIN     → toate din dreapta + potrivirile             ║
║  FULL OUTER     → toate din ambele tabele                     ║
║  SELF JOIN      → tabelul cu el însuși (ierarhii)             ║
║  CROSS JOIN     → produs cartezian (toate combinațiile)       ║
║  NATURAL JOIN   → auto-join pe coloane cu același nume        ║
║  NONEQUIJOIN    → alte operatori (<, >, BETWEEN)              ║
║                                                               ║
╠═══════════════════════════════════════════════════════════════╣
║  SINTAXE:                                                     ║
║                                                               ║
║  WHERE E.X = D.X          → veche Oracle                      ║
║  JOIN ... ON E.X = D.X    → ANSI (recomandată) ⭐             ║
║  JOIN ... USING (X)       → ANSI concisă                      ║
║                                                               ║
╠═══════════════════════════════════════════════════════════════╣
║  REGULI DE AUR:                                               ║
║                                                               ║
║  ✅ n tabele → minim n-1 JOIN-uri                            ║
║  ✅ Alias-uri obligatorii cu ON                               ║
║  ✅ Fără alias-uri cu USING                                   ║
║  ✅ Nu prefixa coloana din USING                              ║
║  ✅ LEFT JOIN(A,B) = RIGHT JOIN(B,A)                          ║
║  ✅ Evită (+) → folosește SQL3                                ║
║                                                               ║
╠═══════════════════════════════════════════════════════════════╣
║  DEBUGGING RAPID:                                             ║
║                                                               ║
║  Produs cartezian?     → Verifică numărul de JOIN-uri         ║
║  NULL-uri neașteptate? → Folosește OUTER JOIN                 ║
║  Column ambiguous?     → Elimină prefixul din USING           ║
║  Eroare (+)?           → Folosește LEFT/RIGHT/FULL            ║
║                                                               ║
╚═══════════════════════════════════════════════════════════════╝
```

---
