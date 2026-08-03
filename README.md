# Amazon IoT Device Defender (amazon-iot-device-defender)

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

AWS IoT Device Defender is a security service that lets you continuously audit your IoT configurations to detect deviations from security best practices. It also lets you detect abnormal device behavior through ML-based anomaly detection and take actions to mitigate security risks.

**URL:** [Visit APIs.json URL](https://raw.githubusercontent.com/api-evangelist/amazon-iot-device-defender/refs/heads/main/apis.yml)

**Run:** [Capabilities Using Naftiko](https://github.com/naftiko/fleet?utm_source=api-evangelist&utm_medium=readme&utm_campaign=company-api-evangelist&utm_content=repo)

## Tags:

 - AWS, Compliance, IoT, Security, Vulnerability Management

## Timestamps

- **Created:** 2026-03-16
- **Modified:** 2026-04-19

## APIs

### AWS IoT Device Defender API
The AWS IoT Device Defender API provides programmatic access to security profiles, audit configurations, anomaly detection, and violation management for IoT fleet security.

**Human URL:** [https://aws.amazon.com/iot-device-defender/](https://aws.amazon.com/iot-device-defender/)

#### Tags:

 - Compliance, IoT, Security

#### Properties

- [Documentation](https://docs.aws.amazon.com/iot/latest/developerguide/device-defender.html)
- [OpenAPI](openapi/amazon-iot-device-defender-openapi-original.yml)
- [GettingStarted](https://docs.aws.amazon.com/iot/latest/developerguide/device-defender-getting-started.html)
- [Pricing](https://aws.amazon.com/iot-device-defender/pricing/)
- [FAQ](https://aws.amazon.com/iot-device-defender/faqs/)

## Common Properties

- [Portal](https://aws.amazon.com/iot-device-defender/)
- [Website](https://aws.amazon.com/iot-device-defender/)
- [Documentation](https://docs.aws.amazon.com/iot/latest/developerguide/device-defender.html)
- [TermsOfService](https://aws.amazon.com/service-terms/)
- [PrivacyPolicy](https://aws.amazon.com/privacy/)
- [Support](https://aws.amazon.com/premiumsupport/)
- [Blog](https://aws.amazon.com/blogs/iot/tag/aws-iot-device-defender/)
- [GitHubOrganization](https://github.com/aws)
- [Console](https://console.aws.amazon.com/iot/home#/devicedefender)
- [SignUp](https://portal.aws.amazon.com/billing/signup)
- [Login](https://signin.aws.amazon.com/)
- [StatusPage](https://health.aws.amazon.com/health/status)
- [Contact](https://aws.amazon.com/contact-us/)

## Features

| Name | Description |
|------|-------------|
| Configuration Audit | Continuously audit IoT configurations against security best practices. |
| ML Anomaly Detection | Detect abnormal device behavior using machine learning models. |
| Security Profiles | Define expected behaviors for device metrics and receive alerts on violations. |
| Automated Mitigation | Automatically take actions to mitigate security violations. |

## Use Cases

| Name | Description |
|------|-------------|
| IoT Compliance | Ensure IoT deployments meet security compliance requirements. |
| Threat Detection | Detect compromised devices exhibiting abnormal communication patterns. |
| Security Auditing | Audit IoT policies and certificates against security best practices. |

## Integrations

| Name | Description |
|------|-------------|
| AWS IoT Core | Monitors all IoT Core device connections and policies. |
| Amazon CloudWatch | Sends security metrics and alerts to CloudWatch. |
| AWS Security Hub | Publishes IoT security findings to Security Hub. |

## Artifacts

Machine-readable API specifications organized by format.

### OpenAPI

- [AWS IoT Device Defender API](openapi/amazon-iot-device-defender-openapi-original.yml)

### JSON Schema

200 schema files covering key resources and operations.

### JSON Structure

200 JSON Structure files converted from JSON Schema.

### JSON-LD

- [Amazon IoT Device Defender Context](json-ld/amazon-iot-device-defender-context.jsonld)

### Examples

200 example JSON files generated from schemas.

## Capabilities

Naftiko capabilities organized as shared per-API definitions composed into customer-facing workflows.

### Shared Per-API Definitions

- [AWS IoT Device Defender API](capabilities/shared/iot-device-defender.yaml) — operations for amazon iot device defender management

### Workflow Capabilities

| Workflow | APIs Combined | Tools | Persona |
|----------|--------------|-------|---------|
| [Iot Security Monitoring](capabilities/iot-security-monitoring.yaml) | Amazon IoT Device Defender | 8 | Security Engineer, IoT Developer |

## Vocabulary

- [Amazon IoT Device Defender Vocabulary](vocabulary/amazon-iot-device-defender-vocabulary.yaml) — Unified taxonomy mapping resources, actions, workflows, and personas

## Rules

- [Amazon IoT Device Defender Spectral Rules](rules/amazon-iot-device-defender-spectral-rules.yml) — 14 rules across 6 categories enforcing Amazon IoT Device Defender API conventions

## Maintainers

**FN:** Kin Lane

**Email:** kin@apievangelist.com
