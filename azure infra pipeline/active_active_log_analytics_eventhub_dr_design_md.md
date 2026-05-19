# Simplified Active-Active DR Design for Log Analytics Export

## 1. Target Architecture

```text
Log Analytics Workspace
    ├── Data Export Rule A
    │       ↓
    │   Event Hub A / East Asia
    │       ↓
    │   GCP Services
    │
    └── Data Export Rule B
            ↓
        Event Hub B / Southeast Asia
            ↓
        GCP Services
```

Architecture diagram:  
![Active-Active DR Architecture](sandbox:/mnt/data/wide_clean_infographic_diagram_on_white_background.png)

## 2. Solution Overview

This design implements a lightweight and operationally simple Active-Active disaster recovery strategy for Azure Monitor log streaming.

A single Log Analytics Workspace exports the same log datasets simultaneously to two independent Azure Event Hub deployments located in different Azure regions. Each export path is fully isolated and independently consumable by downstream GCP services.

The architecture intentionally avoids:

- Event Hub Geo-DR Alias
- Metadata failover mechanisms
- Azure Function relay layers
- Manual DR switching procedures

This minimizes operational complexity while ensuring continuous log delivery during regional failures.

## 3. Core Implementation

Two independent Event Hub namespaces are deployed in separate Azure regions:

- East Asia
- Southeast Asia

Within the same Log Analytics Workspace, two Data Export Rules are configured:

| Export Rule | Destination |
|---|---|
| Export Rule A | Event Hub A |
| Export Rule B | Event Hub B |

Both rules export the same tables, for example:

```text
AzureDiagnostics
AppTraces
AppRequests
AppExceptions
```

Each rule targets a dedicated Event Hub authorization rule with Send permissions.

This creates an Active-Active streaming topology:

```text
LAW
 ├── EH-A
 └── EH-B
```

## 4. DR Behavior

Under normal operation:

```text
Event Hub A receives logs
Event Hub B receives logs
```

If Region A becomes unavailable:

```text
Event Hub A stops receiving logs
Event Hub B continues receiving logs
```

If Region B becomes unavailable:

```text
Event Hub B stops receiving logs
Event Hub A continues receiving logs
```

Because both export paths are continuously active, no failover operation is required.

## 5. DR Validation Procedure

As the environment is internally networked, DR validation does not rely on disabling public network access.

Instead, DR simulation is performed by temporarily removing one export rule.

```bash
az monitor log-analytics workspace data-export delete \
  --resource-group <law-rg> \
  --workspace-name <law-name> \
  --name <export-rule-a>
```

Expected validation results:

```text
Event Hub A:
Incoming Messages stops increasing

Event Hub B:
Incoming Messages continues increasing

GCP Services:
Continue receiving logs from Event Hub B
```

This confirms that the secondary streaming path remains operational independently of the primary path.

## 6. Recovery Procedure

After validation is completed, the deleted Data Export Rule is restored through Terraform.

```bash
terraform apply
```

Terraform recreates the removed export rule and restores the dual-stream export topology.

Post-recovery validation:

```text
Event Hub A:
Incoming Messages resumes increasing

Event Hub B:
Continues receiving logs

GCP Services:
Both ingestion paths operational
```

If the Event Hub service itself experiences a temporary outage, no Log Analytics reconfiguration is required. Once the Event Hub namespace becomes available again, the existing export rule automatically resumes log delivery.

## 7. Architectural Rationale

This approach is preferred over Event Hub Metadata Geo-DR for log streaming workloads because Metadata Geo-DR only replicates namespace metadata and does not replicate event data.

The proposed dual-export architecture provides:

- True multi-region log availability
- Independent regional ingestion paths
- No dependency on alias failover
- Minimal operational overhead
- Straightforward DR testing and recovery
- Simplified Terraform-based restoration workflow

This design is particularly suitabl