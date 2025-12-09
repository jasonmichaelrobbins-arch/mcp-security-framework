[README-MCP.md](https://github.com/user-attachments/files/24049563/README-MCP.md)
# MCP Security Framework

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](http://makeapullrequest.com)
[![MCP](https://img.shields.io/badge/MCP-Security-blue.svg)](https://modelcontextprotocol.io)

A practical security controls framework for Model Context Protocol (MCP) deployments—comparing internal (self-hosted) vs. external (vendor-hosted) architectures with prioritized controls, validation questions, and decision guidance.

---

## Why This Exists

MCP is becoming the standard interface between AI agents and the tools/data they need to operate. That's powerful—and dangerous.

Security teams are scrambling to answer basic questions:
- What controls do we need before putting this in production?
- How do we evaluate vendor-hosted MCP services vs. running our own?
- What's P1 vs. "nice to have"?

This framework provides opinionated, prioritized guidance based on real-world implementation experience. It's not theoretical—it's what you actually need to think about before your agents start calling tools.

**This is a living document.** MCP security practices are evolving rapidly. Contributions welcome.

---

## Table of Contents

- [Priority Model](#priority-model)
- [Shared Concerns](#shared-concerns)
- [Internal vs. External Comparison](#internal-vs-external-comparison)
- [Security Validation Questions](#security-validation-questions)
- [Decision Framework](#decision-framework)
- [Contributing](#contributing)
- [License](#license)
- [Author](#author)

---

## Priority Model

| Priority | Level | Description |
|:--------:|-------|-------------|
| **P1** | Severe | Must have before production. Fundamental security controls. |
| **P2** | High | Should have for production. Important defense-in-depth. |
| **P3** | Medium | Plan for implementation. Governance and operational maturity. |
| **P4** | Recommended | Nice to have. Advanced capabilities for mature programs. |

---

## Shared Concerns

These controls are essential **regardless of deployment model**—internal or external.

| Priority | Area | Control | Key Question |
|:--------:|------|---------|--------------|
| **P1** | Kill switches | Ability to quickly disable tools, endpoints or entire MCP connections | Can you disable any agent/tool within five minutes? |
| **P1** | Authentication | Strong identity verification for all callers (humans, agents, services) | Can an unauthenticated request ever reach the MCP? |
| **P1** | Logging | Capture prompts, tool calls and responses with appropriate redaction | Can you reconstruct what an agent did and why? |
| **P1** | Prompt injection defense | Input sanitization, injection detection, output validation | What happens if malicious content is in retrieved documents? |
| **P2** | Human-in-the-loop | Require approval for high-risk, irreversible or financial actions | Which actions can an agent take without human approval? |
| **P2** | Data classification | Label and control data by sensitivity before it reaches MCP | Do you know what data sensitivity levels the MCP can access? |
| **P2** | Tool minimization | Only enable tools necessary for the use case; treat each tool as a risk | Is there a tool enabled that isn't actively needed? |
| **P3** | Lifecycle governance | Catalog, risk-tier and manage MCP servers/connections through lifecycle | Do you have a single source of truth for all MCP connections? |
| **P3** | Incident response | AI-specific playbooks for containment, evidence preservation and communication | Do you have an incident response playbook specifically for AI/agent incidents? |
| **P3** | Red-teaming | Test for prompt injection, data exfiltration, tool abuse, jailbreaks | When did you last red-team your MCP integrations? |

---

## Internal vs. External Comparison

The full comparison across 18 security domains.

### P1 — Severe (Must Have)

| Area | Internal (Self-hosted) | External (Vendor-hosted) |
|------|------------------------|--------------------------|
| **Scope and risk tier** | Classify by data sensitivity, integration depth, agent capability (read-only → supervised → autonomous). Use 4-tier priority model. | Classify by what data you'll send and what actions the vendor can perform. Start restrictive, expand only with justification. |
| **Network** | No public IPs. Private subnets only. Access via VPN/ExpressRoute. Inbound only from gateway/bastion. | All traffic via controlled egress (API management/gateway). Allowlist vendor FQDNs. Use private link if available. |
| **Identity and auth** | Entra ID + OAuth 2.1. Validate JWT issuer/audience exactly. Short-lived tokens. Token binding for replay prevention. | Prefer your identity provider (Entra) over vendor accounts. Separate keys per environment. Verify vendor token handling. Confirm no sub-processor access. |
| **Authorization** | Enforce at gateway AND backend. Object-level authz. Map Entra claims to MCP roles (admin/owner/user/read-only). | Map internal roles to vendor permissions. Avoid broad admin scopes. Require approval workflows for high-risk actions. |
| **Secrets and non-human identities** | Unique service principal per MCP/agent. Managed identities preferred. Vault for secrets. Rotation + expiration alerts. | Keys only in vault, injected at gateway. Never expose keys to users/agents. Rotate keys on schedule and staff departure. |
| **Input validation** | Direct and indirect prompt injection defense. Input sanitization. Output validation. Schema validation. Encoding checks. | DLP/redaction at gateway. Prompt shielding. Strip secrets before sending. Context length limits. |

### P2 — High (Should Have)

| Area | Internal (Self-hosted) | External (Vendor-hosted) |
|------|------------------------|--------------------------|
| **Tool execution** | Container/VM isolation. Resource limits (CPU/memory/time). Network segmentation. Filesystem isolation. Recursive call limits. | Vendor tools are black boxes. Broker via your APIs. No direct write to core systems. Monitor for capability drift. |
| **Data protection** | Classify data. Segment sensitive indexes. Access checks at retrieval. Data lineage. Unlearning/deletion. Poisoning detection. | Define allowed data classes. Start with non-sensitive data. Synthetic data for testing. Minimal vendor logging. Differential privacy. |
| **Session/memory** | Encrypt context at rest. Session timeouts. Cross-session isolation. Bound memory size. Automatic clearing. | Verify vendor session isolation. Confirm no cross-tenant leakage. Test for context persistence across sessions. |
| **Gateway layer** | Centralized gateway for all MCP access. JWT validation at edge. Rate limiting. Schema validation. Request signing. | Force all traffic through your gateway. Inject vendor keys at gateway. Log everything. Rate limit per user/agent. |
| **Supply chain** | Assess model dependencies, libraries, base images. Monitor for vulnerabilities in tool integrations. | Map vendor's model providers and sub-processors. Understand inference location. Require notification of upstream changes. |
| **Logging and monitoring** | Full tracing with redaction. Log integrity (WORM). Oversight agents. Metrics. Correlation IDs. Alert on anomalies. | Log everything on your side. Ingest vendor signals. Correlation IDs. Behavioral baselines. Response integrity monitoring. |
| **Availability/SLAs** | Define RTO/RPO. Backup configs and indices. Geographic redundancy. Disaster recovery testing. | Negotiate SLAs (uptime, latency). Degradation plans. Failover options. Monitor vendor status. |

### P3 — Medium (Plan For)

| Area | Internal (Self-hosted) | External (Vendor-hosted) |
|------|------------------------|--------------------------|
| **Governance** | Catalog MCP servers/agents. Risk tier → capabilities. Change management. Promotion gates (shadow → autonomous). | AI-flavored vendor risk assessment. SOC 2/ISO. Risk tier per vendor. Concentration risk. Financial health. |
| **Portability** | Version control configs. Documented deployment. Reproducible infrastructure. | Assess lock-in. API compatibility with alternatives. Abstraction layers. Migration runbooks. Data export. |
| **Integrity verification** | Request signing. Replay prevention. TLS everywhere, including internal. | Response authenticity. Cert pinning. Detect response modifications. Monitor for vendor-side injection. |
| **Legal/compliance** | Data security posture management (DSPM) for data residency. Regulatory mapping. Multi-tenancy controls if applicable. | DPA/BAA. AI clauses (no training, IP ownership). Cyber insurance. Liability terms. Regulatory fit. |
| **Testing and validation** | Threat model. Red team. Config review. Continuous security testing in CI/CD. Resource exhaustion tests. | Sandbox with synthetic data. Red-team integration. Continuous evaluation. Canary deployments. A/B testing. Cross-tenant tests. |

### P4 — Recommended (Nice to Have)

| Area | Internal (Self-hosted) | External (Vendor-hosted) |
|------|------------------------|--------------------------|
| **Multi-tenancy** | Tenant isolation at MCP. Scoped indices. Per-tenant keys. Per-tenant rate limits and audit logs. | Verify vendor tenant isolation. Test for cross-tenant access. Contractual isolation guarantees. |

---

## Security Validation Questions

Quick-hit questions to assess your MCP security posture.

### Internal MCP Servers

- [ ] Can requests reach MCP without going through the gateway?
- [ ] What happens if a token is stolen?
- [ ] Can one user access another user's data by manipulating IDs?
- [ ] What tools can an agent call without human approval?
- [ ] How quickly can you disable a misbehaving agent?
- [ ] Where are secrets for this MCP server stored?
- [ ] Can you reconstruct what an agent did last week?
- [ ] What's in your backup for this MCP and when was it tested?

### External MCP Servers

- [ ] Can any user/agent call the vendor directly (bypassing your gateway)?
- [ ] Where does the vendor store your API keys and for how long?
- [ ] What data is the vendor allowed to log and retain?
- [ ] Which of the vendor's tools can write to your systems?
- [ ] How quickly can you cut all traffic to this vendor?
- [ ] What happens to your data if the vendor is acquired?
- [ ] Does the vendor train models on your prompts/responses?
- [ ] What's your plan if this vendor has a major outage?

---

## Decision Framework

**When to choose internal vs. external MCP servers.**

| Favor Internal MCP Servers When... | External May Be Acceptable When... |
|------------------------------------|-------------------------------------|
| Processing PII, PHI, PCI or highly confidential data | Working with public data or low-sensitivity internal data |
| Regulatory requirements mandate data residency control | Vendor meets all compliance requirements with attestation |
| Need full control over model behavior and guardrails | Vendor guardrails are sufficient for use case |
| Actions have high financial or operational impact | Actions are read-only or easily reversible |
| Require deep integration with internal systems | Integration is limited to specific, well-bounded use cases |
| Long-term strategic capability requiring investment | Rapid experimentation or time-to-value is critical |

---

## Contributing

This framework will evolve as MCP matures and attack patterns emerge. Contributions welcome:

- **Issues:** Found a gap? Have a question? Open an issue.
- **Pull Requests:** Additions, clarifications, real-world examples—all welcome.
- **Experience reports:** Implemented this? I'd love to hear what worked and what didn't.

---

## License

This work is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). 

You are free to:
- **Share** — copy and redistribute in any medium or format
- **Adapt** — remix, transform, and build upon the material for any purpose

With attribution.

---

## Author

**Jason Robbins**

---

*Built for the security community. Not paywalled.*
