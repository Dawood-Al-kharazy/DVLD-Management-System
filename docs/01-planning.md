# Phase 1: Project Planning & Architectural Baseline

## 1. Project Overview
* **Project Name:** Driving & Vehicle License Department (DVLD) Management System
* **System Type:** Enterprise Desktop Application
* **Domain:** Government Registry, Citizen Licensing, Testing, and Compliance Management
* **Primary Engineering Objectives:**
  * Architect and implement a production-grade, decoupled desktop system following industry software engineering standards.
  * Achieve high throughput, low memory overhead, and sub-second responsiveness across large datasets (hundreds of thousands of records and concurrent business processes).
  * Establish a clean, maintainable codebase suitable for long-term scalability and professional portfolio demonstration.

---

## 2. Technology Stack & Architectural Pattern

### 2.1 Technology Stack
| Layer / Component | Technology | Role |
| :--- | :--- | :--- |
| **Presentation Layer (UI)** | C# / .NET Windows Forms | Desktop user interface and user input validation |
| **Business Logic Layer (BLL)** | C# / .NET Framework | Core domain models, business rules, and workflow orchestration |
| **Data Access Layer (DAL)** | ADO.NET | Low-level, high-control database communication and data mapping |
| **Database Engine** | Microsoft SQL Server | Relational storage, indexing, constraints, and transactional integrity |
| **Version Control** | Git & GitHub | Source control, feature branching, and release tracking |

### 2.2 Architectural Style: Strict 3-Tier (N-Tier) Architecture
The system enforces strict separation of concerns across three isolated layers:
1. **Presentation Layer (UI):** Responsible solely for rendering screens and capturing user input. Contains **zero** SQL statements and **zero** core business rules. Communicates exclusively with the BLL.
2. **Business Logic Layer (BLL):** Acts as the core engine of the application. Encapsulates domain entities, business validations, state transitions, and authorization checks. Communicates exclusively with the DAL.
3. **Data Access Layer (DAL):** Handles all SQL Server interactions via ADO.NET. Exposes clean CRUD and query methods to the BLL without leaking database-specific objects (`SqlDataReader`, `DataTable`, etc.) upward where avoidable.

> **Architectural Decoupling Goal:** The BLL and DAL must remain completely independent of Windows Forms so that the presentation layer could be replaced (e.g., with a Web API or modern desktop UI) without rewriting business logic or database access.

---

## 3. Software Development Life Cycle (SDLC) Methodology

This project adopts an **Architecture-First Iterative Hybrid** methodology tailored for a solo engineer building a complex, data-centric system with defined requirements:

### Stage A: Waterfall Foundation (Upfront Engineering)
Before writing application code, the structural foundation is fully designed and verified:
1. **Planning:** Define scope, technical constraints, risk mitigations, and Git workflow.
2. **Requirements Engineering & Analysis:** Extract all functional requirements, business rules, edge cases, and entity relationships.
3. **Database & System Architecture:** Design a normalized SQL Server schema (3NF), establish primary/foreign keys, define indexing strategies, and lock down the 3-Tier contracts.

### Stage B: Iterative Construction (Vertical Slicing)
Once the database schema is locked, implementation proceeds in **vertical slices** rather than building an entire layer in isolation:
* Each functional module (e.g., *People Management*, *User Authentication*, *Local Driving License Applications*, *Tests & Appointments*, *License Issuance & Detentions*) is built end-to-end across **SQL -> DAL -> BLL -> UI**.
* Every slice is tested, validated against edge cases, and merged before starting the next slice, ensuring early architectural validation and continuous working software.

---

## 4. Non-Functional Requirements & Quality Targets

* **Scalability & Performance:** Designed to handle hundreds of thousands of citizen, application, test, and license records efficiently through proper relational normalization, targeted SQL indexing, and selective column querying (avoiding `SELECT *` in production paths).
* **Memory Efficiency:** Strict deterministic resource cleanup using `using` blocks for all unmanaged ADO.NET resources (`SqlConnection`, `SqlCommand`, `SqlDataReader`) alongside SQL Server connection pooling.
* **UI Responsiveness:** All I/O-bound database operations execute asynchronously (`async` / `await`) to prevent blocking the Windows Forms UI thread.
* **Auditability & Data Integrity:** Because licensing records constitute legal history, the database enforces strict referential integrity, user action attribution (`CreatedByUserID` and timestamps), and soft-deactivation/archiving rather than destructive deletion of historical records.
* **Security:** Mandatory use of parameterized SQL commands across all DAL methods to prevent SQL injection, paired with secure password hashing for system users.

---

## 5. Risk Management & Mitigation Plan

| Risk | Impact | Mitigation Strategy |
| :--- | :--- | :--- |
| **UI Thread Freezing** during heavy queries or reports | High (Poor UX, unresponsive app) | Enforce asynchronous (`async`/`await`) database calls in the DAL/BLL and UI event handlers. |
| **Memory Leaks & Connection Pool Exhaustion** | Critical (App crash under load) | Wrap every `SqlConnection`, `SqlCommand`, and `SqlDataReader` in `using` blocks to guarantee immediate disposal. |
| **SQL Injection & Data Corruption** | Critical (Security breach, invalid records) | Use parameterized queries exclusively; wrap multi-step financial/licensing operations in SQL Transactions (`SqlTransaction`). |
| **Tight Coupling ("Spaghetti Code")** | High (Hard to debug, scale, or maintain) | Enforce strict project references: `UI -> BLL -> DAL`. Never reference DAL directly from UI. |

---

## 6. Version Control & Repository Governance

* **Repository Hygiene:** Configured with a standard Visual Studio C# `.gitignore` to exclude build artifacts (`bin/`, `obj/`), IDE cache (`.vs/`), and local environment secrets/connection strings.
* **Branching Strategy (Feature Branch Workflow):**
  * `main`: Protected baseline branch containing only tested, stable, working code.
  * `feature/<slice-name>`: Dedicated branch created for each vertical slice or architectural milestone (e.g., `feature/db-schema`, `feature/people-management`), merged into `main` upon completion and verification.
* **Conventional Commit Standard:**
  * `docs:` Documentation, ERD diagrams, and architectural specifications
  * `db:` Database schema scripts, constraints, indexes, and stored procedures
  * `feat:` New vertical slice features (DAL, BLL, UI)
  * `fix:` Bug fixes and validation corrections
  * `refactor:` Structural code improvements without altering external behavior

---

## 7. Project Roadmap & Milestones

1. **Milestone 1: Planning & Environment Setup** *(Completed)* — SDLC selection, risk analysis, Git standards, and architectural baseline.
2. **Milestone 2: Requirements Engineering & Analysis** *(Next)* — Domain breakdown, use cases, business rules, and entity identification.
3. **Milestone 3: Database Architecture** — Conceptual ERD, 3NF relational schema, constraints, indexing plan, and SQL Server implementation.
4. **Milestone 4: Core Infrastructure Setup** — Visual Studio multi-project solution structure (`DVLD.UI`, `DVLD.BLL`, `DVLD.DAL`), configuration management, and base data access utilities.
5. **Milestone 5: Iterative Vertical Slice Development** — End-to-end implementation and testing of each functional module.
6. **Milestone 6: Final QA, Performance Tuning & Documentation** — End-to-end integration testing, query profiling, and final README/architecture polish.