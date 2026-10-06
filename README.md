# TrendAI Vision One - Microsoft Sentinel Data Connectors

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

**TrendAI Vision One (Workbench Alerts + OAT Detections)** &nbsp; [![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Ftrendmicro%2Ftrendai-sentinel-ccf-data-connector%2Fmain%2FSolutions%2FTrendAI%2520Vision%2520One%2528CCF%2529%2FPackage%2FmainTemplate.json/createUIDefinitionUri/https%3A%2F%2Fraw.githubusercontent.com%2Ftrendmicro%2Ftrendai-sentinel-ccf-data-connector%2Fmain%2FSolutions%2FTrendAI%2520Vision%2520One%2528CCF%2529%2FPackage%2FcreateUiDefinition.json) [![Deploy to Azure US Gov](https://aka.ms/deploytoazuregovbutton)](https://portal.azure.us/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Ftrendmicro%2Ftrendai-sentinel-ccf-data-connector%2Fmain%2FSolutions%2FTrendAI%2520Vision%2520One%2528CCF%2529%2FPackage%2FmainTemplate.json/createUIDefinitionUri/https%3A%2F%2Fraw.githubusercontent.com%2Ftrendmicro%2Ftrendai-sentinel-ccf-data-connector%2Fmain%2FSolutions%2FTrendAI%2520Vision%2520One%2528CCF%2529%2FPackage%2FcreateUiDefinition.json)

> **Note:** this repository is not yet public. The link above will start working once the repo's visibility is flipped to public (see `.github/GO_LIVE_CHECKLIST.md`). Until then, deploy via the pre-built zip at [`Solutions/TrendAI Vision One(CCF)/Package/`](<Solutions/TrendAI Vision One(CCF)/Package/>) (Azure Portal → "Deploy a custom template" → "Build your own template" → upload) or install from **Microsoft Sentinel Content Hub** once the solution is published there.

Production-ready data connector solution for ingesting **TrendAI Vision One** security data into **Microsoft Sentinel** using Azure's Codeless Connector Platform (CCP). One deployment installs both connectors below, plus the shared parsers, analytic rule, and workbook.

## 🚀 Quick Deploy

Click the Deploy button above to install the full solution in one pass:

| Connector | Description | Data Volume |
|-----------|-------------|-------------|
| **Workbench Alerts** | Security incidents, investigations, and alerts with IOC extraction | Medium |
| **OAT (Observed Attack Techniques)** | MITRE ATT&CK mapped detections with full process trees | High |

## 📋 Prerequisites

Before deploying, ensure you have:

- ✅ **Azure Subscription** with Owner or Contributor role
- ✅ **Log Analytics Workspace** with Microsoft Sentinel enabled (or workspace will be enabled during deployment)
- ✅ **TrendAI Vision One API Token** with view permissions for Workbench and/or Observed Attack Techniques

### Getting Your API Token

1. Log in to **TrendAI Vision One Console**
2. Navigate to **Administration → API Keys**
3. Click **Add API Key**
4. Select a role with permissions: **Workbench (View)** and/or **Observed Attack Techniques (View)**
5. Copy the token immediately — it will not be shown again

## 🎯 Deployment Process

### Step 1: Click Deploy Button

Click one of the blue/green Deploy buttons above

### Step 2: Fill Azure Portal Form

The custom deployment UI will appear with:
- **Subscription** - Select your Azure subscription
- **Resource Group** - Select or create new
- **Workspace** - Dropdown of your Sentinel workspaces
- **Region** - Your TrendAI Vision One region (US, EU, SG, JP, AU, IN, MEA, UK, CA, ZA — see [Supported Regions](#-supported-regions))

### Step 3: Deploy Resources

Azure will automatically deploy:
1. Microsoft Sentinel solution (if not enabled)
2. Custom log tables (`TrendAI_XDR_WORKBENCH_V2_CL` and `TrendAI_XDR_OAT_V2_CL`)
3. Data Collection Endpoint (DCE)
4. Data Collection Rules (DCR) with data transformation
5. Connector definitions in Sentinel portal (Workbench Alerts, OAT Detections)
6. Parser functions (`TrendAIWorkbench_Complete`, `TrendAIOAT_Complete`)
7. Analytic rule template (disabled by default, Workbench only)
8. Workbook dashboard for monitoring (Workbench only)

⏱️ **Deployment time**: 3-5 minutes

### Step 4: Connect the Data Source

After deployment completes:

1. Navigate to **Microsoft Sentinel → Data connectors**
2. Search for **"TrendAI Vision One - Workbench Alerts"** or **"TrendAI Vision One - OAT Detections"**
3. Click **Open connector page**
4. Enter your API token and API Domain (e.g. `api.xdr.trendmicro.com`)
5. Click **Connect**

🎉 **Data will start flowing in 5-10 minutes!**

### Step 5: Enable the Analytic Rule (Optional)

The solution includes one **Analytic Rule**, for Workbench Alerts, that automatically creates incidents:

1. Navigate to **Microsoft Sentinel → Analytics**
2. Search for **"TrendAI Vision One"**
3. Find the rule: **"TrendAI Vision One - Create Incident for Workbench Alerts"**
4. Click the rule → **Edit** → **Enable** → **Save**

**What it does:**
- Creates incidents from Workbench alerts with mapped entities
- Includes MITRE ATT&CK mappings
- Adds custom details for investigations

### Step 6: View the Dashboard (Optional)

The solution includes one **Workbook** dashboard, for Workbench Alerts:

1. Navigate to **Microsoft Sentinel → Workbooks**
2. Click **My workbooks** tab
3. Find: **"TrendAIVisionOneWorkbenchOverview"**
4. Click **View saved workbook**

**Visualizations include:**
- Alert trends over time
- Severity distribution
- Top affected hosts/endpoints
- Investigation status breakdown

## 📊 Verify Data Ingestion

### Workbench Alerts

```kql
TrendAI_XDR_WORKBENCH_V2_CL
| where TimeGenerated > ago(1h)
| project TimeGenerated, workbenchId_s, severity_s, workbenchName_s
| take 10
```

### OAT Detections

```kql
TrendAI_XDR_OAT_V2_CL
| where TimeGenerated > ago(1h)
| project TimeGenerated, detail_endpointHostName_s, detail_filterRiskLevel_s, detail_ruleName_s
| take 10
```

## 🏗️ Architecture

This solution follows Microsoft's standard **Sentinel solution package** pattern: a single `mainTemplate.json` embeds every resource as ARM content templates, deployed in one pass.

```
mainTemplate.json
  ├─> data connector definitions (Workbench Alerts + OAT Detections)
  ├─> custom tables, DCE, DCRs (ingestion-time transforms)
  ├─> parser functions (TrendAIWorkbench_Complete, TrendAIOAT_Complete)
  ├─> analytic rule (Workbench incident creation)
  └─> workbook (Workbench Overview dashboard)
```

**Benefits:**
- ✅ Single-click deployment via Azure Portal
- ✅ Matches the Microsoft Sentinel Content Hub packaging format
- ✅ Easy to troubleshoot and maintain

## 📖 Data Schemas

### Workbench Alerts (`TrendAI_XDR_WORKBENCH_V2_CL`, 56 columns)

| Category | Fields |
|----------|--------|
| **Core** | workbenchId_s, severity_s, investigationStatus_s, alertProvider_s |
| **IOCs** | FileName_s, FileHashValue_s, IPAddress, DomainName_s, URL_s |
| **Entities** | HostHostName_s, UserAccountName_s, MailboxPrimaryAddress_s |
| **Dynamic** | indicators, entities (for advanced parsing via `TrendAIWorkbench_Complete()`) |

### OAT Detections (`TrendAI_XDR_OAT_V2_CL`, 156 columns)

| Category | Fields |
|----------|--------|
| **Core** | detail_filterRiskLevel_s, detail_ruleName_s, detectionTime_t |
| **Endpoint** | detail_endpointHostName_s |
| **Process** | detail_processCmd_s, detail_processFileHashSha256_s |
| **Parent** | detail_parentCmd_s, detail_parentFileHashSha256_s, detail_parentName_s |
| **Network** | detail_src_s, detail_dst_s, detail_dpt_d, detail_spt_d |
| **File** | detail_fileName_s, detail_fileHash_s, detail_filePathName_s |

## 🔍 Sample Queries

### Workbench: High Severity Alerts with File IOCs

```kql
TrendAIWorkbench_Complete()
| where severity_s in ("high", "critical")
| where isnotempty(FileHashValue_s)
| project TimeGenerated, workbenchName_s, FileName_s, FileHashValue_s, HostHostName_s
```

### OAT: High-Risk Detections with Process Details

```kql
TrendAI_XDR_OAT_V2_CL
| where detail_filterRiskLevel_s == "high"
| project TimeGenerated,
    Endpoint = detail_endpointHostName_s,
    Rule = detail_ruleName_s,
    CommandLine = detail_processCmd_s,
    SHA256 = detail_processFileHashSha256_s
```

### OAT: Process Tree Analysis

```kql
TrendAI_XDR_OAT_V2_CL
| where isnotempty(detail_processCmd_s)
| project TimeGenerated,
    Endpoint = detail_endpointHostName_s,
    CommandLine = detail_processCmd_s,
    ParentCommandLine = detail_parentCmd_s,
    ParentHash = detail_parentFileHashSha256_s
```

## 🌍 Supported Regions

| Region | API Domain |
|--------|------------|
| United States | `api.xdr.trendmicro.com` |
| Europe | `api.eu.xdr.trendmicro.com` |
| Singapore | `api.sg.xdr.trendmicro.com` |
| Japan | `api.xdr.trendmicro.co.jp` |
| Australia | `api.au.xdr.trendmicro.com` |
| India | `api.in.xdr.trendmicro.com` |
| Middle East & Africa | `api.mea.xdr.trendmicro.com` |
| United Kingdom | `api.uk.xdr.trendmicro.com` |
| Canada | `api.ca.xdr.trendmicro.com` |
| South Africa | `api.za.xdr.trendmicro.com` |

## 📁 Repository Structure

```
Solutions/TrendAI Vision One(CCF)/
├── Analytic Rules/         # Workbench incident creation rule
├── Data/                   # Solution manifest & metadata
├── Data Connectors/
│   ├── TrendAIVisionOneWorkbench_ccp/   # Connector definition, DCR, table schema
│   └── TrendAIVisionOneOAT_ccp/         # Connector definition, DCR, table schema
├── Package/
│   ├── mainTemplate.json   # Full ARM template (deployable unit)
│   ├── createUiDefinition.json
│   ├── package.sh          # Builds the deployable .zip from the above
│   └── *.zip                # Packaged solution versions
├── Parsers/                # TrendAIWorkbench_Complete, TrendAIOAT_Complete KQL functions
├── Workbooks/               # Workbench Overview dashboard
└── README.md                # Solution-level reference (install, params, schemas, troubleshooting)
```

## 🛠️ Advanced Deployment

### Using Azure CLI

```bash
az deployment group create \
  --resource-group <your-rg> \
  --template-uri "https://raw.githubusercontent.com/trendmicro/trendai-sentinel-ccf-data-connector/main/Solutions/TrendAI%20Vision%20One(CCF)/Package/mainTemplate.json" \
  --parameters workspace=<workspace-name>
```

## 🔧 Troubleshooting

### No data after 10 minutes

1. **Check API token**: Verify token has the required view permissions and is not expired
2. **Check region**: Ensure correct TrendAI Vision One API domain selected
3. **Check connector status**: Sentinel → Data connectors → View connector health
4. **Check DCR ingestion**: Azure Monitor → Data Collection Rules → View metrics

### Connection fails

- Regenerate API token if expired
- Ensure no extra spaces when copying the token
- Verify workspace has Sentinel enabled

### Missing IOC fields (Workbench)

- Use the parser function: `TrendAIWorkbench_Complete()`
- Parser extracts IOCs from dynamic columns automatically

## 📚 Documentation

For a deeper reference on installation, parameters, data schemas, and troubleshooting, see the solution's own [README](<Solutions/TrendAI Vision One(CCF)/README.md>).

## 🤝 Support

- **Trend Vision One API**: [Trend Micro Support](https://www.trendmicro.com/support)
- **Azure Sentinel**: [Microsoft Sentinel Documentation](https://learn.microsoft.com/azure/sentinel)
- **Issues**: [GitHub Issues](https://github.com/trendmicro/trendai-sentinel-ccf-data-connector/issues)
- **Contributing**: see [CONTRIBUTING.md](CONTRIBUTING.md)

## 🤝 Contributing

Pull requests are welcome — see [CONTRIBUTING.md](CONTRIBUTING.md) and our [Code of Conduct](CODE_OF_CONDUCT.md).

## 🔒 Security

Found a vulnerability? Please **do not** open a public issue — see [SECURITY.md](SECURITY.md) for the private reporting process.

## 📝 License

This project is licensed under the [MIT License](LICENSE) — see the LICENSE file for details.

## 🏆 Credits

Built following Microsoft's [Codeless Connector Platform (CCP)](https://learn.microsoft.com/azure/sentinel/create-codeless-connector) best practices and [SentinelOne reference implementation](https://github.com/Azure/Azure-Sentinel/tree/master/Solutions/SentinelOne/Data%20Connectors/SentinelOne_ccp).

---

**Version**: 3.0.2  
**Maintained by**: Trend Micro
