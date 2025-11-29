# Penetration Test Report - Booking System

## 1️⃣ Introduction

### Tester(s):
**Name:** Ali Haji, Onyeisi Chidiebere  

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
Testing revealed multiple critical vulnerabilities including SQL injection, path traversal, and missing security headers that could lead to complete system compromise and data breach.

**Overall risk level:** 🔴 **High**

**Top 5 immediate actions:**
1. Implement parameterized queries to prevent SQL injection
2. Add input validation to prevent path traversal attacks
3. Implement CSRF protection tokens to all forms
4. Add security headers (CSP, X-Frame-Options, X-Content-Type-Options)
5. Implement proper error handling to prevent information disclosure

## 3️⃣ Severity scale & definitions

| Severity Level | Description | Recommended Action |
|---------------|-------------|-------------------|
| 🔴 High | Serious vulnerability leading to system compromise or data breach | Immediate fix required |
| 🟠 Medium | Significant issue requiring specific conditions | Fix ASAP |
| 🟡 Low | Minor issue or configuration weakness | Fix soon |
| 🔵 Info | No direct risk, useful for hardening | Monitor and fix in maintenance |

## 4️⃣ Findings

| ID | Severity | Finding | Description | Evidence |
|----|----------|---------|-------------|----------|
| F-01 | 🔴 High | SQL Injection | Username field vulnerable to SQL injection | ZAP Alert: SQL Injection with payload `'` |
| F-02 | 🔴 High | Path Traversal | Username field allows path traversal attacks | ZAP Alert: 2 instances of Path Traversal |
| F-03 | 🟠 Medium | No CSRF Protection | Forms lack anti-CSRF tokens | ZAP Alert: Absence of Anti-CSRF Tokens |
| F-04 | 🟠 Medium | Format String Error | Username field vulnerable to format string attacks | ZAP Alert: Format String Error |
| F-05 | 🟠 Medium | Missing Security Headers | No CSP, X-Frame-Options headers | ZAP: Multiple header alerts |

## 5️⃣ OWASP ZAP Test Report

**ZAP Scan Summary:**
- **Total Alerts:** 9
- **High Risk:** 2
- **Medium Risk:** 4
- **Low Risk:** 2
- **Informational:** 1

**Key Findings from ZAP:**
1. **SQL Injection** (High) - Username parameter vulnerable
2. **Path Traversal** (High) - 2 instances in username field
3. **Absence of Anti-CSRF Tokens** (Medium)
4. **Format String Error** (Medium) - Username parameter
5. **Content Security Policy Header Not Set** (Medium)
6. **Missing Anti-clickjacking Header** (Medium)
7. **Application Error Disclosure** (Low)
8. **X-Content-Type-Options Header Missing** (Low)

**Full ZAP Report:** [ZAP-Reports/zap_report.md](ZAP-Reports/zap_report.md)

---

*Report generated on: 28.11.2025_
