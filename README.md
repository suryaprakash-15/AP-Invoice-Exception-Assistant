# AP-Invoice-Exception-Assistant
Build a complete, working full-stack application called:

# AP Invoice Exception Assistant

This project is for an AI Employee assessment.

Do NOT create only a frontend mockup or static demo. Build the complete working application with document upload, invoice extraction, PO comparison, exception detection, source-grounded explanations, conversational chat, testing, documentation, and GitHub-ready setup.

---

# 1. PROJECT OBJECTIVE

Build an AI-powered Accounts Payable Invoice Exception Assistant.

The application must allow a reviewer to:

1. Upload a vendor invoice as a PDF or image.
2. Upload/provide a mock Purchase Order (PO).
3. Extract structured invoice data.
4. Extract invoice line items.
5. Compare invoice data against the PO.
6. Detect mismatches in:

   * Price
   * Quantity
   * Tax
7. Flag each exception clearly.
8. Explain in plain English why each exception was flagged.
9. Cite the actual invoice and PO fields used for the comparison.
10. Provide a conversational chat interface.
11. Allow the reviewer to ask questions such as:

    * "Why was invoice #123 flagged?"
    * "Why is line item 2 an exception?"
    * "What is the price difference?"
    * "Why did the tax fail?"
    * "Which items have quantity mismatches?"
12. Return source-grounded answers based on the actual uploaded invoice and PO data.

The system must never invent invoice or PO values.

---

# 2. CORE ASSESSMENT REQUIREMENTS

The implementation must satisfy these requirements:

## Requirement 1 — Uploaded Invoice and PO

Accept:

* PDF invoice
* PNG invoice
* JPG/JPEG invoice

Accept a mock PO as:

* JSON
* CSV
* PDF
* or structured form input

Prefer supporting JSON PO input initially because it makes testing deterministic.

Also provide a sample PO file.

---

# 3. INVOICE EXTRACTION

Extract structured information from the invoice.

The extraction should identify:

## Invoice-level fields

* invoiceNumber
* invoiceDate
* vendorName
* vendorAddress
* purchaseOrderNumber
* subtotal
* tax
* total

## Line-item fields

For every line item extract:

* lineNumber
* description
* productCode/SKU
* quantity
* unitPrice
* taxRate
* taxAmount
* lineTotal

Example:

{
"invoiceNumber": "INV-123",
"invoiceDate": "2026-08-20",
"vendorName": "ABC Supplies",
"purchaseOrderNumber": "PO-456",
"subtotal": 1000,
"tax": 180,
"total": 1180,
"lineItems": [
{
"lineNumber": 1,
"description": "Laptop Stand",
"productCode": "LS-100",
"quantity": 10,
"unitPrice": 100,
"taxRate": 18,
"taxAmount": 180,
"lineTotal": 1180
}
]
}

Use actual values extracted from the uploaded invoice.

---

# 4. EXTRACTION ROBUSTNESS

The application must handle common invoice layouts.

Support:

* Tables
* Different column names
* Currency symbols
* Decimal numbers
* Thousands separators
* Multiple line items
* Different PDF layouts
* Scanned invoices/images where possible

Normalize values before comparison.

For example:

"₹1,250.00"

should become:

1250.00

Handle:

* INR
* USD
* EUR
* Other common currency formats

Do not silently guess missing fields.

If a field cannot be extracted confidently, mark it as:

"Not detected"

and show a warning.

---

# 5. OCR

For image invoices and scanned PDFs, implement OCR.

Use a practical open-source/local solution if possible, such as:

* Tesseract OCR

For PDFs:

1. Determine whether text is extractable.
2. If text is available, extract it directly.
3. If the PDF is scanned, render pages to images and use OCR.

Make the extraction service modular so another OCR/document AI provider can be added later.

---

# 6. DOCUMENT EXTRACTION ARCHITECTURE

Create a modular extraction pipeline:

Uploaded File
↓
File Type Detection
↓
PDF Text Extraction / OCR
↓
Raw Invoice Text
↓
Invoice Parser
↓
Structured Invoice JSON
↓
Validation
↓
Exception Engine

