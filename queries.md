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
SELECT user_id, role
FROM users
WHERE email = :email
  AND is_active = TRUE;
~~~

**Reason:** Validates user existence and role-based access.

**Action Query:** N/A  

**On-click Action:** Navigate to Dashboard on successful authentication.

---

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
    p.priority_name
FROM tickets t
JOIN customers c
  ON t.customer_id = c.customer_id
JOIN ticket_status s
  ON t.status_id = s.status_id
JOIN ticket_priority p
  ON t.priority_id = p.priority_id
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

~~~sql
UPDATE tickets
SET status_id = :status_id,
    updated_at = CURRENT_TIMESTAMP
WHERE ticket_id = :ticket_id;
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

---

## Additional Notes
- Role-based access control (Agent vs Customer)
- Closed or Cancelled tickets restrict further updates
- Ticket comments act as a complete audit trail
- Filters such as status, priority, and date range can be applied in ticket listing screens
