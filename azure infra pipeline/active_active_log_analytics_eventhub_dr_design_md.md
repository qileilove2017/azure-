# Simplified Active-Active DR Design for Log Analytics Export

## 1. Target Architecture

```text
Log Analytics Workspace
    ├── Data Export Rule A
    │       ↓
    │   Event Hub A / East Asia
    │       ↓
    │   Azure Function A
    │       ↓
    │   GCP Services
    │
    └── Data Export Rule B
            ↓
        Event Hub B / Southeast Asia
            ↓
        Azure Function B
            ↓
        GCP Services
```

Architecture diagram:  
![Active-Active DR Architecture](sandbox:/mnt/data/wide_clean_infographic_diagram_on_white_background.png)

## 2. Solution Overview

This design implements a lightweight and operationally simple Active-Active disaster recovery strategy for Azure Monitor log streaming.

A single Log Analytics Workspace exports the same log datasets simultaneously to two independent Azure Event Hub deployments located in different Azure regions. Each Event Hub is consumed by a dedicated Azure Function in the same regional path. The Azure Functions then forward or process the events before integrating with downstream GCP services.

Each path is isolated end to end:

```text
Path A: Log Analytics → Export Rule A → Event Hub A → Azure Function A → GCP Services
Path B: Log Analytics → Export Rule B → Event Hub B → Azure Function B → GCP Services
```

The architecture intentionally avoids:

- Event Hub Geo-DR Alias
- Metadata failover mechanisms
- Cross-region Event Hub dependency
- Manual DR switching for log export

This minimizes operational complexity while ensuring continuous log delivery during regional or single-path failures.

## 3. Core Implementation

Two independent Event Hub namespaces are deployed in separate Azure regions:

- East Asia
- Southeast Asia

Two Azure Functions are deployed as regional consumers:

- Azure Function A consumes Event Hub A
- Azure Function B consumes Event Hub B

Within the same Log Analytics Workspace, two Data Export Rules are configured:

| Export Rule | Destination | Consumer | Downstream |
|---|---|---|---|
| Export Rule A | Event Hub A | Azure Function A | GCP Services |
| Export Rule B | Event Hub B | Azure Function B | GCP Services |

Both rules export the same log tables, for example:

```text
AzureDiagnostics
AppTraces
AppRequests
AppExceptions
```

Each rule targets a dedicated Event Hub authorization rule with Send permission. Each Azure Function uses the corresponding Event Hub trigger or Event Hub connection configuration to consume events from its regional Event Hub.

This creates an Active-Active streaming topology:

```text
LAW
 ├── EH-A → Function A → GCP
 └── EH-B → Function B → GCP
```

## 4. DR Behavior

Under normal operation:

```text
Event Hub A receives logs → Function A processes logs → GCP receives logs
Event Hub B receives logs → Function B processes logs → GCP receives logs
```

If Region A or Path A becomes unavailable:

```text
Event Hub A / Function A stops processing
Event Hub B / Function B continues processing
GCP Services continue receiving logs from Path B
```

If Region B or Path B becomes unavailable:

```text
Event Hub B / Function B stops processing
Event Hub A / Function A continues processing
GCP Services continue receiving logs from Path A
```

Because both export and processing paths are continuously active, no Event Hub failover operation is required.

## 5. DR Validation Procedure

As the environment is internally networked, DR validation does not rely on disabling public network access.

The validation approach is to temporarily disable one Data Export Rule through Terraform to simulate a single log export path failure.

For example, set Export Rule A `enabled` to `false`:

```hcl
resource "azurerm_log_analytics_data_export_rule" "export_eastasia" {
  name                  = "export-to-eh-eastasia"
  resource_group_name   = azurerm_resource_group.rg.name
  workspace_resource_id = azurerm_log_analytics_workspace.law.id

  table_names = [
    "AzureDiagnostics",
    "AppTraces",
    "AppRequests",
    "AppExceptions"
  ]

  destination_resource_id = azurerm_eventhub_namespace_authorization_rule.ea_send.id

  enabled = false
}
```

Apply the change:

```bash
terraform apply
```

Expected validation results:

```text
Event Hub A:
Incoming Messages stops increasing

Azure Function A:
New event processing stops after existing backlog is drained

Event Hub B:
Incoming Messages continues increasing

Azure Function B:
Continues processing new events

GCP Services:
Continue receiving logs from Path B
```

This confirms that the secondary streaming and processing path remains operational independently when the primary export path is stopped.

## 6. Recovery Procedure

After validation is completed, re-enable the disabled Data Export Rule through Terraform.

For example, set Export Rule A `enabled` back to `true`:

```hcl
enabled = true
```

Apply the change:

```bash
terraform apply
```

Terraform updates the Export Rule state and restores the dual-stream export topology.

Post-recovery validation:

```text
Event Hub A:
Incoming Messages resumes increasing

Azure Function A:
Resumes processing events from Event Hub A

Event Hub B:
Continues receiving logs

Azure Function B:
Continues processing events from Event Hub B

GCP Services:
Both ingestion paths operational
```

If Event Hub A or Azure Function A experiences a temporary outage, no Log Analytics Workspace reconfiguration is required. Once the affected Event Hub or Function becomes available again, the existing Export Rule and Function trigger resume the normal event flow for that path.

## 7. Azure Function Considerations

When both Azure Functions send data to the same GCP destination, the downstream integration should be designed to handle duplicate or overlapping records if both paths export the same log tables.

Recommended controls:

- Use idempotent writes on the GCP side where possible
- Include a stable event identifier or composite key in the forwarded payload
- Track processing failures and retry behavior separately for Function A and Function B
- Monitor Function execution count, failures, retry count, and Event Hub trigger lag
- Ensure both Functions use independent app settings, managed identities, secrets, and network paths where required

If duplicate delivery is not acceptable, the GCP ingestion layer should perform deduplication based on fields such as timestamp, resource ID, category, operation name, and log record identifier.

## 8. Architectural Rationale

This approach is preferred over Event Hub Metadata Geo-DR for log streaming workloads because Metadata Geo-DR only replicates namespace metadata and does not replicate event data.

The proposed dual-export and dual-function architecture provides:

- True multi-region log availability
- Independent regional ingestion and processing paths
- No dependency on alias failover
- Minimal operational overhead
- Straightforward DR testing and recovery
- Standardized Terraform-based enablement and recovery workflow

This design is suitable for centralized observability, audit logging, and cross-cloud log ingestion scenarios where continuous log delivery and simple operational recovery are prioritized over complex failover orchestration.

