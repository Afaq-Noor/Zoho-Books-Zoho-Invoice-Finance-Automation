# Payment Reminder Automation - Zoho Books

## Project Overview

### Objective
Build an automated overdue payment reminder system in Zoho Books that sends email reminders to customers when invoices become 3, 7, 14, or 30 days overdue, ensuring each reminder is sent only once per invoice.

### Business Requirements

| Requirement | Description |
|-------------|-------------|
| Trigger | Invoice becomes overdue (due date passed and invoice not paid) |
| Reminder Intervals | 3 days, 7 days, 14 days, 30 days overdue |
| Delivery | Email sent to customer's email address |
| Content | Invoice number, amount, due date, days overdue, payment link |
| Duplication Prevention | Each reminder sent only once per invoice |
| Automation | Fully automated, no manual intervention required |

### Key Components

| Component | Count | Description |
|-----------|-------|-------------|
| Email Templates | 4 | 3-Day, 7-Day, 14-Day, 30-Day |
| Email Alerts | 4 | One for each template |
| Custom Fields | 5 | 4 tracking + 1 trigger |
| Workflow Rules | 4 | One for each reminder stage |
| Custom Function | 1 | `Send_Overdue_Reminder` |
| Scheduler | 1 | Daily execution at 9:00 AM |

### Success Criteria

- ✅ Invoice automatically triggers reminder emails when overdue
- ✅ Correct email sent for each interval (3, 7, 14, 30 days)
- ✅ No duplicate reminders for the same interval
- ✅ Tracking fields accurately reflect which reminders were sent
- ✅ Workflow logs show successful execution

---

## System Architecture

### Complete Process Flow

Invoice becomes overdue (due date passed)
│
▼
Daily Scheduler runs at 9:00 AM
│
▼
Custom Function: Send_Overdue_Reminder
│
▼
Function calculates days overdue
│
▼
Days Overdue ≥ 3?
│
┌───────┴───────┐
▼ ▼
Yes No
│ │
▼ ▼
Days ≥3 & <7? No Reminder
│
┌─────┴─────┐
▼ ▼
3-Day 7,14,30-Day
Reminder (calculated)
│
▼
Function checks tracking checkbox
│
┌─────┴─────┐
▼ ▼
Not Sent Already Sent
│ │
▼ ▼
Update: Skip (No Duplicate)

cf_reminder_trigger = "X-day"

Tracking checkbox = true
│
▼
Workflow Rule detects trigger
│
▼
Workflow sends Email Alert
│
▼
Workflow clears trigger field
│
▼
Customer receives reminder email
│
▼
Tracking checkbox stays checked
│
▼
Future executions skip the same reminder

text

### Email Content Strategy

| Overdue Days | Subject | Tone | Key Message |
|--------------|---------|------|-------------|
| 3 Days | "Reminder: Invoice #[Number] is 3 Days Overdue" | Friendly | "Your payment is 3 days overdue." |
| 7 Days | "Reminder: Invoice #[Number] is 7 Days Overdue" | Polite | "Your payment is now 7 days overdue." |
| 14 Days | "URGENT: Invoice #[Number] is 14 Days Overdue" | Urgent | "Payment is 14 days overdue. Clear immediately." |
| 30 Days | "FINAL NOTICE: Invoice #[Number] is 30 Days Overdue" | Final | "Final notice. Immediate action required." |

---

## Pre-Implementation Checklist

### Prerequisites

| Requirement | Status | Notes |
|-------------|--------|-------|
| Zoho Books subscription | ✅ Required | Must have admin access |
| Invoice module enabled | ✅ Required | Sales module must be active |
| Email settings configured | ✅ Required | Sender email must be verified |
| Admin permissions | ✅ Required | To create workflows, fields, and functions |

### Information Needed

| Information | Value | Where to Find |
|-------------|-------|---------------|
| Organization ID | `928196532` | Settings > Organization > Profile |
| Connection Name | `books_connection` | Settings > Developer Space > Connections |
| Custom Field API Names | `cf_*` | Settings > Custom Fields > Invoices |

---

## Step-by-Step Implementation Guide

### 5.1 Create Custom Fields

**Navigation:** `Settings > Custom Fields > Invoices`

#### Tracking Fields (Checkbox)

