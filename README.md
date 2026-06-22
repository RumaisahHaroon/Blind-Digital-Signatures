# Cryptographic Core — Chaum's Blind Signature
### Member 1 | InfoSec Project | Blind Digital Signatures

---

## Overview

This module implements **Chaum's RSA-based Blind Signature Scheme** — the cryptographic foundation for the anonymous e-voting system.

### Protocol (in plain English)

```
Voter                              Authority
  |                                   |
  |-- 1. blind(vote, pub_key) ------->|  ← signer never sees real vote
  |                                   |
  |<- 2. sign_blinded(blinded, priv) -|  ← signs without knowing content
  |                                   |
  |-- 3. unblind(blind_sig, r) ------>  ← voter removes blinding factor
  |
  |-- 4. verify(vote, sig, pub_key)      ← anyone can verify
```

---

## Setup

```bash
pip install -r requirements.txt
```

---

## Usage (for other team members)

```python
from crypto import generate_keys, blind_message, sign_blinded, unblind_signature, verify_signature

# Authority: generate keys once, publish public key
kp = generate_keys(key_size=2048)

# Voter: blind their vote
vote = b"vote:candidate_alice"
blinded_msg = blind_message(vote, kp.public_key)

# Authority: sign the blinded message (never sees actual vote)
blind_sig = sign_blinded(blinded_msg.blinded, kp.private_key)

# Voter: remove blinding factor
signature = unblind_signature(blind_sig, blinded_msg, kp.public_key)

# Anyone: verify the vote is authentic
is_valid = verify_signature(vote, signature, kp.public_key)
print(is_valid)  # True
```

## Sharing Public Key (PEM) with Other Modules

```python
from crypto import export_public_key_pem, import_public_key_pem

# Export (Member 1 / Authority)
pem = export_public_key_pem(kp)

# Import (Member 2, 3 — backend/frontend)
pub_key = import_public_key_pem(pem)
```

---

## Running Tests

```bash
python tests/test_crypto.py
```

---

## Backend & System Integration — Member 2

The backend is a Flask server that acts as the Election Authority. It exposes API routes for the frontend to call, uses crypto functions internally, and enforces all system-level security rules.

### What it does

- Generates an RSA key pair on startup (private key never leaves the server)
- Exposes the public key so voters can blind their messages
- Signs blinded messages without ever seeing the real vote
- Verifies unblinded signatures
- Prevents double voting via token tracking

### How to run

```bash
# Install dependencies
pip install flask flask-cors pycryptodome requests

# Start the server (Terminal 1)
python backend/app.py

# Run integration tests (Terminal 2)
python backend/test_backend.py
```

### API Routes

| Route | Method | Purpose |
|---|---|---|
| `/public-key` | GET | Returns RSA public key (n, e, PEM) |
| `/sign` | POST | Signs a blinded message — server stays blind |
| `/verify` | POST | Checks if a signature is valid |
| `/submit-vote` | POST | Verifies + records vote + blocks duplicates |
| `/results` | GET | Returns live vote tally |

### Tests

```
7/7 passed — double voting blocked, forgery blocked, tampering blocked
```

## File Structure

```
crypto/
├── crypto.py        ← All 5 core functions + helpers
├── threat_model.md  ← Formal threat analysis
└── __init__.py      ← Clean public API

tests/
└── test_crypto.py   ← 11 tests covering all cases

```
backend/
├── app.py            ← Flask server (5 routes)
├── test_backend.py   ← Integration test suite (7 tests)
└── requirements.txt  ← flask, flask-cors, pycryptodome, requests
```
