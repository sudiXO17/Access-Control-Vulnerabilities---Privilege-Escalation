# 🔐 Access Control Vulnerabilities & Privilege Escalation

## 🔐 What is Access Control?

Access control means deciding who can do what in an application. It’s how a system restricts access to data or actions based on user identity and role.

**For example:**

- A normal user should only view their own account.
- An admin can delete users or manage settings.

**Access control depends on:**

- **Authentication**: Are you logged in?
- **Authorization**: What are you allowed to do?

---

## ❌ Broken Access Control — Definition

Broken Access Control occurs when users can act outside their intended permissions, gaining access to unauthorized data or functionality.

---

## 🔄 Types of Access Control

### 1. Horizontal Access Control

- **Definition**: Users can only access their own data or resources.
- **Example**: A user should view only their own bank account.

### 2. Vertical Access Control

- **Definition**: Different roles (e.g., user, admin) have different permissions.
- **Example**: Admin can modify/delete any user account; normal users cannot.

---

## 🚨 Types of Privilege Escalation

### 🕽️ Horizontal Privilege Escalation (User A ➔ User B)

#### 💡 What It Means:

A normal user (User A) accesses or performs actions on resources that belong to another user (User B) by modifying parameters like IDs in the request.

#### ✅ Example 1:

```
https://insecure-website.com/myaccount?id=123
```

- Here, id=123 is User A's account.
- If User A changes this to id=124, and sees User B’s account without any server-side check, it’s a horizontal privilege escalation.

#### ✅ Example 2:

- Web app redirects users after login.
- Sensitive data (user info, tokens) is in the **body** of the redirect.
- Risk:
  - Other users may intercept or replay it.
  - Sensitive data may be cached or leaked.

---

## 🧱 Insecure Direct Object References (IDOR)

#### 💡 What It Means:

IDOR occurs when user-controlled input (like an ID or filename) is used to access backend data without proper access validation.

#### ✅ Example 1:

```
GET /my-account?id=administrator
```

- A regular user tries to access the admin's account.
- If the server doesn't validate ownership, it's an IDOR.

#### ✅ Example 2:

User chat logs stored like:

```
/chats/user123.txt
```

- Attacker guesses:

```
/chats/user456.txt
```

- Downloads other users' data without auth.

---

## 🔐 Key Difference Between the Two:

| Aspect                   | Horizontal Privilege Escalation | IDOR                                     |
| ------------------------ | ------------------------------- | ---------------------------------------- |
| What it targets          | Other users’ data or actions    | Any object like files, records, accounts |
| Based on                 | Access level comparison         | Direct reference in URLs or parameters   |
| Requires authentication? | Usually yes                     | Often yes, but still exploitable         |
| Main issue               | No proper authorization check   | No validation of object ownership        |

---

## 🔼 Vertical Privilege Escalation (Low-privileged user ➔ High-privileged user)

#### 💡 What It Means:

A regular user performs actions reserved for higher-privileged users like admins or moderators — usually because the system fails to check their role properly.

---

## 🚨 Common Attack Vectors:

### 1. Unprotected Admin Functionality

- Admin pages exposed without authentication or role checks.
- **Examples:**
  - `https://insecure-website.com/admin`
  - `https://insecure-website.com/robots.txt`

**🛠️ Fix**: Restrict such paths with server-side role checks.

---

### 2. Parameter-Based Access Control

- URL parameters control access. Easily tampered.
- **Example:**
  - `https://insecure-website.com/login/home.jsp?admin=true`

**🛠️ Fix**: Never trust client-side input for permissions.

---

### 3. Platform Misconfiguration / Header Manipulation

- Exploiting headers like:
  - `X-Original-URL`
  - `X-Rewrite-URL`

**Exploit Example:**

```
GET /anything HTTP/1.1
X-Original-URL: /admin/deleteUser
```

**🛠️ Fix**: Avoid trusting such headers. Validate after rewrites.

---

### 4. URL Matching Discrepancies

- Loose pattern matching allows bypass via case or file extension.
- **Examples:**
  - `/ADMIN/DELETEUSER` matches `/admin/deleteUser`
  - `/admin/deleteUser.anything` matches `/admin/deleteUser*`

**🛠️ Fix**: Normalize URLs, enforce case sensitivity, strict matching.

---

## 🧠 Summary Table

| Technique                 | Risk                              | Example                       |
| ------------------------- | --------------------------------- | ----------------------------- |
| Unprotected Functionality | Direct access to admin pages      | /admin, found via robots.txt  |
| Parameter Tampering       | Bypass via query param            | ?admin=true                   |
| Header Misuse             | Access via headers                | X-Original-URL, X-Rewrite-URL |
| URL Pattern Tricks        | Match using uppercase, extensions | /ADMIN/DELETEUSER.any         |

---

## 🧹 Other Access Control Weaknesses

| Weakness                             | Description                                    |
| ------------------------------------ | ---------------------------------------------- |
| 🔃 Multi-step flaws                  | Step 1 and 2 secured, but Step 3 is vulnerable |
| ⬆️ Horizontal to Vertical escalation | Replay admin request as normal user            |
| 🔗 Referer-based access control      | Can be bypassed by spoofing Referer header     |
| 🌍 Location-based access             | Bypassed using VPNs, Proxies, or GPS spoofing  |

---

## 📡 Header-based Access Control Testing

### ✅ 1. Headers to Inject in Requests

| Header                            | Risk                    | Check                            |
| --------------------------------- | ----------------------- | -------------------------------- |
| X-Forwarded-For                   | Bypass IP-based control | Send spoofed IP                  |
| X-Original-URL                    | Override routing        | Inject and observe               |
| X-Rewrite-URL                     | Rewrite URL paths       | Same as above                    |
| Referer                           | Referer spoofing        | Send fake Referer                |
| Access-Control-Allow-Origin       | CORS misconfig          | If \*, test unrestricted origins |
| Access-Control-Allow-Credentials  | CORS bypass             | If true with \*, it's a risk     |
| X-Permitted-Cross-Domain-Policies | Cross-domain data leak  | Set to all = dangerous           |

---

### 📅 2. Headers to Check in Responses

| Header                           | Purpose                             |
| -------------------------------- | ----------------------------------- |
| X-Frame-Options                  | Prevents clickjacking               |
| Content-Security-Policy (CSP)    | Prevents XSS by restricting scripts |
| Strict-Transport-Security (HSTS) | Enforces HTTPS usage                |
| Content-Disposition              | Prevents inline file execution      |
| Content-Type                     | Prevents MIME sniffing attacks      |

---

## 🛡️ Pro Tip:

Always validate access on **server side**, not just UI or JavaScript. **Client-side checks can always be bypassed**.

