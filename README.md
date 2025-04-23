<h1>Error-Aware Logging via Angular Interceptor and DSPS Notification Detection</h1>

<h2>Table of Contents</h2>
<ul>
  <li><a href="#overview">Overview</a></li>
  <li><a href="#why-this-was-built">Why This Was Built</a></li>
  <li><a href="#component-dependencies">Component Dependencies</a></li>
  <li><a href="#how-it-works">How It Works</a>
    <ul>
      <li><a href="#api-error-detection">1. API Error Detection</a></li>
      <li><a href="#dom-error-message-extraction">2. DOM Error Message Extraction</a></li>
      <li><a href="#logging-trigger">3. Logging Trigger</a></li>
      <li><a href="#no-notification-no-log">4. No Notification = No Log</a></li>
    </ul>
  </li>
  <li><a href="#benefits">Benefits</a></li>
  <li><a href="#implementation-notes">Implementation Notes</a></li>
  <li><a href="#next-steps">Next Steps</a></li>
</ul>

<h2 id="overview">Overview</h2>
<p>
This document outlines a custom error logging mechanism developed to enhance error traceability in our Angular applications.
The solution pairs backend API failures with corresponding user-facing error notifications (rendered via <code>&lt;dsps-notification&gt;</code>) and sends structured logs to a central logging server. 
Logging occurs <strong>only</strong> when an API error is acknowledged and surfaced to the user.
</p>

<h2 id="why-this-was-built">Why This Was Built</h2>
<ul>
  <li><strong>Improve observability:</strong> Understand which backend errors are actually being shown to users.</li>
  <li><strong>Reduce noise:</strong> Avoid logging errors that never reach the user interface.</li>
  <li><strong>Enable UX validation:</strong> Detect when error messages are missing despite failed API calls.</li>
</ul>

<h2 id="component-dependencies">Component Dependencies</h2>
<ul>
  <li><strong>dsps-notification:</strong> Custom notification component used across the platform to display messages. It supports types like <code>error</code>, <code>success</code>, etc.</li>
  <li><strong>Angular's HttpInterceptor:</strong> Intercepts all HTTP requests/responses to globally manage API error logging.</li>
</ul>

<h2 id="how-it-works">How It Works</h2>

<h3 id="api-error-detection">1. API Error Detection</h3>
<ul>
  <li>The Angular interceptor listens for failed HTTP responses (status <code>&gt;= 400</code>).</li>
  <li>On error, it triggers a <em>delayed DOM scan</em> to allow error notifications to render.</li>
</ul>

<h3 id="dom-error-message-extraction">2. DOM Error Message Extraction</h3>
<ul>
  <li>After a short timeout, the tool scans the DOM for any matching component:</li>
</ul>
<pre>&lt;dsps-notification type="error"&gt;...&lt;/dsps-notification&gt;</pre>
<ul>
  <li>If one or more matching elements are found, their inner text is extracted.</li>
</ul>

<h3 id="logging-trigger">3. Logging Trigger</h3>
<ul>
  <li>If a UI error message is detected, the tool pairs it with the corresponding API error.</li>
  <li>The following structured log is sent to the logging server:</li>
</ul>
<pre>{
  "timestamp": "2025-04-23T14:32:45.123Z",
  "apiError": {
    "url": "/api/rooms/123",
    "status": 404,
    "message": "Room not found"
  },
  "uiErrorMessage": "The requested room could not be located."
}</pre>

<h3 id="no-notification-no-log">4. No Notification = No Log</h3>
<ul>
  <li>If no <code>&lt;dsps-notification type="error"&gt;</code> is found, logging is skipped.</li>
</ul>

<h2 id="benefits">Benefits</h2>
<table>
  <thead>
    <tr>
      <th>Feature</th>
      <th>Impact</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Accurate Logging</strong></td>
      <td>Only logs errors that users actually see</td>
    </tr>
    <tr>
      <td><strong>Reduced Log Noise</strong></td>
      <td>Avoids background or silent errors that aren’t surfaced</td>
    </tr>
    <tr>
      <td><strong>Supports Debugging</strong></td>
      <td>Helps identify mismatched or missing UI error handling</td>
    </tr>
    <tr>
      <td><strong>Passive Integration</strong></td>
      <td>Requires no changes in existing feature modules</td>
    </tr>
  </tbody>
</table>

<h2 id="implementation-notes">Implementation Notes</h2>
<ul>
  <li><strong>Delay Mechanism:</strong> The interceptor uses <code>setTimeout</code> (or RxJS <code>delay</code>) to wait ~300ms before checking the DOM.</li>
  <li><strong>Multiple Notifications:</strong> It supports logging multiple notifications if they appear after the error.</li>
  <li><strong>Matching Heuristics:</strong> Currently assumes the first notification after the failure corresponds to the intercepted API request.</li>
</ul>

<h2 id="next-steps">Next Steps</h2>
<ul>
  <li>Optional: Improve matching between API endpoints and messages (e.g., by tagging requests or tracking notification timestamps).</li>
  <li>Optional: Add e2e test cases with mocked API failures and UI error detection.</li>
  <li>Optional: Wrap this logic into a reusable <code>ErrorLogService</code> with configuration for debounce delay, log format, and custom filters.</li>
</ul>