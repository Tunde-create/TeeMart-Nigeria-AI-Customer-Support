# TeeMart Nigeria — Node-by-Node Technical Guide

> Source: sanitized 21-node n8n workflow JSON, documented 24 October 2026. Configuration values shown below are taken from that export; placeholder account values must be replaced after import.

## End-to-end workflow

1. Gmail Trigger polls a selected Gmail label for incoming customer mail.
2. Edit Fields extracts customer name, email, subject, message snippet and Gmail message ID.
3. Text Classifier, powered by the first OpenAI Chat Model, routes the enquiry into one of six categories.
4. The appropriate category Edit Fields node adds an `intent`; all branches converge at Append row in sheet.
5. Google Sheets creates a `Pending` record; AI Agent and the second OpenAI Chat Model generate a proposed response.
6. Gmail creates a draft; Google Sheets updates the same record to `Draft Ready` and stores the AI response.
7. Gmail Send a message sends an approval request to the supervisor and waits for a decision.
8. The If node checks `$json.data.approved`: true leads to `Approved` → customer Gmail reply → `Sent`; false leads to `Rejected` and stops.

**Important:** Rejection means the supervisor rejected the proposed AI draft. The exported workflow does **not** send a rejection email to the customer.

## n8n expression basics

- `={{ ... }}` evaluates JavaScript-like n8n expressions at execution time.
- `$json` means the current incoming item.
- `$('Node name').item.json` reads the paired item from a named earlier node.
- `$now.toISO()` generates a current timestamp.
- `||` supplies a fallback when a value is missing or empty; `?.[1]` safely reads a regular-expression capture.

## Complete node reference

### 1. Gmail Trigger

**Node type:** `n8n-nodes-base.gmailTrigger`  
**Purpose:** Starts the automation by polling Gmail every minute, restricted to a configured label. This is polling, not an immediate push notification.

**Exported settings and expressions (account values sanitized):**

```json
{
  "pollTimes": {
    "item": [
      {
        "mode": "everyMinute"
      }
    ]
  },
  "filters": {
    "labelIds": [
      "REPLACE_WITH_YOUR_GMAIL_LABEL_ID"
    ]
  }
}
```

### 2. Edit Fields

**Node type:** `n8n-nodes-base.set`  
**Purpose:** Normalizes incoming Gmail data into five named fields. The customer name expression first looks for “my name is ...” in the message, then falls back to the sender display name or address. The message uses Gmail `snippet`, which may be truncated.

**Exported settings and expressions (account values sanitized):**

```json
{
  "assignments": [
    {
      "name": "customer_name",
      "value": "={{\n  (\n    ($json.Text || $json.text || $json.snippet || \"\")\n      .match(/\\bmy name is\\s+([A-Za-z]+)/i)?.[1]\n    ||\n    ($json.From || \"\").split(\"<\")[0].trim()\n    ||\n    ($json.From || \"\").match(/<([^>]+)>/)?.[1]\n    ||\n    \"Customer\"\n  ).trim()\n}}",
      "type": "string"
    },
    {
      "name": "customer_email",
      "value": "={{ ($json.From || \"\").match(/<([^>]+)>/)?.[1] || ($json.From || \"\").trim() }}",
      "type": "string"
    },
    {
      "name": "email_subject",
      "value": "={{ $json.Subject }}",
      "type": "string"
    },
    {
      "name": "customer_message",
      "value": "={{ $json.snippet || \"\" }}",
      "type": "string"
    },
    {
      "name": "message_id",
      "value": "={{ $json.id }}",
      "type": "string"
    }
  ],
  "options": {}
}
```

### 3. Text Classifier

**Node type:** `@n8n/n8n-nodes-langchain.textClassifier`  
**Purpose:** Combines subject and customer message, then uses the connected language model to choose one of six intent categories. Category descriptions guide the model; classification is probabilistic.

**Category definitions:**

