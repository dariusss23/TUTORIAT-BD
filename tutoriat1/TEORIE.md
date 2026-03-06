# 🗄️ Gestiunea Bazelor de Date - Concepte și SQL

Acest repository conține fundamentele teoretice și practice pentru lucrul cu baze de date relaționale și limbajul SQL.

---

## 📑 Cuprins
1. [Introducere în Bazele de Date](#1-introducere)
2. [Sisteme de Gestiune (SGBD)](#2-sgbd)
3. [Limbajul SQL și Categorii de Instrucțiuni](#3-sql)
4. [Analiza Sintaxei SELECT](#4-sintaxa-select)
5. [Depanare Cod SQL (Studiu de caz)](#5-depanare)

---

## 1. Introducere
O **bază de date** este un ansamblu structurat de date coerente, organizat fără redundanță inutilă. Aceasta este concepută pentru a fi accesată în mod concurent de către mai mulți utilizatori, asigurând integritatea informației.



## 2. Ce este un SGBD?
Un **Sistem de Gestiune a Bazelor de Date (SGBD)** este produsul software care permite interacțiunea cu datele. 
**Funcții principale:**
* **Definirea** structurii datelor.
* **Consultarea** (extragerea) informațiilor.
* **Actualizarea** și întreținerea consistenței datelor.

---

## 3. Limbajul SQL
**SQL (Structured Query Language)** este limbajul standard neprocedural utilizat pentru comunicarea cu bazele de date relaționale.

### Categorii de instrucțiuni:
Instrucțiunile sunt grupate în sub-limbaje specifice:

| Limbaj | Acronim | Comenzi | Descriere |
| :--- | :--- | :--- | :--- |
| **Definire (LDD)** | DDL | `CREATE`, `ALTER`, `DROP` | Modifică structura tabelelor. |
| **Prelucrare (LMD)** | DML | `INSERT`, `UPDATE`, `DELETE`, `SELECT` | Gestionează datele din tabele. |
| **Control (LCD)** | TCL | `COMMIT`, `ROLLBACK` | Gestionează tranzacțiile și salvarea. |

> **Notă:** SQL include și instrucțiuni pentru controlul sesiunii, al sistemului și instrucțiuni încapsulate.

---

## 4. Analiza Sintaxei SELECT

Sintaxa generală a comenzii `SELECT` urmează această structură:

```sql
SELECT { [ {DISTINCT | UNIQUE} | ALL] lista_campuri | *} 
FROM [nume_schemă.]nume_obiect 
[, [nume_schemă.]nume_obiect ...] 
[WHERE condiție_clauza_where] 
[START WITH condiție_clauza_start_with
 CONNECT BY condiție_clauza_connect_by] 
[GROUP BY expresie [, expresie ...] 
[HAVING condiție_clauza_having] 
[ORDER BY {expresie | poziţie} [, {expresie | poziţie} ...] ]; 