| # | Field Label | API Name | Purpose |
|---|-------------|----------|---------|
| 1 | Track if 3-day reminder sent | `cf_track_if_3_day_reminder_sent` | Track 3-day reminder |
| 2 | Track if 7-day reminder sent | `cf_track_if_7_day_reminder_sent` | Track 7-day reminder |
| 3 | Track if 14-day reminder sent | `cf_track_if_14_day_reminder_sent` | Track 14-day reminder |
| 4 | Track if 30-day reminder sent | `cf_track_if_30_day_reminder_sent` | Track 30-day reminder |

**Settings for each:** Is Mandatory: No, Show in All PDFs: No

#### Trigger Field (Text)

| # | Field Label | API Name | Purpose |
|---|-------------|----------|---------|
| 5 | Reminder Trigger | `cf_reminder_trigger` | Triggers workflow rule |

**Settings:** Is Mandatory: No, Show in All PDFs: No, Help Text: "Stores the alert name to send"

**How to Create:**
1. Click "+ New Custom Field"
2. Enter Label Name
3. Select Data Type (Checkbox or Text)
4. Click Save
5. Note the API Name

---

### 5.2 Create Email Templates

**Navigation:** `Settings > Email Templates > + New Email Template` (Module: Invoice)

#### Template 1: 3-Day Overdue Reminder

**Name:** 3-Day Overdue Reminder
**Subject:** Reminder: Invoice #{{invoice.invoice_number}} is 3 Days Overdue
Dear {{customer.display_name}},

This is a friendly reminder that Invoice #{{invoice.invoice_number}}
for {{invoice.total}} was due on {{invoice.due_date}}.
It is currently 3 days overdue.

Please make the payment at your earliest convenience.

You can pay online using this link:
{{invoice.payment_link}}

Thank you for your business.

Best regards,
Your Company Name

text

#### Template 2: 7-Day Overdue Reminder

**Name:** 7-Day Overdue Reminder
**Subject:** Reminder: Invoice #{{invoice.invoice_number}} is 7 Days Overdue
Dear {{customer.display_name}},

This is a reminder that Invoice #{{invoice.invoice_number}}
for {{invoice.total}} is now 7 days overdue.
The invoice was due on {{invoice.due_date}}.

Please process the payment as soon as possible.

You can pay online using this link:
{{invoice.payment_link}}

Thank you,
Your Company Name

text

#### Template 3: 14-Day Overdue Reminder

**Name:** 14-Day Overdue Reminder
**Subject:** URGENT: Invoice #{{invoice.invoice_number}} is 14 Days Overdue
Dear {{customer.display_name}},

This is an urgent reminder that Invoice #{{invoice.invoice_number}}
for {{invoice.total}} is now 14 days overdue.
The due date was {{invoice.due_date}}.

Please clear the payment immediately to avoid any service disruption.

You can pay online using this link:
{{invoice.payment_link}}

Regards,
Your Company Name

text

#### Template 4: 30-Day Overdue Reminder

**Name:** 30-Day Overdue Reminder
**Subject:** FINAL NOTICE: Invoice #{{invoice.invoice_number}} is 30 Days Overdue
Dear {{customer.display_name}},

This is a FINAL NOTICE that Invoice #{{invoice.invoice_number}}
for {{invoice.total}} is now 30 days overdue.
The due date was {{invoice.due_date}}.

If payment is not received within the next 48 hours,
we may have to take further action.

Please make the payment immediately.

You can pay online using this link:
{{invoice.payment_link}}

Sincerely,
Your Company Name

text

---

### 5.3 Create Email Alerts

**Navigation:** `Settings > Automation > Email Alerts > + New Email Alert` (Module: Invoice)

#### Alert 1: 3-Day Overdue Alert

| Field | Value |
|-------|-------|
| Name | `3-Day Overdue Alert` |
| Module | Invoice |
| From | Your organization's email |
| Email Template | 3-Day Overdue Reminder |
| Attachments | ✅ Attach Invoice PDF |
| Email Recipients | Contact Email |

#### Alert 2: 7-Day Overdue Alert

| Field | Value |
|-------|-------|
| Name | `7-Day Overdue Alert` |
| Module | Invoice |
| From | Your organization's email |
| Email Template | 7-Day Overdue Reminder |
| Attachments | ✅ Attach Invoice PDF |
| Email Recipients | Contact Email |

#### Alert 3: 14-Day Overdue Alert

| Field | Value |
|-------|-------|
| Name | `14-Day Overdue Alert` |
| Module | Invoice |
| From | Your organization's email |
| Email Template | 14-Day Overdue Reminder |
| Attachments | ✅ Attach Invoice PDF |
| Email Recipients | Contact Email |

#### Alert 4: 30-Day Overdue Alert