Keep raw extracted text available for debugging and source references.

---

# 7. PURCHASE ORDER

Create sample PO data.

Example:

{
"poNumber": "PO-456",
"vendorName": "ABC Supplies",
"currency": "INR",
"lineItems": [
{
"lineNumber": 1,
"productCode": "LS-100",
"description": "Laptop Stand",
"quantity": 10,
"unitPrice": 100,
"taxRate": 18
},
{
"lineNumber": 2,
"productCode": "KB-200",
"description": "Wireless Keyboard",
"quantity": 5,
"unitPrice": 1500,
"taxRate": 18
}
]
}

Create:

data/sample-po.json

Also create at least one sample invoice that intentionally contains exceptions.

---

# 8. LINE ITEM MATCHING

Implement robust matching between invoice line items and PO line items.

Prefer matching using:

1. Product code/SKU
2. PO line number
3. Description similarity as fallback

Do not match incorrectly just because two descriptions are vaguely similar.

If an invoice line cannot be matched to a PO line, flag:

"UNMATCHED_LINE_ITEM"

and explain why.

---

# 9. EXCEPTION ENGINE

Create a deterministic comparison engine.

Compare:

## Quantity

Invoice:

quantity = 12

PO:

quantity = 10

Flag:

QUANTITY_MISMATCH

Explanation:

"Invoice line 1 contains 12 units, while PO line 1 allows 10 units. The invoice quantity exceeds the PO quantity by 2 units."

---

## Price

Invoice:

unitPrice = 125

PO:

unitPrice = 100

Flag:

PRICE_MISMATCH

Explanation:

"Invoice line 1 has a unit price of ₹125, while the PO specifies ₹100. The invoice price is ₹25 higher per unit."

---

## Tax

Compare:

* Invoice tax rate
* PO tax rate
* Invoice tax amount
* Expected tax amount

Example:

Invoice tax rate = 18%

PO tax rate = 12%

Flag:

TAX_MISMATCH

Explanation:

"Invoice line 2 uses an 18% tax rate, while the PO specifies 12%. This creates a tax-rate mismatch."

---

# 10. TOLERANCE

Implement configurable tolerance.

Create:

config/comparisonConfig.js

Example:

PRICE_TOLERANCE_PERCENT = 0
QUANTITY_TOLERANCE = 0
TAX_TOLERANCE_PERCENT = 0

Allow the values to be changed easily.

If tolerance is applied, explicitly mention it in the explanation.

---

# 11. EXCEPTION DATA MODEL

Each exception should contain:

{
"exceptionId": "EXC-001",
"invoiceNumber": "INV-123",
"poNumber": "PO-456",
"lineNumber": 2,
"type": "PRICE_MISMATCH",
"severity": "HIGH",
"field": "unitPrice",
"invoiceValue": 1250,
"poValue": 1000,
"difference": 250,
"differencePercent": 25,
"reason": "Invoice unit price is higher than the PO unit price.",
"invoiceSource": {
"field": "lineItems[1].unitPrice",
"value": 1250
},
"poSource": {
"field": "lineItems[1].unitPrice",
"value": 1000
}
}

Use the actual values from the uploaded documents.

---

# 12. SOURCE-GROUNDED REASONING

This is extremely important.

Every exception explanation must cite the actual source fields.

Do NOT generate generic responses like:

"The invoice was flagged because the price doesn't match."

Instead return:

"Line 2 was flagged because the invoice lists a unit price of ₹1,250, while PO line 2 specifies ₹1,000. This is a ₹250 difference, or 25% above the PO price."

Then display:

Invoice source:
lineItems[1].unitPrice = ₹1,250

PO source:
lineItems[1].unitPrice = ₹1,000

The reasoning must always be traceable to actual extracted fields.

---

# 13. EXCEPTION SUMMARY

After processing an invoice, show:

Invoice:
INV-123

PO:
PO-456

Status:
EXCEPTIONS FOUND

Summary:

Total line items: 5
Matched: 5
Exceptions: 3

Quantity exceptions: 1
Price exceptions: 1
Tax exceptions: 1

---

# 14. RESULTS TABLE

Create a professional comparison table:

