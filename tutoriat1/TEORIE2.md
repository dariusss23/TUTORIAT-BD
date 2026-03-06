# 🗄️ Gestiunea Bazelor de Date - Concepte, Relații & Diagrame

---

## 1. Modelul Relațional — Concepte de Bază

O **bază de date relațională** organizează datele în **tabele (relații)**, formate din rânduri și coloane.

### Terminologie:

| Termen Matematic | Termen Practic | Descriere |
| :--- | :--- | :--- |
| **Relație** | Tabel | Structura care stochează datele |
| **Tuplu** | Rând / Înregistrare | O instanță a datelor |
| **Atribut** | Coloană / Câmp | O proprietate a entității |
| **Domeniu** | Tip de date | Mulțimea valorilor posibile ale unui atribut |
| **Grad** | Număr de coloane | Câte atribute are relația |
| **Cardinalitate** | Număr de rânduri | Câte tupluri are relația la un moment dat |

---

## 2. Chei în Modelul Relațional

Cheile identifică în mod unic tuplurile dintr-o relație și stabilesc legăturile dintre tabele.

---

### 2.1. Cheie Candidată (Candidate Key)

Un set **minimal** de atribute care identifică unic fiecare tuplu.

> Exemplu: În tabelul `EMPLOYEES`, atât `EMPLOYEE_ID` cât și `EMAIL` pot fi chei candidate (ambele sunt unice).

---

### 2.2. Cheie Primară (Primary Key — PK)

Cheia candidată **aleasă** ca identificator principal al tabelului.

- ❌ Nu poate conține valori `NULL`
- ❌ Nu poate conține valori duplicate
- ✅ Fiecare tabel are **o singură** cheie primară

```
EMPLOYEES(EMPLOYEE_ID*, FIRST_NAME, LAST_NAME, EMAIL, SALARY)
              ^
           PRIMARY KEY
```

---

### 2.3. Cheie Externă (Foreign Key — FK)

Un atribut (sau set de atribute) dintr-un tabel care **referențiază cheia primară** a altui tabel.

- ✅ Poate conține valori `NULL` (dacă relația este opțională)
- ✅ Poate repeta valori
- ⚠️ Valoarea trebuie să **existe** în tabelul referențiat (sau să fie `NULL`)

```
EMPLOYEES.DEPARTMENT_ID → DEPARTMENTS.DEPARTMENT_ID
              ^                          ^
           Foreign Key               Primary Key
```

---

### 2.4. Cheie Supercheie (Superkey)

Orice mulțime de atribute care identifică unic tuplurile — **nu neapărat minimală**.

> Orice cheie candidată este o supercheie, dar nu orice supercheie este o cheie candidată.

---

### 2.5. Cheie Artificială (Surrogate Key)

O cheie primară **generată artificial** (ex: `ID` auto-increment), fără semnificație în lumea reală.

> Folosită când nu există un identificator natural bun.

---

## 3. Integritatea Datelor

Constrângerile de integritate asigură că datele rămân **corecte și consistente**.

---

### 3.1. Integritatea entității

Cheia primară **nu poate fi NULL** — fiecare rând trebuie să fie identificabil.

---

### 3.2. Integritatea referențială

O cheie externă trebuie să facă referire la o valoare care **există** în tabelul referențiat.

```
EMPLOYEES.DEPARTMENT_ID = 99
→ DEPARTMENTS trebuie să aibă un rând cu DEPARTMENT_ID = 99
```

---

### 3.3. Integritatea domeniului

Valorile unui atribut trebuie să respecte **tipul de date și restricțiile** definite.

> Exemplu: `SALARY` nu poate fi negativ; `EMAIL` trebuie să conțină `@`.

---

### 3.4. Integritatea definită de utilizator

Reguli de business specifice aplicației.

> Exemplu: Un angajat nu poate avea salariul mai mare decât managerul său.

---

## 4. Diagrama Entitate-Relație (ERD)

**ERD (Entity-Relationship Diagram)** este o reprezentare vizuală a structurii bazei de date: entități, atribute și relațiile dintre ele.

---

### 4.1. Entitate

Reprezintă un **obiect din lumea reală** despre care stocăm informații.

```
┌─────────────┐
│  EMPLOYEES  │
└─────────────┘
```

- **Entitate tare**: Există independent (ex: `EMPLOYEES`, `DEPARTMENTS`)
- **Entitate slabă**: Depinde de o altă entitate pentru identificare (ex: `DEPENDENTS` față de `EMPLOYEES`)

