---
name: formstep-requests
description: Ask a customer for information through Formstep and read the verified answers back, with a published form or one built in the same call. Use when you need a confirmation, a document, a signature, a payment, a booking or missing details from one named person.
---

# Formstep requests

A request is one form assigned to one recipient: its own link, its own prefilled values, its own expiry, its own outcome. Use `formShareLink_create` instead when anyone may answer.

1. Find the form with `form_list`. Ask the user which one if several match. If no form fits, build one in the request instead (see below).
2. Call `fields_list` with the form id. It returns every field key with its value type, option keys and a `usage` line saying whether it goes in `prefill` (visible questions) or `context` (hidden fields). Prefill takes option keys, never labels.
3. Call `request_create` with `formId` or `form`, `recipient` (email, name), `prefill` for values you know, `readonly` for prefilled values the recipient must not change, `context` for hidden fields, and `delivery` (`"email"` lets Formstep send the invitation, which needs Pro and `recipient.email`; `"none"`, the default, returns the link for you to send). Pass `reminders` (offsets like `["2d","5d"]`, Pro, needs `recipient.email`) when the user asks for them, or `[]` for none; omitting it keeps the form's own schedule. Pass `expiresAt` (epoch ms, default 30 days) when the user gives a deadline, and `callbackUrl` if the user has a webhook to receive the answers.
4. Give the user the request id and link. Poll `request_get` when asked, or let the callback tell you. Status moves from `pending` to `completed`, `expired` or `canceled`; a completed request carries `answers` keyed by field key.
5. `request_remind` sends one reminder now (Pro, at most 8 reminders per request, one manual send per 10 minutes). `request_cancel` closes the link and cannot be undone; it is marked destructive, so confirm with the user first. `request_replayCallback` re-sends a callback that never landed.

## No form yet: build it in the request

Pass `form` instead of `formId` and `request_create` builds an ordinary form, publishes it and creates the request in one call. Skip steps 1 and 2: the keys are the ones you give the blocks.

```json
{
  "form": {
    "name": "Documents for your application",
    "baseline": "<form id to copy the logo, theme and emails from>",
    "blocks": [
      { "type": "paragraph", "content": "Hi {{first_name}}, please upload your payslips." },
      { "type": "hiddenField", "key": "first_name" },
      { "type": "file", "key": "payslips", "title": "Your last three payslips", "maxFiles": 3 },
      { "type": "select", "key": "employment", "title": "Employment", "options": ["Employed", "Self-employed"] }
    ]
  },
  "recipient": { "email": "ada@example.com", "name": "Ada" },
  "context": { "first_name": "Ada" },
  "prefill": { "employment": "self_employed" }
}
```

- Each block takes the parameters of its `editor_insert*` tool, without `formId` and `after`, plus `key`. Set `key` on every question; the answers come back under it.
- Keep the definition the same for every recipient: the same `form` reuses the published form (`reused: true`). Personalise through `prefill` and `context` (`{{key}}` in text shows a value), never through the name or a block.
- Prefill a choice question with option keys derived from the labels: "Self-employed" becomes `self_employed`.
- Before a `payment` or `scheduleAppointment` block, call `workspaceConnection_list`. If Stripe or Cal.com is missing, give the user its `connectUrl`; you cannot connect it yourself. `scheduler_listEventTypes` lists the event types a booking can name.
- The response adds `form: { id, name, reused, fields }`. Pass `form.id` to `formSubmission_list` to read every answer to that form at once.

Do not invent field keys. Re-read `fields_list` after any `form_publish`. Call `load_skill` with `requests` for the full reference; section 2c lists every block type and form option.
