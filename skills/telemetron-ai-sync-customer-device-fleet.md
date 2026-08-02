---
name: Sync customers and devices into Telemetron
description: >-
  Register customers and devices in Telemetron and map device ownership so
  telemetry is correlated to the right customer for AI-powered hardware support.
api: openapi/telemetron-ai-ext-v1-openapi.yml
operations: [createOrUpdateCustomer, createOrUpdateDevice, assignDeviceToOwners, assignDevicesToOwner]
generated: '2026-07-21'
method: generated
---

# Sync customers and devices into Telemetron

Base URL: `https://admin.telemetron.ai/api/ext-v1`. Authenticate every request
with the organization API key in the `x-api-key` header (issued in the
dashboard under Settings > Integrations). All requests and responses are JSON.

## Steps

1. **Upsert the customer** — `POST /customer/createOrUpdateCustomer`
   (`createOrUpdateCustomer`). `email` is required and is the unique
   identifier. Omit `telemetronCustomerId` to create (a `409` means the email
   already exists — re-run with the existing `telemetronCustomerId` to update).
   Capture `telemetronCustomerId` from the response.
2. **Upsert each device** — `POST /device` (`createOrUpdateDevice`). Send a
   unique `identifier` (serial, MAC, etc.); `type` is required only when
   creating. Optional `properties` is a free-form metadata object.
3. **Map ownership** — one device to one or more owners with
   `POST /deviceAssignment/assignDeviceToOwners` (`assignDeviceToOwners`,
   `deviceIdentifier` + `customerEmails[]`), or many devices to one customer
   with `POST /deviceAssignment/assignDevicesToOwner`
   (`assignDevicesToOwner`, `customerEmail` + `deviceIdentifiers[]`).
   Both auto-create unknown customers; set `autoCreateDevice(s): true` plus
   `deviceType` to auto-create devices in the same call.

## Rules

- Writes are natural upserts (keyed on email / device identifier), so retries
  converge on the same state — there is no `Idempotency-Key` header.
- Errors come back as a flat `{ "error": "<description>" }` envelope; branch on
  the HTTP status (`400` bad params, `401` bad key, `409` duplicate email,
  `500` retry with backoff).
- Multi-owner (shared/family) devices are supported: assigning does not remove
  existing owners.