| Line | Item | Invoice Qty | PO Qty | Invoice Price | PO Price | Invoice Tax | PO Tax | Status |
| ---- | ---- | ----------- | ------ | ------------- | -------- | ----------- | ------ | ------ |

Use visual status indicators:

* PASS
* WARNING
* EXCEPTION

Allow the reviewer to click a row to see detailed reasoning.

---

# 15. CHAT INTERFACE

Create a conversational reviewer interface.

The reviewer should be able to ask:

"Why was invoice #123 flagged?"

The assistant should answer using the actual exception data.

Example:

"Invoice INV-123 was flagged for 3 exceptions:

1. Line 2 price mismatch:
   Invoice price: ₹1,250
   PO price: ₹1,000
   Difference: ₹250 (25%)

2. Line 3 quantity mismatch:
   Invoice quantity: 12
   PO quantity: 10
   Difference: 2 units

3. Line 4 tax mismatch:
   Invoice tax rate: 18%
   PO tax rate: 12%

These exceptions were identified by comparing the extracted invoice fields with the corresponding PO fields."

---

# 16. FOLLOW-UP QUESTIONS

Support follow-up questions.

Example:

User:
Why was invoice INV-123 flagged?

Assistant:
It has 3 exceptions...

User:
Which one has the largest difference?

Assistant:
The largest monetary difference is the price mismatch on line 2: ₹250.

User:
What was the PO price?

Assistant:
The PO price for line 2 was ₹1,000.

The assistant must maintain conversation context.

---

# 17. UNKNOWN INFORMATION

If the reviewer asks something that cannot be determined from the invoice or PO:

"I don't have enough information in the uploaded invoice and purchase order to answer that."

Never invent data.

---

# 18. FRONTEND

Use:

* React
* Vite
* JavaScript
* CSS

Create a modern enterprise AP dashboard.

Pages/components:

### Upload

Allow:

* Invoice upload
* PO upload
* Process button

Show:

* File name
* File type
* Upload progress
* Processing status

### Invoice Details

Display:

* Invoice number
* Vendor
* Invoice date
* PO number
* Subtotal
* Tax
* Total

### Exception Dashboard

Display:

* Total lines
* Passed lines
* Exceptions
* Price mismatches
* Quantity mismatches
* Tax mismatches

### Comparison Table

Show invoice vs PO fields.

### Exception Details

When a reviewer selects an exception, show:

* Exception type
* Severity
* Invoice value
* PO value
* Difference
* Explanation
* Source fields

### AI Chat

Provide a reviewer chat interface.

---

# 19. USER EXPERIENCE

Make the UI look like a professional Accounts Payable enterprise application.

Use:

* Clean dashboard
* Cards
* Tables
* Status badges
* Exception highlighting
* Expandable details
* Responsive design
* Loading indicators
* Error messages
* Empty states

Do NOT make it look like a simple college project.

---

# 20. BACKEND

Use:

* Node.js
* Express
* JavaScript
* Multer for uploads
* PDF parser
* OCR
* Validation
* REST APIs

Create APIs:

POST /api/invoices/upload

POST /api/po/upload

POST /api/process

GET /api/invoices/:invoiceId

GET /api/exceptions/:invoiceId

GET /api/comparison/:invoiceId

POST /api/chat

GET /api/health

---

# 21. CHAT API

Create:

POST /api/chat

Request:

{
"invoiceId": "INV-123",
"message": "Why was invoice INV-123 flagged?",
"conversationId": "conversation-001"
}

Response:

{
"message": "...",
"sources": [
{
"type": "invoice",
"field": "lineItems[1].unitPrice",
"value": 1250
},
{
"type": "purchaseOrder",
"field": "lineItems[1].unitPrice",
"value": 1000
}
]
}

The response must be grounded in actual processed data.

---

# 22. AI SYSTEM PROMPT

Create an internal AI system prompt with these rules:

You are an Accounts Payable Invoice Exception Assistant.

Your job is to explain invoice exceptions using only the extracted invoice data, purchase order data, and calculated exception results.

Rules:

