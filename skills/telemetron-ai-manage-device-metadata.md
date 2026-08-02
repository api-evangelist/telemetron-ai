---
name: Manage device metadata at fleet scale
description: >-
  Keep Telemetron device metadata current — single-device upserts or
  full-replacement bulk updates for up to 1,000 devices per request.
api: openapi/telemetron-ai-ext-v1-openapi.yml
operations: [createOrUpdateDevice, bulkUpdateDeviceMetadata, unassignDeviceFromOwner, unassignDevicesFromOwner]
generated: '2026-07-21'
method: generated
---

# Manage device metadata at fleet scale

Base URL: `https://admin.telemetron.ai/api/ext-v1`, auth via `x-api-key`
header.

## Steps

1. **Single device** — `POST /device` (`createOrUpdateDevice`) with
   `identifier` plus the fields to change; include `type` only when creating.
   `properties` is a free-form metadata object.
2. **Bulk metadata** — `POST /device/bulk-metadata`
   (`bulkUpdateDeviceMetadata`) with a `devices` map of
   `identifier -> metadata object`, 1–1,000 entries. This is a **full
   replacement** of each device's metadata, not a merge — send the complete
   object every time. It never creates devices; the response reports each
   identifier as `updated` or `not_found` with `updated`/`notFound` counts.
3. **Retire ownership when devices change hands** —
   `POST /deviceAssignment/unassignDeviceFromOwner`
   (`unassignDeviceFromOwner`, one device/one customer) or
   `POST /deviceAssignment/unassignDevicesFromOwner`
   (`unassignDevicesFromOwner`, many devices/one customer). Records remain;
   only the association (and telemetry routing) is removed.

## Rules

- Bulk metadata replaces, single-device upsert merges field-by-field at the
  top level — choose accordingly.
- Cap bulk requests at 1,000 devices; chunk larger fleets.
- Errors are `{ "error": "<description>" }` with standard status codes
  (`400`, `401`, `404`, `500`); retry `500`s with backoff.
