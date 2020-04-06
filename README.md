# OWASP WebGoat

Selected solutions for [OWASP WebGoat](https://owasp.org/www-project-webgoat/) (8.0.0.M26).

- **(A1) Injection**
    - [SQL Injection (advanced)](01-sqli_advanced.md)
    - SQL Injection (mitigation)
    - [Path traversal](01-path-traversal.md)
- **(A2) Broken Authentication**
    - [Authentication bypasses](02-auth_bypasses.md)
    - [JWT tokens](02-jwt-tokens.md)
    - [Password reset](02-password-reset.md)
- **[(A4) XML External Entities (XXE)](04-xxe.md)**
- **(A5) Broken Access Control**
    - [Insecure Direct Object References](05-idor.md)
- **[(A7) Cross-Site Scripting (XSS)](07-xss.md)**
- **[(A8) Insecure Deserialization](08-insecure-deser.md)**
- **[(A9) Vulnerable Components](09-vuln_components.md)**

## General tips
- Check out source code
    - [WebGoat container](https://github.com/WebGoat/WebGoat/tree/develop/webgoat-container/src/main/java/org/owasp/webgoat)
    - [Lessons](https://github.com/WebGoat/WebGoat/tree/develop/webgoat-lessons)
- Peek into database, and if necessary (for example to overcome a bug), you can modify it
    - Database is saved onto your disk under `c:\Users\USER\.webgoat-v8.0.0-SNAPSHOT\data\`