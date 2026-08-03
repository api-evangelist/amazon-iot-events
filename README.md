# Amazon IoT Events (amazon-iot-events)

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

AWS IoT Events is a managed service that makes it easy to detect and respond to events from IoT sensors and applications. You can use it to build complex event detection logic, create state machines for IoT workflows, and trigger alerts or actions when specific conditions are met.

**URL:** [Visit APIs.json URL](https://raw.githubusercontent.com/api-evangelist/amazon-iot-events/refs/heads/main/apis.yml)

**Run:** [Capabilities Using Naftiko](https://github.com/naftiko/fleet?utm_source=api-evangelist&utm_medium=readme&utm_campaign=company-api-evangelist&utm_content=repo)

## Tags:

 - AWS, Event Detection, IoT, State Machine, Automation

## Timestamps

- **Created:** 2026-03-16
- **Modified:** 2026-04-19

## APIs

### AWS IoT Events API
The AWS IoT Events API provides access to detector models, inputs, alarms, and event detection configurations for building IoT event-driven workflows.

**Human URL:** [https://aws.amazon.com/iot-events/](https://aws.amazon.com/iot-events/)

#### Tags:

 - Event Detection, IoT, State Machine

#### Properties

- [Documentation](https://docs.aws.amazon.com/iotevents/latest/apireference/)
- [OpenAPI](openapi/amazon-iot-events-openapi-original.yml)
- [GettingStarted](https://docs.aws.amazon.com/iotevents/latest/developerguide/getting-started-iotevents.html)
- [Pricing](https://aws.amazon.com/iot-events/pricing/)
- [FAQ](https://aws.amazon.com/iot-events/faqs/)

## Common Properties

- [Portal](https://aws.amazon.com/iot-events/)
- [Website](https://aws.amazon.com/iot-events/)
- [Documentation](https://docs.aws.amazon.com/iotevents/)
- [TermsOfService](https://aws.amazon.com/service-terms/)
- [PrivacyPolicy](https://aws.amazon.com/privacy/)
- [Support](https://aws.amazon.com/premiumsupport/)
- [Blog](https://aws.amazon.com/blogs/iot/tag/aws-iot-events/)
- [GitHubOrganization](https://github.com/aws)
- [Console](https://console.aws.amazon.com/iotevents/)
- [SignUp](https://portal.aws.amazon.com/billing/signup)
- [Login](https://signin.aws.amazon.com/)
- [StatusPage](https://health.aws.amazon.com/health/status)
- [Contact](https://aws.amazon.com/contact-us/)

## Features

| Name | Description |
|------|-------------|
| Detector Models | Create state machines to detect complex event patterns across IoT data streams. |
| Alarm Management | Built-in alarm management for monitoring IoT sensor thresholds. |
| Event Inputs | Define structured event inputs and route IoT data to detector models. |
| Multi-Trigger Actions | Trigger actions to SNS, SQS, Lambda, and other services when events are detected. |

## Use Cases

| Name | Description |
|------|-------------|
| Industrial Alarm Management | Detect equipment failures and trigger maintenance workflows automatically. |
| Complex Event Processing | Detect patterns across multiple sensor streams over time. |
| Predictive Maintenance | Alert operations teams when device metrics indicate impending failure. |

## Integrations

| Name | Description |
|------|-------------|
| AWS IoT Core | Receives message data from IoT Core for event detection. |
| Amazon SNS | Sends alerts and notifications when events are detected. |
| AWS Lambda | Triggers Lambda functions to execute response workflows. |

## Artifacts

Machine-readable API specifications organized by format.

### OpenAPI

- [AWS IoT Events API](openapi/amazon-iot-events-openapi-original.yml)

### JSON Schema

143 schema files covering key resources and operations.

### JSON Structure

143 JSON Structure files converted from JSON Schema.

### JSON-LD

- [Amazon IoT Events Context](json-ld/amazon-iot-events-context.jsonld)

### Examples

143 example JSON files generated from schemas.

## Capabilities

Naftiko capabilities organized as shared per-API definitions composed into customer-facing workflows.

### Shared Per-API Definitions

- [AWS IoT Events API](capabilities/shared/iot-events.yaml) — operations for amazon iot events management

### Workflow Capabilities

| Workflow | APIs Combined | Tools | Persona |
|----------|--------------|-------|---------|
| [Iot Event Management](capabilities/iot-event-management.yaml) | Amazon IoT Events | 8 | IoT Developer, Solutions Architect |

## Vocabulary

- [Amazon IoT Events Vocabulary](vocabulary/amazon-iot-events-vocabulary.yaml) — Unified taxonomy mapping resources, actions, workflows, and personas

## Rules

- [Amazon IoT Events Spectral Rules](rules/amazon-iot-events-spectral-rules.yml) — 14 rules across 6 categories enforcing Amazon IoT Events API conventions

## Maintainers

**FN:** Kin Lane

**Email:** kin@apievangelist.com
