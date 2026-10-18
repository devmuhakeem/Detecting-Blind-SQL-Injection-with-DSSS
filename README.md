# Detecting Blind SQL Injection with DSSS

A hands-on lab using Damn Small SQLi Scanner (DSSS) to automatically detect a blind SQL injection vulnerability in an authenticated section of a target web application, then confirming the finding manually.

## Scenario
A target site's View Profile page, accessible only after login, accepted an `id` URL parameter. The question was whether that parameter was vulnerable to SQL injection, specifically the blind variety, where no error message or visible data leak confirms the flaw directly.

## What I did

### 1. Captured an authenticated session
- Logged into the target application through the browser as a registered user
- Navigated to the View Profile page and noted its URL (including the `id` parameter)
- Opened browser dev tools, used the Console tab to run `document.cookie`, and copied the returned session cookie value

### 2. Ran DSSS against the authenticated endpoint
- From a terminal on Parrot Security, navigated to the DSSS directory and ran:
```
python3 dsss.py -u "http://[target]/viewprofile.aspx?id=1" --cookie="[session cookie]"
```
- `-u` set the target URL and parameter to test; `--cookie` passed the authenticated session so the scanner could reach the logged-in-only page
- DSSS reported the target was vulnerable to **blind SQL injection** and provided a crafted URL demonstrating it

### 3. Confirmed the finding manually
- Opened the DSSS-generated URL directly in the browser
- The page returned profile information for multiple user accounts instead of just the one the session was logged in as, confirming the injected condition had altered the underlying query's result set

## Key takeaways
- Blind SQL injection produces no visible error or obvious data leak — confirming it requires observing subtle behavioral differences across many requests, which is exactly the kind of systematic probing an automated scanner is built for
- Authentication state has to be part of a scan's scope: a scanner that can't carry a session past login is blind to any vulnerability sitting behind authentication, which is most of a real application's attack surface
- A scanner's output is a lead, not a conclusion — manually loading the generated URL and observing the actual data exposure is what turns a tool's finding into a verified vulnerability
- The absence of an error message says nothing about query safety; a silently-failing injection point can persist unnoticed longer than a loud one
- Fix: parameterized queries/prepared statements everywhere user input reaches a database call, strict input validation as a secondary layer, and routine authenticated (not just unauthenticated) vulnerability scanning

## Tools
Damn Small SQLi Scanner (DSSS), Python 3, Mozilla Firefox (dev tools), Parrot Security OS

---
*Completed in an isolated EC-Council training lab environment against a designated practice web application.*
