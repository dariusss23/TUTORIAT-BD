# 🔗 Cereri Multi-Relație — JOIN în SQL Oracle

---

## 1. Ce este un JOIN?

**JOIN** este operația de regăsire a datelor din **două sau mai multe tabele**, pe baza valorilor comune ale unor coloane. De obicei, aceste coloane reprezintă **cheia primară (PK)** și **cheia externă (FK)** ale tabelelor.

### Reguli generale:
- Pentru a uni **n tabele**, sunt necesare cel puțin **n – 1 condiții de join**.
- Se recomandă **prefixarea coloanelor** cu alias-ul tabelului — pentru claritate și performanță.
- Dacă același nume de coloană apare în mai mult de două tabele, prefixarea este **obligatorie**.
- Dacă un alias este atribuit unui tabel în `FROM`, el **trebuie** să înlocuiască toate aparițiile numelui tabelului în instrucțiune.
- Alias-urile pot avea maxim **30 de caractere**, dar se recomandă să fie scurte și sugestive.

---

## 2. Tipuri de JOIN

| Tip | Descriere |
| :--- | :--- |
| **INNER JOIN** (equijoin) | Returnează doar liniile cu valori egale pe coloanele de join |
| **NONEQUIJOIN** | Condiția de join folosește alți operatori decât `=` |
| **LEFT OUTER JOIN** | Toate liniile din tabelul stâng + potrivirile din dreapta |
| **RIGHT OUTER JOIN** | Toate liniile din tabelul drept + potrivirile din stânga |
| **FULL OUTER JOIN** | LEFT OUTER JOIN + RIGHT OUTER JOIN |
| **SELF JOIN** | Un tabel unit cu el însuși |
| **CROSS JOIN** | Produsul cartezian (toate combinațiile posibile) |
| **NATURAL JOIN** | Equijoin automat pe toate coloanele cu același nume |

---

## 3. INNER JOIN (Equijoin) — trei variante echivalente

Returnează doar înregistrările care au **valori egale** pe coloanele de join în ambele tabele.

---

### 3.1. Varianta cu WHERE (sintaxă clasică Oracle)

```sql
SELECT E.EMPLOYEE_ID, D.DEPARTMENT_NAME
FROM EMPLOYEES E, DEPARTMENTS D
WHERE E.DEPARTMENT_ID = D.DEPARTMENT_ID;
```

---

### 3.2. Varianta cu JOIN ... ON (SQL3)

```sql
SELECT E.EMPLOYEE_ID, D.DEPARTMENT_NAME
FROM EMPLOYEES E
JOIN DEPARTMENTS D ON E.DEPARTMENT_ID = D.DEPARTMENT_ID;
```

> ⚠️ Când folosești `JOIN ... ON`, alias-urile pentru tabele sunt **obligatorii**.

---

### 3.3. Varianta cu JOIN ... USING (SQL3)

```sql
SELECT EMPLOYEE_ID, DEPARTMENT_NAME
FROM EMPLOYEES
JOIN DEPARTMENTS USING (DEPARTMENT_ID);
```

> ⚠️ Când folosești `USING`:
> - **Nu** se atribuie alias-uri tabelelor.
> - Coloana din `USING` trebuie să aibă **același nume** în ambele tabele.
> - Coloana din `USING` **nu se prefixează** cu alias sau nume de tabel în nicio apariție.

---

### 💡 Cele trei variante de mai sus sunt echivalente:

```
WHERE  E.DEPARTMENT_ID = D.DEPARTMENT_ID
ON     E.DEPARTMENT_ID = D.DEPARTMENT_ID    → alias-uri obligatorii
USING  (DEPARTMENT_ID)                       → fără alias-uri, fără prefix pe coloana de join
```

---

## 4. NONEQUIJOIN

Condiția de join folosește **alți operatori decât `=`** (ex: `>`, `<`, `BETWEEN` etc.).

```sql
-- Perechi de angajați unde primul a fost angajat DUPĂ al doilea
SELECT E.EMPLOYEE_ID, E2.EMPLOYEE_ID
FROM EMPLOYEES E
JOIN EMPLOYEES E2 ON E.HIRE_DATE > E2.HIRE_DATE;
```