1. Never invent invoice values.
2. Never invent PO values.
3. Never invent exception types.
4. Always cite the source fields used.
5. Explain differences using actual numbers.
6. Use plain English.
7. If information is missing, say that it is unavailable.
8. Maintain conversation context.
9. Do not override deterministic comparison results.
10. Treat the exception engine as the source of truth for mismatch detection.
11. The LLM should explain results, not independently make up financial calculations.
12. When calculations are needed, use backend-calculated values.

---

# 23. IMPORTANT ARCHITECTURE RULE

Separate deterministic comparison logic from the LLM.

Use:

Document Extraction
↓
Structured Invoice JSON
↓
PO JSON
↓
Deterministic Comparison Engine
↓
Exception Results
↓
LLM Explanation Layer
↓
Reviewer

The LLM should NOT be responsible for deciding whether numbers match.

The comparison engine should calculate:

* Quantity difference
* Price difference
* Tax difference
* Percentage difference
* Exception type

The LLM only explains those results conversationally.

---

# 24. SAMPLE FILES

Create:

data/sample-invoice.pdf

and:

data/sample-po.json

The sample invoice should intentionally contain:

1. One price mismatch
2. One quantity mismatch
3. One tax mismatch
4. At least one correctly matched line

This will make the application easy to demonstrate.

If generating a binary PDF is difficult, create a sample invoice source file and a script that generates the PDF automatically.

---

# 25. DATA VALIDATION

Validate extracted data.

Check:

* Required invoice number
* Invoice date
* Vendor
* PO number
* Line items
* Quantity
* Unit price
* Tax
* Total

Show extraction warnings if information is missing.

Do not silently assume missing values.

---

# 26. SECURITY

Implement:

* File type validation
* File size limits
* Safe file names
* No arbitrary file access
* Environment variables for API keys
* Input validation
* No API key exposure to frontend
* No secrets committed to Git

---

# 27. ERROR HANDLING

Handle:

* Invalid invoice file
* Unsupported file type
* OCR failure
* PDF parsing failure
* Missing fields
* Invalid PO
* Unmatched line item
* API errors
* LLM errors
* Processing errors

Show understandable error messages.

---

# 28. PROJECT STRUCTURE

Create a clean structure:

ap-invoice-exception-assistant/
│
├── client/
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── services/
│   │   ├── hooks/
│   │   ├── App.jsx
│   │   └── main.jsx
│   └── package.json
│
├── server/
│   ├── src/
│   │   ├── routes/
│   │   ├── controllers/
│   │   ├── services/
│   │   ├── extraction/
│   │   ├── ocr/
│   │   ├── comparison/
│   │   ├── chat/
│   │   └── app.js
│   ├── package.json
│   └── .env.example
│
├── data/
│   ├── sample-invoice.pdf
│   └── sample-po.json
│
├── config/
│   └── comparisonConfig.js
│
├── tests/
│
├── README.md
├── .gitignore
└── package.json

You may improve the structure if required.

---

# 29. ENVIRONMENT VARIABLES

Create:

server/.env.example

Include:

PORT=5000
CLIENT_URL=http://localhost:5173
OPENAI_API_KEY=

Never hardcode API keys.

Add `.env` to `.gitignore`.

---

# 30. TESTING

Create automated tests for:

1. Invoice extraction
2. PO parsing
3. Quantity mismatch detection
4. Price mismatch detection
5. Tax mismatch detection
6. Correctly matched line
7. Unmatched line
8. Exception calculations
9. Chat explanation
10. API endpoints

Use deterministic test data.

Example:

PO quantity = 10
Invoice quantity = 12

Expected:

QUANTITY_MISMATCH

Difference:

2

---

# 31. README

Create a professional GitHub README.md.

Include:

# AP Invoice Exception Assistant

## Overview

## Problem Statement

## Solution

## Key Features

## Architecture

## Tech Stack

## Document Extraction

## OCR

## Invoice Parsing

## PO Comparison

## Exception Detection

## Source-Grounded Reasoning

## Chat Interface

## Project Structure

## Installation

## Environment Variables

## Running Locally

## API Documentation

## Sample Invoice

## Sample Conversation

## Testing

## Screenshots

## Future Improvements