- **Price Enquiry:** Customer enquiries primarily asking about product prices, costs, discounts, promotions, or payment amounts. If a message asks about several topics, choose this category when pricing is the main request.
- **Product Enquiry:** Customer enquiries primarily asking about product availability, specifications, features, brands, models, colours, sizes, or product recommendations. Use this category when product information is the main request rather than pricing, ordering, or delivery.
- **Order Placement:** Customer messages primarily requesting to place a new order, purchase a product, confirm a purchase, or proceed with checkout. Use this category when the customer clearly intends to buy a product rather than simply asking about its price or availability.
- **Delivery Enquiry:** Customer enquiries primarily asking about delivery locations, shipping fees, estimated delivery times, dispatch status, tracking information, or delivery arrangements. Use this category when delivery or shipping is the main concern rather than product pricing, product information, or order placement.
- **Complaint:** Customer messages primarily reporting dissatisfaction, damaged or defective products, incorrect orders, missing items, failed payments, delayed deliveries, refund requests, or poor customer service. Classify as Complaint when the customer is reporting a problem or requesting a resolution, even if the message also mentions delivery, pricing, or product information.
- **General Enquiry:** Customer messages primarily asking general questions about TeeMart Nigeria, including business location, opening hours, contact information, company policies, or other enquiries that do not fit into Price Enquiry, Product Enquiry, Order Placement, Delivery Enquiry, or Complaint. Use this category only when none of the other five categories is more appropriate.

**Exported settings and expressions (account values sanitized):**

```json
{
  "inputText": "={{ $json.email_subject + \"\\n\" + $json.customer_message }}",
  "options": {}
}
```

### 4. OpenAI Chat Model

**Node type:** `@n8n/n8n-nodes-langchain.lmChatOpenAi`  
**Purpose:** Provides GPT-5-mini to the Text Classifier via an AI model connection; it is not a standalone sequential processing step.

**Exported settings and expressions (account values sanitized):**

```json
{
  "model": {
    "__rl": true,
    "mode": "list",
    "value": "gpt-5-mini"
  },
  "builtInTools": {},
  "options": {}
}
```

### 5. Edit Fields1

**Node type:** `n8n-nodes-base.set`  
**Purpose:** On the Price Enquiry output, sets `intent` to Price Enquiry while preserving other fields.

**Exported settings and expressions (account values sanitized):**

```json
{
  "assignments": [
    {
      "name": "intent",
      "value": "Price Enquiry",
      "type": "string"
    }
  ],
  "includeOtherFields": true,
  "options": {}
}
```

### 6. Edit Fields2

**Node type:** `n8n-nodes-base.set`  
**Purpose:** On the Product Enquiry output, sets `intent` to Product Enquiry while preserving other fields.

**Exported settings and expressions (account values sanitized):**

```json
{
  "assignments": [
    {
      "name": "intent",
      "value": "Product Enquiry",
      "type": "string"
    }
  ],
  "includeOtherFields": true,
  "options": {}
}
```

### 7. Edit Fields3

**Node type:** `n8n-nodes-base.set`  
**Purpose:** On the Order Placement output, sets `intent` to Order Placement while preserving other fields.

**Exported settings and expressions (account values sanitized):**

```json
{
  "assignments": [
    {
      "name": "intent",
      "value": "Order Placement",
      "type": "string"
    }
  ],
  "includeOtherFields": true,
  "options": {}
}
```

### 8. Edit Fields4

**Node type:** `n8n-nodes-base.set`  
**Purpose:** On the Delivery Enquiry output, sets `intent` to Delivery Enquiry while preserving other fields.

**Exported settings and expressions (account values sanitized):**

```json
{
  "assignments": [
    {
      "name": "intent",
      "value": "Delivery Enquiry",
      "type": "string"
    }
  ],
  "includeOtherFields": true,
  "options": {}
}
```

### 9. Edit Fields5

**Node type:** `n8n-nodes-base.set`  
**Purpose:** On the Complaint output, sets `intent` to Complaint while preserving other fields.

**Exported settings and expressions (account values sanitized):**

```json
{
  "assignments": [
    {
      "name": "intent",
      "value": "Complaint",
      "type": "string"
    }
  ],
  "includeOtherFields": true,
  "options": {}
}
```

### 10. Edit Fields6

**Node type:** `n8n-nodes-base.set`  
**Purpose:** On the General Enquiry output, sets `intent` to General Enquiry while preserving other fields.

**Exported settings and expressions (account values sanitized):**

