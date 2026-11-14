# Utility Server

A small, lightweight FastAPI server that was created to manage tasks I found in common across domains, and the schema I used for implementation.

It's installed as a systemd service for TOTP authentication, domain/policy management, token issuance, and rate limiting.

## Web UI

![Console showing domain, TOTP, and rate-limit management](assets/utility-server/admin-console.webp)

The console provides:

- TOTP login for admin access.
- Domain registration/status management.
- TOTP policy creation, editing, deletion, and QR provisioning for single-subject domains.
- Rate-limit policy management.
- Connectivity checks through `/util/heartbeat`.
- Policy dumps through `/admin/policy/dump` and `/admin/rate-limit/dump` for display on the web page.

The deployed file layout differs from this repo structure. The related server and admin UI files are grouped here for convenience.

## Backend Files

- `util_server.py`: FastAPI app for utility endpoints.
- `rate_limit.py`: database-backed rate-limit helper used by the API.
- `cleanup.py`: cron-friendly cleanup for old rate-limit, token, and TOTP rows.
- `utility-server-watchdog.sh`: cron-friendly service watchdog for the FastAPI process.
- `utility_schema.sql`: current schema dump plus required `rate_limit_action` bootstrap rows.
- `utility_user.sql`: local database user/grant SQL split out from the main schema and ignored by git.
- `config.json`: local runtime configuration with MySQL and JWT secrets, ignored by git.

## Endpoints

- `GET /heartbeat`: returns an empty success response so mobile apps and other clients can tell whether they are connected.
- `GET /uuid`: returns a new UUIDv4.
- `GET /time`: returns the server time as Unix seconds and UTC ISO text.
- `POST /domain`: registers, reads, updates, or deletes a managed domain.
- `POST /policy`: creates, reads, updates, or deletes TOTP policies for a domain and purpose.
- `POST /totp/provision-single`: creates or reuses a single-subject TOTP secret and returns the authenticator URI.
- `POST /totp/confirm-single`: confirms a pending single-subject TOTP secret with an authenticator code.
- `POST /totp/verify-single`: verifies an active single-subject TOTP code and records attempts/rate limits.
- `POST /token`: issues, validates, or revokes utility tokens.
- `POST /rate-limit/check`: checks and records a rate-limit event for a registered action.
- `POST /rate-limit/event/success`: marks a previously recorded rate-limit event as successful.
- `POST /rate-limit/policy`: creates, reads, updates, or deletes rate-limit policies.
- `GET /admin/policy/dump`: returns TOTP policy rows for display on the admin web page.
- `POST /admin/policy/refresh`: placeholder refresh endpoint for the admin web page.
- `GET /admin/rate-limit/dump`: returns rate-limit actions and policies for display on the admin web page.

## Cleanup

`cleanup.py` removes old operational rows and is intended for cron. The script currently documents this daily example:

```cron
17 3 * * * cd /path/to/utility-server && .venv/bin/python cleanup.py >> /var/log/utility-cleanup.log 2>&1
```
