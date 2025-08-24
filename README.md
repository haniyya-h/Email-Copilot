# 📧 n8n Email Copilot Workflow

This repository contains an exported **n8n workflow JSON** that implements a “Silent Email Copilot” for customer support. It monitors Gmail for new support messages, generates AI-based draft replies, and lets human agents approve or reject drafts. Rejected drafts are automatically re-written and improved until accepted.

---

## ⚙️ How the Workflow Works

1. **📥 Gmail Trigger**
   - Watches a Gmail inbox for new incoming emails.
   - Fetches full message content (both plain text and HTML).

2. **📊 Ticket Logging (Google Sheets)**
   - Appends email details into a `tickets` sheet:  
     `timestamp, messageId, threadId, from, subject, plainText, html, draft, status, agentEmail, decision, decisionAt, rejectCount, promptVersion…`

3. **🤖 AI Draft Generation (Gemini)**
   - Uses Google Gemini AI (via the AI Agent node) to generate a concise, friendly draft reply.
   - Draft is saved back into the `tickets` sheet.

4. **👩‍💻 Agent Review (Gmail Send)**
   - Agent receives an email with:
     - The AI-generated draft.
     - Two action links:  
       ✅ **Approve & Send**  
       ❌ **Reject & Re-draft**

5. **✅ Approve Path (Webhook `/approve`)**
   - Updates ticket as *approved/sent* in the sheet.
   - Sends the draft as a reply to the customer in the same Gmail thread.
   - Logs decision and timestamp.

6. **❌ Reject Path (Webhook `/reject`)**
   - Increments `rejectCount` and bumps `promptVersion` (e.g. v1 → v2).
   - Re-generates a new draft using Gemini, including the rejection reason.
   - Updates the sheet with new draft + status `revised_pending_review`.
   - Sends the agent a new review email with updated Approve/Reject links.
   - Logs rejection reasons in a `prompt_library` sheet for long-term prompt improvement.

---

## 📂 Repository Contents

- `Email Copilot.json` → the exported n8n workflow.
- `README.md` → this documentation.

---

## 🚀 How to Use

### 1. Import into n8n
- Open your n8n dashboard → **Workflows** → **Import from File**.  
- Select `Email Copilot.json`.

### 2. Set up credentials
- **Gmail OAuth2** → for triggers and sending replies.  
- **Google Sheets OAuth2** → for ticket logging.  
- **Google Gemini API (PaLM)** → for AI drafting.

### 3. Prepare Google Sheet
Create a spreadsheet called **`Email Copilot`** with two tabs:
- **`tickets`**: must include columns like:  
  `timestamp, messageId, threadId, from, subject, plainText, html, draft, status, agentEmail, decision, decisionAt, rejectCount, promptVersion`
- **`prompt_library`**: include columns for global learning:  
  `id, tone, length, style, lastUpdate, notes`

### 4. Activate the workflow
- Turn the workflow **ON**.  
- This registers the production webhook URLs (e.g., `/approve`, `/reject`).  

### 5. Test it
1. Send a test email to your Gmail inbox.  
2. A row should appear in the `tickets` sheet.  
3. You (the agent) will receive an AI draft email.  
4. Click **Approve** → the draft is sent to the customer, ticket is marked approved.  
5. Click **Reject** → a new draft is generated and sent back for review.

---

## 🔁 Learning Loop

- Every **Approve** marks success, resets status to `sent`.
- Every **Reject** increments counters, logs reasons, and triggers a re-draft.  
- Over time, you can refine the AI prompts using data in `prompt_library`.

---

## 🛠️ Tech Stack

- [n8n](https://n8n.io) — workflow automation  
- Gmail API — triggers & replies  
- Google Sheets API — ticket logging  
- Google Gemini (PaLM) — AI draft generation  

---

## ✨ Future Improvements
- Aggregate rejection reasons from `prompt_library` to auto-adjust prompts.  
- Add analytics dashboard to visualize approve/reject rates.  
- Fine-tune prompts for specific agents or ticket categories.

---

