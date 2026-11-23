RFC-001: Ephemeral Agent Credential Delegation & Budget-Bounded Authorization
Author: V.I.M. (Virtual Identity Manager)
Repository: virtualidentitydev/open-virtual-identity-core
Status: Draft / Request for Comments
Date: November 17, 2026
1. Abstract
This Request for Comments (RFC) defines a standardized protocol for delegating scoped, time-bound, and budget-bounded authority from a sovereign root Decentralized Identifier (DID) to autonomous AI agents. By combining W3C Verifiable Credentials (VCs) with ephemeral cryptographic session keys, open-virtual-identity-core eliminates the need for agents to hold long-lived master private keys or static API secrets.
2. Motivation
Autonomous AI agents operating within multi-step tool environments require cryptographic proof of identity and authorization. Current industry practices rely on static bearer tokens or unrestricted API keys, introducing severe security vulnerabilities:
Key Exposure: Compromise of an agent sandbox exposes master credentials with indefinite validity.
Unbounded Execution: Without protocol-level spend and scope limits, looping agents can exhaust financial or compute budgets.
Lack of Verifiable Provenance: Downstream services cannot cryptographically verify whether an agent is authorized by a specific principal without centralized OAuth introspection.
3. Architectural Specification
3.1 Delegation Flow
Agent Session Initialization: The agent runtime generates an ephemeral Ed25519 keypair inside an isolated sandbox (did:key or ephemeral did:web sub-path).
Delegation Request: The runtime submits its ephemeral public key, required tool scopes, requested TTL, and maximum spend ceiling to the Principal Identity Controller.
Credential Issuance: The Principal signs an AgentScopeCredential binding the ephemeral key to explicit operational constraints.
Execution & Verification: The agent presents a Verifiable Presentation (VP) signed with its ephemeral key alongside the delegation VC when invoking external tools or proxies.
3.2 AgentScopeCredential Data Model
The credential extends the W3C Verifiable Credentials Data Model v2.0 with strict execution constraints:
{
  "@context": [
    "https://www.w3.org/ns/credentials/v2",
    "https://virtualidentity.dev/contexts/agent-delegation/v1"
  ],
  "id": "urn:uuid:9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d",
  "type": ["VerifiableCredential", "AgentScopeCredential"],
  "issuer": "did:web:virtualidentity.dev",
  "validFrom": "2026-11-17T16:15:00Z",
  "validUntil": "2026-11-17T17:15:00Z",
  "credentialSubject": {
    "id": "did:key:z6MkpTHR8VNsBxYpjRGGbdPPYwA53V64Mmp4K5G71Q2u",
    "delegatedBy": "did:web:virtualidentity.dev",
    "allowedScopes": [
      "github:discussions:write",
      "social:draft:create"
    ],
    "executionConstraints": {
      "maxBudget": {
        "amount": "5.00",
        "currency": "USD"
      },
      "maxToolCalls": 25,
      "allowedEnvironments": ["personaops-sandbox-v1"]
    }
  },
  "proof": {
    "type": "DataIntegrityProof",
    "cryptosuite": "eddsa-rdfc-2022",
    "created": "2026-11-17T16:15:00Z",
    "verificationMethod": "did:web:virtualidentity.dev#keys-1",
    "proofPurpose": "assertionMethod",
    "proofValue": "z3FXQjecWufY46yg..."
  }
}


4. Security & Revocation Considerations
Short-Lived TTL Enforcement: Recommended maximum validUntil window is 1 hour (3,600 seconds) for interactive sessions and 24 hours for scheduled background workflows.
Cumulative Spend Tracking: Execution proxies (such as personaops-agent-proxy) must maintain deterministic state per credential id to enforce maxBudget across distributed tool calls.
Zero-Knowledge Selective Disclosure: For public verification without revealing internal budget limits or principal metadata, implementations should support BBS+ signatures or SD-JWT encoding.
5. Community Feedback & Next Steps
Comment Period: Open for 14 days in virtualidentitydev/open-virtual-identity-core GitHub Discussions.
Reference Implementation: Map the JSON-LD schema directly into AgentIdentityResolver.ts and add validation middleware to AgentProxySandbox.
Would you like me to save this RFC directly into a new Google Doc or generate the corresponding TypeScript validation interfaces for open-virtual-identity-core?
