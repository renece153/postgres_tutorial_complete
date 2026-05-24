# Relational Databases & Why We Use PostgreSQL

## What Is a Relational Database?

A relational database is a structured system that organizes data into tables connected by relationships, enabling efficient storage, retrieval, and manipulation through SQL. Although vendors like PostgreSQL, MySQL, SQL Server, and Oracle differ in implementation details such as how they handle storage engines, indexing, or transaction management, they all follow a consistent logical architecture.

At its core, every relational database is built around a hierarchy that defines how data is organized and accessed:

**Database → Schema → Database Objects (Tables, Views, Functions, Procedures, etc.)**

The **Database** acts as the top‑level container, holding all data and metadata.  
Within it, **Schemas** serve as logical namespaces that group related objects, providing structure and access control.  
Inside each schema are **Database Objects**, which include the actual data structures (tables, views, indexes) and logic (functions, triggers, stored procedures).

This layered organization ensures consistency across vendors. For example:

- PostgreSQL uses clusters and schemas.  
- MySQL simplifies the model by omitting schemas but still groups tables under databases.  
- SQL Server uses servers and schemas.  

Despite these variations, the conceptual flow remains identical: data is always organized hierarchically from the database level down to individual objects.

Understanding this common structure makes it easier to switch between systems and design portable, well‑organized data models across different relational platforms.

---

## Core Components of an RDBMS

| Component | Description |
|------------|-------------|
| **Database Engine** | Executes SQL, manages memory, and coordinates reads/writes |
| **Storage Layer** | Physical files on disk where data is stored |
| **Schemas** | Logical containers for tables, views, functions |
| **Transaction Manager** | Ensures atomicity, consistency, isolation, durability |
| **Query Optimizer** | Decides the most efficient execution plan |
| **Background Services** | Vacuuming, logging, checkpoints, caching |

A relational database is built around a central component—the **database engine**—and every other subsystem exists to support, inform, or extend what the engine does.  
The engine interprets SQL, manages memory, coordinates reads and writes, and ensures that every operation follows the rules of the system.

It interacts with several tightly connected components:

- Communicates with the **storage layer** to retrieve and persist data.  
- Uses **schemas** as the logical map of the database.  
- Wraps every operation with the **transaction manager** to enforce ACID guarantees.  
- Consults the **query optimizer** to evaluate execution paths.  
- Coordinates **background services** that maintain performance and integrity.

Together, these components form a unified system where the database engine acts as the conductor that coordinates every part to deliver predictable and reliable relational behavior across vendors like PostgreSQL, MySQL, and SQL Server.

---

## What Is ACID?

The **ACID** property describes the four guarantees that every reliable relational database must uphold. These ensure that data remains correct, consistent, and durable even when multiple users, failures, or complex operations are involved.

- **Atomicity** — A transaction is treated as a single unit of work; it either completes fully or has no effect.  
- **Consistency** — Every transaction moves the database from one valid state to another, respecting constraints and rules.  
- **Isolation** — Controls how concurrent transactions interact; each behaves as if running alone.  
- **Durability** — Once committed, changes are permanent even after a crash.

Together, these properties make relational databases predictable and trustworthy. Whether using PostgreSQL, MySQL, or SQL Server, ACID ensures that data remains accurate, consistent, and safe under heavy load or failure.

---

## Differences Between PostgreSQL, MySQL, and SQL Server

| Feature | PostgreSQL | MySQL | SQL Server |
|----------|-------------|--------|------------|
| **SQL Compliance** | High | Medium | High |
| **Extensibility** | Excellent | Limited | Medium |
| **Licensing** | Free | Free (GPL) | Paid |
| **Best For** | Modern apps, analytics, APIs | Simple CRUD, legacy stacks | Enterprise systems |

**PostgreSQL** — Object‑relational, designed for correctness, extensibility, and standards compliance. Ideal for modern applications, analytics, and API‑driven systems.  
**MySQL** — Built for simplicity and speed, especially web workloads. Backbone of countless PHP/LAMP applications.  
**SQL Server** — Enterprise‑focused, tightly integrated with Windows ecosystem, strong tooling (SSMS, SSIS, SSRS).

Despite differences, all share the same logical structure:

**Database → Schema → Tables and Objects**

Cross‑database access varies:

- PostgreSQL: **Foreign Data Wrappers (FDWs)** for querying external sources.  
- SQL Server: **Linked Servers** for cross‑server queries.  
- MySQL: Limited native cross‑database 
