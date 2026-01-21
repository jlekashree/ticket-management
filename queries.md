# Ticket Management System

## Screen-wise Application Flow

### 1. Login Screen
**Purpose:** Authenticate the user and determine authorization level.

**Data Required:**
- User ID  
- Role  
- Account status  

**Data Query:**

~~~sql
SELECT user_id, role,customer_id
FROM users
WHERE email = :email
  AND password_hash = :password_hash
  AND is_active = TRUE;
~~~

**Reason:** Validates user existence and role-based access.

**Action Query:** N/A  

**On-click Action:** Navigate to the appropriate dashboard based on user role:

Customer → Customer Dashboard

Support/Admin → Support Dashboard

---
### CUSTOMERS
### 2. Customer Dashboard
**Purpose:** Display all tickets raised by the logged-in customer and provide quick access to ticket details.

**Data Required:**
- Ticket ID
- Category Name
- Status Name
- Priority Name
- Created At

**Data Query:**

~~~sql
SELECT t.ticket_id,
       tc.category_name,
       ts.status_name,
       tp.priority_name,
       t.created_at
FROM tickets t
JOIN ticket_category tc ON t.category_id = tc.category_id
JOIN ticket_status ts ON t.status_id = ts.status_id
JOIN ticket_priority tp ON t.priority_id = tp.priority_id
WHERE t.customer_id = :customer_id
ORDER BY t.created_at DESC;

~~~

**Reason:** Ensures that only tickets belonging to the logged-in customer are displayed and provides necessary details for overview.

**Action Query:** N/A  

**On-click Action:** 
- Click on a ticket → Navigate to Ticket Details Screen
- Click on “New Ticket” → Navigate to Create Ticket Screen

---
### 3. Create Ticket Screen
**Purpose:** Allow the customer to raise a new ticket with required details.

**Data Required:**
- Customer ID (from session)
- Category ID
- Priority ID
- Description

**Data Query:** N/A
**Action Query:**
~~~sql
INSERT INTO tickets (
    ticket_id,
    customer_id,
    category_id,
    priority_id,
    status_id,
    created_by,
    description,
    created_at
)
VALUES (
    :ticket_id,
    :customer_id,
    :category_id,
    :priority_id, --[open by default]
    :1,
    :user_id,            
    :description,       
    NOW()
);

~~~

**Reason:** Creates a new ticket linked to the customer, sets the default status as “Open”, and tracks who created it.   

**On-click Action:** 
Submit → Insert ticket into database and Show success message.

---
### 4. Ticket Details Screen
**Purpose:** Display full details of a ticket on clicking the ticket.

**Data Required:**
- Ticket ID
- Description
- Status Name
- Priority Name
- Category Name
- Comments

**Data Query:**

~~~sql
SELECT t.ticket_id,
       t.description,
       ts.status_name,
       tp.priority_name,
       tc.category_name,
       c.comment_id,
       c.commented_by,
       c.comment_text,
       c.created_at
FROM tickets t
JOIN ticket_status ts ON t.status_id = ts.status_id
JOIN ticket_priority tp ON t.priority_id = tp.priority_id
JOIN ticket_category tc ON t.category_id = tc.category_id
LEFT JOIN ticket_comments c ON t.ticket_id = c.ticket_id
WHERE t.ticket_id = :ticket_id
  AND t.customer_id = :customer_id;
~~~

**Reason:** Ensures only the owner of the ticket can view it; provides full ticket history.

**Action Query:** N/A  

**On-click Action:** Click “Add Comment” → Navigate to Add Comment Screen

---
### Employee
### 2. Dashboard
**Purpose:** Display a high-level overview of tickets assigned to the logged-in user.

**Data Required:**
- Ticket status  
- Ticket count per status  

**Data Query:**

~~~sql
SELECT
    s.status_name,
    COUNT(*) AS ticket_count
FROM tickets t
JOIN ticket_status s
  ON t.status_id = s.status_id
WHERE t.assigned_to = :user_id
AND t.status_id != 6
GROUP BY s.status_name
ORDER BY ticket_count DESC;
~~~

**Reason:** Helps users quickly assess workload and prioritize tasks.

**Action Query:** N/A  

**On-click Action:** Selecting a status navigates to *My Tickets* filtered by that status.

---

### 3. My Tickets
**Purpose:** Display all tickets assigned to the logged-in user.

**Data Required:**
- Ticket ID  
- Customer name  
- Priority  
- Status  
- Created date  

**Data Query:**

~~~sql
SELECT
    t.ticket_id,
    c.customer_name,
    p.priority_name,
    s.status_name,
    t.created_at
