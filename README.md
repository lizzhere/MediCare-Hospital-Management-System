# MediCare: Hospital Patient Management System (SQL Project)

A relational database project in **MySQL** that models the day-to-day operations of a hospital: patients, doctors, appointments, prescriptions and billing. It covers the full process, from design and normalization to table creation, sample data and 21 analytical queries.

- **Database name:** `MediCare`
- **RDBMS:** MySQL
- **Subject:** SQL / Database Management Systems

---

## Table of Contents

1. [Project Objective](#project-objective)
2. [Concepts Applied](#concepts-applied)
3. [Development Process](#development-process)
4. [Entity Relationship Diagram](#entity-relationship-diagram)
5. [Database Schema](#database-schema)
6. [Sample Data](#sample-data)
7. [Queries Solved](#queries-solved)
8. [How to Run](#how-to-run)
9. [Possible Improvements](#possible-improvements)

---

## Project Objective

Give front-desk staff, physicians and the accounts team one consistent source of data to:

- register patients and doctors,
- schedule appointments,
- record prescribed medicines,
- generate bills and track payments,
- produce management reports (revenue by specialization, top-spending patients, payment-method trends, etc.).

---

## Concepts Applied

| Concept | How it is used in this project |
|---|---|
| **Normalization (up to 3NF)** | Patient and doctor details are stored once and referenced by ID from other tables, so there is no duplicated data. |
| **Primary keys** | Every table has a unique, non-null ID. |
| **Foreign keys** | Link Appointments to Patients and Doctors, and Prescriptions and Billing to Appointments. |
| **Relationships** | 1:M and 1:1, as described in the ER section below. |
| **Joins** | Used to combine patients, doctors, appointments and billing in the reports. |
| **Aggregation** | `SUM`, `AVG`, `COUNT`, `GROUP BY`, `HAVING`. |
| **Subqueries** | Used in the average-bill and "doctors with no appointments" queries. |
| **Window functions** | Cumulative daily revenue using `SUM() OVER (ORDER BY ...)`. |
| **Indexes, views, transactions and ACID** | Studied and documented in the project report as supporting theory. |

---

## Development Process

1. **Requirement analysis.** Identified what a hospital must track: who the patients are, which doctor they see, when, what is prescribed and what is billed.
2. **Conceptual design.** Defined five entities and their attributes, then worked out the relationships between them.
3. **Normalization.** Split the data into five tables so every non-key attribute depends only on its table's key. A surrogate key (`Prescription_ID`) lets one appointment have several prescription lines.
4. **ER diagram.** Drew the diagram to show entities, keys and cardinalities before writing any SQL.
5. **Database creation.** Created the `MediCare` database and the five tables in dependency order (parents before children) so the foreign keys resolve.
6. **Data population.** Inserted realistic sample data and verified each table with `SELECT *`.
7. **Querying and reporting.** Wrote 21 queries of increasing complexity, from simple filters to joins, subqueries and window functions, and checked the output of each.

---

## Entity Relationship Diagram

| Relationship | Type | Meaning |
|---|---|---|
| Patients → Appointments | 1:M | One patient can book many appointments. |
| Doctors → Appointments | 1:M | One doctor can attend many appointments. |
| Appointments → Prescriptions | 1:M | One appointment can result in several prescribed medicines. |
| Appointments → Billing | 1:1 | Each appointment generates exactly one bill. |

```
Patients ──< Appointments >── Doctors
                 │
        ┌────────┴────────┐
   Prescriptions        Billing
      (1:M)              (1:1)
```

The full diagram images are in the project report (`SQL_Project_Hospital_Patient_Management_System.docx`).

---

## Database Schema

| Table | Columns |
|---|---|
| **Patients** | `Patient_ID` (PK), `Name`, `Age`, `Gender`, `Contact`, `Email`, `Address` |
| **Doctors** | `Doctor_ID` (PK), `Name`, `Specialization`, `Fee`, `Experience_Years` |
| **Appointments** | `Appointment_ID` (PK), `Patient_ID` (FK), `Doctor_ID` (FK), `Appointment_Date`, `Status` |
| **Prescriptions** | `Prescription_ID` (PK), `Appointment_ID` (FK), `Medicine`, `Dosage`, `Duration_Days` |
| **Billing** | `Bill_ID` (PK), `Appointment_ID` (FK), `Amount`, `Payment_Method`, `Payment_Status` |

Example DDL:

```sql
CREATE DATABASE MediCare;
USE MediCare;

CREATE TABLE Patients (
    Patient_ID INT PRIMARY KEY,
    Name VARCHAR(100),
    Age INT,
    Gender VARCHAR(10),
    Contact VARCHAR(15),
    Email VARCHAR(100),
    Address VARCHAR(100)
);

CREATE TABLE Appointments (
    Appointment_ID INT PRIMARY KEY,
    Patient_ID INT,
    Doctor_ID INT,
    Appointment_Date DATE,
    Status VARCHAR(20),
    FOREIGN KEY (Patient_ID) REFERENCES Patients(Patient_ID),
    FOREIGN KEY (Doctor_ID)  REFERENCES Doctors(Doctor_ID)
);
```

---

## Sample Data

| Table | Rows | Notes |
|---|---|---|
| Patients | 20 | Patients from cities across India |
| Doctors | 15 | 15 specializations, from General Medicine to Oncology |
| Appointments | 10 | January 2024, all `Completed` |
| Prescriptions | 13 | Some appointments have 2 medicines |
| Billing | 10 | Payment methods: Credit Card, Debit Card, Cash, UPI |

---

## Queries Solved

| # | Task | Techniques |
|---|---|---|
| 1 | Appointments scheduled after `2024-01-08` | `WHERE` |
| 2 | Payments made via Credit Card | `WHERE` |
| 3 | Top 5 doctors by consultation fee | `ORDER BY`, `LIMIT` |
| 4 | 3 most recent appointments | `ORDER BY`, `LIMIT` |
| 5 | Medicines prescribed to Rohit Sharma | Multi-table `JOIN` |
| 6 | Bills above the average amount | Subquery |
| 7 | Appointments booked per patient | `GROUP BY`, `COUNT` |
| 8 | Average fee per specialization | `GROUP BY`, `AVG` |
| 9 | Number of completed appointments | `COUNT` |
| 10 | Patient who spent the most | `JOIN`, `SUM`, `LIMIT 1` |
| 11 | Doctors with no appointments | `NOT IN` subquery |
| 12 | Cumulative revenue per day | Window function |
| 13 | Revenue by payment method | `GROUP BY` |
| 14 | Total billed per patient | `JOIN`, `GROUP BY` |
| 15 | Appointments with more than one medicine | `HAVING` |
| 16 | Patients who paid via UPI | `JOIN`, `DISTINCT` |
| 17 | Average billing amount per month | `MONTH()`, `AVG` |
| 18 | Highest-revenue specialization | 3-table `JOIN`, `LIMIT 1` |
| 19 | Top 3 patients by total billing | `JOIN`, `SUM`, `LIMIT 3` |
| 20 | Most-used payment method | `COUNT`, `LIMIT 1` |
| 21 | Total billing per specialization | 3-table `JOIN`, `GROUP BY` |

Example (Q18):

```sql
SELECT d.Specialization, SUM(b.Amount) AS Total_Revenue
FROM Doctors d
JOIN Appointments a ON d.Doctor_ID = a.Doctor_ID
JOIN Billing b      ON a.Appointment_ID = b.Appointment_ID
GROUP BY d.Specialization
ORDER BY Total_Revenue DESC
LIMIT 1;
```

---

## How to Run

1. Install MySQL 8.0 or later (window functions need 8.0+).
2. Open the MySQL client (or Workbench).
3. Run, in order:
   - `CREATE DATABASE MediCare; USE MediCare;`
   - the `CREATE TABLE` statements: Patients, Doctors, Appointments, Prescriptions, Billing
   - the `INSERT` statements for each table
4. Run the queries from the Queries Solved section and compare with the outputs in the report.

---

## Possible Improvements

- Add `NOT NULL`, `UNIQUE` (e.g. on `Email`) and `CHECK` constraints.
- Add `UNIQUE` on `Billing.Appointment_ID` to enforce the 1:1 relationship.
- Use `AUTO_INCREMENT` for IDs.
- Add indexes on `Appointments(Patient_ID, Doctor_ID)` and `Appointment_Date`.
- Create views, for example a patient billing summary.
- Wrap booking and billing in transactions (`START TRANSACTION` / `COMMIT` / `ROLLBACK`).
- Add more appointments with varied statuses (Scheduled, Cancelled) and `Pending` payments.