---

### 4.2. Atribute

Proprietățile unei entități:

| Tip Atribut | Descriere | Exemplu |
| :--- | :--- | :--- |
| **Simplu** | Valoare atomică, indivizibilă | `SALARY`, `AGE` |
| **Compus** | Format din mai multe sub-atribute | `ADDRESS` (strada, oraș, cod) |
| **Derivat** | Calculat din alte atribute | `AGE` din `BIRTH_DATE` |
| **Multivaluat** | Poate avea mai multe valori | `PHONE_NUMBERS` |
| **Cheie** | Identifică unic entitatea | `EMPLOYEE_ID` |

---

### 4.3. Relații între Entități

O **relație** descrie asocierea dintre două sau mai multe entități.

```
EMPLOYEES ────────── lucrează_în ────────── DEPARTMENTS
```

---

## 5. Cardinalitatea Relațiilor

Cardinalitatea specifică **câte instanțe** ale unei entități pot fi asociate cu instanțele altei entități.

---

### 5.1. Unu-la-Unu (1:1)

O înregistrare din A este asociată cu **cel mult una** din B, și viceversa.

```
EMPLOYEES (1) ──────── are ──────── (1) PASSPORT
```

> Exemplu: Un angajat are un singur pașaport; un pașaport aparține unui singur angajat.

---

### 5.2. Unu-la-Mulți (1:N)

O înregistrare din A este asociată cu **mai multe** din B, dar fiecare B aparține unui singur A.

```
DEPARTMENTS (1) ──────── conține ──────── (N) EMPLOYEES
```

> Exemplu: Un departament are mulți angajați; un angajat aparține unui singur departament.

---

### 5.3. Mulți-la-Mulți (M:N)

O înregistrare din A poate fi asociată cu **mai multe** din B, și invers.

```
EMPLOYEES (M) ──────── participă_la ──────── (N) PROJECTS
```

> ⚠️ În modelul relațional, o relație M:N se implementează printr-un **tabel de legătură** (junction table):

```
EMPLOYEES ──── EMPLOYEE_PROJECTS ──── PROJECTS
   (PK)      (FK_emp | FK_proj)         (PK)
```

---

## 6. Notații ERD

Există mai multe stiluri de notație pentru ERD:

---

### 6.1. Notația Chen (clasică)

```
[EMPLOYEES] ──<lucrează_în>── [DEPARTMENTS]
     │                               │
  atribute                        atribute
```

- **Dreptunghiuri** → Entități
- **Elipse** → Atribute
- **Romburi** → Relații
- **Linii** → Conexiuni

---

### 6.2. Notația Crow's Foot (modernă, folosită în DataGrip, Lucidchart etc.)

```
DEPARTMENTS ||──o{ EMPLOYEES
```

| Simbol | Semnificație |
| :---: | :--- |
| `\|\|` | Exact unul (obligatoriu) |
| `\|o` | Zero sau unul (opțional) |
| `{` sau `}` | Mai mulți (many) |
| `o{` | Zero sau mai mulți |
| `\|{` | Unul sau mai mulți |

**Exemplu complet:**

```
DEPARTMENTS ||──o{ EMPLOYEES : "angajează"
EMPLOYEES   }o──o{ PROJECTS  : "participă la"
```

---

## 7. Normalizarea Bazelor de Date

**Normalizarea** este procesul de organizare a datelor pentru a reduce redundanța și a îmbunătăți integritatea.

---

### 7.1. Dependența Funcțională

Atributul B este **dependent funcțional** de A dacă, pentru orice valoare a lui A, există o singură valoare a lui B.

```
A → B
EMPLOYEE_ID → FIRST_NAME  (ID-ul determină numele)
```

---

### 7.2. Forma Normală 1 (1NF)

**Condiții:**
- Toate atributele sunt **atomice** (indivizibile)
- Nu există grupuri repetitive
- Fiecare coloană conține valori de același tip

❌ **Înainte de 1NF:**
```
| EMP_ID | PHONES                    |
|--------|---------------------------|
| 101    | 0721111111, 0722222222    |
```

✅ **După 1NF:**
```
| EMP_ID | PHONE      |
|--------|------------|
| 101    | 0721111111 |
| 101    | 0722222222 |
```

---

### 7.3. Forma Normală 2 (2NF)