FROM tickets t
JOIN customers c
  ON t.customer_id = c.customer_id
JOIN ticket_priority p
  ON t.priority_id = p.priority_id
JOIN ticket_status s
  ON t.status_id = s.status_id
WHERE t.assigned_to = :user_id
ORDER BY p.priority_id DESC, t.created_at ASC;
~~~

**Reason:** Displays actionable tickets sorted by urgency and age.

**Action Query:** N/A  

**On-click Action:** Selecting a ticket navigates to the Ticket Details screen.

---

### 4. Ticket Details
**Purpose:** Display complete information about a selected ticket.

**Data Required:**
- Ticket description  
- Customer details  
- Status and priority  
- Timestamps  

**Data Query:**

~~~sql
SELECT
    t.ticket_id,
    t.description,
    t.created_at,
    t.updated_at,
    c.customer_name,
    c.address,
    s.status_name,
    p.priority_name,
    cmt.comment_id,
    cmt.commented_by,
    cmt.comment_text,
    cmt.created_at
FROM tickets t
JOIN customers c
  ON t.customer_id = c.customer_id
JOIN ticket_status s
  ON t.status_id = s.status_id
JOIN ticket_priority p
  ON t.priority_id = p.priority_id
LEFT JOIN ticket_comments cmt
  ON t.ticket_id = c.ticket_id
WHERE t.ticket_id = :ticket_id;
~~~

**Reason:** Provides full context to understand and resolve the issue.

**On-click Actions:**
- Update Status → Update Ticket Status screen  
- Add Comment → Add Comment modal  
- Cancel Ticket → Cancel Ticket screen  

---

### 5. Update Ticket Status
**Purpose:** Allow agents to update the ticket lifecycle stage.

**Data Required:**
- Available statuses  

**Data Query:**

~~~sql
SELECT status_id, status_name
FROM ticket_status
ORDER BY status_id;
~~~

**Action Query:**
**update status:**
~~~sql
UPDATE tickets
SET status_id = :status_id,
    updated_at = CURRENT_TIMESTAMP
WHERE ticket_id = :ticket_id;
~~~
**Insert Closed ticket to history:**
~~~sql
INSERT INTO ticket_history (
    ticket_id,
    customer_id,
    category_id,
    priority_id,
    status_id,
    created_by,
    assigned_to,
    description,
    created_at,
    closed_at
)
SELECT
    ticket_id,
    customer_id,
    category_id,
    priority_id,
    status_id,
    created_by,
    assigned_to,
    description,
    created_at,
    updated_at
FROM tickets
WHERE ticket_id = :ticket_id
  AND status_id = 5;
~~~
**Delete Closed ticket in tickets table:**
~~~sql
DELETE FROM tickets
WHERE ticket_id = :ticket_id
  AND status_id = 5;
~~~

**On-click Action:** Save changes and return to Ticket Details screen.

---

### 6. Add Comment
**Purpose:** Maintain communication and audit trail for a ticket.

**Data Required:**
- Comment text  

**Action Query:**

~~~sql
INSERT INTO ticket_comments (
    ticket_id,
    commented_by,
    comment_text,
    created_at
)
VALUES (
    :ticket_id,
    :user_id,
    :comment,
    CURRENT_TIMESTAMP
);
~~~

**On-click Action:** Submit and refresh Ticket Details view.

---

### 7. Cancel Ticket
**Purpose:** Cancel a ticket while preserving audit history.

**Data Required:**
- Ticket ID  
- Cancellation reason  

**Action Queries (Transactional):**

~~~sql
BEGIN;

UPDATE tickets
SET status_id = 6, -- Cancelled
    updated_at = CURRENT_TIMESTAMP
WHERE ticket_id = :ticket_id;

INSERT INTO ticket_comments (
    ticket_id,
    commented_by,
    comment_text,
    created_at
)
VALUES (
    :ticket_id,
    :user_id,
    :cancel_reason,
    CURRENT_TIMESTAMP
);

COMMIT;
~~~

**On-click Action:** Navigate back to My Tickets screen.

---

### 8. Create New Ticket
**Purpose:** Allow users to raise a new support ticket.

**Data Required:**
- Customer  
- Category  
- Priority  
- Description  

**Action Query:**

~~~sql
INSERT INTO tickets (
    customer_id,
    category_id,
    priority_id,
    status_id,
    created_by,
    created_at,
    description
)
VALUES (
    :customer_id,
    :category_id,
    :priority_id,
    1, -- Open
    :user_id,
    CURRENT_TIMESTAMP,
    :description
);
~~~

**On-click Action:** Navigate to My Tickets and highlight the newly created ticket.

