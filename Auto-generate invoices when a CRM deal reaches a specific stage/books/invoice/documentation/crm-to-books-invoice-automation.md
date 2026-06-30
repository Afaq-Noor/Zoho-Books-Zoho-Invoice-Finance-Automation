# Zoho CRM to Zoho Books: Auto-Invoice Generation

## 📌 Task Overview

**Objective:** Automatically create an invoice in Zoho Books when a deal in Zoho CRM reaches the "Closed Won" stage.

**Scope:**
- Zoho CRM (Deals module)
- Zoho Books (Invoices module)
- Zoho Deluge (Custom Function)
- Zoho Workflow Rules

**Status:** ✅ Complete and Tested

---

## 🔧 Prerequisites

| Requirement | Status | Notes |
|-------------|--------|-------|
| Zoho CRM and Zoho Books connected via Zoho Finance Suite | ✅ | Organization: "Student" |
| CRM Deals have linked Contact | ✅ Required | Contact must have valid Email |
| Zoho Books Organization ID | ✅ | ID: 928196532 |
| Admin access to CRM | ✅ | To create functions and workflows |

---

## 🏗️ Architecture
┌─────────────────────┐
│ Zoho CRM │
│ Deal Updated to │
│ "Closed Won" │
└──────────┬──────────┘
│
▼
┌─────────────────────┐
│ Workflow Rule │
│ Triggers on: │
│ Stage = Closed Won │
└──────────┬──────────┘
│
▼
┌─────────────────────┐
│ Deluge Function │
│ Create_Invoice_ │
│ from_Closed_Deal │
└──────────┬──────────┘
│
▼
┌─────────────────────────────────────────────┐
│ Zoho Books │
│ 1. Search/Find Customer by Email │
│ 2. Create Customer if not found │
│ 3. Create Invoice with Line Items │
│ 4. Return Invoice ID │
└─────────────────────────────────────────────┘


---

## 📝 Implementation Steps

### Step 1: Verify Zoho Finance Suite Connection

1. Navigate to **Zoho CRM > Setup > Marketplace > All**
2. Find **Zoho Finance Suite**
3. Verify connection status:
   - Email: afaqn846@gmail.com
   - Organization: Student
   - Modules: Invoice, Quote, Sales Order, Purchase Order ✅
   - Sync Settings: Active ✅

**Screenshot Reference:**
![Zoho Finance Suite](screenshots/finance-suite.png)

---

### Step 2: Get Organization ID

1. Go to **Zoho Books > Setup > Organization > Profile**
2. Copy the **Organization ID** from the URL or settings
3. Our Organization ID: `928196532`

---

### Step 3: Create Deluge Function

#### 3.1 Navigate to Functions
- **Path:** Zoho CRM > Setup > Developer Space > Functions
- Click **"Create New Function"**

#### 3.2 Function Details
| Field | Value |
|-------|-------|
| **Function Name** | Create_Invoice_from_Closed_Deal |
| **Display Name** | Create Invoice from Closed Deal |
| **Category** | automation |
| **Return Type** | void |

#### 3.3 Function Code
See file: `functions/Create_Invoice_from_Closed_Deal.deluge`

#### 3.4 Key Logic Explained

| Section | Description |
|---------|-------------|
| **Get Deal Details** | Fetches Deal Name, Amount, and linked Contact |
| **Get Contact Email** | Extracts email from linked Contact |
| **Search/Find Customer** | Searches Books for existing customer by email |
| **Create Customer** | If not found, creates new customer in Books |
| **Build Invoice** | Constructs invoice with customer_id and line items |
| **Create Invoice** | Creates invoice in Books using zoho.books.createRecord |
| **Return Success** | Logs invoice ID on success |

---

### Step 4: Create Workflow Rule

#### 4.1 Navigate to Workflow Rules
- **Path:** Zoho CRM > Setup > Automation > Workflow Automation > Rules
- Click **"Create Rule"**

#### 4.2 Workflow Configuration

| Setting | Value |
|---------|-------|
| **Module** | Deals |
| **Rule Name** | Generate Invoice on Closed Won |
| **Trigger** | On Record Action (Create & Edit) |
| **Condition** | Potential Stage is Closed Won |
| **Execute Trigger** | ⚠️ DO NOT check "Execute workflow whenever this condition is met" |

#### 4.3 Action Configuration

| Setting | Value |
|---------|-------|
| **Action Type** | Function |
| **Function** | Create_Invoice_from_Closed_Deal |
| **Parameter** | ID (from Deals module) |

#### 4.4 Workflow Rule JSON
See file: `workflows/workflow-rule-crm-to-books-invoice.json`

---

### Step 5: Test the Automation

#### 5.1 Create Test Data
1. Create a **Contact** in CRM with a valid Email
2. Create a **Deal** linked to that Contact
3. Set Deal Amount and Name

#### 5.2 Trigger the Workflow
1. Edit the Deal
2. Change **Stage** to **"Closed Won"**
3. Save the Deal

#### 5.3 Verify Execution
1. Go to **Zoho CRM > Setup > Automation > Workflow Automation > Execution**
2. Find your workflow execution log
3. Check for:
   - `✅ SUCCESS! Invoice Created with ID: xxx`

#### 5.4 Verify in Zoho Books
1. Go to **Zoho Books > Invoices**
2. Look for a new invoice with:
   - Customer: Your test Contact
   - Reference: Your Deal Name
   - Amount: Your Deal Amount

---

## 🧪 Test Results

| Test Case | Expected Result | Actual Result | Status |
|-----------|----------------|---------------|--------|
| Deal with Contact and Email | Invoice created in Books | ✅ Invoice generated | PASS |
| Deal with no Contact | Function skips with error | ✅ Logged error | PASS |
| Deal with Contact no Email | Function skips with error | ✅ Logged error | PASS |
| Customer exists in Books | Uses existing customer | ✅ Found customer | PASS |
| Customer doesn't exist in Books | Creates new customer | ✅ Customer created | PASS |

---

## 🐛 Troubleshooting Guide

### Error: "Connection 'conn' does not exist"
**Solution:** Remove the connection parameter OR use available connection name:
```deluge
// Option 1: No connection (auto-detect)
customerSearch = zoho.books.getRecords("contacts", orgID, {"email": contactEmail});

// Option 2: Use existing connection
conn = "Zoho OAuth"; // or "internal-zohobooks"
customerSearch = zoho.books.getRecords("contacts", orgID, {"email": contactEmail}, conn);