| Field | Value |
|-------|-------|
| Name | `30-Day Overdue Alert` |
| Module | Invoice |
| From | Your organization's email |
| Email Template | 30-Day Overdue Reminder |
| Attachments | ✅ Attach Invoice PDF |
| Email Recipients | Contact Email |

---

### 5.4 Create OAuth Connection

**Navigation:** `Settings > Developer Space > Connections > Create Connection`

| Field | Value |
|-------|-------|
| Name | `books_connection` |
| Application | Zoho Books |
| Authorization | OAuth 2.0 |

**Steps:**
1. Click "Create Connection"
2. Search for Zoho Books
3. Click "Connect"
4. Authorize the connection
5. Grant necessary permissions
6. Copy the Connection Name (`books_connection`)

---

### 5.5 Create Custom Function

**Navigation:** `Settings > Automation > Functions > + New Function`

| Field | Value |
|-------|-------|
| Function Name | `Send_Overdue_Reminder` |
| Module | Invoice |
| Return Type | `void` |
| Arguments | None (Zoho Books passes `invoice`, `organization`, `user` automatically) |

#### Function Code

```deluge
// ============================================================
// FUNCTION: Send_Overdue_Reminder
// DESCRIPTION: Calculates days overdue and triggers email alert
// MODULE: Invoice
// CONNECTION: books_connection
// ============================================================

info("=== STARTING SEND_OVERDUE_REMINDER ===");

// 1. Get invoice data from the map
invoiceID = invoice.get("invoice_id");
dueDate = invoice.get("due_date");
invoiceStatus = invoice.get("status");

info("Invoice ID: " + invoiceID);
info("Due Date: " + dueDate);
info("Invoice Status: " + invoiceStatus);

// 2. Check if invoice is overdue
if(invoiceStatus != "overdue" && invoiceStatus != "partially_paid")
{
    info("Status not overdue. Status: " + invoiceStatus);
    return;
}

// 3. Check due date
if(dueDate == null || dueDate == "")
{
    info("ERROR: Due date not found.");
    return;
}

// 4. Calculate days overdue
dueDateObj = dueDate.toDate();
todayObj = zoho.currentdate;
daysOverdue = (todayObj.toLong() - dueDateObj.toLong()) / (24 * 60 * 60 * 1000);
info("Days Overdue: " + daysOverdue);

// 5. Determine which reminder to send
trackingField = "";
triggerValue = "";

if(daysOverdue >= 3 && daysOverdue < 7)
{
    trackingField = "cf_track_if_3_day_reminder_sent";
    triggerValue = "3-day";
}
else if(daysOverdue >= 7 && daysOverdue < 14)
{
    trackingField = "cf_track_if_7_day_reminder_sent";
    triggerValue = "7-day";
}
else if(daysOverdue >= 14 && daysOverdue < 30)
{
    trackingField = "cf_track_if_14_day_reminder_sent";
    triggerValue = "14-day";
}
else if(daysOverdue >= 30)
{
    trackingField = "cf_track_if_30_day_reminder_sent";
    triggerValue = "30-day";
}
else
{
    info("Not overdue enough. Days: " + daysOverdue);
    return;
}

info("Tracking Field: " + trackingField);
info("Trigger Value: " + triggerValue);

// 6. Check if already sent
trackingFieldValue = invoice.get(trackingField);
if(trackingFieldValue == true)
{
    info("Reminder already sent. Skipping.");
    return;
}

// 7. Update the invoice using custom_fields array
orgID = "928196532";
connectionName = "books_connection";

// Build custom_fields array
customFields = List();
customFields.add({
    "api_name": "cf_reminder_trigger",
    "value": triggerValue
});
customFields.add({
    "api_name": trackingField,
    "value": true
});

// Create update map
updateMap = Map();
updateMap.put("custom_fields", customFields);

info("Update Map: " + updateMap.toString());

// Update the invoice
zoho.books.updateRecord("Invoices", orgID, invoiceID, updateMap, connectionName);

info("Tracking field updated. Workflow rules will send the email.");
info("=== FUNCTION COMPLETED ===");
5.6 Create Workflow Rules
Navigation: Settings > Automation > Workflow Rules > + New Workflow Rule

Common Settings for All Rules
Field	Value
Module	Invoice
Workflow Type	Event Based
Action Type	Edited
Execute the workflow when	Any field is updated
Execute when the record is	Edited each time
Rule 1: Send 3-Day Overdue Reminder
Name: Send 3-Day Overdue Reminder

Condition: cf_reminder_trigger is 3-day

Actions:

Email Alerts: 3-Day Overdue Alert

Field Updates: Clear Reminder Trigger

Rule 2: Send 7-Day Overdue Reminder
Name: Send 7-Day Overdue Reminder

Condition: cf_reminder_trigger is 7-day

Actions:

Email Alerts: 7-Day Overdue Alert

Field Updates: Clear Reminder Trigger

Rule 3: Send 14-Day Overdue Reminder
Name: Send 14-Day Overdue Reminder

Condition: cf_reminder_trigger is 14-day

Actions:

Email Alerts: 14-Day Overdue Alert

Field Updates: Clear Reminder Trigger

Rule 4: Send 30-Day Overdue Reminder
Name: Send 30-Day Overdue Reminder

Condition: cf_reminder_trigger is 30-day

Actions:

Email Alerts: 30-Day Overdue Alert

Field Updates: Clear Reminder Trigger

Field Update: Clear Reminder Trigger
Navigation: Settings > Automation > Field Updates > + New Field Update

Field	Value
Name	Clear Reminder Trigger
Module	Invoice
Field	cf_reminder_trigger
Value	Empty
5.7 Create Scheduler
Navigation: Settings > Automation > Schedules > + New Schedule

Field	Value
Name	Send Overdue Reminders
Module	Invoice
Action Type	Function
Function	Send_Overdue_Reminder
Schedule Settings:

Field	Value
Frequency	Daily
Time	09:00 AM
Time Zone	Organization's Time Zone
Status	Active
Testing Guide
Test Invoices
Create 4 test invoices with different due dates:

Test	Due Date (relative to today)	Expected Reminder
Test 1	5 days ago	3-Day Reminder
Test 2	10 days ago	7-Day Reminder
Test 3	20 days ago	14-Day Reminder
Test 4	35 days ago	30-Day Reminder
Test Steps
Create Test Invoices:

Customer: Use a test customer with valid email

Set due date to past date

Save and send the invoice

Execute Function Manually:

Go to Functions > Send_Overdue_Reminder

Click "Custom Execution"

Enter the Invoice ID

Click "Execute"

Verify Results:

Check invoice custom fields (tracking checkbox should be checked)

Check customer email (should receive the reminder)

Check workflow logs (should show "Executed" not "Skipped")

Test Scenarios
Scenario	Expected Result	Pass/Fail
3-Day Reminder	3-Day email sent, tracking field checked	☐
7-Day Reminder	7-Day email sent, tracking field checked	☐
14-Day Reminder	14-Day email sent, tracking field checked	☐
30-Day Reminder	30-Day email sent, tracking field checked	☐
No Duplicate	Only one email sent, tracking field remains checked	☐
Paid Invoice	No email sent, function skips	☐
Troubleshooting Guide
Common Issues & Solutions
Issue	Cause	Solution
Fields not updating	Custom fields not in custom_fields array	Use custom_fields array format
Authentication error	No OAuth connection	Create books_connection
Workflow not triggering	Condition mismatch	Verify condition uses correct API name
Workflow skipped	Field cleared before condition check	Email first, then Clear
Duplicate reminders	Tracking checkbox not checked	Verify tracking field is set to true
Debugging Steps
Check Function Logs: Settings > Automation > Functions > View Logs

Check Workflow Logs: Settings > Automation > Workflow Rules > View Logs

Check Email Alerts: Settings > Automation > Email Alerts

Check Custom Fields: Open invoice and verify fields

Key Technical Discovery
Zoho Books requires custom fields to be updated using the custom_fields array format.

❌ Wrong:

deluge
updateMap.put("cf_reminder_trigger", "3-day");
✅ Correct:

deluge
customFields = List();
customFields.add({
    "api_name": "cf_reminder_trigger",
    "value": "3-day"
});
updateMap.put("custom_fields", customFields);
Function Call Syntax
deluge
zoho.books.updateRecord("Invoices", orgID, invoiceID, updateMap, connectionName);
Appendices
Appendix A: Custom Fields Reference
Field Label	API Name	Type	Purpose
Reminder Trigger	cf_reminder_trigger	Text	Triggers workflow
Track if 3-day reminder sent	cf_track_if_3_day_reminder_sent	Checkbox	3-day tracking
Track if 7-day reminder sent	cf_track_if_7_day_reminder_sent	Checkbox	7-day tracking
Track if 14-day reminder sent	cf_track_if_14_day_reminder_sent	Checkbox	14-day tracking
Track if 30-day reminder sent	cf_track_if_30_day_reminder_sent	Checkbox	30-day tracking
Appendix B: Email Templates Reference
Template Name	Subject	Tone
3-Day Overdue Reminder	"Reminder: Invoice #[Number] is 3 Days Overdue"	Friendly
7-Day Overdue Reminder	"Reminder: Invoice #[Number] is 7 Days Overdue"	Polite
14-Day Overdue Reminder	"URGENT: Invoice #[Number] is 14 Days Overdue"	Urgent
30-Day Overdue Reminder	"FINAL NOTICE: Invoice #[Number] is 30 Days Overdue"	Final
Appendix C: Email Alerts Reference
Alert Name	Template	Recipient
3-Day Overdue Alert	3-Day Overdue Reminder	Contact Email
7-Day Overdue Alert	7-Day Overdue Reminder	Contact Email
14-Day Overdue Alert	14-Day Overdue Reminder	Contact Email
30-Day Overdue Alert	30-Day Overdue Reminder	Contact Email
Appendix D: Workflow Rules Reference
Rule Name	Condition	Actions
Send 3-Day Overdue Reminder	cf_reminder_trigger = "3-day"	Email Alert + Clear Trigger
Send 7-Day Overdue Reminder	cf_reminder_trigger = "7-day"	Email Alert + Clear Trigger
Send 14-Day Overdue Reminder	cf_reminder_trigger = "14-day"	Email Alert + Clear Trigger
Send 30-Day Overdue Reminder	cf_reminder_trigger = "30-day"	Email Alert + Clear Trigger
Quick Reference Card
text
┌─────────────────────────────────────────────────────────────────────────────┐
│                    QUICK REFERENCE CARD                                     │
│                    Overdue Payment Reminder Automation                     │
│                                                                             │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │  CUSTOM FIELDS (Invoice Module)                                      │ │
│  │  cf_reminder_trigger              - Text (Trigger)                   │ │
│  │  cf_track_if_3_day_reminder_sent  - Checkbox (3-Day)                │ │
│  │  cf_track_if_7_day_reminder_sent  - Checkbox (7-Day)                │ │
│  │  cf_track_if_14_day_reminder_sent - Checkbox (14-Day)               │ │
│  │  cf_track_if_30_day_reminder_sent - Checkbox (30-Day)               │ │
│  └───────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │  EMAIL ALERTS                                                         │ │
│  │  3-Day Overdue Alert  → 3-Day Overdue Reminder                       │ │
│  │  7-Day Overdue Alert  → 7-Day Overdue Reminder                       │ │
│  │  14-Day Overdue Alert → 14-Day Overdue Reminder                      │ │
│  │  30-Day Overdue Alert → 30-Day Overdue Reminder                      │ │
│  └───────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │  WORKFLOW RULES                                                        │ │
│  │  Send 3-Day Overdue Reminder  → cf_reminder_trigger = "3-day"        │ │
│  │  Send 7-Day Overdue Reminder  → cf_reminder_trigger = "7-day"        │ │
│  │  Send 14-Day Overdue Reminder → cf_reminder_trigger = "14-day"       │ │
│  │  Send 30-Day Overdue Reminder → cf_reminder_trigger = "30-day"       │ │
│  └───────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │  FUNCTION                                                             │ │
│  │  Send_Overdue_Reminder                                                │ │
│  │  Connection: books_connection                                         │ │
│  │  Schedule: Daily at 9:00 AM                                          │ │
│  └───────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────┘
Project Files Summary
File Name	Folder	Description
payment-reminder-automation.md	docs/	This complete documentation
Send_Overdue_Reminder.deluge	functions/	Custom function code
workflow-rules-config.json	workflows/	All 4 workflow rules configuration
custom-fields-list.json	custom-fields/	All custom fields with API names
3-day-overdue-reminder.txt	email-templates/	3-Day email template
7-day-overdue-reminder.txt	email-templates/	7-Day email template
14-day-overdue-reminder.txt	email-templates/	14-Day email template
30-day-overdue-reminder.txt	email-templates/	30-Day email template
email-alerts-config.json	email-alerts/	All 4 email alerts configuration
Final Status
✅ Automation Complete and Tested

All components are configured and working together as intended.

Email Templates: ✅ Created

Email Alerts: ✅ Created

Custom Fields: ✅ Created

Custom Function: ✅ Working

Workflow Rules: ✅ Active

Scheduler: ✅ Configured

End-to-End Test: ✅ Passed

