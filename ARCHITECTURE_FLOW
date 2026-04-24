# Azure GRC Platform - Detailed Architecture Flow

## 1. COMPLETE END-TO-END FLOW

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         ENTRY POINTS TO AZURE FUNCTION                       │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                    ┌───────────────────┴───────────────────┐
                    │                                       │
        ┌───────────▼──────────┐              ┌──────────▼──────────┐
        │  HTTP TRIGGER        │              │  TIMER TRIGGER      │
        │  /analyze endpoint   │              │  Scheduled hourly   │
        │  ?hours=24           │              │  (0 0 * * * *)      │
        │  &framework=SOC2     │              │                      │
        └───────────┬──────────┘              └──────────┬───────────┘
                    │                                    │
                    └────────────────┬───────────────────┘
                                     │
                    ┌────────────────▼───────────────────┐
                    │   compliance_http_trigger()        │
                    │   compliance_timer_trigger()       │
                    │         (function_app.py)          │
                    │                                     │
                    │  Extract parameters:               │
                    │  • hours (default: 24)             │
                    │  • framework (default: SOC2)       │
                    └────────────────┬───────────────────┘
                                     │
                    ┌────────────────▼───────────────────┐
                    │    run_and_report()                │
                    │  (Main Orchestration Function)     │
                    └────────────────┬───────────────────┘
                                     │
        ┌────────────────────────────┼────────────────────────────┐
        │                            │                            │
        │                            │                            │
   STEP 1              STEP 2            STEP 3          STEP 4 (PARALLEL)
        │                            │                            │
        ▼                            ▼                            ▼
  ┌──────────────┐           ┌──────────────┐           ┌──────────────┐
  │  MAPPER      │           │  GET NIST    │           │   ANALYZE    │
  │ INITIALIZATION           │  CONTROL     │           │  FINDINGS    │
  │              │           │  STATUS      │           │              │
  │ MapperFactory │           │              │           │              │
  │ .get_all()    │           │ get_nist_    │           │ analyze_     │
  │ Creates:      │           │ control_     │           │ findings()   │
  │ • SOC2Mapper  │           │ status()     │           │              │
  │ • NISTMapper  │           │              │           │ For each     │
  │              │           │ Queries:     │           │ finding:     │
  │ Returns dict  │           │ • Regulatory │           │ • Get title  │
  │ of mappers    │           │   Compliance │           │ • Severity   │
  │              │           │   Standards  │           │ • Type       │
  └──────────────┘           │ • Control    │           │ • Resource   │
        │                     │   status     │           │              │
        │                     │   by family  │           │ Uses OpenAI  │
        │                     │              │           │ to analyze   │
        │                     │ Returns:     │           │ impact       │
        │                     │ Dict of      │           │              │
        │                     │ {family: {   │           │ Returns:     │
        │                     │   status...} │           │ analyses{}   │
        │                     │ }            │           │ stats{}      │
        │                     └──────────────┘           └──────────────┘
        │                            │                            │
        └────────────────────────────┼────────────────────────────┘
                                     │
        ┌────────────────────────────▼────────────────────────────┐
        │                 STEP 5: GET FINDINGS                     │
        │              get_findings(hours, framework_id)          │
        │                    (analyzer.py)                         │
        │                                                           │
        │  Calls Azure Security Center API:                        │
        │  ┌────────────────────────────────────────────┐          │
        │  │ client.assessments.list(scope=subscription) │          │
        │  └────────────────────────────────────────────┘          │
        │                        │                                  │
        │         ┌──────────────┴──────────────┐                  │
        │         │                             │                  │
        │    Filters:                      Transforms:             │
        │    • Status == "Unhealthy"       • Title                │
        │    • Time >= (now - hours)       • Severity             │
        │    • Framework matches           • Description           │
        │                                  • Resource ID           │
        │                                  • Finding Type          │
        │                                                           │
        │  Returns: findings{                                       │
        │    "SOC2": [finding1, finding2, ...],                    │
        │    "NIST800-53": [finding1, finding3, ...]              │
        │  }                                                        │
        └────────────────────────────────────────────────────────┘
                                     │
        ┌────────────────────────────▼────────────────────────────┐
        │               STEP 6: MAP FINDINGS TO CONTROLS           │
        │        For each framework_id in frameworks:              │
        │        mapper = mappers[framework_id]                    │
        │        analysis = mapper.map_findings(findings)          │
        │                   (framework_mapper.py)                  │
        │                                                           │
        │  SOC2Mapper.map_findings():                              │
        │  ┌─────────────────────────────────────────────┐        │
        │  │ For each finding:                           │        │
        │  │ 1. Match to SOC 2 control (Trust Service)  │        │
        │  │ 2. Extract severity/impact                 │        │
        │  │ 3. Generate mapping explanation            │        │
        │  │ 4. Return control → findings[] structure   │        │
        │  └─────────────────────────────────────────────┘        │
        │                                                           │
        │  NIST80053Mapper.map_findings():                         │
        │  ┌─────────────────────────────────────────────┐        │
        │  │ For each finding:                           │        │
        │  │ 1. Match to NIST 800-53 control            │        │
        │  │ 2. Map to control family (AC, CM, etc.)    │        │
        │  │ 3. Determine control status impact         │        │
        │  │ 4. Return control → findings[] structure   │        │
        │  └─────────────────────────────────────────────┘        │
        │                                                           │
        │  Returns: analyses{                                       │
        │    "SOC2": "[markdown report of mappings]",              │
        │    "NIST800-53": "[markdown report]",                    │
        │    "combined": "[cross-framework analysis]"              │
        │  }                                                        │
        └────────────────────────────────────────────────────────┘
                                     │
        ┌────────────────────────────▼────────────────────────────┐
        │         STEP 7: GENERATE NIST cATO REPORT                │
        │    generate_nist_cato_report(nist_status)                │
        │           (reporter.py)                                   │
        │                                                           │
        │  Takes control status data from STEP 2:                  │
        │  ┌─────────────────────────────────────────────┐        │
        │  │ For each control family (AC, CM, IA, etc.): │        │
        │  │ • Count passing/failing/N/A controls        │        │
        │  │ • Calculate compliance percentage           │        │
        │  │ • Generate visual indicators                │        │
        │  │ • Format markdown table                     │        │
        │  └─────────────────────────────────────────────┘        │
        │                                                           │
        │  Example output:                                         │
        │  ```                                                      │
        │  ## Control Family Status                                │
        │  ### AC: Access Control                                  │
        │  Total: 25, Passing: 20 (80%), Failing: 3 (12%)         │
        │  ```                                                      │
        │                                                           │
        │  Returns: markdown_report_string                         │
        └────────────────────────────────────────────────────────┘
                                     │
        ┌────────────────────────────▼────────────────────────────┐
        │            STEP 8: BUILD COMPLETE REPORT TEXT            │
        │        Concatenate analyses + cATO report                │
        │                                                           │
        │  report_text = """                                        │
        │  # Azure GRC Compliance Report                           │
        │                                                           │
        │  ## SOC2 Analysis                                        │
        │  [analysis from STEP 6]                                 │
        │                                                           │
        │  ## NIST800-53 Analysis                                 │
        │  [analysis from STEP 6]                                 │
        │                                                           │
        │  ## Control Family Status                               │
        │  [cATO report from STEP 7]                              │
        │  """                                                      │
        │                                                           │
        │  Returns: report_text                                    │
        └────────────────────────────────────────────────────────┘
                                     │
        ┌────────────────────────────▼────────────────────────────┐
        │            STEP 9: GENERATE CSV EXPORT                   │
        │         generate_csv(findings, mappers)                  │
        │            (reporter.py)                                  │
        │                                                           │
        │  Creates tabular export:                                 │
        │  ┌──────────────────────────────────────────────┐       │
        │  │ Finding Title | Framework | Control | Severity        │
        │  │───────────────────────────────────────────────        │
        │  │ ... (one row per finding per framework)     │         │
        │  └──────────────────────────────────────────────┘       │
        │                                                           │
        │  Returns: csv_data (string)                              │
        └────────────────────────────────────────────────────────┘
                                     │
        ┌────────────────────────────▼────────────────────────────┐
        │              STEP 10: SEND EMAIL REPORT                  │
        │         send_email_report(report_text, csv_data)         │
        │             (reporter.py)                                 │
        │                                                           │
        │  Azure Communication Services Email Delivery:            │
        │  ┌────────────────────────────────────────────┐         │
        │  │ client = EmailClient(endpoint, credential) │         │
        │  │ send_message(                              │         │
        │  │   sender="verified@example.com",           │         │
        │  │   to=["recipient@example.com"],            │         │
        │  │   subject="Azure GRC Report",              │         │
        │  │   html_content=render_template(            │         │
        │  │     report_text,                           │         │
        │  │     csv_data                               │         │
        │  │   )                                         │         │
        │  │ )                                            │         │
        │  └────────────────────────────────────────────┘         │
        │                                                           │
        │  Uses email template: deployment/config/                │
        │                       nist_cato_email_template.html      │
        │                                                           │
        │  Returns: email_id or throws exception                  │
        └────────────────────────────────────────────────────────┘
                                     │
        ┌────────────────────────────▼────────────────────────────┐
        │               STEP 11: RETURN RESPONSE                   │
        │                                                           │
        │  If HTTP Trigger:                                        │
        │  ┌────────────────────────────────────────────┐         │
        │  │ HttpResponse {                             │         │
        │  │   status: "success",                       │         │
        │  │   message: "Report generated and sent",    │         │
        │  │   report: report_text                      │         │
        │  │ }                                            │         │
        │  └────────────────────────────────────────────┘         │
        │                                                           │
        │  If Timer Trigger:                                       │
        │  ┌────────────────────────────────────────────┐         │
        │  │ Log completion and exit                    │         │
        │  │ (No HTTP response, just logging)           │         │
        │  └────────────────────────────────────────────┘         │
        │                                                           │
        └────────────────────────────────────────────────────────┘