```json
{
  "assignments": [
    {
      "name": "intent",
      "value": "General Enquiry",
      "type": "string"
    }
  ],
  "includeOtherFields": true,
  "options": {}
}
```

### 11. Append row in sheet

**Node type:** `n8n-nodes-base.googleSheets`  
**Purpose:** Adds one customer enquiry record to Google Sheets, initially with Status Pending. The original Gmail message ID is the later update key.

**Exported settings and expressions (account values sanitized):**

```json
{
  "operation": "append",
  "documentId": {
    "__rl": true,
    "value": "REPLACE_WITH_YOUR_SPREADSHEET_ID",
    "mode": "id"
  },
  "sheetName": {
    "__rl": true,
    "value": "REPLACE_WITH_YOUR_SHEET_NAME_OR_ID",
    "mode": "id"
  },
  "columns": {
    "mappingMode": "defineBelow",
    "matchingColumns": [],
    "value": {
      "Timestamp": "={{ $now.toISO() }}",
      "Customer Name": "={{ $json.customer_name }}",
      "Customer Email": "={{ $json.customer_email }}",
      "Email Subject": "={{ $json.email_subject }}",
      "Customer Message": "={{ $json.customer_message }}",
      "Message ID": "={{ $json.message_id }}",
      "Intent": "={{ $json.intent }}",
      "Status": "Pending"
    }
  },
  "options": {}
}
```

### 12. AI Agent

**Node type:** `@n8n/n8n-nodes-langchain.agent`  
**Purpose:** Generates a customer-facing reply using the logged customer name, subject, category and message. Its prompt explicitly prohibits invented prices, stock, delivery dates, addresses and policies.

**Exported settings and expressions (account values sanitized):**

```json
{
  "promptType": "define",
  "text": "={{ \"You are a professional customer support assistant for TeeMart Nigeria, a retail and e-commerce business.\\n\\nPrepare a polite, clear and professional email reply to the customer.\\n\\nCustomer Name: \" + $json[\"Customer Name\"] + \"\\nEmail Subject: \" + $json[\"Email Subject\"] + \"\\nEnquiry Category: \" + $json[\"Intent\"] + \"\\nCustomer Message: \" + $json[\"Customer Message\"] + \"\\n\\nInstructions:\\n1. Acknowledge the customer's enquiry.\\n2. Address the customer politely.\\n3. Respond according to the enquiry category.\\n4. Do not invent product prices, stock availability, delivery dates, store addresses, or business policies.\\n5. Where information is unavailable, explain that the TeeMart team will confirm it.\\n6. For complaints, show empathy and offer appropriate follow-up.\\n7. Do not claim that an order or payment has been confirmed without verification.\\n8. End with 'TeeMart Nigeria Customer Support'.\\n9. Produce only the email reply, without additional explanations.\" }}",
  "options": {}
}
```

### 13. OpenAI Chat Model1

**Node type:** `@n8n/n8n-nodes-langchain.lmChatOpenAi`  
**Purpose:** Provides GPT-5-mini to the AI Agent through an AI model connection.

**Exported settings and expressions (account values sanitized):**

```json
{
  "model": {
    "__rl": true,
    "mode": "list",
    "value": "gpt-5-mini"
  },
  "builtInTools": {},
  "options": {}
}
```

### 14. Create a draft

**Node type:** `n8n-nodes-base.gmail`  
**Purpose:** Creates a Gmail draft addressed to the original customer; the reply is not yet sent. Subject prepends Re: and body uses the AI Agent output.

**Exported settings and expressions (account values sanitized):**

```json
{
  "resource": "draft",
  "subject": "={{ 'Re: ' + $('Append row in sheet').item.json['Email Subject'] }}",
  "message": "={{ $json.output }}",
  "options": {
    "sendTo": "={{ $('Append row in sheet').item.json['Customer Email'] }}"
  }
}
```

### 15. Update row in sheet

**Node type:** `n8n-nodes-base.googleSheets`  
**Purpose:** Matches the original message ID, stores the generated AI reply in the `Ai Response` column and sets Status Draft Ready.

**Exported settings and expressions (account values sanitized):**

