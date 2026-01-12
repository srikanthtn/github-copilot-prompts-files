# Security Baselines – Global

## Purpose
Define mandatory security requirements applicable across all financial domains.

## Core Security Rules
- All systems must operate deterministically.
- Secrets, credentials, and keys must never be hardcoded.
- Sensitive data must not be exposed through logs or errors.
- Cryptographic operations must use approved libraries only.

## Access Control
- Access must follow the principle of least privilege.
- Authorization checks must be explicit and traceable.
- Privilege escalation must be controlled and auditable.

## Prohibited
- Dynamic code execution
- Reflection-based access to sensitive internals
- Bypassing security controls
- Obfuscation intended to hide behavior

## Enforcement
Violation of this file invalidates system compliance.