```

---

## 2. DETAILED COMPONENT INTERACTION DIAGRAM

```
┌────────────────────────────────────────────────────────────────────┐
│                        CONFIGURATION LOADING                         │
└────────────────────────────────────────────────────────────────────┘

At Startup (when mapper_factory imports):

    load_frameworks()
           │
           ├─► Read: config/frameworks.json
           │   {
           │     "frameworks": [
           │       {"id": "SOC2", "name": "SOC 2", ...},
           │       {"id": "NIST800-53", "name": "NIST 800-53", ...}
           │     ]
           │   }
           │
           └─► Returns: List[Dict] of framework definitions


    MapperFactory.get_all_mappers()
           │
           ├─► SOC2Mapper()
           │   ├─► Read: config/mappings/soc2_mappings.json
           │   │   {
           │   │     "controls": [
           │   │       {"id": "CC6.1", "name": "Change Control", ...}
           │   │     ]
           │   │   }
           │   └─► Store in self.controls_mapping
           │
           └─► NIST80053Mapper()
               ├─► Read: config/mappings/nist800_53_mappings.json
               │   {
               │     "controls": [
               │       {"family": "AC", "id": "AC-1", "name": "...", ...}
               │     ]
               │   }
               └─► Store in self.controls_mapping


┌────────────────────────────────────────────────────────────────────┐
│                   AZURE SERVICE INTEGRATION FLOW                     │
└────────────────────────────────────────────────────────────────────┘

