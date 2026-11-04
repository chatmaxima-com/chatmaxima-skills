# ChatMaxima API v2 endpoints

Base URL `https://chatmaxima.com/api/v2`. Header `Authorization: <API key>` (no `Bearer`). JSON bodies need `Content-Type: application/json`. Every path ends with `/`. Times are UTC. Full pages: https://docs.chatmaxima.com/api. OpenAPI 3.1: https://docs.chatmaxima.com/openapi/chatmaxima-api-v2.json.

## Errors on any endpoint

| HTTP | `error.code` | Meaning |
|---|---|---|
| 401 | `MISSING_API_KEY` | No `Authorization` header |
| 401 | `INVALID_API_KEY` | Wrong or rotated key |
| 403 | `PLAN_NO_API_ACCESS` | Plan has no API access (`details.upgrade_url`) |
| 403 | `PLAN_EXPIRED` | Send endpoints: plan ended |
| 403 | `MESSAGE_LIMIT_REACHED` | Send endpoints: monthly allowance used up |
| 400 | `INVALID_JSON` | Body is not valid JSON |
| 400 | `MISSING_REQUIRED_FIELDS` | A required field is missing |
| 429 | `RATE_LIMIT_EXCEEDED` | Wait `Retry-After` seconds |

## Channels

**`GET /channels/accounts/`** `?platform=WhatsApp,Instagram` (optional, comma-separated: WhatsApp, Instagram, Facebook, Telegram, SMS, Email, Ticket, Web, GoHighLevel, WhatsApp Voice, Twilio Voice, Exotel). Returns `data[]` of `{account_id, account_name, platform_name, account_identifier, account_status (active|pending), account_created_datetime}`, all in one response.

## Messages (any channel)

**`POST /messages/send/`**

