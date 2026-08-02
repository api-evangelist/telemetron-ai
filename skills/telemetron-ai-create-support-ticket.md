---
name: Create a support ticket for a customer
description: >-
  Look up a Telemetron customer and open a support ticket that is
  auto-categorized and routed by the organization's rules.
api: openapi/telemetron-ai-ext-v1-openapi.yml
operations: [queryCustomer, getCustomer, createTicket]
generated: '2026-07-21'
method: generated
---

# Create a support ticket for a customer

Base URL: `https://admin.telemetron.ai/api/ext-v1`, auth via `x-api-key`
header.

## Steps

1. **Resolve the customer (optional)** — `POST /customer/queryCustomer`
   (`queryCustomer`) with `email`, or `GET
   /customer/getCustomer/{telemetronCustomerId}` (`getCustomer`) when you hold
   the ID. The record includes the customer's assigned devices, useful ticket
   context. A `404` means the customer does not exist yet — `createTicket`
   will auto-create them from `customerEmail`.
2. **Create the ticket** — `POST /ticket/createTicket` (`createTicket`).
   `title` (max 255 chars) is required, plus either `customerEmail` or
   `customerId`. Optional: `description`, `priority`
   (`low|medium|high|urgent`, default `medium`), `category`, and a
   `messages[]` transcript (`role`: `customer|agent|bot`, `content`).
3. **Confirm** — a `200` with `{ "success": true }` means the ticket was
   created; Telemetron auto-categorizes and routes it by the organization's
   routing rules.

## Rules

- An unknown `customerEmail` auto-creates the customer record — no separate
  `createOrUpdateCustomer` call is required first.
- Errors are `{ "error": "<description>" }`; `400` = missing title/customer
  reference, `401` = invalid `x-api-key`, `500` = retry with backoff.