AZURE DEFENDER FOR CLOUD (Security Center API)
│
├─► get_security_client()
│   ├─► credential = DefaultAzureCredential()
│   │   (checks: env vars, managed identity, CLI creds, etc.)
│   │
│   └─► SecurityCenter(credential, subscription_id)
│       (AZURE SDK client)
│
└─► client.assessments.list(scope="/subscriptions/{sub_id}")
    │
    ├─► Iterates through assessments
    │   │
    │   └─► For each assessment with status="Unhealthy":
    │       ├─► Title: assessment.display_name
    │       ├─► Severity: assessment.status.cause
    │       ├─► Resource: assessment.resource_details.id
    │       └─► Status: "Unhealthy" (= security finding)
    │
    └─► Filter by framework (if specified)
        └─► Returns: findings{framework_id: [finding1, ...]}


AZURE COMMUNICATION SERVICES (Email Delivery)
│
└─► EmailClient(endpoint, credential)
    │
    └─► send_message()
        ├─► sender: Verified sender email
        ├─► to: [Verified recipient emails]
        ├─► subject: "Azure GRC Compliance Report"
        ├─► html_content: Rendered email template
        │   └─► Uses: deployment/config/nist_cato_email_template.html
        │       Substitutes:
        │       • {REPORT_CONTENT} ← report_text
        │       • {CSV_ATTACHMENT} ← csv_data
        │
        └─► Returns: email_id (success) or raises exception


OPENAI API (Analysis Service)
│
└─► openai.ChatCompletion.create() [in reporter.py]
    │
    └─► Prompt each finding for compliance impact analysis
        ├─► Input: Finding details + Control definitions
        ├─► Task: "Analyze how this security finding affects [framework] controls"
        ├─► Response: AI-generated analysis explaining impact
        │
        └─► Results aggregated into analysis report


┌────────────────────────────────────────────────────────────────────┐
│              MAPPER IMPLEMENTATION DETAIL (FRAMEWORK MAPPER PATTERN)  │
└────────────────────────────────────────────────────────────────────┘

