# Authorization Test Report - Booking System Phase 3

## Basic Information

| Information          | Value                                   |
| :------------------- | :-------------------------------------- |
| **Tester**           | Ali Haji                                |
| **Date**             | 2025-12-18                              |
| **Test Environment** | Docker Desktop, Deno + PostgreSQL       |
| **Target**           | Booking System                          |
| **Docker Image**     | `vheikkiniemi/cybersec-web-phase3:v1.1` |
| **Container ID**     | `9da44acd6cce`                          |
| **Status**           | Container running (over 3 hours)        |

---

## Testing Methodology

Testing was performed using **manual cURL commands** via Docker Desktop's Exec tab. Session cookies were saved to files for management.

**Tested Areas:**

1. Access control to different endpoints
2. CSRF protection functionality
3. Admin role correctness and visibility
4. Logout functionality

---

##  Discovered Endpoints and Their Status

| Endpoint           | HTTP Status     | Description                                              |
| :----------------- | :-------------- | :------------------------------------------------------- |
| `GET /`            | `200 OK`        | Public home page                                         |
| `GET /login`       | `200 OK`        | Public login form                                        |
| `GET /register`    | `200 OK`        | Public registration form                                 |
| `GET /resources`   | `200 OK`        | **CRITICAL ISSUE: Resource management open to everyone** |
| `GET /reservation` | `303 See Other` | Redirects to "Unauthorized" page                         |
| `GET /profile`     | `302 Found`     | Redirects to "Not Found" error                           |
| `GET /admin`       | `302 Found`     | Redirects to "Not Found" error                           |

---

## Critical Security Issues

### **ISSUE 1: Public Access to Resource Management**

**Severity:** VERY HIGH
**Proof:**

```bash
curl -s -w "Status: %{http_code}\n" -o /tmp/test.html http://localhost:8000/resources
# Status: 200
# Content: Form for creating/editing resources is available.


Impact: Any internet user can create, edit, or potentially delete the system's resources.
Priority: FIX IMMEDIATELY

ISSUE 2: Lack of CSRF Protection
Severity: HIGH
Proof:

bash
curl -s -X POST http://localhost:8000/resources \
  -d "resource_name=HACKED&resource_description=No CSRF" \
  -w "Status: %{http_code}"
# Status: 302 (Succeeded without CSRF token)
Impact: An attacker can trick a logged-in user into performing malicious actions (e.g., creating resources).
Priority: FIX IMMEDIATELY

 Things Working Well
1. CSRF Token Generation
bash
curl -s http://localhost:8000/register | grep -o 'value="[^"]*"'
# value="6b2c8bde-769a-4f43-89a0-e32f13680144"
Token is generated and also stored in a cookie (good practice).

2. Login, Registration and Logout
Registration with admin role is successful.

Admin login works and session is created.

Logout function redirects away (302) and session is terminated.

3. Good Security Headers
text
HTTP Headers:
- content-security-policy: default-src 'self'
- x-frame-options: DENY
- x-content-type-options: nosniff
- set-cookie: session_id=XXX; HttpOnly; SameSite=Strict
These effectively prevent XSS, clickjacking, and CSRF attacks.

 Found Bugs and Deficiencies
Bug 1: CSRF Token Not Rendered in Template
Finding:

html
<!-- In /resources page source -->
<input type="hidden" name="csrf_token" value="{{csrf_token}}">
The placeholder {{csrf_token}} is not replaced with an actual value, so CSRF protection is completely broken on this page.

Bug 2: Admin Panel Not Found
All tested admin endpoints (/admin, /admin/dashboard, etc.) return 302 and redirect to "Not Found" error page.

Interpretation: Admin functions are either not implemented or are in a hidden location.

Bug 3: Role Displayed to Everyone (Information Leak)
A logged-in user's home page directly displays their email and user role (e.g., administrator). This reveals sensitive information.

 Lack of Role-Based Access Control
Comparison of admin and regular user rights reveals that role-based access control is absent.

Endpoint	Admin Account	Regular Account
/	        200	    200
/resources200	    200
/reservation200	  200
/admin	  302	    302
Result: Both roles have exactly the same permissions.

 Risk Analysis and Prioritization
Issue	Severity	Priority	Recommended Fix
Public /resources	 VERY HIGH	IMMEDIATE	Add requireAuth middleware to the endpoint.
Lack of CSRF Protection	 HIGH	IMMEDIATE	Implement and require CSRF tokens for all POST requests.
CSRF Template Bug	 MEDIUM	1-2 DAYS	Fix template ({{csrf_token}} → <%= csrfToken %>).
Missing Admin Functions	 MEDIUM	1 WEEK	Implement admin panel and role-based controls.
Role Display (Info Leak)	 LOW	1 WEEK	Remove role display from public user view.
 Summary and Recommendation
Booking System Security Status: POOR

The system contains critical vulnerabilities (public resource management, CSRF) that could allow the entire system to be sabotaged. Basic security settings (CSP, cookies) are on a good track, but application logic security is deficient.

Solution recommendation for the development team:

DO NOT DEPLOY THE SYSTEM TO PRODUCTION BEFORE THESE CRITICAL ISSUES ARE FIXED. Start fixes from the top of the list (public access, CSRF).

 Test Commands and Attachments
Commands Used:

bash
# Public access test
curl http://localhost:8000/resources

# CSRF test
curl -X POST http://localhost:8000/resources -d "resource_name=TEST"

# Admin access test
curl -b admin_session.txt http://localhost:8000/admin
Test Environment Details:

Image: vheikkiniemi/cybersec-web-phase3:v1.1

Ports: 8003:8000 (application), 5435 (PostgreSQL)

Database: PostgreSQL

Status: Container running normally

Summary of Test Results:

Authentication: 90% (functional)

Authorization: 40% (deficient)

Session Security: 80% (good, but template bug)

CSRF Protection: 30% (partially functional)
```