> 💡 Acesta este și un exemplu de **SELF JOIN** — tabelul `EMPLOYEES` este unit cu el însuși folosind alias-uri diferite (`E` și `E2`).

---

## 5. OUTER JOIN

Folosit pentru a include în rezultat și **înregistrările fără corespondent** în celălalt tabel. Acolo unde nu există potrivire, coloanele apar ca `NULL`.

---

### 5.1. LEFT JOIN

Preia în plus și înregistrările din tabelul **stâng** care nu respectă condiția de join.

```sql
SELECT E.EMPLOYEE_ID, D.DEPARTMENT_NAME
FROM EMPLOYEES E
LEFT JOIN DEPARTMENTS D ON E.DEPARTMENT_ID = D.DEPARTMENT_ID;
```

> Rezultat: toți angajații, chiar și cei fără departament (`DEPARTMENT_NAME` va fi `NULL`).

---

### 5.2. RIGHT JOIN

Preia în plus și înregistrările din tabelul **drept** care nu respectă condiția de join.

```sql
SELECT E.EMPLOYEE_ID, D.DEPARTMENT_NAME
FROM EMPLOYEES E
RIGHT JOIN DEPARTMENTS D ON E.DEPARTMENT_ID = D.DEPARTMENT_ID;
```

> Rezultat: toate departamentele, chiar și cele fără angajați (`EMPLOYEE_ID` va fi `NULL`).

---

### 5.3. FULL OUTER JOIN

Preia în plus înregistrările din **ambele tabele** care nu respectă condiția de join.

```sql
SELECT E.EMPLOYEE_ID, D.DEPARTMENT_NAME
FROM EMPLOYEES E
FULL OUTER JOIN DEPARTMENTS D ON E.DEPARTMENT_ID = D.DEPARTMENT_ID;
```

> 💡 **FULL OUTER JOIN** = LEFT JOIN + RIGHT JOIN (fără duplicate).

---

### 5.4. Echivalența LEFT ↔ RIGHT

Cele două query-uri de mai jos returnează **același rezultat**:

```sql
-- Varianta 1: LEFT JOIN
SELECT E.EMPLOYEE_ID, D.DEPARTMENT_NAME
FROM EMPLOYEES E
LEFT JOIN DEPARTMENTS D ON E.DEPARTMENT_ID = D.DEPARTMENT_ID;

-- Varianta 2: RIGHT JOIN (cu tabelele inversate)
SELECT E.EMPLOYEE_ID, D.DEPARTMENT_NAME
FROM DEPARTMENTS D
RIGHT JOIN EMPLOYEES E ON E.DEPARTMENT_ID = D.DEPARTMENT_ID;
```

> 💡 LEFT și RIGHT JOIN sunt **oglinda** unul altuia — schimbând ordinea tabelelor și tipul de join, obții același rezultat.

---

### ⚠️ Restricții pentru operatorul clasic `(+)` (sintaxă veche Oracle):

```sql
-- Echivalent cu LEFT JOIN
WHERE E.DEPARTMENT_ID = D.DEPARTMENT_ID(+)

-- Echivalent cu RIGHT JOIN
WHERE E.DEPARTMENT_ID(+) = D.DEPARTMENT_ID
```

- Nu poate fi plasat **în ambele părți** simultan.
- Nu poate fi combinat cu operatorul **`IN`**.
- Nu poate fi legat de altă condiție prin operatorul **`OR`**.
- ❌ Nu are echivalent pentru **FULL OUTER JOIN**.

> ✅ Se recomandă utilizarea sintaxei SQL3 (`LEFT/RIGHT/FULL OUTER JOIN`) în locul operatorului `(+)`.

---

## 6. SELF JOIN

Un tabel unit **cu el însuși**. Alias-urile sunt **obligatorii** pentru a distinge cele două instanțe.

```sql
-- Fiecare angajat și numele managerului său
SELECT E.FIRST_NAME AS ANGAJAT, M.FIRST_NAME AS MANAGER
FROM EMPLOYEES E
JOIN EMPLOYEES M ON E.MANAGER_ID = M.EMPLOYEE_ID;
```

---

## 7. NATURAL JOIN

Efectuează automat un equijoin pe **toate coloanele cu același nume** din cele două tabele.