FrameworkMapper (base class)
│   def __init__(self, framework_id, mappings_file):
│       self.framework_id = framework_id           ← "SOC2" or "NIST800-53"
│       self.controls_mapping = load_json(file)    ← Load control definitions
│
│   def map_findings(self, findings, ...):
│       findings_by_control = {}
│       for finding in findings:
│           matched_controls = self.find_matching_controls(finding)
│           for control in matched_controls:
│               add_finding_to_control(findings_by_control[control])
│       return findings_by_control


SOC2Mapper extends FrameworkMapper
│
├─► Control Definitions: Trust Service Criteria (CC, TSC, etc.)
│   ├─► CC6: Logical and Physical Access Controls
│   ├─► CC7: System Monitoring
│   └─► CC9: Encryption and Key Management
│
├─► Matching Logic:
│   └─► For each finding:
│       ├─► Parse finding types and keywords
│       ├─► Search self.controls_mapping for matching control
│       ├─► Use similarity scoring if keyword match weak
│       └─► Return best matching control ID
│
└─► Output:
    {
      "CC6.1": [finding1, finding3],
      "CC7.2": [finding2],
      ...
    }


NIST80053Mapper extends FrameworkMapper
│
├─► Control Definitions: NIST 800-53 control families
│   ├─► AC: Access Control (AC-1, AC-2, AC-3, ...)
│   ├─► CM: Configuration Management
│   ├─► IA: Identification and Authentication
│   ├─► SC: System and Communications Protection
│   └─► ... (13 control families total)
│
├─► Matching Logic:
│   └─► For each finding:
│       ├─► Parse finding for security control area
│       ├─► Match to NIST control family (AC, CM, etc.)
│       ├─► Find specific control within family
│       └─► Also store control status from Step 2
│
└─► Special: cATO Status Tracking
    ├─► Maps findings to control status
    ├─► Calculates family-level compliance percentages
    ├─► Identifies control families at risk
    └─► Generates remediation priorities


┌────────────────────────────────────────────────────────────────────┐
│                  FINDING DATA STRUCTURE TRANSFORMATION               │
└────────────────────────────────────────────────────────────────────┘

Azure Assessment (from Defender for Cloud)
├─► name: "checkRestrictedSSHPorts"
├─► display_name: "Restricted SSH port access should be enforced"
├─► status.code: "Unhealthy"
├─► status.cause: "HIGH"
├─► resource_details.id: "/subscriptions/{id}/resources/vm-1"
└─► metadata: {...}

            │
            │ Transform in get_findings()
            ▼

Finding (Internal Representation)
├─► Title: "Restricted SSH port access should be enforced"
├─► Description: "Restricted SSH port access should be enforced"
├─► Severity: {Label: "HIGH"}
├─► Types: ["Software and Configuration Checks"]
├─► Resources: [{Id: "/subscriptions/{id}/resources/vm-1"}]
├─► id: "checkRestrictedSSHPorts"
└─► status: "Unhealthy"

            │
            │ Passed to mappers
            │ SOC2Mapper.map_findings(findings)
            ▼

Mapped Finding (After Framework Mapping)
├─► Original Finding: {...}
├─► Matched Control: "CC6.1"  ← SOC 2 Control
├─► Match Confidence: 0.92    ← Similarity score
├─► Reasoning: "Network access control affects Trust Service Criteria"
└─► Remediation Control: "Enforce SSH key-based auth, disable root login"

            │
            │ Passed to reporter
            │ send_email_report(analyses)
            ▼

Email Content (Rendered)
├─► Subject: "Azure GRC Compliance Report"
├─► HTML Body:
│   ├─► Finding Title: "Restricted SSH port access..."
│   ├─► Control: "CC6.1 - Logical and Physical Access Controls"
│   ├─► Severity: 🔴 HIGH
│   ├─► AI Analysis: "This finding directly impacts...[OpenAI response]"
│   └─► Remediation: "Apply this control..."
│
└─► CSV Attachment:
    ├─► Columns: Finding, Framework, Control, Severity, Resource
    └─► Rows: One per finding-control mapping


┌────────────────────────────────────────────────────────────────────┐
│                    EXECUTION TIMELINE EXAMPLE                        │
└────────────────────────────────────────────────────────────────────┘

T=0s    │ HTTP Request arrives: /analyze?hours=24&framework=SOC2
        │
T=0.1s  │ ├─► Parse parameters
        │ └─► Call run_and_report(hours=24, framework_id="SOC2")
        │
T=0.2s  │ ├─► MapperFactory.get_all_mappers()
        │ │   └─► Load SOC2Mapper + NIST80053Mapper (from JSON configs)
        │ │
        │ └─► get_findings(hours=24, framework_id="SOC2")
        │     ├─► Initialize Azure Security Client
        │     ├─► Query Defender for Cloud assessments
        │     └─► Filter Unhealthy assessments from last 24 hours
        │         ✓ Found 12 findings for SOC2
        │
