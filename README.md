# threat-model-secure-configuration

# Threat Model and Secure Configuration

## Project Information

- **Track:** Cyber Security
- **Level:** Beginner-friendly
- **Estimated Time:** 45–60 minutes
- **Application:** Notely App
- **Category:** Threat Modeling & Application Security

---

## Target Scenario: Small B2B SaaS Application

To make the analysis concrete and practical, this threat model evaluates a standard **Multi-Tenant Note & Document Collaboration SaaS**, called **Notely App**.

The application consists of the following components:

| Component | Technology |
|---|---|
| Frontend | Single Page Application (React) |
| Backend | REST/GraphQL API Gateway (Node.js/Express) |
| Database | Managed PostgreSQL |
| Payment Integration | Stripe |
| Email Integration | SendGrid |
| File Storage | AWS S3 |
| Security & Perimeter | Cloudflare CDN & WAF |
| Identity Provider | Auth0 / OAuth IdP |

The application allows multiple business customers (tenants) to manage notes and collaborate on documents within a shared SaaS environment.

---

# 1. System Architecture & Data Flow Diagram (DFD)

The architecture diagram maps the application's:

- Actors
- Data stores
- External dependencies
- Communication channels
- Trust boundaries

## 1.1 Architecture Overview

```text
========================= UNTRUSTED ZONE =========================

                    [ Browser / Mobile Client ]
                                |
                                | HTTPS / TLS 1.3
                                v

---------------- TRUST BOUNDARY 1 (Internet / Perimeter) ----------

                 [ Cloudflare CDN & WAF / Reverse Proxy ]
                                |
                                | Reverse Proxy (mTLS / Internal VPC)
                                v

========================= APPLICATION DMZ =========================

                      [ API Gateway / Web Server ]
                      - Authentication Middleware
                      - Input Validation & Rate Limiter
                      - Business Logic & RBAC

                          /          |          \
                         /           |           \
                Internal SQL      Internal      HTTPS / SDK
                    / TCP           Logic            |
                   /                |                |
                  v                 v                v

--------- TRUST BOUNDARY 2 ----------- TRUST BOUNDARY 3 -----------

[ Managed PostgreSQL ]     [ Local File Cache / S3 ]    [ Third-Party APIs ]

- Users & Hash PII          - Uploaded Document Blobs   - Stripe (Billing)
- Tenant Document Tables   - Presigned URL Generator   - SendGrid (Email)
- Audit & Auth Logs                                      - Auth0 / OAuth IdP

===================== DATA PERSISTENCE ZONE =======================
```

## 1.2 Application Components

### 1. Browser / Mobile Client
- Acts as the entry point for users.
- Communicates with the application over HTTPS.
- Uses TLS 1.3 for secure communication.

### 2. Cloudflare CDN & WAF / Reverse Proxy
- Acts as the internet-facing security layer.
- Receives incoming client requests.
- Provides CDN and Web Application Firewall (WAF) functionality.
- Forwards requests to the internal application through a reverse proxy connection.

### 3. API Gateway / Web Server
- Built using Node.js / Express.
- Handles incoming API requests.
- Contains authentication middleware.
- Performs input validation and rate limiting.
- Implements business logic and Role-Based Access Control (RBAC).

### 4. Managed PostgreSQL
Stores application-related information:

- Users & Hash PII
- Tenant Document Tables
- Audit & Auth Logs

### 5. Local File Cache / S3
Handles document storage:

- Uploaded Document Blobs
- Presigned URL Generator

### 6. Third-Party APIs

The application integrates with external services:

| Service | Purpose |
|---|---|
| Stripe | Billing |
| SendGrid | Email |
| Auth0 / OAuth IdP | Identity Provider |

---

## 1.3 Trust Boundaries

Trust boundaries represent the separation between different security zones in the application.

| Trust Boundary | Description |
|---|---|
| Trust Boundary 1 | Separates untrusted internet traffic from the application's perimeter and infrastructure. |
| Trust Boundary 2 | Separates the application environment from the managed PostgreSQL database and file storage. |
| Trust Boundary 3 | Separates the application from external third-party APIs and integrations. |

### Key Takeaway

The Data Flow Diagram (DFD) helps identify:

- How data moves through the application.
- Where sensitive information is stored.
- Which external services are involved.
- Where trust boundaries exist.
- Where security controls must be implemented.

**Purpose:** Understanding the system architecture and data flow is the foundation for identifying potential threats and designing a secure application.