**Condiții:**
- Este în **1NF**
- Fiecare atribut non-cheie depinde **complet** de întreaga cheie primară (nu de o parte din ea)

> ⚠️ Relevant doar când cheia primară este **compusă**.

❌ **Problemă (dependență parțială):**
```
(ORDER_ID, PRODUCT_ID) → PRODUCT_NAME
                ^
         depinde doar de PRODUCT_ID, nu de toată cheia
```

✅ **Soluție:** Mută `PRODUCT_NAME` într-un tabel separat `PRODUCTS`.

---

### 7.4. Forma Normală 3 (3NF)

**Condiții:**
- Este în **2NF**
- Nu există **dependențe tranzitive** (un atribut non-cheie nu depinde de alt atribut non-cheie)

❌ **Problemă (dependență tranzitivă):**
```
EMPLOYEE_ID → DEPARTMENT_ID → DEPARTMENT_NAME
                                    ^
                    depinde tranzitiv de EMPLOYEE_ID
```

✅ **Soluție:** Mută `DEPARTMENT_NAME` în tabelul `DEPARTMENTS`.

---

### 7.5. Forma Normală Boyce-Codd (BCNF)

**Condiții:**
- Este în **3NF**
- Pentru orice dependență funcțională `A → B`, A trebuie să fie o **supercheie**

> BCNF este o versiune mai strictă a 3NF, rezolvând anomalii care pot apărea în cazuri rare cu mai multe chei candidate.

---

### Rezumat Normalizare:

```
1NF → date atomice, fără repetiții
  ↓
2NF → eliminare dependențe parțiale
  ↓
3NF → eliminare dependențe tranzitive
  ↓
BCNF → orice determinant este supercheie
```

---

## 8. Anomalii de Actualizare

Redundanța datelor provoacă **anomalii** atunci când datele sunt modificate:

| Anomalie | Descriere | Exemplu |
| :--- | :--- | :--- |
| **Inserare** | Nu poți adăuga date fără date conexe | Nu poți adăuga un departament fără angajați |
| **Ștergere** | Ștergerea unui rând pierde și alte informații | Ștergând ultimul angajat, pierzi și departamentul |
| **Actualizare** | Trebuie modificate mai multe rânduri | Schimbarea numelui departamentului în toate rândurile angajaților |

> ✅ **Normalizarea elimină aceste anomalii.**

---

## 9. Tipuri de Relații Speciale

---

### 9.1. Relație Reflexivă (Auto-relație)

O entitate se asociază **cu ea însăși**.

```
EMPLOYEES ──── supervizează ──── EMPLOYEES
(MANAGER_ID referențiază EMPLOYEE_ID din același tabel)
```

---

### 9.2. Entitate Asociativă (Tabel de Legătură)

Rezolvă relațiile M:N și poate conține atribute proprii.

```
┌──────────┐    ┌──────────────────┐    ┌──────────┐
│ EMPLOYEES│────│ EMPLOYEE_PROJECTS│────│ PROJECTS │
│          │    │  - START_DATE    │    │          │
│          │    │  - ROLE          │    │          │
└──────────┘    └──────────────────┘    └──────────┘
```

---

### 9.3. Ierarhie (Relație IS-A)

O entitate este un **subtip** al alteia (moștenire).

```
         PERSON
        /      \
   EMPLOYEE   CUSTOMER
   /      \
MANAGER  CLERK
```

---

## 10. Diagrama Relațională (Schema Logică)

Schema logică descrie structura tabelelor și relațiile dintre ele, fără detalii fizice de implementare.

**Convenție de notație:**

```
TABEL(atribut1*, atribut2, atribut3#)
         ^                    ^
     Primary Key          Foreign Key
```

---

### Exemplu: Schema HR

```
REGIONS(REGION_ID*, REGION_NAME)

COUNTRIES(COUNTRY_ID*, COUNTRY_NAME, REGION_ID#)
                                          ↑
                                    ref. REGIONS

LOCATIONS(LOCATION_ID*, STREET_ADDRESS, CITY, COUNTRY_ID#)
                                                   ↑
                                             ref. COUNTRIES

DEPARTMENTS(DEPARTMENT_ID*, DEPARTMENT_NAME, MANAGER_ID#, LOCATION_ID#)
                                                  ↑              ↑
                                            ref. EMPLOYEES   ref. LOCATIONS

JOBS(JOB_ID*, JOB_TITLE, MIN_SALARY, MAX_SALARY)

EMPLOYEES(EMPLOYEE_ID*, FIRST_NAME, LAST_NAME, EMAIL, SALARY,
          JOB_ID#, DEPARTMENT_ID#, MANAGER_ID#)
              ↑            ↑             ↑
           ref. JOBS   ref. DEPTS   ref. EMPLOYEES (auto-relație)

JOB_HISTORY(EMPLOYEE_ID#, START_DATE*, JOB_ID#, DEPARTMENT_ID#)
```