T=1.5s  │ ├─► get_nist_control_status()
        │ │   ├─► Query Regulatory Compliance Standards
        │ │   └─► Extract NIST 800-53 control statuses
        │ │       ✓ AC family: 20/25 passing
        │ │       ✓ CM family: 18/22 passing
        │ │
        │ └─► analyze_findings(findings, mappers)
        │     ├─► SOC2Mapper.map_findings(findings)
        │     │   └─► For each of 12 findings: match to SOC2 control
        │     │       └─► 3 findings → CC6.1
        │     │       └─► 2 findings → CC7.2
        │     │       └─► 7 findings → other controls
        │     │
        │     └─► (OpenAI analysis happens in parallel for each finding)
        │         └─► "This finding affects CC6.1 because..."
        │
T=4.2s  │ ├─► generate_nist_cato_report(nist_status)
        │ │   └─► Compile control status by family
        │ │       ```
        │ │       AC: 20/25 (80%)
        │ │       CM: 18/22 (82%)
        │ │       ...
        │ │       ```
        │ │
        │ └─► Build report_text
        │     ```
        │     # Azure GRC Compliance Report
        │     
        │     ## SOC2 Analysis
        │     ... [12 findings mapped to controls]
        │     
        │     ## Control Family Status
        │     ... [NIST cATO status by family]
        │     ```
        │
T=4.5s  │ ├─► generate_csv(findings, mappers)
        │ │   └─► Create tabular export
        │ │
        │ └─► send_email_report(report_text, csv_data)
        │     └─► EmailClient.send_message()
        │         ├─► Render email template
        │         ├─► Attach CSV
        │         └─► Send via Azure Communication Services
        │             ✓ Email sent successfully (ID: xyz123)
        │
T=5.0s  │ └─► Return HttpResponse (200 OK)
        │     {
        │       "status": "success",
        │       "message": "Report generated and sent",
        │       "report": "[full report text]"
        │     }


┌────────────────────────────────────────────────────────────────────┐
│                        ERROR HANDLING FLOW                           │
└────────────────────────────────────────────────────────────────────┘

HTTP Request
    │
    ├─► Exception in get_findings()
    │   └─► Reason: Azure credentials not valid
    │   └─► Caught by: try/except in compliance_http_trigger()
    │   └─► Response: HttpResponse(status=500, message="Error: ...")
    │
    ├─► Exception in analyze_findings()
    │   └─► Reason: OpenAI API rate limit exceeded
    │   └─► Caught by: try/except in run_and_report()
    │   └─► Result: Analysis skipped, email sent with available data
    │
    ├─► Exception in send_email_report()
    │   └─► Reason: Email address not verified in Communication Services
    │   └─► Caught by: try/except in run_and_report()
    │   └─► Response: HttpResponse(status=500, message="Email send failed")
    │
    └─► Success Path
        └─► All steps complete
        └─► Response: HttpResponse(status=200, "success")