## GitHub Setup

## Disclaimer

Include a clear statement:

"The invoice and purchase order data included in this project are sample data created for demonstration and assessment purposes."

---

# 32. SAMPLE DEMO CONVERSATION

Include this in README:

Reviewer:
Why was invoice INV-123 flagged?

AI:
Invoice INV-123 was flagged because 3 exceptions were detected.

Line 2:
Invoice unit price = ₹1,250
PO unit price = ₹1,000
Difference = ₹250

Line 3:
Invoice quantity = 12
PO quantity = 10
Difference = 2 units

Line 4:
Invoice tax rate = 18%
PO tax rate = 12%

Reviewer:
Which exception has the largest price difference?

AI:
Line 2 has the largest price difference. The invoice price is ₹1,250 compared with the PO price of ₹1,000, resulting in a ₹250 difference.

Reviewer:
What was the PO price?

AI:
The PO price for line 2 was ₹1,000.

---

# 33. GITHUB PREPARATION

Prepare the entire project for GitHub.

Create `.gitignore` containing:

node_modules/
.env
.env.local
dist/
coverage/
*.log
uploads/

Initialize Git:

git init

Create an initial commit.

Do NOT ask me for my GitHub password or personal access token.

Do NOT invent my GitHub username.

Use:

YOUR_GITHUB_USERNAME
YOUR_REPOSITORY_NAME

Give me the exact commands:

git remote add origin https://github.com/YOUR_GITHUB_USERNAME/YOUR_REPOSITORY_NAME.git
git branch -M main
git push -u origin main

If GitHub CLI is already authenticated, you may use it. Otherwise give me the commands I need to run manually.

---

# 34. ROOT PACKAGE.JSON

Create a root package.json that allows:

npm install

and:

npm run dev

to start the complete application.

Use concurrently if appropriate.

---

# 35. FINAL VERIFICATION

After creating the project:

1. Install all dependencies.
2. Start backend.
3. Start frontend.
4. Check compilation errors.
5. Fix all errors.
6. Test invoice upload.
7. Test PDF extraction.
8. Test OCR if applicable.
9. Test PO upload.
10. Test invoice parsing.
11. Test line-item matching.
12. Test price mismatch.
13. Test quantity mismatch.
14. Test tax mismatch.
15. Test correctly matched line.
16. Test exception explanations.
17. Test chat.
18. Test follow-up questions.
19. Verify source fields are shown.
20. Verify no values are hallucinated.
21. Run automated tests.
22. Prepare Git repository.

Do not declare the project complete until the core workflow works.

---

# 36. ACCEPTANCE CHECKLIST

Verify all of these:

[ ] Invoice PDF upload works
[ ] Invoice image upload works
[ ] PO input works
[ ] Invoice text extraction works
[ ] OCR fallback works
[ ] Structured invoice JSON is generated
[ ] Invoice line items are extracted
[ ] PO line items are parsed
[ ] Invoice/PO lines are matched
[ ] Price mismatches are detected
[ ] Quantity mismatches are detected
[ ] Tax mismatches are detected
[ ] Correct lines are marked as matched
[ ] Unmatched lines are flagged
[ ] Exception calculations are deterministic
[ ] Actual source fields are displayed
[ ] Plain-English explanations are generated
[ ] Chat interface works
[ ] Multi-turn questions work
[ ] Unknown information is handled safely
[ ] Frontend works
[ ] Backend works
[ ] APIs work
[ ] Tests pass
[ ] README is complete
[ ] .env.example exists
[ ] .gitignore exists
[ ] No secrets are committed
[ ] Git repository is initialized
[ ] Project is ready to push to GitHub

IMPORTANT:

Start by inspecting the current VS Code workspace.

If it is empty, create the entire project.

If code already exists, inspect it and integrate with it.

Do not delete useful existing work.

Do not provide only code snippets.

Actually create the files, install dependencies, run the project, test it, fix errors, and make the final project GitHub-ready.

At the end, show:

1. Final project structure
2. How to run it
3. Test results
4. Demo steps
5. GitHub push commands
6. Any remaining limitations
7. Recommended future improvements

Build the complete application now.
