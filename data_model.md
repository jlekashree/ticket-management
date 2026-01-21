# Ticket Management System – Data Modelling

## Fact Tables

### 1. FACT_TICKETS
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
| location         | VARCHAR   | Issue location |
| created_at       | TIMESTAMP | Creation time |
| updated_at       | TIMESTAMP | Last update time |

**Stores the current state of all tickets.**

A customer can log a ticket in two ways: either by raising it themselves or by contacting the support team, who will create the ticket on their behalf.

---

### 2. FACT_TICKET_COMMENTS
| Column Name        | Data Type  | Description |
|--------------------|-----------|-------------|
| comment_id     | INT    | Unique comment |
| ticket_id     | INT    | Ticket reference |
| commented_by   | INT    | User who commented |
| comment_text       | VARCHAR      | Comment content |
| created_at         | TIMESTAMP | Comment time |

**Tracks all interactions on tickets for maintaining history.**

---

## Dimension Tables

### 4. DIM_USERS
| Column Name   | Data Type | Description |
|--------------|-----------|-------------|
| user_id (PK) | INT    | Unique user ID |
| name         | VARCHAR   | User name |
| role         | VARCHAR   | Technician / Supervisor / Admin |
| phone        | VARCHAR   | Contact number |
| email        | VARCHAR   | Email address |
| is_active    | VARCHAR   | Active flag |

**Stores all user information for assignments and actions.**

---

### 5. DIM_CUSTOMERS
| Column Name        | Data Type | Description |
|-------------------|-----------|-------------|
| customer_id (PK)  | INT    | Customer ID |
| customer_name     | VARCHAR   | Customer name |
| address           | TEXT      | Address |
| phone             | VARCHAR   | Contact |

**Provides customer details linked to tickets.**

---

### 6. DIM_TICKET_STATUS
| Column Name    | Data Type | Description |
|---------------|-----------|-------------|
| status_id (PK)| INT       | Status ID |
| status_name   | VARCHAR   | Open / Assigned / In Progress / Resolved / Closed / Cancelled |

**Defines ticket lifecycle states.**

---

### 7. DIM_TICKET_PRIORITY
| Column Name        | Data Type | Description |
|-------------------|-----------|-------------|
| priority_id (PK)  | INT       | Priority ID |
| priority_name     | VARCHAR   | Low / Medium / High / Critical |

 **Specifies ticket urgency to prioritize work.** 

---

### 8. DIM_TICKET_CATEGORY
| Column Name        | Data Type | Description |
|-------------------|-----------|-------------|
| category_id (PK)  | INT       | Category ID |
| category_name     | VARCHAR   | Network / Hardware / Software / Installation |

**Categorizes type of issues for routing and reporting.**