```

---

## 3. FILE INTERACTION DEPENDENCY DIAGRAM

```
ENTRY POINT
│
├─► function_app.py
│   ├─► IMPORTS:
│   │   ├─► analyzer.py (get_findings, get_nist_control_status, analyze_findings)
│   │   ├─► reporter.py (generate_csv, generate_nist_cato_report, send_email_report)
│   │   └─► mapper_factory.py (MapperFactory)
│   │
│   └─► CALLS: run_and_report()
│       │
│       ├─► [1] mapper_factory.get_all_mappers()
│       │   │
│       │   ├─► IMPORTS:
│       │   │   ├─► mappers/soc2_mapper.py (SOC2Mapper class)
│       │   │   ├─► mappers/nist_mapper.py (NIST80053Mapper class)
│       │   │   └─► framework_mapper.py (FrameworkMapper base)
│       │   │
│       │   ├─► READS:
│       │   │   ├─► config/frameworks.json
│       │   │   ├─► config/mappings/soc2_mappings.json
│       │   │   └─► config/mappings/nist800_53_mappings.json
│       │   │
│       │   └─► RETURNS: {"SOC2": SOC2Mapper(), "NIST800-53": NISTMapper()}
│       │
│       ├─► [2] analyzer.get_findings()
│       │   │
│       │   ├─► IMPORTS:
│       │   │   ├─► azure.identity.DefaultAzureCredential
│       │   │   ├─► azure.mgmt.security.SecurityCenter
│       │   │   └─► mapper_factory.load_frameworks()
│       │   │
│       │   ├─► QUERIES:
│       │   │   └─► Azure Defender for Cloud API
│       │   │       └─► /subscriptions/{sub}/providers/Microsoft.Security/assessments
│       │   │
│       │   └─► RETURNS: {"SOC2": [finding, ...], "NIST800-53": [...]}
│       │
│       ├─► [3] analyzer.get_nist_control_status()
│       │   │
│       │   ├─► QUERIES:
│       │   │   └─► Azure Regulatory Compliance Standards API
│       │   │
│       │   └─► RETURNS: {"AC": {...}, "CM": {...}, ...}
│       │
│       ├─► [4] analyzer.analyze_findings()
│       │   │
│       │   ├─► For each mapper in mappers dict:
│       │   │   │
│       │   │   ├─► mapper.map_findings()
│       │   │   │   │
│       │   │   │   ├─► soc2_mapper.py:
│       │   │   │   │   └─► Loads soc2_mappings.json
│       │   │   │   │   └─► Matches findings to SOC2 controls
│       │   │   │   │
│       │   │   │   ├─► nist_mapper.py:
│       │   │   │   │   └─► Loads nist800_53_mappings.json
│       │   │   │   │   └─► Matches findings to NIST controls
│       │   │   │   │
│       │   │   │   └─► CALLS: openai.ChatCompletion.create()
│       │   │   │       └─► For analysis of each finding
│       │   │   │
│       │   │   └─► RETURNS: {control_id: [findings], ...}
│       │   │
│       │   └─► RETURNS: {"SOC2": analysis_text, "NIST800-53": analysis_text, ...}
│       │
│       ├─► [5] reporter.generate_nist_cato_report()
│       │   │
│       │   ├─► IMPORTS:
│       │   │   ├─► analyzer (already imported)
│       │   │   └─► utils.format_datetime()
│       │   │
│       │   ├─► PROCESSES: nist_control_status from [3]
│       │   │
│       │   └─► RETURNS: markdown report of control family status
│       │
│       ├─► [6] reporter.generate_csv()
│       │   │
│       │   └─► PROCESSES: findings + mappers
│       │   └─► RETURNS: CSV string (findings by framework and control)
│       │
│       └─► [7] reporter.send_email_report()
│           │
│           ├─► IMPORTS:
│           │   └─► azure.communication.email.EmailClient
│           │
│           ├─► READS:
│           │   └─► deployment/config/nist_cato_email_template.html
│           │
│           ├─► CALLS:
│           │   └─► EmailClient.send_message()
│           │       └─► To verified email addresses via Azure Communication Services
│           │
│           └─► RETURNS: email_id or raises exception


TESTING STRUCTURE
│
└─► tests/ directory
    │
    ├─► conftest.py
    │   ├─► Fixtures:
    │   │   ├─► mock_security_client
    │   │   ├─► mock_email_client
    │   │   ├─► sample_findings
    │   │   ├─► sample_nist_status
    │   │   └─► mappers_dict
    │   │
    │   └─► Mocks Azure services to avoid real API calls
    │
    ├─► test_app.py
    │   └─► Tests: function_app.py entry points
    │       ├─► HTTP trigger invocation
    │       ├─► Parameter parsing
    │       └─► Response format
    │
    ├─► test_app_findings.py
    │   └─► Tests: analyzer.get_findings()
    │       ├─► Unhealthy assessment filtering
    │       ├─► Framework filtering
    │       └─► Time-based filtering
    │
    ├─► test_app_nist.py
    │   └─► Tests: analyzer.get_nist_control_status()
    │       ├─► Control status retrieval
    │       ├─► Family-level aggregation
    │       └─► Compliance percentage calculation
    │
    ├─► test_soc2_mapper.py
    │   └─► Tests: mappers/soc2_mapper.py
    │       ├─► Finding-to-control matching
    │       ├─► Mapping accuracy
    │       └─► CSV generation
    │
    ├─► test_app_email.py
    │   └─► Tests: reporter.send_email_report()
    │       ├─► Email content rendering
    │       ├─► Attachment handling
    │       └─► Send success/failure
    │
    └─► test_utils.py
        └─► Tests: utils.py utility functions
            ├─► Date formatting
            ├─► Severity parsing
            └─► String utilities

```

---

## 4. CONFIGURATION DATA FLOW

```
┌────────────────────────────────────────────────────────────────────┐
│              CONFIGURATION FILES HIERARCHY                           │
└────────────────────────────────────────────────────────────────────┘

