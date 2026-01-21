# Ticket Management System – Data Modelling

## Fact Tables

### 1. TICKETS
| Column Name      | Data Type  | Description |
|------------------|-----------|-------------|
| ticket_id(PK)    | INT       | Unique ticket identifier |
| customer_id  | INT       | customer ID |
| category_id  | INT       | Issue type |
| priority_id  | INT       | Ticket priority |
| status_id    | INT       | Current ticket status |
| created_by   | INT    | Who created the ticket |
| assigned_to | INT    | Who is currently assigned |
| description      | VARCHAR      | Issue description |
| created_at       | TIMESTAMP | Creation time |
| updated_at       | TIMESTAMP | Last update time |

**Stores the current state of all tickets.**

~~~sql
CREATE TABLE tickets (
    ticket_id INT PRIMARY KEY,
    customer_id INT,
    category_id INT,
    priority_id INT,
    status_id INT,
    created_by INT,
    assigned_to INT,
    description VARCHAR,
    created_at TIMESTAMP,
    updated_at TIMESTAMP,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
    FOREIGN KEY (status_id) REFERENCES ticket_status(status_id),
    FOREIGN KEY (priority_id) REFERENCES ticket_priority(priority_id)
);
~~~

---

### 2. TICKET_COMMENTS
| Column Name        | Data Type  | Description |
|--------------------|-----------|-------------|
| comment_id     | INT    | Unique comment |
| ticket_id     | INT    | Ticket reference |
| commented_by   | INT    | User who commented |
| comment_text       | VARCHAR      | Comment content |
| created_at         | TIMESTAMP | Comment time |

**Tracks all interactions on tickets for maintaining history.**
~~~sql
CREATE TABLE ticket_comments (
    comment_id INT PRIMARY KEY,
    ticket_id INT,
    commented_by INT,
    comment_text VARCHAR,
    created_at TIMESTAMP,
    FOREIGN KEY (ticket_id) REFERENCES tickets(ticket_id)
);
~~~
---

## Dimension Tables

### 4. USERS
| Column Name   | Data Type | Description |
|--------------|-----------|-------------|
| user_id (PK) | INT    | Unique user ID |
| name         | VARCHAR   | User name |
| role         | VARCHAR   | Technician / Supervisor / Admin |
| phone        | VARCHAR   | Contact number |
| email        | VARCHAR   | Email address |
| is_active    | VARCHAR   | Active flag |

**Stores all user information for assignments and actions.**
~~~sql
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(150) UNIQUE,
    role VARCHAR(50), -- [Agent / Customer / Admin]
    is_active BOOLEAN,
    created_at TIMESTAMP
);
~~~
---

### 5. CUSTOMERS
| Column Name        | Data Type | Description |
|-------------------|-----------|-------------|
| customer_id (PK)  | INT    | Customer ID |
| customer_name     | VARCHAR   | Customer name |
| address           | VARCHAR      | Address |
| phone             | VARCHAR   | Contact |

**Provides customer details linked to tickets.**
~~~sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(150),
    address VARCHAR,
    phone VARCHAR(20)
);
~~~

---

### 6. TICKET_STATUS
| Column Name    | Data Type | Description |
|---------------|-----------|-------------|
| status_id (PK)| INT       | Status ID |
| status_name   | VARCHAR   | Open / Assigned / In Progress / Resolved / Closed / Cancelled |

**Defines ticket lifecycle states.**
~~~sql
CREATE TABLE ticket_status (
    status_id INT PRIMARY KEY,
    status_name VARCHAR(50)
);
~~~

---

### 7. TICKET_PRIORITY
| Column Name        | Data Type | Description |
|-------------------|-----------|-------------|
| priority_id (PK)  | INT       | Priority ID |
| priority_name     | VARCHAR   | Low / Medium / High / Critical |

 **Specifies ticket urgency to prioritize work.**
 ~~~sql
CREATE TABLE ticket_priority (
    priority_id INT PRIMARY KEY,
    priority_name VARCHAR(50)
);
~~~

---

### 8. TICKET_CATEGORY
| Column Name        | Data Type | Description |
|-------------------|-----------|-------------|
| category_id (PK)  | INT       | Category ID |
| category_name     | VARCHAR   | Network / Hardware / Software / Installation |

**Categorizes type of issues for routing and reporting.**

~~~sql
CREATE TABLE ticket_category (
    category_id INT PRIMARY KEY,
    category_name VARCHAR(100)
);
~~~




















