# Rootly GraphQL Schema

## Overview

This document describes the conceptual GraphQL schema for the Rootly incident management platform. Rootly provides a REST API for automating incident response workflows, managing on-call schedules, configuring escalation policies, and integrating with tools such as Slack, PagerDuty, and OpsGenie.

The schema below models the core resources exposed by the Rootly REST API (https://rootly.com/api) as GraphQL types, enabling a unified graph view over incident lifecycle management.

## Schema Source

- **Provider**: Rootly
- **API Docs**: https://rootly.com/api
- **Schema File**: rootly-schema.graphql
- **Type**: Conceptual GraphQL schema derived from REST API surface

## Core Domains

### Incidents

The central resource in Rootly is the `Incident`. Each incident carries status, severity, affected environments, teams, services, and functionalities. Incidents accumulate a timeline of entries, custom fields, tags, labels, attachments, and links.

- `Incident` — the root incident record
- `IncidentStatus` — lifecycle state (e.g., started, mitigating, resolved)
- `IncidentSeverity` — severity level with color coding and priority ordering
- `IncidentEnvironment` — the environment (production, staging, etc.) affected
- `IncidentTeam` — the team assigned to the incident
- `IncidentService` — the service impacted by the incident
- `IncidentFunctionality` — the product functionality affected
- `IncidentTimestamps` — structured timestamps (started, detected, mitigating, resolved)
- `IncidentCustomField` — user-defined fields attached to incidents
- `IncidentTag` — free-form tagging for categorization
- `IncidentLabel` — structured label pairs (key/value)
- `IncidentSummary` — AI-generated or manual summary of the incident
- `IncidentAttachment` — files or screenshots attached to the incident
- `IncidentLink` — external URLs linked from the incident
- `IncidentType` — classification type for the incident

### Action Items

Action items track follow-up tasks arising from incidents, retrospectives, or playbooks.

- `ActionItem` — a task created during or after an incident
- `ActionItemStatus` — status of the action item (open, in_progress, done)
- `ActionItemAssignee` — the user or team responsible for the action item

### Timeline

The incident timeline captures the chronological sequence of events.

- `Timeline` — the full timeline for an incident
- `TimelineEntry` — a single timestamped event or note in the timeline
- `EntrySource` — the origin of the timeline entry (manual, automation, integration)

### Automations

Rootly automations fire actions in response to incident events.

- `Automation` — an automation rule configuration
- `AutomationTrigger` — the event that fires the automation
- `AutomationAction` — the action to perform when triggered
- `AutomationCondition` — conditions that must be met for the automation to fire

### On-Call

On-call management covers schedules, rotations, and escalation policies.

- `OnCall` — the current on-call state
- `OnCallSchedule` — a named schedule defining coverage windows
- `Rotation` — a rotation layer within a schedule
- `RotationParticipant` — a user or team in a rotation
- `EscalationPolicy` — an ordered set of escalation rules
- `EscalationRule` — a single step in an escalation policy

### Users and Teams

- `User` — a Rootly platform user
- `UserDetails` — extended profile information for a user
- `UserRole` — the RBAC role assigned to the user
- `Team` — a group of users
- `TeamDetails` — extended team metadata

### Services and Environments

- `Service` — a software service tracked in Rootly
- `ServiceSeverity` — severity mapping for a specific service
- `Environment` — a deployment environment (production, staging, dev)

### Workflows and Runbooks

- `Workflow` — a structured incident response workflow
- `PlaybookStep` — an individual step within a workflow
- `Runbook` — a reference runbook document linked to workflows

### Retrospectives and Postmortems

- `Retrospective` — a post-incident retrospective record
- `PostmortemSection` — a named section within the retrospective document

### Dashboards and Widgets

- `Dashboard` — a named metrics or status dashboard
- `Widget` — a visual component within a dashboard
- `WidgetConfig` — configuration parameters for a widget

### Webhooks and Integrations

- `Webhook` — a webhook subscription configuration
- `WebhookEndpoint` — a target URL for webhook delivery
- `Integration` — a third-party integration configured in Rootly
- `Slack` — Slack workspace integration settings
- `PagerDuty` — PagerDuty integration settings
- `OpsGenie` — OpsGenie integration settings

### Authentication

- `APIKey` — an API key credential for programmatic access
- `Token` — a short-lived bearer token

## Query Examples

```graphql
# Fetch a single incident with its timeline
query GetIncident($id: ID!) {
  incident(id: $id) {
    id
    title
    status {
      name
      color
    }
    severity {
      name
      priority
    }
    teams {
      id
      name
    }
    services {
      id
      name
    }
    timeline {
      entries {
        id
        occurredAt
        description
        source {
          type
          name
        }
      }
    }
    actionItems {
      id
      title
      status {
        name
      }
      assignee {
        user {
          id
          name
          email
        }
      }
    }
  }
}

# List open incidents by severity
query ListOpenIncidents($severityId: ID) {
  incidents(filters: { statusCategory: "open", severityId: $severityId }) {
    nodes {
      id
      title
      createdAt
      severity {
        name
        priority
      }
      status {
        name
      }
    }
    pageInfo {
      hasNextPage
      endCursor
    }
  }
}

# Get the current on-call user for a schedule
query OnCallNow($scheduleId: ID!) {
  onCallSchedule(id: $scheduleId) {
    id
    name
    currentOnCall {
      user {
        id
        name
        email
      }
    }
  }
}
```

## Mutation Examples

```graphql
# Create a new incident
mutation CreateIncident($input: CreateIncidentInput!) {
  createIncident(input: $input) {
    incident {
      id
      title
      status {
        name
      }
      severity {
        name
      }
    }
  }
}

# Update incident status
mutation UpdateIncidentStatus($id: ID!, $statusId: ID!) {
  updateIncident(id: $id, input: { statusId: $statusId }) {
    incident {
      id
      status {
        name
      }
    }
  }
}

# Add a timeline entry
mutation AddTimelineEntry($incidentId: ID!, $description: String!, $occurredAt: DateTime!) {
  createTimelineEntry(incidentId: $incidentId, input: { description: $description, occurredAt: $occurredAt }) {
    entry {
      id
      description
      occurredAt
    }
  }
}
```
