# ticket-management - Screens & Flow

## Customer Flow

1. **Login Screen**  
   - Authenticate the user (Customer)  
   - Redirect to Customer Dashboard after login  

2. **Customer Dashboard**  
   - Show all tickets raised by the logged-in customer  
   - Quick overview of ticket status (Open, In Progress, Resolved, etc.)  
   - Navigate to **My Tickets** or **Create Ticket**  

3. **My Tickets**  
   - List all tickets created by the customer  
   - Filter or sort tickets by status, priority, or creation date  
   - Select a ticket to view details  

4. **Ticket Details**  
   - Show full ticket information including:  
     - Description  
     - Status, Priority, Category  
     - Comments / history  

5. **Create Ticket**  
   - Raise a new support ticket  
   - Provide **Category, Priority, Description**  
   - Ticket is created with **status = Open** and **assigned_to = NULL**  
 

---

## Employee Flow

1. **Login Screen**  
   - Authenticate the user (Technician, Supervisor, or Admin)  
   - Redirect to Employee Dashboard after login  

2. **Dashboard**  
   - Show summary of tickets assigned to the logged-in employee  
   - Navigate to My Tickets  

3. **My Tickets**  
   - List all tickets assigned to the employee  
   - Filter or sort tickets by priority or creation date  
   - Select a ticket to view details  

4. **Ticket Details**  
   - Show full ticket information including customer details  
   - Actions available:  
     - **Update Ticket Status** → Change progress (In Progress, Resolved, Closed)  
     - **Add Comment** → Log activity or updates  
     - **Cancel Ticket** → Cancel ticket with reason  

5. **Update Ticket Status**  
   - Change the status of a ticket to reflect progress  

6. **Add Comment**  
   - Add notes or activity updates to the ticket  

7. **Cancel Ticket**  
   - Cancel the ticket and record the reason for cancellation  
