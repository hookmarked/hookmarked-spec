# RFC 0001: HookMarked Protocol v0.1

- **Author:** Anthony Busulwa (@ajbusulwa)
- **Status:** Draft
- **Created:** 2025-06-23
- **Target Release:** v0.1.0

---

## 🎯 Summary

This RFC proposes the v0.1 release of the **HookMarked Protocol**, a community-driven specification for defining, discovering, securing, and delivering webhooks in a consistent, validated, and replayable way.

---

## 📦 Motivation

Webhooks power system-to-system automation, but the current ecosystem suffers from:

- No standard way to **describe payloads**
- Inconsistent **signing and security**
- Lack of **discovery**
- Manual or ad hoc **delivery retry logic**
- Poor debugging, especially for failed deliveries

**HookMarked** aims to make webhooks interoperable, testable, and trusted — similar to what OpenAPI did for REST.

---

## 📐 Specification Overview

### 1. Webhook Discovery

  - Services expose a JSON document at:  
    `/.well-known/hooks.json`
  - This file lists all available webhook types, their payload schemas, retry metadata, and signing expectations.
  
  Example:
  
  ```json
  {
    "version": "0.1.0",
    "events": [
      {
        "event": "user.created",
        "schema": "https://example.com/schemas/user.created.json",
        "delivery": {
          "retry_policy": "exponential",
          "max_attempts": 5
        }
      }
    ]
  }


**### 2. Payload Schema Definition**

  Hook payloads must follow JSON Schema Draft-07 or newer
  
  Required root keys:
  
    event: string
  
    timestamp: ISO-8601
  
    payload: object

  Example user.created schema (abridged):
  {
    "$schema": "http://json-schema.org/draft-07/schema#",
    "title": "user.created",
    "type": "object",
    "required": ["event", "timestamp", "payload"],
    "properties": {
      "event": { "const": "user.created" },
      "timestamp": { "type": "string", "format": "date-time" },
      "payload": {
        "type": "object",
        "properties": {
          "id": { "type": "string" },
          "email": { "type": "string", "format": "email" }
        },
        "required": ["id", "email"]
      }
    }
  }

**### 3. Security & Signing**

  Delivery requests must include an HMAC-SHA256 signature in the X-Hookmarked-Signature header

  Signature computed over the raw JSON body using a shared secret

  Optional:

    X-Hookmarked-Timestamp for replay protection

    X-Hookmarked-Nonce to prevent duplication


**### 4. Delivery Rules**
  Webhooks are delivered via HTTP POST

  Expected receiver responses:

    2xx: delivery succeeded

    4xx: delivery failed and should NOT be retried

    5xx: retry with backoff

  Suggested retry policy:

    Exponential backoff with jitter

    Max attempts configurable per hook type

  HookMarked-compliant senders must persist and retry deliverable events according to policy.



**### 5. Versioning**
  The discovery file and schema URLs must indicate a protocol or payload version

  Clients should declare support in X-Hookmarked-Version or Accept headers


🧩 Extensibility
  Future features may include:
  
    Delivery receipts
    
    Event chaining
    
    Encryption of payloads
    
    Spec is versioned to allow backward-compatible extensions

📘 Tooling
  hookmarked-cli: Validate discovery files, schemas, and simulate deliveries

  hookmarked-site: Web-based docs, playground, and reference validator

  SDKs in Node.js, Python, Go (planned)

🧠 Adoption Goals
  SaaS platforms
  
  Developer APIs
  
  Internal microservices
  
  Event-driven backends

📅 Changelog
  2025-06-23: Initial draft of HookMarked Protocol v0.1

🪪 License
  MIT
