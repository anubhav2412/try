h1. Error-Aware Logging via Angular Interceptor and DSPS Notification Detection

h2. Overview

This document outlines a custom error logging mechanism developed to enhance error traceability in our Angular applications. The solution pairs backend API failures with corresponding user-facing error notifications (rendered via `<dsps-notification>`) and sends structured logs to a central logging server. Logging occurs *only* when an API error is acknowledged and surfaced to the user.

h2. Why This Was Built

* *Improve observability*: Understand which backend errors are actually being shown to users.
* *Reduce noise*: Avoid logging errors that never reach the user interface.
* *Enable UX validation*: Detect when error messages are missing despite failed API calls.

h2. Component Dependencies

* *dsps-notification*: Custom notification component used across the platform to display messages. It supports types like `error`, `success`, etc.
* *Angular's HttpInterceptor*: Intercepts all HTTP requests/responses to globally manage API error logging.

h2. How It Works

h3. 1. API Error Detection
* The Angular interceptor listens for failed HTTP responses (`status >= 400`).
* On error, it triggers a *delayed DOM scan* to allow error notifications to render.

h3. 2. DOM Error Message Extraction
* After a short timeout (to allow rendering), the tool scans the DOM for any:
{code:html}
<dsps-notification type="error">...</dsps-notification>
{code}
* If one or more matching elements are found, their inner text is extracted.

h3. 3. Logging Trigger
* If a UI error message is detected, the tool pairs the message with the corresponding API error.
* A structured log entry is then sent to the logging server in the following format:
{code:json}
{
  "timestamp": "2025-04-23T14:32:45.123Z",
  "apiError": {
    "url": "/api/rooms/123",
    "status": 404,
    "message": "Room not found"
  },
  "uiErrorMessage": "The requested room could not be located."
}
{code}

h3. 4. No Notification = No Log
* If no `<dsps-notification type="error">` is found, logging is skipped.

h2. Benefits

|| Feature || Impact ||
| *Accurate Logging* | Only logs errors that users actually see |
| *Reduced Log Noise* | Avoids background or silent errors that aren’t surfaced |
| *Supports Debugging* | Helps identify mismatched or missing UI error handling |
| *Passive Integration* | Requires no changes in existing feature modules |

h2. Implementation Notes

* *Delay Mechanism*: The interceptor uses `setTimeout` (or RxJS `delay`) to wait ~300ms before checking the DOM.
* *Multiple Notifications*: It supports multiple error notifications by logging all that appear in response to the error.
* *Matching Heuristics*: Current implementation assumes the first `<dsps-notification type="error">` after the error corresponds to the intercepted request.

h2. Next Steps

* Optional: Improve matching between API endpoints and messages (e.g., by tagging requests or tracking notification timestamps).
* Optional: Add e2e test cases with mocked API failures and UI error detection.
* Optional: Wrap this logic into a reusable `ErrorLogService` with configuration for debounce delay, log format, and custom filters.