Application Startup
    │
    ├─► Environment Variables
    │   ├─► AZURE_SUBSCRIPTION_ID        (Required)
    │   ├─► AZURE_TENANT_ID              (Optional, DefaultAzureCredential)
    │   ├─► OPENAI_API_KEY               (Optional, for analysis)
    │   ├─► COMMUNICATION_SERVICES_ENDPOINT (Optional, Email)
    │   └─► COMMUNICATION_SERVICES_KEY   (Optional, Email)
    │
    ├─► config/frameworks.json (Define available frameworks)
    │   │
    │   ├─► JSON Structure:
    │   │   {
    │   │     "frameworks": [
    │   │       {
    │   │         "id": "SOC2",
    │   │         "name": "SOC 2",
    │   │         "description": "Service Organization Control 2",
    │   │         "mappingsFile": "config/mappings/soc2_mappings.json"
    │   │       },
    │   │       {
    │   │         "id": "NIST800-53",
    │   │         "name": "NIST 800-53",
    │   │         "description": "NIST SP 800-53 Revision 5",
    │   │         "mappingsFile": "config/mappings/nist800_53_mappings.json"
    │   │       }
    │   │     ]
    │   │   }
    │   │
    │   └─► Used by: mapper_factory.load_frameworks()
    │
    ├─► config/mappings/soc2_mappings.json
    │   │
    │   ├─► JSON Structure:
    │   │   {
    │   │     "framework": "SOC2",
    │   │     "controls": [
    │   │       {
    │   │         "id": "CC6.1",
    │   │         "category": "Logical and Physical Access Controls",
    │   │         "description": "...",
    │   │         "keywords": ["access", "authentication", "ssh", "port"],
    │   │         "severity_threshold": "MEDIUM"
    │   │       },
    │   │       {
    │   │         "id": "CC7.2",
    │   │         "category": "System Monitoring and Incident Detection",
    │   │         "description": "...",
    │   │         "keywords": ["audit", "logging", "monitoring", "detection"],
    │   │         "severity_threshold": "LOW"
    │   │       },
    │   │       ... (more SOC2 controls)
    │   │     ]
    │   │   }
    │   │
    │   └─► Used by: SOC2Mapper.__init__()
    │
    ├─► config/mappings/nist800_53_mappings.json
    │   │
    │   ├─► JSON Structure:
    │   │   {
    │   │     "framework": "NIST800-53",
    │   │     "families": [
    │   │       {
    │   │         "id": "AC",
    │   │         "name": "Access Control",
    │   │         "controls": [
    │   │           {
    │   │             "id": "AC-1",
    │   │             "title": "Policy and Procedures",
    │   │             "description": "...",
    │   │             "keywords": ["policy", "procedures", "access"],
    │   │             "severity_threshold": "HIGH"
    │   │           },
    │   │           ... (more AC controls)
    │   │         ]
    │   │       },
    │   │       {
    │   │         "id": "CM",
    │   │         "name": "Configuration Management",
    │   │         "controls": [...],
    │   │       },
    │   │       ... (other NIST families)
    │   │     ]
    │   │   }
    │   │
    │   └─► Used by: NIST80053Mapper.__init__()
    │
    └─► deployment/config/nist_cato_email_template.html
        │
        ├─► HTML Template with placeholders:
        │   ├─► {REPORT_CONTENT}      ← report_text (markdown converted to HTML)
        │   ├─► {CSV_ATTACHMENT}      ← csv_data (base64 encoded)
        │   ├─► {GENERATED_DATE}      ← Current timestamp
        │   └─► {TOTAL_FINDINGS}      ← Count of findings
        │
        └─► Used by: reporter.send_email_report()


┌────────────────────────────────────────────────────────────────────┐
│            CONTROL MAPPING MATCHING ALGORITHM EXAMPLE                │
└────────────────────────────────────────────────────────────────────┘

Finding from Defender for Cloud:
{
  "Title": "Restrict SSH port access should be enforced",
  "Description": "SSH port should not be open to all IPs",
  "Severity": {"Label": "HIGH"},
  "Types": ["Network Security"],
  "Resources": [{"Id": "/subscriptions/.../vm-1"}]
}