```json
{
  "operation": "update",
  "documentId": {
    "__rl": true,
    "value": "REPLACE_WITH_YOUR_SPREADSHEET_ID",
    "mode": "id"
  },
  "sheetName": {
    "__rl": true,
    "value": "REPLACE_WITH_YOUR_SHEET_NAME_OR_ID",
    "mode": "id"
  },
  "columns": {
    "mappingMode": "defineBelow",
    "matchingColumns": [
      "Message ID"
    ],
    "value": {
      "Message ID": "={{ $('Append row in sheet').item.json['Message ID'] }}",
      "Ai Response": "={{ $('AI Agent').item.json.output }}",
      "Status": "Draft Ready"
    }
  },
  "options": {}
}
```

**Why the matching column matters:** The update finds the existing record by `Message ID`, instead of creating another row. Unmapped fields should remain untouched.

### 16. Send a message

**Node type:** `n8n-nodes-base.gmail`  
**Purpose:** Sends the supervisor a Gmail approval request and waits for Approve/Reject. The address in the sanitized export is a placeholder.

**Exported settings and expressions (account values sanitized):**

```json
{
  "operation": "sendAndWait",
  "sendTo": "supervisor@example.com",
  "subject": "TeeMart Nigeria - Customer Reply Approval Required",
  "message": "={{ 'Hello TeeMart Supervisor,\\n\\nAn AI-generated customer reply is awaiting your approval.\\n\\nPROPOSED REPLY:\\n\\n' + $('AI Agent').item.json.output + '\\n\\nPlease review the response and approve or reject it using the response options provided.\\n\\nTeeMart Nigeria - AI Customer Support' }}",
  "approvalOptions": {
    "values": {
      "approvalType": "double",
      "disapproveLabel": "Reject"
    }
  },
  "options": {}
}
```

### 17. If

**Node type:** `n8n-nodes-base.if`  
**Purpose:** Checks the boolean `data.approved`. True follows the approval branch; false follows rejection.

**Exported settings and expressions (account values sanitized):**

```json
{
  "conditions": {
    "options": {
      "caseSensitive": true,
      "leftValue": "",
      "typeValidation": "strict",
      "version": 3
    },
    "conditions": [
      {
        "id": "afdf8e0e-65e5-49a9-9e25-5aa0b83e4a2d",
        "leftValue": "={{ $json.data.approved }}",
        "rightValue": "",
        "operator": {
          "type": "boolean",
          "operation": "true",
          "singleValue": true
        }
      }
    ],
    "combinator": "and"
  },
  "options": {}
}
```

### 18. Update row in sheet1

**Node type:** `n8n-nodes-base.googleSheets`  
**Purpose:** Rejection branch: sets the matched Google Sheets record Status to Rejected. No customer email follows this node in the export.

**Exported settings and expressions (account values sanitized):**

```json
{
  "operation": "update",
  "documentId": {
    "__rl": true,
    "value": "REPLACE_WITH_YOUR_SPREADSHEET_ID",
    "mode": "id"
  },
  "sheetName": {
    "__rl": true,
    "value": "REPLACE_WITH_YOUR_SHEET_NAME_OR_ID",
    "mode": "id"
  },
  "columns": {
    "mappingMode": "defineBelow",
    "matchingColumns": [
      "Message ID"
    ],
    "value": {
      "Message ID": "={{ $('Append row in sheet').item.json['Message ID'] }}",
      "Status": "Rejected"
    }
  },
  "options": {}
}
```

**Why the matching column matters:** The update finds the existing record by `Message ID`, instead of creating another row. Unmapped fields should remain untouched.

### 19. Update row in sheet2

**Node type:** `n8n-nodes-base.googleSheets`  
**Purpose:** Approval branch: sets the matched record Status to Approved before sending.

**Exported settings and expressions (account values sanitized):**

```json
{
  "operation": "update",
  "documentId": {
    "__rl": true,
    "value": "REPLACE_WITH_YOUR_SPREADSHEET_ID",
    "mode": "id"
  },
  "sheetName": {
    "__rl": true,
    "value": "REPLACE_WITH_YOUR_SHEET_NAME_OR_ID",
    "mode": "id"
  },
  "columns": {
    "mappingMode": "defineBelow",
    "matchingColumns": [
      "Message ID"
    ],
    "value": {
      "Message ID": "={{ $('Append row in sheet').item.json['Message ID'] }}",
      "Status": "Approved"
    }
  },
  "options": {}
}
```

