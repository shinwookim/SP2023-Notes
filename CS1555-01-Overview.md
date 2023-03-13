# Overview of Database Management Systems
## Data vs Information
Given a set of bits, how do we know what it represents? $$\text{bits} + \text{type} = \text{(primitive) data}$$
If we add **semantics** to the data, we get **information**: $$\text{data}+\text{semantics}=\text{information}$$
From this information, we can get **knowledge**, by adding a **context model**:$$\text{information}+\text{context model} = \text{knowledge}$$
How knowledge can be represented in this way is a key question for **data science**.
## What is a database?
A **database** is a very large, integrated collection of *related* data
- **Data** is raw facts on some aspect of the world
- Database *models* a real-world enterprise (e.g., a university)
	- Consists of **Entities** (e.g., students, courses)
	- And **Relationships** (e.g., Bob took CS 1550)

**Integrated data** is all data that is stored and manipulated in a *uniform way* on a secondary storage
- Databases store large amounts of data that cannot fit in main memory (such as RAM)
- Data is stored for long and indefinite periods
- Data is shared across multiple applications

## What is a database management system?
A **Database Management System (DBMS)** is a general purpose software package designed to store and manage databases *conveniently and efficiently*.
- Some common examples are: Oracle, IBM DB2, SQLServer, MySQL, PostgreSQL, and so many more
The database, together with the DBMS and application logic is called a **database system**.
$$\text{Database system}=\text{DB}+\text{DBMS}+\text{Application logic}$$
- Application logic include resource planning applications (such as PeopleSoft), web=based applications (such as E-Bay and Google), and etcetera.
![center](Database%20System.png)
- We create a database system to model the real world in order to solve a problem!
There are two main *approaches* to management of data:
1. The **file system approach** utilizes traditional (flat) files and (C, Java, ...) programs to access them 
	- Our applications simply write to text files
	- However, this is often slow, and we must constantly enforce the layout of data (enforce all programs follow our layout)
	- Harder for multiple instances of applications to access the same data, and has more disastrous consequences if an application crashes.
2. The **database approach** aims to overcome the drawbacks of the file system approach by providing:
	1. **Abstraction**
	2. **Reliability** (High availability: short recovery time and trusted/quality data)
	3. **Efficiency/Performance**
		- High throughput: committed transactions per unit time
		- Short of bounded response time
		- Energy Efficiency

## Abstraction
### Data Abstraction
Data is structured in a way meaningful to applications
- A **data model** is a collection of high-level data description constructs that hide low-level storage details
	- E.g., Relational, Object-oriented, XML, Graphs
- The **Relational/Object-Relational Model** is the most widely used data model today.
	- In it, the main construct is a **relation** (represented by a table of records) which each has a **schema** (relation name, field names, field types)

Suppose a database example:
| SID    | Name  | Age | GPA  |
| ------ | ----- | --- | ---- |
| 546007 | Susan | 18  | 3.8  |
| 546100 | Bob   | 19  | 3.65 |
| 546500 | Bill  | 20  | 3.7  |
- Each row is called a **record** or **tuple**
- Each column is called the **field** or **attribute**
* The header row which defined all the fields is called the **schema**
	* There are various ways to define the schema for the database above:
		* `STUDENTS(sid: string, name: string, age: integer, gpa: real)`
		* `STUDENTS(sid: integer, fname: string, lname:string dob: date, gpa: real)`
		* Here, the second approach may be more appropriate because:
			* Having `sid` be an integer allows use to sort, etc. more easily.
			* Separating the first and last name allows use to conduct more operations more easily (and we can always stitch them back if we need to).
			* Having the age as a `dob date` means we don't have to worry about updating them every year.
		* Hence, it is important to create a good schema!

The data is a DBMS is described at three levels of abstraction
1. **Conceptual Schema** (DDL: Data definition language) 
2. **Physical Schema** (SDL: Storage definition language)
3. **External Schema** (VDL: View definition language)
	- Allows the data access to be customized and authorized at the user-level
		- Defined in terms of data model
		- Consists of a collection of **views** (E.g., `CourseEnroll(cid: string, enrollment: int)`)
		- It is guided by end-user requirements and computed as needed.
		- Multiple views of data allows each user/application to get different perspective of the database
![center](data-abstractions.png)
The 3-level architecture:
1. **View level** (`CSMajors`, `MathMajors`)
2. **Logical level**: entire database schema
	```
	Courses (CourseNo,CourseName,Credits,Dept)
	Students (StudentID,Lname,Name,Class,Major)
	GradeReport (StudentID,CourseNo,Grade,Term)	
	```
3. **Physical level**: How are these tables are stored? How many bytes and attribute, etc?
### Execution Abstraction
A **transaction** is a **logical unit of work** in a DBMS
- It is the execution of a **program segment** that performs some function or task by accessing shared data (such as a database)
- Logical grouping of query and update requests needed to perform a task
Examples include:
- Deposit, withdrawal, transfer money (in a bank)
- Reserving seats on a flight
- Printing monthly payment checks (business transaction)
- Updating inventory (inventory transaction)

#### ACID Properties
1. **Atomicity** (alias failure atomicity)  
	- Either all the operations associated with a transaction happen or none of them happens  
2. **Consistency Preservation**  
	- A transaction is a correct program segment. It satisfies the integrity constraints on the database at the transaction's  boundaries  
3. **Isolation** (alias concurrency atomicity / serializability) 
	- Transactions are independent, the result of the execution of concurrent transactions is the same as if transactions were executed serially, one after the other  
4. **Durability** (alias persistence / permanence)  
	- The effects of completed transactions become permanent surviving any subsequent failures

Start --> All operations --> Commit
OR Start --> Some operations --> Rollback

NoSQL databases have **BaSE** property:
1. **Basic Availability**: The database appears to work most of the time.  
2. **Soft-state**: Stores don’t have to be write-consistent, nor do different replicas have to be mutually consistent all the time.  
3. **Eventual consistency**: Stores exhibit consistency at some later point (e.g., lazily at read time).

Derived from Cap theorem:
Relationals are from CA side; NoSQL are from the AP side
![](Pasted%20image%2020230307124218.png)


## Reliability
Reliability means enforcing **integrity constraints**  
- E.g., data type, relationship between values
- Data in DB *must satisfy* the integrity constraints  
- Transactions are committed only if they do not violate any integrity constraint  
- Integrity constraints are stored in the catalog  
Ensuring data integrity despite failures  
- Data is not lost when the system or a transaction fails for whatever reason
Ensuring data integrity
!Security
§Encryption & Private Information Retrieval
§Authentication
§Data Domains, Compartmentalization
!Access control
§Who (user/role), what (data), how (operations)
§Views and access permissions in the catalog

Performance/Efficiency