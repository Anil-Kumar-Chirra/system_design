# Complete Authentication & Authorization Guide

## Table of Contents
1. [Introduction](#introduction)
2. [Authentication vs Authorization](#authentication-vs-authorization)
3. [Authentication Methods](#authentication-methods)
4. [Authorization Models](#authorization-models)
5. [Token-Based Authentication](#token-based-authentication)
6. [OAuth 2.0 & OpenID Connect](#oauth-20--openid-connect)
7. [Session Management](#session-management)
8. [Security Best Practices](#security-best-practices)
9. [Common Vulnerabilities](#common-vulnerabilities)
10. [Python Implementation Overview](#python-implementation-overview)
11. [Tools & Libraries](#tools--libraries)
12. [References](#references)

---

## Introduction

Authentication and authorization are fundamental security concepts that control access to applications and resources. This guide provides comprehensive coverage of both concepts, implementation strategies, and best practices at a high level.

**Key Definitions:**
- **Authentication**: "Who are you?" - Verifying the identity of a user
- **Authorization**: "What can you do?" - Determining what an authenticated user can access

---

## Authentication vs Authorization

### Authentication (AuthN)
```
User Credentials → Verification Process → Identity Confirmed
```

**Purpose**: Verify user identity  
**Question**: "Are you who you claim to be?"  
**Methods**: Passwords, biometrics, certificates, tokens  
**Result**: User identity established  

### Authorization (AuthZ)
```
Authenticated User → Permission Check → Access Granted/Denied
```

**Purpose**: Control access to resources  
**Question**: "What are you allowed to do?"  
**Methods**: Roles, permissions, policies, ACLs  
**Result**: Access decision made  

### Flow Diagram
```
┌─────────────┐    ┌──────────────┐    ┌─────────────┐
│   User      │───▶│Authentication│───▶│Authorization│
│ Credentials │    │   Process    │    │   Check     │
└─────────────┘    └──────────────┘    └─────────────┘
                          │                    │
                          ▼                    ▼
                   ┌─────────────┐    ┌─────────────┐
                   │  Identity   │    │   Access    │
                   │ Established │    │   Decision  │
                   └─────────────┘    └─────────────┘
```

---

## Authentication Methods

### 1. Password-Based Authentication
- **Traditional**: Username + password combination
- **Strengths**: Simple, widely understood
- **Weaknesses**: Vulnerable to attacks, password fatigue
- **Improvements**: Password policies, hashing (bcrypt, scrypt, argon2)

### 2. Multi-Factor Authentication (MFA)
- **Something you know**: Password, PIN
- **Something you have**: Phone, hardware token, smart card
- **Something you are**: Fingerprint, face, voice, iris

**Types**:
- **SMS/Email OTP**: One-time passwords sent via message
- **TOTP**: Time-based tokens (Google Authenticator, Authy)
- **Hardware tokens**: YubiKey, RSA SecurID
- **Biometric**: Fingerprint, facial recognition

### 3. Certificate-Based Authentication
- **Digital certificates**: X.509 certificates for identity
- **PKI**: Public Key Infrastructure for certificate management
- **Use cases**: Enterprise environments, API authentication

### 4. Social Authentication
- **OAuth providers**: Google, Facebook, GitHub, Microsoft
- **Benefits**: No password management, user convenience
- **Considerations**: Dependency on third-party, privacy concerns

### 5. Passwordless Authentication
- **Magic links**: Email-based authentication
- **WebAuthn**: FIDO2 standard for passwordless login
- **Push notifications**: Mobile app-based approval

---

## Authorization Models

### 1. Role-Based Access Control (RBAC)

**Concept**: Users assigned to roles, roles have permissions

```
User → Role → Permissions → Resources
```

**Components**:
- **Users**: Individual entities
- **Roles**: Job functions (admin, editor, viewer)
- **Permissions**: Actions (read, write, delete)
- **Resources**: Protected objects

**Benefits**:
- Easy to understand and manage
- Scalable for organizations
- Principle of least privilege

**Limitations**:
- Role explosion in complex systems
- Difficult to handle exceptions

### 2. Attribute-Based Access Control (ABAC)

**Concept**: Access decisions based on attributes of users, resources, and environment

**Attributes**:
- **Subject**: User department, clearance level, location
- **Object**: Resource classification, owner, type
- **Action**: Read, write, execute, delete
- **Environment**: Time, location, network, device

**Policy Structure**:
```
IF (user.department == "Finance" AND 
    resource.classification == "Internal" AND 
    time BETWEEN 9:00-17:00)
THEN ALLOW
```

**Benefits**:
- Fine-grained control
- Dynamic decisions
- Flexible policies

**Challenges**:
- Complex to implement
- Performance overhead
- Policy management complexity

### 3. Access Control Lists (ACL)

**Concept**: Direct assignment of permissions to users for specific resources

```
Resource → [(User1, Permissions), (User2, Permissions), ...]
```

**Use cases**:
- File systems (Unix/Linux permissions)
- Network access control
- Simple applications

**Benefits**:
- Direct control
- Easy to understand
- No abstraction layer

**Limitations**:
- Difficult to scale
- Maintenance overhead
- No centralized management

### 4. Discretionary Access Control (DAC)

**Concept**: Resource owners control access to their resources

**Characteristics**:
- Owner can grant/revoke access
- Flexible but potentially insecure
- Common in file systems

### 5. Mandatory Access Control (MAC)

**Concept**: System-enforced access control based on security labels

**Characteristics**:
- Security labels (Top Secret, Secret, Confidential)
- Users cannot override system policies
- Used in high-security environments

---

## Token-Based Authentication

### JSON Web Tokens (JWT)

**Structure**: `header.payload.signature`

**Header** (Algorithm and token type):
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

**Payload** (Claims):
```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "role": "admin",
  "iat": 1516239022,
  "exp": 1516242622
}
```

**Signature**: Verification of token integrity

**JWT Types**:
- **Access Token**: Short-lived (15 minutes - 1 hour)
- **Refresh Token**: Long-lived (days to months)
- **ID Token**: User identity information

**Benefits**:
- Stateless
- Self-contained
- Cross-domain support
- Standard format

**Considerations**:
- Token size
- No server-side revocation
- Secret key management

### Other Token Types

**Opaque Tokens**:
- Random strings
- Server-side lookup required
- Easy to revoke

**Bearer Tokens**:
- Simple to use
- No cryptographic proof of possession
- Vulnerable if intercepted

---

## OAuth 2.0 & OpenID Connect

### OAuth 2.0 Overview

**Purpose**: Authorization framework for third-party access

**Key Concepts**:
- **Resource Owner**: User who owns the data
- **Client**: Application requesting access
- **Authorization Server**: Issues tokens
- **Resource Server**: Hosts protected resources

### OAuth 2.0 Flows

#### 1. Authorization Code Flow
```
User → Client → Auth Server → User Consent → Auth Code → Client → Access Token
```
**Use case**: Web applications with server-side code

#### 2. Implicit Flow (Deprecated)
```
User → Client → Auth Server → Access Token (in URL fragment)
```
**Use case**: Single-page applications (now use PKCE)

#### 3. Client Credentials Flow
```
Client → Auth Server → Access Token
```
**Use case**: Machine-to-machine communication

#### 4. Resource Owner Password Credentials (Not recommended)
```
Client → Username/Password → Auth Server → Access Token
```
**Use case**: Trusted first-party applications only

#### 5. Authorization Code + PKCE
**PKCE**: Proof Key for Code Exchange
- Adds security for public clients
- Code verifier and code challenge
- Prevents authorization code interception

### OpenID Connect (OIDC)

**Purpose**: Authentication layer on top of OAuth 2.0

**Additional Components**:
- **ID Token**: User identity (JWT format)
- **UserInfo Endpoint**: Additional user claims
- **Discovery**: Automatic configuration

**Claims**:
- **Standard**: sub, name, email, picture
- **Custom**: Application-specific information

---

## Session Management

### Traditional Sessions

**Server-side storage**:
- Session data stored on server
- Session ID sent to client (cookie)
- Stateful approach

**Benefits**:
- Server control
- Easy revocation
- Secure data storage

**Challenges**:
- Scalability issues
- Load balancing complexity
- Memory usage

### Distributed Sessions

**Solutions**:
- **Sticky sessions**: Route to same server
- **Session replication**: Copy across servers
- **External store**: Redis, database
- **Stateless tokens**: JWT approach

### Cookie Security

**Attributes**:
- **HttpOnly**: Prevent XSS access
- **Secure**: HTTPS only
- **SameSite**: CSRF protection
- **Domain/Path**: Scope limitation

---

## Security Best Practices

### Authentication Security

1. **Password Security**:
   - Strong password policies
   - Password hashing (bcrypt, scrypt, argon2)
   - Salt usage
   - Rate limiting

2. **Multi-Factor Authentication**:
   - Implement for sensitive operations
   - Support multiple MFA methods
   - Backup codes for recovery

3. **Account Security**:
   - Account lockout policies
   - Suspicious activity detection
   - Email notifications for security events

### Authorization Security

1. **Principle of Least Privilege**:
   - Grant minimum necessary permissions
   - Regular access reviews
   - Time-bound access

2. **Defense in Depth**:
   - Multiple authorization layers
   - Resource-level checks
   - Audit logging

3. **Secure by Default**:
   - Deny by default
   - Explicit permissions required
   - Fail securely

### Token Security

1. **JWT Security**:
   - Strong signing keys
   - Short expiration times
   - Proper key rotation
   - Validate all claims

2. **Storage Security**:
   - Secure storage (not localStorage for sensitive tokens)
   - Proper transmission (HTTPS)
   - Token revocation mechanisms

### General Security

1. **Communication**:
   - HTTPS everywhere
   - Certificate pinning
   - HSTS headers

2. **Input Validation**:
   - Validate all inputs
   - Sanitize data
   - Parameterized queries

3. **Monitoring**:
   - Security logging
   - Anomaly detection
   - Real-time alerts

---

## Common Vulnerabilities

### Authentication Vulnerabilities

1. **Brute Force Attacks**:
   - **Mitigation**: Rate limiting, account lockout, CAPTCHA

2. **Credential Stuffing**:
   - **Mitigation**: MFA, device fingerprinting, behavioral analysis

3. **Session Fixation**:
   - **Mitigation**: Regenerate session ID after login

4. **Session Hijacking**:
   - **Mitigation**: HTTPS, secure cookies, IP validation

### Authorization Vulnerabilities

1. **Privilege Escalation**:
   - **Horizontal**: Access other users' data
   - **Vertical**: Gain higher privileges
   - **Mitigation**: Proper access controls, input validation

2. **Insecure Direct Object References**:
   - **Example**: `/user/123/profile` accessible by changing ID
   - **Mitigation**: Authorization checks, indirect references

3. **Missing Function Level Access Control**:
   - **Example**: Admin functions accessible to regular users
   - **Mitigation**: Consistent authorization checks

### Token Vulnerabilities

1. **Token Interception**:
   - **Mitigation**: HTTPS, secure storage, short expiration

2. **Token Replay**:
   - **Mitigation**: Short-lived tokens, refresh token rotation

3. **JWT Attacks**:
   - **Algorithm confusion**: `alg: none` attacks
   - **Weak secrets**: Dictionary attacks on HMAC keys
   - **Mitigation**: Algorithm whitelisting, strong keys

---

## Python Implementation Overview

### Key Libraries for Authentication & Authorization

```python
# Authentication
import bcrypt          # Password hashing
import pyotp          # TOTP/HOTP implementation
import jwt            # JWT tokens
from passlib.context import CryptContext  # Password hashing utilities

# Authorization
from functools import wraps  # Decorators
import casbin            # Authorization library (RBAC/ABAC)
from flask_principal import Principal, Permission, RoleNeed  # Flask authorization

# OAuth & Social Auth
from authlib.integrations.flask_client import OAuth  # OAuth client
import requests_oauthlib  # OAuth for requests
from flask_dance.contrib.google import make_google_blueprint  # Social auth

# Session Management
from flask_session import Session  # Server-side sessions
import redis              # Session storage
from flask_login import LoginManager  # User session management
```

### Framework-Specific Solutions

**Flask**:
```python
from flask_login import LoginManager, login_required
from flask_principal import Principal, Permission, RoleNeed
from flask_jwt_extended import JWTManager, create_access_token
```

**Django**:
```python
from django.contrib.auth import authenticate, login
from django.contrib.auth.decorators import login_required, permission_required
from rest_framework.permissions import IsAuthenticated
from rest_framework_simplejwt.tokens import RefreshToken
```

**FastAPI**:
```python
from fastapi.security import HTTPBearer, OAuth2PasswordBearer
from fastapi import Depends, HTTPException, status
from passlib.context import CryptContext
import python-jose  # JWT handling
```

### High-Level Implementation Patterns

1. **Authentication Middleware**:
   - Intercept requests
   - Validate credentials/tokens
   - Set user context

2. **Authorization Decorators**:
   - Protect functions/routes
   - Check permissions
   - Return appropriate responses

3. **User Management**:
   - User model/schema
   - Password hashing
   - Profile management

4. **Token Management**:
   - Token generation
   - Validation and refresh
   - Blacklisting/revocation

---

## Tools & Libraries

### Python Authentication Libraries

| Library | Purpose | Features |
|---------|---------|----------|
| **bcrypt** | Password hashing | Secure password hashing with salt |
| **passlib** | Password utilities | Multiple hashing algorithms, migration |
| **PyJWT** | JWT tokens | Create, verify, decode JWT tokens |
| **pyotp** | OTP generation | TOTP, HOTP for 2FA |
| **cryptography** | Cryptographic operations | Low-level crypto primitives |

### Python Authorization Libraries

| Library | Purpose | Features |
|---------|---------|----------|
| **casbin** | Authorization | RBAC, ABAC, ACL models |
| **flask-principal** | Flask authorization | Role-based permissions |
| **django-guardian** | Django object permissions | Per-object permissions |
| **authlib** | OAuth client/server | OAuth 1.0/2.0, OpenID Connect |

### Framework Integration

| Framework | Auth Libraries | Features |
|-----------|---------------|----------|
| **Flask** | Flask-Login, Flask-Principal, Flask-JWT-Extended | Session management, permissions, JWT |
| **Django** | Built-in auth, DRF permissions | User model, permissions, middleware |
| **FastAPI** | python-jose, passlib, OAuth2 | Dependency injection, OAuth2 flows |

### External Services

| Service | Type | Features |
|---------|------|----------|
| **Auth0** | Identity platform | Complete auth solution, SDKs |
| **Firebase Auth** | Google service | Social login, phone auth |
| **AWS Cognito** | AWS service | User pools, identity pools |
| **Okta** | Enterprise IdP | SSO, MFA, user management |
| **Keycloak** | Open source IdP | SSO, OAuth, SAML |

---

## References

### Official Documentation

- **OAuth 2.0**: [RFC 6749](https://tools.ietf.org/html/rfc6749)
- **OpenID Connect**: [OpenID Connect Core](https://openid.net/specs/openid-connect-core-1_0.html)
- **JWT**: [RFC 7519](https://tools.ietf.org/html/rfc7519)
- **PKCE**: [RFC 7636](https://tools.ietf.org/html/rfc7636)
- **WebAuthn**: [W3C WebAuthn](https://www.w3.org/TR/webauthn/)

### Security Standards

- **OWASP Authentication Cheat Sheet**: [OWASP Auth](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- **NIST Digital Identity Guidelines**: [NIST SP 800-63](https://pages.nist.gov/800-63-3/)
- **SAML 2.0**: [OASIS SAML](https://docs.oasis-open.org/security/saml/v2.0/)

### Python-Specific Resources

- **Django Authentication**: [Django Docs](https://docs.djangoproject.com/en/stable/topics/auth/)
- **Flask-Login**: [Flask-Login Docs](https://flask-login.readthedocs.io/)
- **FastAPI Security**: [FastAPI Security](https://fastapi.tiangolo.com/tutorial/security/)
- **Authlib Documentation**: [Authlib Docs](https://docs.authlib.org/)

### Best Practice Guides

- **Auth0 Blog**: [Auth0 Resources](https://auth0.com/blog/)
- **OWASP Top 10**: [OWASP](https://owasp.org/www-project-top-ten/)
- **Google Identity Best Practices**: [Google Identity](https://developers.google.com/identity)
- **Microsoft Identity Platform**: [Microsoft Docs](https://docs.microsoft.com/en-us/azure/active-directory/develop/)

---

## Summary

This guide covers the essential concepts of authentication and authorization:

1. **Authentication** verifies "who you are" through various methods (passwords, MFA, certificates)
2. **Authorization** determines "what you can do" using models like RBAC, ABAC, and ACL
3. **Token-based systems** (JWT, OAuth 2.0) provide scalable, stateless solutions
4. **Security best practices** include proper hashing, HTTPS, rate limiting, and monitoring
5. **Python ecosystem** offers comprehensive libraries for implementing these concepts

The key to successful implementation is understanding the trade-offs between security, usability, and performance, then choosing the appropriate methods for your specific use case and threat model.