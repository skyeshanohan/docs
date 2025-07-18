# Security Questionnaire - Supplemental Information & FAQ

## Overview

This document provides supplemental information and frequently asked questions (FAQ) for the security questionnaire in Jira. This questionnaire helps assess the security implications and compliance requirements for new system development or modifications.

---

## Question 1: Compliance Requirements Awareness

**Question:** Are you aware of any compliance requirements for this system?

### What this means:
This question helps identify if your system needs to meet specific regulatory or industry compliance standards.

### Guidance:
- **FedRAMP**: Required when your system will be used by federal agencies or processes federal data
- **Hitrust**: Required for systems handling sensitive data, especially when working with partners who require Hitrust certification
- **PCI**: Required when your system directly processes credit card payments (not required if using third-party payment processors)
- **Other**: If your initiative falls under specific state or other regulatory requirements
- **No**: System has no specific compliance requirements
- **Unsure**: Select this if you need to consult with compliance teams


---

## Question 2: Sensitive Data Handling

**Question:** Does the new development handle sensitive data?

### What this means:
This determines if your system processes, stores, or transmits sensitive information that requires special protection.

### Guidance:
**Select "Yes" if your system handles:**
- Personally Identifiable Information (PII)
- Financial data (credit cards, bank accounts)
- Health information (PHI)

**Select "No" if:**
- System only handles public information
- No personal or sensitive data is processed
- System is purely informational with no data collection

---

## Question 3: System Access Methods

**Question:** How is the system accessed?

### What this means:
This identifies the access patterns and security boundaries for your system.

### Guidance:
- **Unauthenticated - public users**: Anyone can access without login (e.g., public websites)
- **Authenticated - External users (public internet)**: External users with login credentials
- **External users (VPN/Controlled)**: External parties access through secure VPN
- **Internal users (public internet)**: Company employees access from outside the corporate network
- **Internal users (VPN/Controlled)**: Company employees access through corporate VPN
- **Integration with existing service/application**: System-to-system communication via APIs


**Note:** You can select multiple options if your system has different access patterns for different user types.

---

## Question 4: IAM Permissions Changes

**Question:** Are any new roles or IAM permissions being added or modified?

### What this means:
This identifies if your system introduces new user roles, permissions, or access controls.

### Guidance:
**Select "Yes" if:**
- Creating new user roles or groups
- Adding new permissions to existing roles
- Implementing role-based access control (RBAC)
- Creating service accounts with specific permissions
- Modifying existing permission structures

**Select "No" if:**
- Using existing roles and permissions
- No changes to access control structure
- System inherits permissions from existing infrastructure

---

## Question 5: Authentication Method Changes

**Question:** Are any new authentication methods being added or modified?

### What this means:
This identifies if your system introduces new ways for users to prove their identity.

### Guidance:
**Select "Yes" if:**
- Implementing new authentication providers (OAuth, SAML, etc.)
- Adding multi-factor authentication (MFA)
- Creating new login systems
- Integrating with new identity providers
- Modifying existing authentication flows
- Implementing single sign-on (SSO)

**Select "No" if:**
- Using existing authentication systems
- No changes to login processes
- System inherits authentication from existing infrastructure

---

## Question 6: Network Controls Changes

**Question:** Are any new network controls being added or modified (e.g., public subnets)?

### What this means:
This identifies if your system requires changes to network architecture or security controls.

### Guidance:
**Select "Yes" if:**
- Adding public-facing components (ALB, API Gateway, etc.)
- Implementing new network access controls
- Creating new subnets or VPCs

**Select "No" if:**
- Using existing security groups and network infrastructure
- No changes to network architecture
- System operates within existing network boundaries

---

### Contact Information:
- **Secure Product & Infrastructure**: [add slack channel]



---

*This FAQ is designed to help teams provide accurate and complete responses to the security questionnaire. If you have questions not covered here, please contact SPI for guidance.* 