---

## 11. Vizualizare ERD — Schema HR

```
REGIONS
  ↑ (1:N)
COUNTRIES
  ↑ (1:N)
LOCATIONS
  ↑ (1:N)
DEPARTMENTS ←────────────── EMPLOYEES (MANAGER_ID, auto-relație)
  ↑ (1:N)                      ↑
EMPLOYEES ────────────────── JOBS
  ↑ (1:N)                      ↑
JOB_HISTORY ─────────────────┘
```

---

## 12. Diferența dintre Modelele de Date

| Model | Descriere | Exemplu SGBD |
| :--- | :--- | :--- |
| **Relațional** | Date organizate în tabele cu relații | Oracle, MySQL, PostgreSQL |
| **Ierarhic** | Date organizate ca arbore (părinte-copil) | IBM IMS |
| **Rețea** | Extensie a ierarhicului, permite mai mulți părinți | IDMS |
| **Obiect-relațional** | Combină OOP cu relațional | PostgreSQL, Oracle |
| **NoSQL** | Fără schemă fixă (document, key-value, graf) | MongoDB, Redis, Neo4j |

---

## 13. Concepte Avansate: Vederi și Indecși

---

### 13.1. Vedere (View)

O **vedere** este o interogare stocată care se comportă ca un tabel virtual.

- ✅ Simplifică interogările complexe
- ✅ Restricționează accesul la date sensibile
- ✅ Nu stochează date fizic (de obicei)

```
VIEW: EMPLOYEES_PUBLIC
  → Afișează doar FIRST_NAME, JOB_ID, DEPARTMENT_ID
  → Ascunde SALARY, EMAIL etc.
```

---

### 13.2. Index

Un **index** este o structură de date auxiliară care **accelerează căutările**.

- ✅ Îmbunătățește performanța `SELECT`-urilor pe coloane indexate
- ⚠️ Încetinește `INSERT`, `UPDATE`, `DELETE` (indexul trebuie actualizat)
- ✅ Creat automat pe **cheia primară**

```
Index pe EMPLOYEES.LAST_NAME
→ Căutarea după nume devine mult mai rapidă
```

---

## 14. Tranzacții și Proprietăți ACID

O **tranzacție** este o unitate logică de lucru, formată din una sau mai multe operații.

### Proprietăți ACID:

| Proprietate | Descriere |
| :--- | :--- |
| **Atomicitate** | Toate operațiile se execută sau niciuna (totul sau nimic) |
| **Consistență** | Baza de date trece dintr-o stare validă în alta validă |
| **Izolare** | Tranzacțiile concurente nu interferează una cu alta |
| **Durabilitate** | Odată confirmată (COMMIT), modificarea este permanentă |

```
START TRANSACTION
  → Operație 1: Scade 1000 RON din contul A
  → Operație 2: Adaugă 1000 RON în contul B
COMMIT  ← ambele reușesc, sau ROLLBACK ← niciuna nu se aplică
```

---

## 15. Glosar Rapid

| Termen | Definiție |
| :--- | :--- |
| **Entitate** | Obiect din lumea reală reprezentat în baza de date |
| **Atribut** | Proprietate a unei entități |
| **Tuplu** | Un rând dintr-un tabel |
| **Relație** | Tabel în modelul relațional |
| **Cheie primară (PK)** | Identificator unic al unui rând |
| **Cheie externă (FK)** | Referință la PK-ul altui tabel |
| **ERD** | Diagramă Entitate-Relație |
| **Normalizare** | Proces de eliminare a redundanței |
| **1NF / 2NF / 3NF** | Forme normale de organizare a datelor |
| **ACID** | Proprietățile unei tranzacții corecte |
| **Cardinalitate** | Tipul relației: 1:1, 1:N, M:N |
| **Integritate referențială** | Regulă care asigură consistența FK → PK |
| **View** | Tabel virtual bazat pe o interogare |
| **Index** | Structură auxiliară pentru accelerarea căutărilor |
