# AI-SDR-Installation-N8N

An advanced **n8n AI SDR (Sales Development Representative) workflow** designed to automatically handle **email replies**, detect booking intent, check **Calendly availability**, book appointments, generate human-like email responses, and send replies back through **Instantly.ai**.

This workflow is ideal for **AI appointment setters**, **podcast guest booking systems**, and **sales outreach reply automation**.

---

## Features

- Receives inbound email replies via **Webhook**
- Detects whether a lead is:
  - interested
  - not interested
  - requesting booking info
- Extracts and normalizes appointment dates from email replies
- Supports relative date understanding such as:
  - tomorrow
  - coming Tuesday
  - next week
- Uses **Anthropic Claude models** for intent analysis and reply generation
- Checks **Calendly availability**
- Books appointments automatically when a slot is available
- Generates polished **HTML email replies**
- Preserves and corrects weekday/date formatting
- Sends replies back using **Instantly.ai**
- Includes memory for ongoing conversation context
- Uses Google Docs as a **knowledge base tool**

---

## Workflow Overview

This workflow follows the process below:

1. A lead replies to an outbound email campaign
2. **Instantly.ai** sends the reply to the n8n **Webhook**
3. The workflow extracts the latest user message from the email thread
4. AI analyzes whether the lead is interested and whether a booking date/time was mentioned
5. If a booking date is found:
   - the workflow checks Calendly availability
   - prepares booking context
6. The main AI SDR agent responds as **Mickey**
7. If the requested slot is available:
   - the workflow books the appointment
8. If the slot is unavailable:
   - the AI suggests alternate times
9. The email is cleaned and validated into final HTML
10. The workflow sends the reply back to the lead through **Instantly.ai**

---

## Main Use Case

This workflow is built for **AI SDR automation** where leads reply to cold outreach emails and the system handles:

- qualification
- scheduling
- objection handling
- follow-up
- appointment setting

A key use case in this workflow is **podcast guest booking**.

---

## Nodes Used

### 1. Webhook: Reply Received
The workflow begins with a **Webhook** node:

- **Path:** `hermes-ai-sdr-v2`

This receives reply events from **Instantly.ai** whenever a lead responds to an outreach email.

---

### 2. Edit Fields3
This node normalizes and stores important fields such as:

- sender email
- sender name
- Calendly API token
- Instantly API token
- message body

It acts as the central preparation layer before AI processing.

---

### 3. Code in JavaScript3
This node extracts only the **newest reply content** from the full email thread.

It removes quoted thread history like:

- previous replies
- forwarded content
- email chain context

This ensures AI only analyzes the lead’s latest message.

---

### 4. AI Agent4
This AI agent analyzes the lead’s message and determines:

- whether the lead is interested
- whether a booking time/date is mentioned

Expected structured output:

```json
{
  "status": "Interested",
  "start_time": "2026-01-20T00:00:00Z"
}
```

This is the first decision layer in the SDR pipeline.

---

### 5. Code in JavaScript4
This node extracts clean JSON from the AI response and ensures the workflow always receives usable output.

If parsing fails, it defaults to:

```json
{
  "status": "Not Interested",
  "start_time": null
}
```

---

### 6. If1
This node checks whether:

- `start_time` exists

If yes, the workflow continues into the **availability checking path**.

If not, it skips direct scheduling logic and moves toward AI email reply generation.

---

## Calendly Scheduling Flow

If the lead mentioned a specific date/time, the workflow continues into the booking logic.

---

### 7. Date & Time1
This node adds a **2-hour window** to the requested start time.

This helps define the booking search range.

---

### 8. URI GET Calendly1
Fetches the authenticated Calendly user profile using the stored API token.

---

### 9. Get List Events1
Fetches active event types from Calendly for the target user.

This helps determine which event type to use for availability and booking.

---

### 10. Event Type Available Times 1
Checks whether the requested slot is available in Calendly.

It uses:

- event type URI
- requested start time
- generated end time
- timezone: `America/Denver`

---

### 11. Edit Fields2
Stores normalized booking fields such as:

- event type URI
- user URI
- start time
- end time

This prepares data for downstream booking logic.

---

## Main AI SDR Agent

### 12. AI Agent2
This is the **core AI sales assistant** of the workflow.

It is heavily instructed to act as:

- **Mickey**
- not as an assistant
- not as AI
- not as a bot

It handles:

- appointment scheduling replies
- availability discussions
- alternate slot suggestions
- natural sales-style email communication

### Core responsibilities:
- respond like a human SDR
- check availability using tools
- book appointments when possible
- ask for better times if needed
- maintain professional and persuasive tone

---

## AI Tools Connected to Main Agent

### 13. get_availability_in_my_calender
A tool that checks available slots in Calendly over a defined timeframe.

It is used when the AI needs to:

- confirm requested times
- suggest alternate times
- provide available slots

---

### 14. appointment_book1
A tool used by the AI agent to book a Calendly appointment through a webhook-based booking endpoint.

It includes:

- event type
- start time
- invitee name
- invitee email
- timezone
- Zoom conference location

---

### 15. Code Tool
A helper tool used to correctly determine:

- weekday name
- formatted date

This ensures responses like:

```text
Tuesday, January 20 at 1:30 PM MT
```

are always accurate.

---

### 16. Get a Document in Google Docs1
This node acts as a **knowledge base tool** for the AI.

It allows the AI SDR to retrieve internal instructions or context from a connected Google Doc.