SOC2Mapper.map_findings(findings)
    │
    ├─► For each finding:
    │   │
    │   ├─► Extract keywords: ["SSH", "port", "restrict", "access"]
    │   │
    │   ├─► Iterate through self.controls (from soc2_mappings.json):
    │   │   │
    │   │   ├─► Control CC6.1 (Logical and Physical Access Controls)
    │   │   │   Keywords: ["access", "authentication", "ssh", "port"]
    │   │   │   Match Score: 4/4 keywords match = 100% ✓ BEST MATCH
    │   │   │
    │   │   ├─► Control CC7.2 (System Monitoring)
    │   │   │   Keywords: ["audit", "logging", "monitoring", "detection"]
    │   │   │   Match Score: 0/4 keywords match = 0%
    │   │   │
    │   │   └─► Control CC9.1 (Encryption and Key Management)
    │   │       Keywords: ["encryption", "key", "certificate", "tls"]
    │   │       Match Score: 0/4 keywords match = 0%
    │   │
    │   ├─► Selected Control: CC6.1 (highest score)
    │   │
    │   └─► Add to results:
    │       findings_by_control["CC6.1"].append({
    │         "original_finding": {...},
    │         "match_confidence": 1.0,
    │         "reasoning": "Multiple keywords match: access, ssh, port"
    │       })
    │
    └─► Return: findings_by_control = {
          "CC6.1": [finding1],
          "CC7.2": [],
          "CC9.1": [],
          ...
        }

```

---

## 5. ERROR HANDLING & RETRY LOGIC

```
┌────────────────────────────────────────────────────────────────────┐
│                   ERROR HANDLING IN FUNCTION_APP.PY                  │
└────────────────────────────────────────────────────────────────────┘

compliance_http_trigger(req)
    │
    └─► try:
            │
            ├─► Parse parameters
            ├─► Call run_and_report()
            │
            └─► Return HttpResponse(status=200)
        
        except Exception as e:
            │
            ├─► Log error: logging.error(f"Error: {e}")
            │
            └─► Return HttpResponse(
                    f"Error: {e}",
                    status_code=500
                )


run_and_report() - INTERNAL ERROR HANDLING
    │
    ├─► get_findings()
    │   └─► Exception: AZURE_SUBSCRIPTION_ID not set
    │       └─► Caught: By function_app try/except
    │       └─► Response: 500 error to user
    │
    ├─► analyze_findings()
    │   └─► Exception: OpenAI API rate limit
    │       └─► Caught: May be caught here, analysis skipped
    │       └─► Behavior: Email sent with partial data
    │
    ├─► send_email_report()
    │   └─► Exception: Email address not verified
    │       └─► Caught: By function_app try/except
    │       └─► Response: 500 error, instructions to verify email
    │
    └─► Success: All exceptions handled gracefully


RETRY LOGIC IN CI/CD PIPELINE (.github/workflows/ci.yml)
    │
    ├─► pip install --retries 5 --timeout 60
    │   └─► If pip fails, retry up to 5 times
    │   └─► Wait up to 60 seconds
    │
    └─► Applied to:
        ├─► requirements.txt installation
        └─► debug_requirements.txt installation (if exists)

```

---

## 6. MEMORY & PERFORMANCE CHARACTERISTICS

```
┌────────────────────────────────────────────────────────────────────┐
│              DATA STRUCTURES IN MEMORY DURING EXECUTION              │
└────────────────────────────────────────────────────────────────────┘

startup:
    mappers dict (in memory):
    ├─► "SOC2" → SOC2Mapper object
    │   ├─► controls_mapping: JSON loaded from disk (cached)
    │   ├─► ~50 KB for SOC2 controls
    │   └─► Reused for entire function lifetime
    │
    └─► "NIST800-53" → NIST80053Mapper object
        ├─► controls_mapping: JSON loaded from disk (cached)
        ├─► ~80 KB for NIST 800-53 controls
        └─► Reused for entire function lifetime


per request:
    findings dict:
    ├─► Size: ~100 KB per 100 findings
    ├─► Structure:
    │   {
    │     "SOC2": [finding1, finding2, ...],
    │     "NIST800-53": [finding1, finding3, ...]
    │   }
    └─► Freed after email sent

    report_text string:
    ├─► Size: ~50-200 KB depending on findings count
    ├─► Contains: All analyses + cATO report
    └─► Freed after email sent

    csv_data string:
    ├─► Size: ~20-100 KB depending on findings count
    └─► Freed after email sent

    analyses dict:
    ├─► Size: ~100 KB
    ├─► Contains: Analysis for each framework
    └─► Freed after concatenation to report_text


EXECUTION TIME BREAKDOWN (typical):
    Get findings:           1-2 seconds
    Get NIST status:        1-2 seconds
    Map findings:           1-3 seconds
    OpenAI analysis:        5-15 seconds (depends on finding count)
    Generate reports:       1-2 seconds
    Send email:             1-2 seconds
    ─────────────────────────────────────
    TOTAL:                  10-26 seconds per request

Azure Function Cold Start Overhead:
    Python runtime:         1-2 seconds
    Module imports:         1-2 seconds
    Mapper initialization:  0.5-1 second
    ─────────────────────────────────────
    Total cold start:       2.5-5 seconds

```

