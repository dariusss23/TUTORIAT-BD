# 📑 Ședința 01: Introducere în Baze de Date Relaționale

Bun venit la primul tutoriat! Astăzi punem bazele teoretice pentru a înțelege cum funcționează "creierul" oricărei aplicații moderne: **Baza de Date**.

---

## 🧊 1. Ce este o Bază de Date (BD)?
O bază de date este o colecție organizată de date, structurată astfel încât să permită regăsirea, inserarea și modificarea rapidă a informațiilor.

* **Fără BD:** Datele sunt împrăștiate (fișiere Excel diferite), se repetă (redundanță) și sunt greu de interconectat.
* **Cu BD:** Totul este centralizat și guvernat de un **SGBD** (Sistem de Gestiune a Bazelor de Date - ex: Oracle, MySQL, PostgreSQL).

---

## 🏛️ 2. Arhitectura pe 3 Niveluri (Modelul ANSI/SPARC)
Există o separare clară între utilizator și datele propriu-zise:
1.  **Nivelul Extern (Vedere):** Ce vede utilizatorul final (ex: interfața Facebook).
2.  **Nivelul Conceptual (Logic):** Structura tabelelor și relațiile dintre ele. **Aici lucrăm noi!**
3.  **Nivelul Intern (Fizic):** Cum sunt salvați biții pe hard-disk.

[Image of ANSI-SPARC three-level architecture diagram]

---

## 🗂️ 3. Modelul Relațional (Tabelele)
În bazele de date relaționale, datele sunt stocate în **Tabele**.

| Termen Relational | Termen Comun | Explicație |
| :--- | :--- | :--- |
| **Relatie** | Tabel | Structura principală. |
| **Tuplu** | Rând / Înregistrare | O singură instanță (ex: Studentul "Popescu"). |
| **Atribut** | Coloană / Câmp | O proprietate (ex: "Email", "Data_Nasterii"). |
| **Domeniu** | Tip de date | Ce fel de valori permitem (ex: INTEGER, VARCHAR, DATE). |

---

## 🔑 4. Constrângeri de Integritate
Fără aceste reguli, datele devin corupte. Cele mai importante sunt:

### 🆔 Cheia Primară (Primary Key - PK)
* Identifică **unic** fiecare rând dintr-un tabel.
* **Reguli:** Nu poate fi NULL și nu poate avea duplicate.
* *Exemplu:* `id_student`, `cod_produs`.

### 🔗 Cheia Externă (Foreign Key - FK)
* Este coloana care face legătura cu