**Why the matching column matters:** The update finds the existing record by `Message ID`, instead of creating another row. Unmapped fields should remain untouched.

### 20. Send a message1

**Node type:** `n8n-nodes-base.gmail`  
**Purpose:** Sends the AI Agent reply as a plain-text email to the original customer after supervisor approval.

**Exported settings and expressions (account values sanitized):**

```json
{
  "sendTo": "={{ $('Append row in sheet').item.json['Customer Email'] }}",
  "subject": "={{ 'Re: ' + $('Append row in sheet').item.json['Email Subject'] }}",
  "emailType": "text",
  "message": "={{ $('AI Agent').item.json.output }}",
  "options": {}
}
```

### 21. Update row in sheet3

**Node type:** `n8n-nodes-base.googleSheets`  
**Purpose:** After Gmail sending, updates the matched record Status to Sent without overwriting other columns.

**Exported settings and expressions (account values sanitized):**

```json
{
  "operation": "update",
  "documentId": {
    "__rl": true,
    "value": "REPLACE_WITH_YOUR_SPREADSHEET_ID",
    "mode": "id"
  },
  "sheetName": {
    "__rl": true,
    "value": "REPLACE_WITH_YOUR_SHEET_NAME_OR_ID",
    "mode": "id"
  },
  "columns": {
    "mappingMode": "defineBelow",
    "matchingColumns": [
      "Message ID"
    ],
    "value": {
      "Message ID": "={{ $('Append row in sheet').item.json['Message ID'] }}",
      "Status": "Sent"
    }
  },
  "options": {}
}
```

**Why the matching column matters:** The update finds the existing record by `Message ID`, instead of creating another row. Unmapped fields should remain untouched.

## Google Sheets columns and statuses

The sheet uses `Timestamp`, `Customer Name`, `Customer Email`, `Email Subject`, `Customer Message`, `Message ID`, `Intent`, `Status`, and `Ai Response` (capitalization must match your actual sheet).

| Status | Meaning |
|---|---|
| Pending | New enquiry appended |
| Draft Ready | AI reply generated and recorded |
| Approved | Supervisor approved; send step follows |
| Rejected | Supervisor rejected the proposed draft; no automatic customer message |
| Sent | Customer sending node completed and sheet updated |

## Worked test — 24 October 2026

A customer asked about Samsung Galaxy A55 price and availability. The classifier selected Price Enquiry. The AI Agent drafted a response without asserting an unverified price or stock level. The supervisor approved; the customer received the Gmail reply; the corresponding Sheets record reached `Sent` with `Ai Response` preserved. This confirms the approved branch in that test, not every failure scenario.

## Setup after import

1. Create your own Gmail label and replace `REPLACE_WITH_YOUR_GMAIL_LABEL_ID`.
2. Connect your own Gmail OAuth2 credentials to all Gmail nodes.
3. Connect your own Google Sheets credentials; replace spreadsheet and sheet placeholders in all Sheets nodes.
4. Connect your OpenAI credentials to both model nodes.
5. Replace `supervisor@example.com` with an authorized supervisor address.
6. Create the required sheet columns with matching capitalization.
7. Test classification, approval, rejection, sending and status updates using non-sensitive sample data.
8. Keep the unsanitized original backup private.

## Known limitations and safeguards

- A rejected draft is logged but not sent to the customer or revised automatically.
- A Gmail send failure may leave the status at `Approved`; add explicit error handling and retries before production use.
- No duplicate-email detection is implemented in this export.
- Gmail message snippets may truncate longer emails.
- The reply uses a new subject with `Re:`; threading into the original conversation is not guaranteed.
- The AI-generated response may still include unsupported promises (such as a 24–48-hour turnaround) despite prompt instructions; human review remains essential.
- Publishing sanitized workflow JSON does not configure working credentials for other users.

---

**Repository:** `TeeMart-Nigeria-AI-Customer-Support`  
**Source of truth:** `workflow/TeeMart-Nigeria-Workflow-Sanitized.json`
