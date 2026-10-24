# TeeMart Nigeria – AI Customer Support Automation

**An AI-powered email customer support workflow built with n8n, OpenAI, Gmail, and Google Sheets, featuring human-in-the-loop approval before customer replies are sent.**

## 1. Project Overview

TeeMart Nigeria is a fictional Nigerian retail and e-commerce business used as a practical AI automation capstone project.

The system automates customer email intake, enquiry classification, response drafting, supervisor approval, customer email delivery, and enquiry tracking.

The project demonstrates how AI and low-code automation can help retail businesses manage customer enquiries while retaining human oversight.

## 2. Business Problem

Retail and e-commerce businesses frequently receive customer emails about:

- Product prices
- Product availability and specifications
- New orders
- Delivery arrangements
- Complaints and refunds
- General business enquiries

Handling these enquiries manually can lead to delays, inconsistent categorisation, and poor tracking.

This workflow provides a structured AI-assisted customer support process.

## 3. Technologies Used

| Technology | Purpose |
|---|---|
| n8n | Workflow automation and orchestration |
| OpenAI GPT-5-mini | Customer enquiry classification and response generation |
| Gmail | Incoming emails, draft creation, supervisor approval and customer replies |
| Google Sheets | Customer enquiry logging and status tracking |
| GitHub | Source control, documentation and portfolio presentation |

## 4. Workflow Architecture

The workflow follows this process:

Customer Email → Gmail Trigger → Edit Fields → AI Text Classifier → Category Routing → Google Sheets Logging → AI Agent → Gmail Draft → Supervisor Approval → Decision Routing

**If approved:**

Update Status to Approved → Send Customer Email → Update Status to Sent

**If rejected:**

Update Status to Rejected → Stop automated processing

The current workflow contains **21 nodes**.

## 5. Customer Enquiry Categories

The AI Text Classifier identifies six categories:

1. Price Enquiry
2. Product Enquiry
3. Order Placement
4. Delivery Enquiry
5. Complaint
6. General Enquiry

Each category is routed through its own Edit Fields node before the enquiry is logged in Google Sheets.

## 6. Human-in-the-Loop Approval

The AI Agent prepares a proposed customer response using the enquiry details.

A Gmail draft is created, and the proposed response is stored in Google Sheets.

A supervisor receives an approval request with two options:

- **Approve:** The workflow sends the proposed response to the customer.
- **Reject:** The workflow records the rejection in Google Sheets and stops automated sending.

**Important:** The current version does not automatically send a rejection notification to the customer. Rejected drafts require further human handling.

## 7. Google Sheets Logging

The customer support log contains the following columns:

| Column | Field |
|---|---|
| A | Timestamp |
| B | Customer Name |
| C | Customer Email |
| D | Email Subject |
| E | Customer Message |
| F | Message ID |
| G | Intent |
| H | Status |
| I | Ai Response |

The workflow uses the original Gmail Message ID to identify the correct record when updating its status.

Supported workflow statuses include:

- Pending
- Draft Ready
- Approved
- Rejected
- Sent

## 8. Testing and Validation

The six enquiry categories were tested during development.

A successful end-to-end test was completed on **24 October 2026**, involving an enquiry about Samsung Galaxy A55 pricing and availability.

The test confirmed:

- Customer email received by Gmail Trigger
- AI enquiry classification
- Customer information logged in Google Sheets
- AI-generated customer response
- Supervisor approval
- Approved reply sent through Gmail
- Customer receipt of the reply
- Final Google Sheets status updated to Sent
- AI-generated response preserved in the log

**Result: Successful end-to-end approved-response test.**

## 9. Project Structure

The repository is intended to contain:

- `README.md` – Project overview
- `workflow/` – Sanitized n8n workflow JSON
- `docs/` – Technical documentation and node-by-node explanations
- `screenshots/` – Redacted workflow and testing screenshots

Files will be added progressively.

## 10. Installation and Setup

To reproduce the project:

1. Install or access a compatible n8n instance.
2. Import the sanitized workflow JSON.
3. Configure Gmail OAuth2 credentials.
4. Configure Google Sheets OAuth2 credentials.
5. Configure OpenAI model access.
6. Create a Google Sheets support log with the required columns.
7. Update spreadsheet references, Gmail labels and supervisor email settings.
8. Test the workflow with sample customer enquiries before activating it.

The published workflow must not contain working credentials, personal account identifiers or customer data.

## 11. Current Limitations

The following improvements are planned:

- Supervisor revision and resubmission of rejected AI drafts
- Error handling and fallback notifications
- Duplicate email detection
- Improved email conversation threading
- Stronger controls against unsupported AI response-time promises
- Further end-to-end testing across all six enquiry categories

## 12. Project Status

**Core approved-response workflow: Successfully implemented and tested.**

The project remains a capstone demonstration and requires further reliability and security improvements before production deployment.

## 13. Learning Outcomes

This project demonstrates practical experience in:

- Designing n8n automation workflows
- Integrating Gmail and Google Sheets
- Applying AI-powered text classification
- Building AI-assisted customer support systems
- Implementing human approval controls
- Writing n8n expressions
- Testing and debugging multi-node workflows
- Documenting automation projects on GitHub

---

**Project:** TeeMart Nigeria – AI Customer Support Capstone  
**Platform:** n8n Cloud  
**Latest documented milestone:** 24 October 2026
