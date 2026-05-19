# Trend Micro Vision One Azure Sentinel Data Connector

[![ARM Template Validation](https://github.com/yourusername/TrendAI_Sentinel_public_data_Connector/workflows/ARM%20Template%20Validation/badge.svg)](https://github.com/yourusername/TrendAI_Sentinel_public_data_Connector/actions)

A production-grade Azure Resource Manager (ARM) template-based data connector that ingests Trend Micro Vision One XDR data (Workbench alerts and Observed Attack Techniques) into Microsoft Sentinel.

## Features

- **Automated Data Ingestion**: Poll Trend Micro Vision One API every 5 minutes
- **Multiple Data Sources**: 
  - Workbench Alerts (security incidents and investigations)
  - Observed Attack Techniques (OAT) - MITRE ATT&CK mapped detections
- **Enriched IOC Extraction**: Automatic extraction and normalization of:
  - File hashes (SHA1, SHA256, MD5)
  - IP addresses and domains
  - URLs and file paths
  - Registry keys and values
  - Process command lines
  - User accounts and hostnames
  - Email addresses
- **Production-Ready**: 
  - Secure API key management via Azure Key Vault
  - Managed Identity authentication
  - Data Collection Rules (DCR) for structured ingestion
  - Regional API endpoint support (US, UK, SG, CA, JP)
- **ASIM Compatible**: Normalized field names for Azure Sentinel Advanced Security Information Model

## Architecture

```
Trend Micro Vision One API
          ↓
  Azure Logic App (5min polling)
          ↓
  Data Collection Endpoint (DCE)
          ↓
  Data Collection Rule (DCR)
          ↓
  Log Analytics Custom Tables
    - TrendMicro_XDR_WORKBENCH_CL
    - TrendMicro_XDR_OAT_CL
          ↓
  Microsoft Sentinel Analytics Rules
```

## Prerequisites

- **Azure Subscription** with Microsoft Sentinel enabled
- **Log Analytics Workspace** configured for Sentinel
- **Trend Micro Vision One Account** with:
  - API access enabled
  - API key with `SIEM` role permissions ([Generate API Key](https://docs.trendmicro.com/en-us/enterprise/trend-vision-one/administrative-setti/administration/api-keys.aspx))
- **Azure Permissions**:
  - `Contributor` role on the resource group
  - `Microsoft.OperationalInsights/workspaces/write`
  - `Microsoft.Logic/workflows/write`
  - `Microsoft.Insights/dataCollectionRules/write`

## Quick Start

### 1. Deploy Workbench Connector

```bash
az deployment group create \
  --resource-group <your-resource-group> \
  --template-file templates/arm-template-workbench-compatible-v2.json \
  --parameters \
    workspace="<your-workspace-name>" \
    workspace-location="<region>" \
    trendaiRegion="US" \
    apikey="<your-trend-vision-one-api-key>"
```

**Example:**
```bash
az deployment group create \
  --resource-group sentinel-rg \
  --template-file templates/arm-template-workbench-compatible-v2.json \
  --parameters \
    workspace="sentinel-law" \
    workspace-location="eastus" \
    trendaiRegion="US" \
    apikey="eyJhbGciOiJSUzI1NiIsInR5..."
```

### 2. Deploy OAT Connector (Optional)

```bash
az deployment group create \
  --resource-group <your-resource-group> \
  --template-file templates/arm-template-oat-complete.json \
  --parameters \
    workspace="<your-workspace-name>" \
    location="<region>" \
    subscription="<subscription-id>" \
    resourceGroupName="<resource-group>"
```

### 3. Verify Deployment

After 5-10 minutes, check for data in Log Analytics:

```kql
TrendMicro_XDR_WORKBENCH_CL
| where TimeGenerated > ago(1h)
| take 10
```

## Available Templates

| Template | Purpose | Deployment Time | Polling Interval |
|----------|---------|-----------------|------------------|
| `arm-template-workbench-compatible-v2.json` | Workbench alerts with IOC enrichment | ~3-5 min | 5 minutes |
| `arm-template-workbench-PRODUCTION-SAFE.json` | Production workbench connector | ~3-5 min | 5 minutes |
| `arm-template-oat-complete.json` | Observed Attack Techniques (MITRE ATT&CK) | ~3-5 min | 5 minutes |

## Data Enrichment

The connector automatically extracts and normalizes Indicators of Compromise (IOCs) from raw Trend Micro data:

### Extracted Fields

- **File Indicators**: `FileName_s`, `FileHashValue_s`, `FileDirectory_s`
- **Network Indicators**: `IPAddress`, `DomainName_s`, `URL_s`
- **Host Indicators**: `HostHostName_s`, `ProcessCommandLine_s`
- **Registry Indicators**: `RegistryKey_s`, `RegistryValue_s`, `RegistryValueName_s`
- **User Indicators**: `UserAccountName_s`, `UserAccountNTDomain_s`, `MailboxPrimaryAddress_s`

### Using Enriched Data

**Option 1: Saved Query Function (Recommended)**
```kql
// Save examples/WORKING-QUERY.kql as a function named "WorkbenchEnriched"
WorkbenchEnriched
| where severity_s == "critical"
| where isnotempty(FileHashValue_s)
```

**Option 2: Direct Query**
```kql
TrendMicro_XDR_WORKBENCH_CL
| extend indicators_array = parse_json(indicators_s)
| mv-expand indicator = indicators_array
| where indicator.type == "file_sha256"
```

See [docs/DEPLOYMENT-GUIDE-ENRICHED-TABLE.md](docs/DEPLOYMENT-GUIDE-ENRICHED-TABLE.md) for detailed enrichment options.

## Configuration

### Regional API Endpoints

The connector supports multiple Trend Micro Vision One regions:

| Region | Parameter Value | API Endpoint |
|--------|----------------|--------------|
| United States | `US` | api.xdr.trendmicro.com |
| United Kingdom | `UK` | api.uk.xdr.trendmicro.com |
| Singapore | `SG` | api.sg.xdr.trendmicro.com |
| Canada | `CA` | api.ca.xdr.trendmicro.com |
| Japan | `JP` | api.jp.xdr.trendmicro.com |

### Polling Frequency

Default: **5 minutes**

To modify, edit the Logic App recurrence trigger after deployment:
1. Azure Portal → Logic Apps → `TrendMicro-XDR-Workbench-Connector`
2. Logic App Designer → Recurrence trigger
3. Change `Frequency` and `Interval`

### Data Retention

Data is retained according to your Log Analytics workspace retention policy (default: 30-90 days).

To modify:
```bash
az monitor log-analytics workspace table update \
  --resource-group <rg> \
  --workspace-name <workspace> \
  --name TrendMicro_XDR_WORKBENCH_CL \
  --retention-time 90
```

## Monitoring and Troubleshooting

### Check Logic App Execution

```bash
az logic workflow show \
  --resource-group <rg> \
  --name TrendMicro-XDR-Workbench-Connector
```

### View Run History

Azure Portal → Logic Apps → `TrendMicro-XDR-Workbench-Connector` → Overview → Runs history

### Common Issues

| Issue | Solution |
|-------|----------|
| No data after 10 minutes | Check API key validity, verify Trend Vision One region |
| `401 Unauthorized` | Regenerate API key with SIEM role |
| `Schema validation failed` | Ensure using latest ARM template version |
| Missing IOC fields | Deploy enriched table or use saved query function |

### Enable Diagnostic Logging

```bash
az monitor diagnostic-settings create \
  --resource <logic-app-resource-id> \
  --name "LogicAppDiagnostics" \
  --workspace <workspace-id> \
  --logs '[{"category":"WorkflowRuntime","enabled":true}]'
```

## Integration with Sentinel Analytics

### Sample Analytics Rule

```kql
// High-severity Workbench alerts with file IOCs
TrendMicro_XDR_WORKBENCH_CL
| where TimeGenerated > ago(5m)
| where severity_s in ("high", "critical")
| extend indicators_array = parse_json(indicators_s)
| mv-expand indicator = indicators_array
| where indicator.type in ("file_sha256", "file_sha1", "md5")
| project 
    TimeGenerated,
    workbenchName_s,
    severity_s,
    FileHash = tostring(indicator.value),
    HostHostName_s,
    workbenchLink_s
```

See [docs/COMPLETE-FIELD-MAPPING.md](docs/COMPLETE-FIELD-MAPPING.md) for all available fields.

## Deployment Scripts

### PowerShell

```powershell
# scripts/deploy.ps1
.\scripts\deploy.ps1 `
  -ResourceGroup "sentinel-rg" `
  -WorkspaceName "sentinel-law" `
  -Location "eastus" `
  -TrendRegion "US" `
  -ApiKey "your-api-key"
```

### Bash

```bash
# scripts/deploy.sh
./scripts/deploy.sh \
  --resource-group sentinel-rg \
  --workspace sentinel-law \
  --location eastus \
  --trend-region US \
  --api-key your-api-key
```

## Roadmap

- [ ] Azure DevOps pipeline for automated deployment
- [ ] Terraform module version
- [ ] Bicep template conversion
- [ ] Workbook templates for Trend Micro data visualization
- [ ] Pre-built Analytics Rules package
- [ ] Support for additional Trend Micro Vision One APIs (Endpoint Activity, Email Activity)

## Contributing

Contributions are welcome! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Support

- **Issues**: [GitHub Issues](https://github.com/yourusername/TrendAI_Sentinel_public_data_Connector/issues)
- **Trend Micro Vision One Docs**: [Official Documentation](https://docs.trendmicro.com/en-us/enterprise/trend-vision-one.aspx)
- **Microsoft Sentinel Docs**: [Azure Sentinel Documentation](https://learn.microsoft.com/en-us/azure/sentinel/)

## Acknowledgments

- Trend Micro Vision One API Team
- Microsoft Sentinel Community
- Azure ARM Template Contributors

## Related Resources

- [Trend Micro Vision One API Reference](https://automation.trendmicro.com/xdr/api-v3)
- [Azure Sentinel Data Connectors](https://learn.microsoft.com/en-us/azure/sentinel/connect-data-sources)
- [MITRE ATT&CK Framework](https://attack.mitre.org/)
- [Azure Monitor Data Collection Rules](https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/data-collection-rule-overview)

---

**Maintained by**: [Your Name/Organization]  
**Last Updated**: May 2026  
**Version**: 2.0.1
