---
title: Azure Monitoring
date: 2024-07-10
tags:
  - azure
  - monitoring

---

## Monitor Tools

- Azure Activity Log is equivalent to AWS CloudTrail.
- Azure Advisor: Provides recommendations for cost savings and best practices.
- Traffic Analytics: showing statistics, summary, numbers of the network

### Application Insight

It is a powerful application performance management tool.

It offer two main methods of implementation:

1. Agent-based (Codeless):
- No code changes required
- Supports multiple languages (e.g., .NET, Java, Node.js)
- Limited customization options

2. SDK-based:
- Requires code changes
- Supports multiple languages (Java, Python, JavaScript, etc.)
- Offers more control and customization
- Implemented as middleware in the application

**Features:**

- Logging:
	- Sniff any logger.warning(f"foo bar") and send to cloud
	- Centralized log collection and analysis
- Metrics:
	- Similar to Prometheus client library
	- Custom metrics
- Tracing:
	- Monitors requests from start to end
	- Measures duration and identifies bottlenecks
- Dependency tracking
- Exception monitoring
- Real-time dashboards
- Integrate with Microsoft Sentinel (a platform to consolidate all security related data)

### VM Inslights

- Monitors CPU, memory, disk, and network usage

### Container Inslights

- Monitor containerized applications
- Works with AKS and ACI
- Provides container-specific metrics and logs

### Metrics / Insight

Azure Monitor is similar to AWS CloudWatch.

To setup the monitoring, we need to

- Install Azure Monitor Agent for collect metrics and logs.
- Setup Data Collection Rule (DCR) to send metrics and logs from the agent to Log Analytics Workspaces.

Components of a DCR:
- **Resources:** Specify VMs from which to collect data.
- **Data Sources:** Include Performance Data, Syslog, and Event Logs gathered by the agent.
- **Destination:** Log Analytics Workspace.

Notice that:
- One DCR can collect data from multiple VMs
- One VM can contribute data to multiple DCRs

This setup allows centralized management and analysis of metrics and logs from Azure VMs using Azure Monitor and Log Analytics Workspaces.

**Legacy Way will use Log Analytics:**
1. Create a Log Analytics workspace.
2. Install the Log Analytics agent.
3. Under the Log Analytics workspace, configure performance counters.

### Monitoring Agents

**Monitor Agent:**
- Capable of sending logs and metrics to multiple Log Analytics Workspaces.
- Recommended as a replacement for other agents.
- Does not support custom logs or IIS logs.

**Windows/Linux Diagnostics Agent:**
- Found in the `Diagnostics` tab.
- Can send logs and metrics to multiple destinations such as Log Analytics Workspaces, Storage Accounts, and Event Hubs.
- Provides more robust functionality compared to Monitor Agent.

Both agents support metrics and logs and are designed for use with VMs.

**Log Analytics Agent:**
- Supports sending logs only.
- Configuration is decentralized (needs to be configured on the VM itself).
- Can be used outside of Azure environments.

## Alerts

### ITSM Integration

One of the alert destinations is send the alert to ITSM via IT Service Management Connector.

But this will be retired in 2025. Currently, it can only be created via API.

### Alert Rate Limits

- **SMS/Voice Call:** Every 5 minutes.
- **Email:** Less than 100 per hour.

Emails will only be sent to individual users, not to groups or service principals.

### Administrative Operations Alerts

You can configure alerts for administrative operations logged in Azure Activity, which essentially covers any API call. Examples include:
- Writing a tag to resource and VM
- Creating a disk
- Attaching a disk to VM

In the activity log, these operations are identified by `"eventSource": "Administrative"`.

### Suppressed Notifications

When an alert processing rule suppresses notifications, the alert will still be listed in the portal, but no notification will be sent.

### Alert Components

- **Resource (Scope):** The subscription or resource the alert pertains to.
- **Condition (Signal):** The source of data and the threshold that triggers the alert.
- **Signal Source:** Typically a Log Analytics Workspace.
- **Action Group:** Defines the notifications and actions to be taken when the alert is triggered.

### User Response States

- If the user response is "New", it can be changed to "Acknowledged" or "Closed."
- If the user response is "Closed", it cannot be changed.
