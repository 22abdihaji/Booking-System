# Penetration Test Report - Booking System

## 1️⃣ Introduction

### Tester(s):

**Name:** Ali Haji, Onyeisi Chidiebere  
**Student ID:**

### Purpose:

Identify security vulnerabilities and functionality flaws in the Booking System web application, focusing on the registration and authentication mechanisms.

### Scope:

**Tested components:**

- User registration page (/register)
- Authentication flows
- Session management
- Input validation
- Database interactions

**Exclusions:**

- Denial of Service (DoS) testing
- Physical security testing

**Test approach:** Gray-box (partial knowledge of application structure)

**Test environment & dates:**

- **Start:** 28.11.2025
- **End:** 28.11.2025
- **Test environment:**
  - OS: Windows
  - Browser: Firefox
  - Tools: OWASP ZAP, manual testing
  - Application: Deno + Oak + PostgreSQL

**Assumptions & constraints:**

- Limited to registration functionality
- Time-constrained testing
- No source code access

## 2️⃣ Executive Summary

**Short summary:**
Testing revealed multiple high-risk vulnerabilities including SQL injection, weak input validation, and missing security headers that could lead to complete system compromise.

**Overall risk level:** 🔴 **High**

**Top 5 immediate actions:**

1. Implement parameterized queries to prevent SQL injection
2. Add CSRF protection tokens to all forms
3. Implement security headers (CSP, X-Frame-Options)
4. Enforce strong password policy
5. Add proper input validation and sanitization

## 3️⃣ Severity scale & definitions

| Severity Level | Description                                                       | Recommended Action             |
| -------------- | ----------------------------------------------------------------- | ------------------------------ |
| 🔴 High        | Serious vulnerability leading to system compromise or data breach | Immediate fix required         |
| 🟠 Medium      | Significant issue requiring specific conditions                   | Fix ASAP                       |
| 🟡 Low         | Minor issue or configuration weakness                             | Fix soon                       |
| 🔵 Info        | No direct risk, useful for hardening                              | Monitor and fix in maintenance |

## 4️⃣ Findings

| ID   | Severity  | Finding                  | Description                                | Evidence                                    |
| ---- | --------- | ------------------------ | ------------------------------------------ | ------------------------------------------- |
| F-01 | 🔴 High   | SQL Injection in Email   | Email field accepts SQL injection payloads | Payload: `test' OR '1'='1' -- @example.com` |
| F-02 | 🔴 High   | No CSRF Protection       | Forms lack anti-CSRF tokens                | ZAP Alert: Absence of Anti-CSRF Tokens      |
| F-03 | 🟠 Medium | Missing Security Headers | No CSP, X-Content-Type-Options headers     | ZAP: Multiple header alerts                 |
| F-04 | 🟠 Medium | Weak Password Policy     | Accepts single character passwords         | Manual testing                              |
| F-05 | 🟡 Low    | Information Disclosure   | Error messages reveal application details  | ZAP: Application Error Disclosure           |

## 5️⃣ OWASP ZAP Test Report

**ZAP Scan Summary:**

- **Total Alerts:** 8
- **Medium Risk:** 3
- **Low Risk:** 2
- **Informational:** 0

**Key Findings from ZAP:**

1. **Absence of Anti-CSRF Tokens** (Medium)
2. **Content Security Policy Header Not Set** (Medium)
3. **Missing Anti-clickjacking Header** (Medium)
4. **Application Error Disclosure** (Low)
5. **X-Content-Type-Options Header Missing** (Low)

**Full ZAP Report:** [ZAP-Reports/zap_report.md](ZAP-Reports/zap_report.md)

---

_Report generated on: 28.11.2025_