```sql
SELECT *
FROM EMPLOYEES
NATURAL JOIN DEPARTMENTS;
```

> ⚠️ Coloanele comune **nu trebuie prefixate** cu alias sau nume de tabel.  
> ⚠️ Dacă tipurile de date ale coloanelor cu același nume sunt **diferite**, se returnează eroare.  
> ⚠️ `NATURAL JOIN` și `USING` **nu pot coexista** în aceeași instrucțiune.

---

## 8. CROSS JOIN — Produs Cartezian

Returnează **toate combinațiile posibile** de linii din cele două tabele. Nu are condiție de join.

```sql
SELECT E.FIRST_NAME, D.DEPARTMENT_NAME
FROM EMPLOYEES E
CROSS JOIN DEPARTMENTS D;
```

> ⚠️ Dacă `EMPLOYEES` are 107 linii și `DEPARTMENTS` are 27, rezultatul va fi **107 × 27 = 2.889 linii**.

---

## 9. JOIN pe mai multe tabele

```sql
-- Angajați, departamentul lor și locația departamentului
SELECT E.FIRST_NAME, D.DEPARTMENT_NAME, L.CITY
FROM EMPLOYEES E
JOIN DEPARTMENTS D ON E.DEPARTMENT_ID = D.DEPARTMENT_ID
JOIN LOCATIONS L   ON D.LOCATION_ID   = L.LOCATION_ID;
```

> 💡 Pentru **n tabele** → cel puțin **n – 1 condiții de join**.

---

## 10. Comparație: WHERE vs. ON vs. USING

| | `WHERE` (clasic) | `JOIN ... ON` | `JOIN ... USING` |
| :--- | :--- | :--- | :--- |
| **Alias-uri obligatorii** | Nu | ✅ Da | ❌ Nu |
| **Prefix pe coloana de join** | Da | Da | ❌ Nu |
| **Coloane cu nume diferit** | ✅ | ✅ | ❌ (trebuie același nume) |
| **Separare join / filtrare** | ❌ (amestecate) | ✅ (ON + WHERE) | ✅ |
| **FULL OUTER JOIN** | ❌ (cu `(+)` nu merge) | ✅ | ✅ |

---

## 11. Rezumat Rapid

```
INNER JOIN     → doar liniile cu potrivire în ambele tabele
LEFT JOIN      → toate din stânga + potrivirile din dreapta (restul NULL)
RIGHT JOIN     → toate din dreapta + potrivirile din stânga (restul NULL)
FULL OUTER     → toate liniile din ambele tabele
SELF JOIN      → tabelul unit cu el însuși (alias-uri obligatorii)
CROSS JOIN     → produs cartezian (toate combinațiile, fără condiție)
NATURAL JOIN   → equijoin automat pe toate coloanele cu același nume
ON             → condiție de join explicită (alias-uri obligatorii)
USING          → equijoin pe coloana specificată (fără prefix, fără alias)

LEFT JOIN (A, B) ≡ RIGHT JOIN (B, A)
```

---

## 12. Capcane Frecvente ⚠️

| Situație | Problemă | Soluție |
| :--- | :--- | :--- |
| `JOIN ... ON` fără alias | Eroare sau ambiguitate | Alias-urile sunt **obligatorii** cu `ON` |
| `USING` cu prefix pe coloana de join | Eroare Oracle | Nu prefixezi coloana din `USING` **niciodată** |
| `(+)` pe ambele părți | Eroare Oracle | `(+)` merge doar pe **o singură parte** |
| `(+)` cu `IN` sau `OR` | Eroare Oracle | Folosește `LEFT/RIGHT OUTER JOIN` |
| `NATURAL JOIN` tipuri diferite | Eroare | Folosește `JOIN ... USING` sau `JOIN ... ON` |
| `NATURAL JOIN` + `USING` simultan | Eroare | Nu pot coexista în aceeași instrucțiune |
| Produs cartezian accidental | Uitarea condiției de join | Verifică să ai cel puțin **n-1 condiții** pentru n tabele |
| Confuzie LEFT ↔ RIGHT | Rezultate greșite | `LEFT JOIN(A,B)` = `RIGHT JOIN(B,A)` — schimbi ordinea tabelelor |