Useful for:

- sales messaging guidance
- booking policy
- brand tone
- FAQ references

---

### 17. Simple Memory1
Maintains conversation memory for the lead using their email address as session key.

This helps the AI stay context-aware during ongoing reply chains.

---

## Email Formatting & Validation Layer

### 18. AI Agent3
This agent receives the generated subject + HTML email and performs **final validation**.

Its main role is:

- preserve HTML exactly
- validate weekday names against actual dates
- fix weekday only if incorrect
- avoid rewriting the message

It outputs clean structured JSON:

```json
{
  "subject": "Re: Example Subject",
  "email_html": "<p>Example HTML email</p>"
}
```

---

### 19. Structured Output Parser
Ensures the final output from AI Agent3 is valid structured JSON.

---

## Instantly.ai Reply Flow

### 20. Get Thread Id of Emaiil1
Searches Instantly.ai for the email thread using the sender email.

This identifies the correct thread for reply continuation.

---

### 21. Get UUID1
Fetches the specific email record from the thread to determine:

- reply target UUID
- campaign context

---

### 22. Reply an Email1
Sends the final reply email back through **Instantly.ai**.

It includes:

- email account
- thread reply target
- recipient email
- subject
- final HTML body

This completes the SDR automation cycle.

---

## AI Models Used

This workflow uses multiple **Anthropic Claude models**:

- **Claude Haiku 3**
- **Claude Sonnet 4**
- **Claude Sonnet 4.5**

These are used for:

- reply intent detection
- booking analysis
- email generation
- HTML correction

---

## Example Input (Webhook Payload)

This workflow expects a webhook payload similar to:

```json
{
  "body": {
    "email": "operations@systemapic.agency",
    "reply_text": "Could you Book me for Tuesday",
    "reply_subject": "Re: <> Would you be open to being a guest?",
    "email_account": "mpendergast@tryschooleymitchell.com",
    "lead_email": "operations@systemapic.agency"
  }
}
```

---

## Example AI Intent Output

```json
{
  "status": "Interested",
  "start_time": "2026-01-20T00:00:00Z"
}
```

---

## Example Final AI Reply Behavior

If a lead says:

> “Could you book me for Tuesday?”

The workflow can:

1. understand intent
2. resolve the date
3. check availability
4. book the slot if available
5. send a human-style reply

Example response style:

```html
<p>Perfect — you're confirmed for <strong>Tuesday, January 20 at 1:30 PM MT</strong>.</p>
<p>You'll receive a calendar invite shortly with all the details.</p>
<p>Looking forward to it.</p>
<p>— Mickey</p>
```

---

## Requirements

To run this workflow successfully, you need:

- **n8n**
- **Anthropic API credentials**
- **Calendly API token**
- **Instantly.ai API token**
- **Google Docs OAuth2 credentials**

---

## External Services Used

This workflow integrates with:

- **Anthropic Claude**
- **Calendly API**
- **Instantly.ai API**
- **Google Docs**

---

## Setup Instructions

1. Import the JSON workflow into n8n
2. Connect your **Anthropic API credentials**
3. Replace the hardcoded **Calendly API token**
4. Replace the hardcoded **Instantly API token**
5. Connect your **Google Docs account**
6. Update the Google Docs knowledge base document if needed
7. Configure your webhook in Instantly.ai
8. Activate the workflow
9. Send a test email reply
10. Confirm that:
   - intent is detected correctly
   - date extraction works
   - Calendly availability is checked
   - appointment booking works
   - reply is sent correctly through Instantly.ai

---

## Important Notes

- The workflow is currently **inactive**
- Timezone handling is based on:
  - `America/Denver`
- The AI agent is strongly persona-driven and responds as **Mickey**
- The system is optimized for **email-based appointment setting**
- It includes strict rules to avoid:
  - wrong weekday names
  - incorrect year assumptions
  - AI-style robotic wording

---

## Security Notes

Before deploying this workflow publicly, you should:

- remove all hardcoded API keys and tokens
- move secrets into n8n credentials or environment variables
- secure the webhook endpoint
- validate all incoming payloads
- restrict tool access where needed

---

## Suggested Improvements

You can improve this workflow further by adding:

- CRM sync after booking
- lead stage updates
- no-show follow-up automation
- cancellation / reschedule logic
- multi-calendar routing
- timezone auto-detection
- internal Slack alerts
- booking analytics dashboard
- fallback handling for ambiguous replies
- sentiment scoring for lead qualification

---

## Recommended Workflow Pairing

This workflow works very well when paired with:

- outbound cold email workflows
- lead enrichment systems
- CRM contact creation workflows
- AI follow-up sequences
- podcast guest intake pipelines

A full automation system could work like this:

1. outbound email sent
2. lead replies
3. this AI SDR handles the conversation
4. appointment is booked automatically
5. CRM and follow-up systems continue the pipeline

This creates a complete **AI SDR sales automation engine**.

---

## Real-World Applications

This workflow is ideal for:

- podcast guest outreach automation
- sales appointment setting
- inbound lead booking
- AI SDR agencies
- high-ticket offer qualification
- founder-led outreach systems
- B2B cold email reply handling

---

## Author

Built as an **n8n AI SDR and appointment-setting automation workflow** for outbound sales systems and email-based booking funnels.

---

## License

You can add your preferred license here, such as:

- MIT
- Apache-2.0
- Proprietary
