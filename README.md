# Amazon IoT Events (amazon-iot-events)
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
