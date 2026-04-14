# Azure Shared Context

All Azure advisory skills MUST read and apply this context before giving advice.

## Organisation Profile

- **Institution**: Lancaster University
- **Sector**: UK Higher Education
- **Network**: JANET (Jisc-managed academic network)
- **Jisc membership**: Full member. Jisc frameworks, procurement routes, and shared services are available and preferred where they reduce cost or complexity.

## UK Region Requirements

- **Primary region**: UK South (`uksouth`)
- **DR / secondary region**: UK West (`ukwest`)
- **Data residency**: All data MUST reside in UK regions unless there is an explicit, documented exception (e.g., a global CDN edge or a service with no UK region availability). If a service is not available in UK South/West, flag it and propose alternatives.
- **Rationale**: GDPR, institutional data governance policy, and sector expectations.

## Compliance Baseline

| Framework | Status | Notes |
|-----------|--------|-------|
| GDPR / UK GDPR | Required | All personal data processing must comply |
| Cyber Essentials | Held | Current certification; architecture must not break CE controls |
| NCSC Cloud Security Principles | Follow | 14 principles for evaluating cloud services; reference when selecting services |
| Jisc Security Framework | Follow | Sector-specific security guidance |
| ISO 27001 | Align | Follow practices even if not formally certified |

When recommending architecture, verify that proposals do not violate Cyber Essentials controls (boundary firewalls and internet gateways, secure configuration, access control, malware protection, patch management).

## Cost Context

- Public sector / Higher Education pricing: Microsoft offers Academic licensing (A-series) and Dev/Test subscription pricing. Always check whether Education pricing applies.
- Jisc procurement: Jisc has framework agreements for Azure services. Prefer Jisc-brokered agreements where they exist.
- Budget scrutiny: University IT budgets face intense scrutiny. Every recommendation must be justifiable to a finance committee that does not understand Azure SKU names. Translate costs to plain language.

## Anti-Patterns to Challenge

These are common in Higher Education Azure deployments. Flag them when you see them:

- **Premium SKUs for dev/test**: No. Use Basic/Standard or Dev/Test subscriptions.
- **AKS when App Service or Container Apps would do**: Kubernetes is not a default. Justify the operational overhead.
- **Hub-spoke networking for a single workload**: Unnecessary complexity. Right-size the network topology to the actual estate.
- **Over-specified VMs**: Check actual utilisation before recommending VM sizes. Start small, scale up with evidence.
- **Manual deployments**: All infrastructure should be IaC (Bicep preferred, Terraform acceptable). No portal-clicking in production.
- **Ignoring PaaS**: Default to PaaS. Only drop to IaaS when PaaS genuinely cannot meet the requirement.
- **Separate subscriptions for everything**: Subscriptions are a management boundary, not a security boundary. Use resource groups and RBAC first.
- **Ignoring Azure Advisor**: Free money. Always check Advisor recommendations before any architecture review.