| Field | Notes |
|---|---|
| `account_id` | required |
| `contact_id` / `contact_ref_id` / `to` | exactly one; `to` = phone (WhatsApp Cloud API, SMS) or email (email channels) |
| `text` | up to 4096 chars; required unless `media_url`; always required on email |
| `media_url`, `media_type` (`image`, `video`, `audio`, `file`), `file_name` | non-WhatsApp channels need a URL from `POST /media/` |
| `subject` | email only, required there, up to 250 chars |
| `conversation_id` | web chat only |
| `reply_to` | id of a customer message to answer (e.g. a reply webhook's `data.message_id`); quoted on WhatsApp Cloud API and Messenger |
| `message_ref_id` | idempotency key, 24 h, up to 100 bytes |

201 `data`: `{message_id, provider_message_id, account_id, channel, contact_id, conversation_id, direction, status, error, sent_at, message_ref_id, duplicate}`.

Errors: 404 `ACCOUNT_NOT_FOUND`, `CONTACT_NOT_ON_CHANNEL`, `UNKNOWN_CONVERSATION`; 409 `SEND_IN_PROGRESS`; 422 `CHANNEL_NOT_SUPPORTED`, `ACCOUNT_INACTIVE`, `VALIDATION_ERROR`, `MEDIA_URL_NOT_ALLOWED`, `OUTSIDE_MESSAGING_WINDOW`, `SEND_REJECTED`; 502 `SEND_UNCONFIRMED` (retry with the same `message_ref_id`).

**`GET /messages/{message_id}/`** returns the same shape; `status` is `sent`, `delivered`, `read` or `failed`. 404 `MESSAGE_NOT_FOUND`.

## WhatsApp templates

**`GET /whatsapp/templates/`** `?account_id=&template=` (both optional). `data[]`: `{template_id, template_name, language, language_name, template_type, template_status, account_id, variables{body[], body_named, header_media, header_media_type, header_text, buttons[{index, sub_type, label}]}}`. Send only `approved` ones.

**`POST /whatsapp/templates/send/`**: `mobile_no` (required, with country code), `account_id` (when several numbers), `reference_id`, `broadcast_name`, `template{template_name, body_1.. | <named vars>, header_text, header_media, header_media_filename, button_url_N, copy_code}`. 201 `data`: `{message_id (wamid), mobile_no, account_id, template_name, broadcast_name, reference_id, status, created_datetime}`. Uses the newest approved version of that name on that number. Errors include 422 `MISSING_MOBILE_NO`, `INVALID_MOBILE_NO`, `MISSING_TEMPLATE_NAME`.

## WhatsApp session messages

**`POST /whatsapp/messages/send/`**: `mobile_no`, `account_id`, `reference_id` (needed for webhooks), `message` as a string (text) or `{type: text|image|video|audio|document, text|caption, media_url, file_name}`. 201 `data`: `{message_id, mobile_no, account_id, reference_id, status, created_datetime}`. Errors: 422 `WINDOW_CLOSED`, `UNSUPPORTED_MESSAGE_TYPE`, `MISSING_MESSAGE`, `MISSING_MEDIA_URL`, `INVALID_MEDIA_URL`, `ACCOUNT_RESOLUTION_FAILED`, `WHATSAPP_SEND_REJECTED`; 502 `SEND_UNCONFIRMED` (check status before retrying).

**`GET /whatsapp/messages/{wamid}/status/`** (URL-encode the id): `{message_id, status, error, created_datetime, updated_datetime}`. 404 `NOT_FOUND`.

## WhatsApp webhooks

- **`POST /whatsapp/webhooks/`** `{callback_url (public https), events[] of sent|delivered|read|failed|reply, account_id?}`. 201 returns `secret` once.
- **`GET /whatsapp/webhooks/`** lists subscriptions (no secret).
- **`POST /whatsapp/webhooks/{webhook_id}/test/`** sends a signed `webhook.test` event and reports `delivered`, `http_status`, response start.
- **`DELETE /whatsapp/webhooks/{webhook_id}/`** returns 204.

Errors: 422 `MISSING_CALLBACK_URL`, `INVALID_CALLBACK_URL`, `MISSING_EVENTS`, `INVALID_EVENT`, `INVALID_ACCOUNT`; 404 `NOT_FOUND`.

Event body: `{id, event, occurred_at, data}`.
- `message.delivered|read|failed`: `data{message_id, account_id, template_name, broadcast_name, reference_id, status, error}`.
- `message.reply`: `data{message_id, account_id, mobile_no ("<account_id>_<number>"), text, reference_id, broadcast_name, template_name}` (the last three from the most recent API message to that customer).

Headers: `X-CM-Event`, `X-CM-Signature: sha256=<hmac>`, `User-Agent: ChatMaxima-Webhook/1.0`. Retries at 1 min, 5 min, 30 min, 2 h (5 attempts), 10 s timeout, redirects not followed.

## Media

**`POST /media/`** multipart, field `file`, under 10 MB. Images jpg/jpeg/png/gif/webp; video mp4/mov/webm/3gp/avi; audio mp3/m4a/aac/ogg/opus/wav/amr/flac; documents pdf/doc/docx/xls/xlsx/ppt/pptx/txt/csv/zip. 201 `data{media_url, media_type, file_name}`. Errors: 400 `MISSING_FILE`, 413 `MEDIA_TOO_LARGE`, 422 `MEDIA_TYPE_NOT_ALLOWED`, 502 `UPLOAD_FAILED`.

## Contacts

- **`POST /contacts/`** upsert. Send at least one of `contact_vendor_ref_id`, `contact_mobile` (or `contact_phone`), `contact_email`; optional `contact_name`, `contact_location`, `contact_organization`, `contact_bio`, `contact_image_url`, `contact_attributes{}`. Match order: vendor ref id, then API-created contact with the same mobile and email. 201 created / 200 updated.
- **`POST /contacts/sync/`** `{contacts: [...]}` up to 500; `data{created, updated, failed, total, results[{index, status, contact_id}]}`.
- **`GET /contacts/{id}/`** by `contact_id` or `contact_vendor_ref_id`.
- **`PUT /contacts/{id}/`** changes only the sent fields; 422 `MOBILE_LOCKED` for WhatsApp/SMS contacts' mobile.
- **`GET /contacts/`** `?search=&contact_vendor_ref_id=&contact_mobile=&contact_email=&page=&per_page=(1-100, default 20)`; `meta{page, per_page, total, total_pages, has_next, has_prev}`.

Errors: 400 `MISSING_CONTACTS_ARRAY`; 404 `NOT_FOUND`; 422 `INVALID_CONTACT`, `BATCH_TOO_LARGE`, `MOBILE_LOCKED`. Empty values are ignored on update; a new `contact_attributes` key creates that attribute.

## Conversations export

**`GET /conversations/export/`** `?cursor=<last message_id, default 0>&limit=<1-1000, default 500>&from=&to=` (ISO 8601 or `YYYY-MM-DD HH:MM:SS`; default the current UTC day; at most 3 days; only `from` or only `to` extends 3 days). Page with `meta.next_cursor`. Errors 400 `INVALID_DATE_FORMAT`, `INVALID_DATE_RANGE`, `DATE_RANGE_TOO_LARGE`.
