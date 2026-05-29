
A Flask-based secure document sharing system with encrypted file storage, role-based access control, and comprehensive security controls.

---

## Features

- User registration and authentication with bcrypt password hashing
- Upload and download encrypted documents (Fernet AES encryption)
- Share documents with specific users (viewer/editor roles)
- Role-based access control (Admin, User, Guest)
- Document versioning and audit trail
- Security headers, session management, and rate limiting
- Full security event logging

---

## Requirements

- Python 3.10+
- pip
- Git

---

## Setup Instructions

### 1. Clone the Repository

```bash
git clone https://github.com/KrystianB456/securewebapp.git
cd securewebapp
```

### 2. Create a Virtual Environment

**Windows:**
```powershell
python -m venv .venv
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
.\.venv\Scripts\Activate.ps1
```

**Mac/Linux:**
```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

### 4. Initialize Data Files

```bash
python -c "
import json, os
os.makedirs('data', exist_ok=True)
os.makedirs('logs', exist_ok=True)
os.makedirs('uploads', exist_ok=True)
for f in ['data/users.json', 'data/sessions.json', 'data/documents.json']:
    with open(f, 'w') as file:
        json.dump({}, file)
open('logs/security.log', 'w').close()
open('logs/access.log', 'w').close()
print('Done!')
"
```

### 5. Generate TLS Certificate

**Option A — Using OpenSSL (if installed):**
```bash
openssl req -x509 -newkey rsa:4096 -nodes -out cert.pem -keyout key.pem -days 365
```
Press Enter through all prompts to use defaults.

**Option B — Using Python:**
```bash
pip install pyopenssl
python -c "
from OpenSSL import crypto
k = crypto.PKey()
k.generate_key(crypto.TYPE_RSA, 4096)
cert = crypto.X509()
cert.get_subject().CN = 'localhost'
cert.set_serial_number(1)
cert.gmtime_adj_notBefore(0)
cert.gmtime_adj_notAfter(365*24*60*60)
cert.set_issuer(cert.get_subject())
cert.set_pubkey(k)
cert.sign(k, 'sha256')
open('cert.pem', 'wb').write(crypto.dump_certificate(crypto.FILETYPE_PEM, cert))
open('key.pem', 'wb').write(crypto.dump_privatekey(crypto.FILETYPE_PEM, k))
print('Done!')
"
```

### 6. Create Admin Account

```bash
python -c "
import json, bcrypt, time
with open('data/users.json', 'r') as f:
    users = json.load(f)
password = b'AdminPass1!'
hashed = bcrypt.hashpw(password, bcrypt.gensalt(rounds=12))
users['admin'] = {
    'username': 'admin',
    'email': 'admin@example.com',
    'password_hash': hashed.decode('utf-8'),
    'role': 'admin',
    'created_at': time.time(),
    'failed_attempts': 0,
    'locked_until': None
}
users['guest'] = {
    'username': 'guest',
    'email': 'guest@example.com',
    'password_hash': '',
    'role': 'guest',
    'created_at': time.time(),
    'failed_attempts': 0,
    'locked_until': None
}
with open('data/users.json', 'w') as f:
    json.dump(users, f, indent=2)
print('Admin and Guest accounts created!')
"
```

### 7. Run the Application

```bash
python app.py
```

Open your browser and go to: **https://127.0.0.1:5000**

> **Note:** Your browser will show a security warning because the TLS certificate is self-signed. This is expected for development. Click **Advanced → Proceed to 127.0.0.1** to continue.

---

## Default Accounts

| Username | Password | Role |
|----------|----------|------|
| admin | AdminPass1! | Admin |
| guest | *(no password — use guest link)* | Guest |

> Register a new account for regular user access.

---

## Testing the Application

### Manual Testing

**1. Register a new user**
- Go to `https://127.0.0.1:5000/register`
- Username: 3-20 alphanumeric characters
- Password: 12+ chars with uppercase, lowercase, number, special character

**2. Login**
- Go to `https://127.0.0.1:5000/login`
- Use your registered credentials

**3. Upload a document**
- Go to Documents → Upload New Document
- Allowed types: pdf, txt, docx, png, jpg (max 16MB)

**4. Share a document**
- Click Share next to any document you own
- Enter a username and select viewer or editor role

**5. Guest access**
- Click "Continue as Guest" on the login page
- Guests can only view documents shared with username `guest`

**6. Admin panel**
- Login as `admin` with password `AdminPass1!`
- Click Admin Panel on the dashboard
- View all users and documents, change roles, delete accounts

---

## Running Automated Tests

```bash
pip install pytest
pytest tests/script.py -v
```

Expected output — all 11 tests should pass:
```
tests/script.py::test_register_weak_password PASSED
tests/script.py::test_register_invalid_username PASSED
tests/script.py::test_register_mismatched_passwords PASSED
tests/script.py::test_login_invalid_credentials PASSED
tests/script.py::test_dashboard_requires_auth PASSED
tests/script.py::test_admin_requires_auth PASSED
tests/script.py::test_documents_requires_auth PASSED
tests/script.py::test_security_headers PASSED
tests/script.py::test_xss_in_username PASSED
tests/script.py::test_password_no_uppercase PASSED
tests/script.py::test_password_no_special_char PASSED
11 passed
```

---

## Security Features

| Feature | Implementation |
|---------|---------------|
| Password hashing | bcrypt with cost factor 12 |
| Account lockout | 5 failed attempts → 15 minute lockout |
| Rate limiting | 10 login attempts per IP per minute |
| Session management | Secure tokens with 30-minute timeout |
| Cookie security | HttpOnly, Secure, SameSite=Strict |
| File encryption | Fernet (AES-128-CBC) |
| Transport security | HTTPS/TLS with self-signed certificate |
| Input validation | Whitelist validation + html.escape |
| Security headers | CSP, HSTS, X-Frame-Options, and more |
| Logging | All security events logged to logs/security.log |

---

## Project Structure

```
securewebapp/
├── README.md               # This file
├── requirements.txt        # Python dependencies
├── app.py                  # Main application
├── config.py               # Configuration settings
├── encryption.py           # Fernet file encryption
├── security_logger.py      # Security event logging
├── data/
│   ├── users.json          # User accounts
│   ├── sessions.json       # Active sessions
│   └── documents.json      # Document metadata
├── logs/
│   └── security.log        # Security events
├── static/
│   ├── css/
│   └── js/
├── templates/              # HTML templates
├── docs/                   # Documentation
│   ├── security_design.pdf
│   └── pentest_report.pdf
├── tests/
│   └── script.py           # Automated tests
└── presentation/           # Presentation slides
```

---

## Resetting the Application

To clear all data and start fresh:

```bash
python -c "
import json, os, shutil
for f in ['data/users.json', 'data/sessions.json', 'data/documents.json']:
    with open(f, 'w') as file:
        json.dump({}, file)
if os.path.exists('uploads'):
    shutil.rmtree('uploads')
    os.makedirs('uploads')
open('logs/security.log', 'w').close()
print('Reset complete!')
"
```

Then re-run Step 6 to recreate the admin account.

---

## Viewing Security Logs

```bash
# Windows
type logs\security.log

# Mac/Linux
cat logs/security.log
```

---

## Dependencies

See `requirements.txt` for full list. Key packages:

- `flask` — Web framework
- `bcrypt` — Password hashing
- `cryptography` — Fernet file encryption
- `flask-limiter` — Rate limiting
- `PyJWT` — JSON Web Tokens
- `pyopenssl` — TLS certificate generation
