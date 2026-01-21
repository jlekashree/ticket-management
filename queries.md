# Ticket Management System – Screen-wise SQL Queries

* **Fact Tables:** `tickets`, `ticket_comments`
* **Dimension Tables:** `users`, `customers`, `ticket_status`, `ticket_priority`, `ticket_category`

---

## 1. Login Screen
**Purpose:** Authenticate the user and determine authorization level.

* **Data Query:**
    ```sql
    SELECT user_id, role
    FROM users
    WHERE email = :email
      AND is_active = TRUE;
    ```
* **Reason:** Validates user existence.
* **Action Query:** N/A
* **On-click Action:** **Navigate to Dashboard** on successful authentication.

---

## 2. Dashboard
**Purpose:** Provide a high-level overview of tickets assigned to the logged-in user.

* **Data Query:**
    ```sql
    SELECT
        s.status_name,
        COUNT(*) AS ticket_count
    FROM tickets t
    JOIN ticket_status s
      ON t.status_id = s.status_id
    WHERE t.assigned_to = :user_id
    GROUP BY s.status_name
    ORDER BY ticket_count DESC;
    ```
* **Reason:** Displays ticket distribution by status to help users prioritize work.
* **Action Query:** N/A
* **On-click Action:** **Navigate to My Tickets** (filtered by selected status).

---

## 3. My Tickets
**Purpose:** List all tickets assigned to the logged-in user.

* **Data Query:**
    ```sql
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
    ```
* **Reason:** Shows actionable tickets sorted by priority and age.
* **Action Query:** N/A
* **On-click Action:** **Navigate to Ticket Details** upon selection.

---

## 4. Ticket Details
**Purpose:** Display complete information about a selected ticket.

* **Data Query:**
    ```sql
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
    ```
* **Reason:** Enables agents to understand the issue and customer context fully.
* **On-click Actions:**
    * **Update Status** → Navigate to Update Ticket Status
    * **Add Comment** → Open Add Comment modal
    * **Cancel Ticket** → Navigate to Cancel Ticket screen

---

## 5. Update Ticket Status
**Purpose:** Allow agents to update the ticket lifecycle stage.

* **Data Query:**
    ```sql
    SELECT status_id, status_name
    FROM ticket_status
    ORDER BY status_id;
    ```
* **Action Query:**
    ```sql
    UPDATE tickets
    SET status_id = :status_id,
        updated_at = CURRENT_TIMESTAMP
    WHERE ticket_id = :ticket_id;
    ```
* **On-click Action:** **Return to Ticket Details** with updated status.

---

## 6. Add Comment
**Purpose:** Capture discussion and audit trail for a ticket.

* **Data Query:** N/A (User inputs text)
* **Action Query:**
    ```sql
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
    ```
* **On-click Action:** **Submit & Refresh** Ticket Details view.

---

## 7. Cancel Ticket
**Purpose:** Cancel a ticket with a reason and audit log.

* **Data Query:**
    ```sql
    SELECT ticket_id, status_id
    FROM tickets
    WHERE ticket_id = :ticket_id;
    ```
* **Action Queries (Transactional):**
    ```sql
    -- Update status to 'Cancelled' (Assumes ID 6)
    UPDATE tickets
    SET status_id = 6,
        updated_at = CURRENT_TIMESTAMP
    WHERE ticket_id = :ticket_id;

    -- Record cancellation reason
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
    ```
* **On-click Action:** **Return to My Tickets** list.

---

## 8. Create New Ticket
**Purpose:** Allow users to raise a new support ticket.

* **Data Query:** N/A (UI Form)
* **Action Query:**
    ```sql
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
        1, -- Status: Open
        :user_id,
        CURRENT_TIMESTAMP,
        :description
    );
    ```
* **On-click Action:** **Navigate to My Tickets** and highlight the new entry.

-
