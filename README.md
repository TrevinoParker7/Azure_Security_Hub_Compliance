# Azure GRC Platform: A Comprehensive Beginner's Tutorial

<img width="2816" height="1536" alt="Gemini_Generated_Image_2e78kx2e78kx2e78" src="https://github.com/user-attachments/assets/5c509035-cad5-4e2c-ac77-3bf3fb0baea6" />

## Table of Contents
1. [Introduction](#introduction)
2. [Technology Overview](#technology-overview)
3. [High-Level Workflow](#high-level-workflow)
4. [Step-by-Step Implementation](#step-by-step-implementation)
5. [Detailed Code Review](#detailed-code-review)
6. [Key Concepts to Remember](#key-concepts-to-remember)
7. [Self-Review and Improvements](#self-review-and-improvements)

---

## Introduction

### What is This Project?

The **Azure GRC (Governance, Risk, Compliance) Platform** is an automated system that helps organizations track their security compliance. Think of it like a safety inspector for cloud systems:

- **What it does**: Automatically collects security problems found by Azure Defender (Azure's built-in security scanner)
- **Why it matters**: Maps those problems to compliance rules (like SOC 2 and NIST 800-53) so organizations know if they're meeting standards
- **How it helps**: Generates professional reports and emails them to managers and compliance officers

**Real-world analogy**: Imagine a hospital that must follow government health regulations. This system is like an automated inspector that checks hospital operations daily, identifies problems, and tells the hospital manager exactly which regulations are being violated and how to fix them.

### Why This Technology Stack?

We chose **Azure Functions** + **Python** + **Bicep Infrastructure-as-Code** because:

| Technology | Why It's Used |
|-----------|--------------|
| **Azure Functions** | Serverless computing—the code runs only when needed (on-demand or on schedule), so you pay only for what you use. No servers to manage. |
| **Python** | Excellent for data processing, easy to read, and perfect for compliance/reporting tasks. Fast to write and test. |
| **Azure Defender for Cloud** | Built into Azure; automatically scans cloud infrastructure for security problems. |
| **Bicep** | Infrastructure-as-Code language for Azure; lets you define cloud infrastructure as text, making it reproducible and version-controlled. |
| **OpenAI API** | Analyzes findings and generates human-readable compliance insights automatically. |
| **Azure Communication Services** | Handles secure email delivery to stakeholders. |

**Educational Purpose**: This is designed as a learning platform for GRC (Governance, Risk, Compliance) professionals to understand how compliance automation works while building portfolio artifacts.

---

## Technology Overview

### Core Technologies Explained

#### 1. **Python 3.9+**
- **What it is**: A programming language known for being readable and powerful
- **Our usage**: Main language for business logic (finding analysis, compliance mapping, report generation)
- **Why**: Easy to test, great for data processing, and widely used in security/compliance automation

#### 2. **Azure Functions**
- **What it is**: Serverless computing—code that runs on Azure's servers without you managing infrastructure
- **Our usage**: Two entry points:
  - **HTTP Trigger**: Call `/analyze` endpoint to run compliance analysis on-demand
  - **Timer Trigger**: Automatically run analysis on a schedule (like every hour)
- **Why**: Cost-effective, scalable, and integrates seamlessly with other Azure services

#### 3. **Azure Defender for Cloud**
- **What it is**: Azure's built-in security scanning service that continuously monitors your cloud infrastructure
- **Our usage**: Retrieves unhealthy security assessments (findings) and NIST 800-53 control status
- **Why**: Authoritative source of truth for security problems in your Azure environment

#### 4. **Bicep**
- **What it is**: Azure's Infrastructure-as-Code language—infrastructure defined as text
- **Our usage**: Defines the entire Azure infrastructure (Function App, databases, permissions)
- **Why**: Version-controlled infrastructure, reproducible deployments, and infrastructure reviews

#### 5. **OpenAI API**
- **What it is**: AI service that understands natural language and can analyze text
- **Our usage**: Analyzes security findings and generates compliance impact analysis automatically
- **Why**: Removes human bottleneck in compliance analysis; generates professional insights

#### 6. **Azure Communication Services**
- **What it is**: Azure's email/SMS service
- **Our usage**: Delivers compliance reports via email to stakeholders
- **Why**: Reliable, professional email delivery with proper authentication

#### 7. **pytest**
- **What it is**: Python testing framework
- **Our usage**: Unit tests, integration tests, and coverage reporting
- **Why**: Ensures code quality and prevents regressions

---

## High-Level Workflow

### How the System Works: End-to-End

```
┌─────────────────────────────────────────────────────────────────┐
│                    SYSTEM FLOW DIAGRAM                          │
└─────────────────────────────────────────────────────────────────┘

1. TRIGGER
   ├─ HTTP Request: POST /analyze?framework=SOC2&hours=24
   └─ Timer: Every hour (scheduled)
          │
          ▼

2. FINDINGS COLLECTION (analyzer.py)
   ├─ Query Azure Defender for Cloud
   ├─ Get "Unhealthy" security assessments
   ├─ Get NIST 800-53 control status (for cATO monitoring)
   ├─ Filter by framework type
   └─ Filter by time range (default 24 hours)
          │
          ▼

3. FRAMEWORK MAPPING (mapper_factory.py + mappers/)
   ├─ MapperFactory determines which mapper to use
   ├─ SOC2Mapper: Maps finding to SOC 2 controls
   ├─ NIST80053Mapper: Maps finding to NIST 800-53 controls
   └─ Results: List of findings + which controls they affect
          │
          ▼

4. ANALYSIS & INSIGHTS (reporter.py)
   ├─ Send findings to OpenAI for compliance impact analysis
   ├─ Generate human-readable explanations
   ├─ Create HTML email report with findings and controls
   └─ Export data to CSV format
          │
          ▼

5. DELIVERY (reporter.py)
   ├─ Send email via Azure Communication Services
   └─ Include HTML report + CSV attachment
          │
          ▼

6. STAKEHOLDER RECEIVES
   └─ Professional compliance report showing:
      ├─ Which controls are affected
      ├─ Compliance impact analysis
      ├─ Remediation recommendations
      └─ CSV for further analysis
```

### Step-by-Step Conceptual Flow

**Step 1: System Wakes Up**
- Either a person makes an HTTP request to `/analyze`, or the scheduled timer fires every hour
- Request includes: framework type (SOC2, NIST, or all) and time range

**Step 2: Collect Security Findings**
- Connect to your Azure environment using Azure credentials
- Query Azure Defender for Cloud: "What security problems do we have?"
- Azure Defender returns a list of unhealthy assessments (e.g., "MFA not enabled on all accounts")

**Step 3: Map Findings to Compliance Controls**
- The factory pattern creates the appropriate mapper (SOC2, NIST, etc.)
- Mapper reads JSON configuration files that define which findings map to which controls
- Example: "MFA finding" → maps to SOC 2 control "AC-2.1 User Authentication"

**Step 4: Analyze with AI**
- Send the mapped findings to OpenAI
- OpenAI generates: "This affects your ability to prove compliance with SOC 2 requirement X because..."
- Results come back as human-readable insights

**Step 5: Generate Report**
- Create HTML report with:
  - List of all findings
  - Which controls are affected
  - AI-generated compliance impact analysis
  - CSV export for further analysis

**Step 6: Send Email**
- Email the report to compliance officers and managers
- Uses Azure Communication Services for reliable delivery

---

## Step-by-Step Implementation

### Phase 1: Environment Setup

#### 1.1 Create Python Virtual Environment

A virtual environment is an isolated Python workspace. It prevents package conflicts.

```bash
# Navigate to your project directory
cd C:\Users\Elda\projects2\Azure_security-hub-compliance_CC

# Create virtual environment
python -m venv venv

# Activate it (on Windows)
venv\Scripts\activate

# On Mac/Linux:
# source venv/bin/activate

# You should see (venv) at the start of your terminal prompt
```

**What's happening**: Python creates a folder called `venv` containing its own copy of Python and an empty package list. This keeps your projects isolated.

#### 1.2 Install Dependencies

Dependencies are code libraries others wrote that your project needs.

```bash
# Install required packages
pip install -r azure_grc_platform/src/requirements.txt

# Install development tools (for testing and code quality)
pip install pytest pytest-cov flake8 black isort
```

**What's happening**: `pip` (Python's package manager) downloads libraries from PyPI (Python Package Index) and installs them into your virtual environment.

#### 1.3 Create Configuration Directories

The application needs a place to store compliance mappings.

```bash
# Create directories
mkdir -p config/mappings

# Copy initial mappings (they come with deployment templates)
cp deployment/config/mappings.json config/mappings/soc2_mappings.json
```

**What's happening**: You're setting up the directory structure the application expects. The mappings are JSON files that define how findings map to compliance controls.

#### 1.4 Set Up Azure Credentials

The application needs permission to access your Azure environment.

```bash
# Install Azure CLI if you don't have it
# Then authenticate
az login

# Set your subscription ID
$env:AZURE_SUBSCRIPTION_ID = "your-subscription-id-here"
```

**What's happening**: Azure CLI authenticates you with Azure. Azure Functions will use this credential context to access your resources securely.

### Phase 2: Understanding the Project Structure

```
azure_grc_platform/src/
├── function_app.py              # Entry point (HTTP + Timer triggers)
├── analyzer.py                  # Collects findings from Azure Defender
├── framework_mapper.py          # Base class for mappers (abstract pattern)
├── mapper_factory.py            # Creates the right mapper
├── reporter.py                  # Generates reports and sends email
├── utils.py                     # Helper functions
├── mappers/
│   ├── soc2_mapper.py          # Specific implementation for SOC 2
│   ├── nist_mapper.py          # Specific implementation for NIST
│   └── __init__.py
└── tests/                       # Test suite (critical for reliability)
    ├── conftest.py             # Test fixtures and mocks
    ├── test_app.py
    ├── test_app_findings.py
    └── ... (other test files)

config/
├── frameworks.json             # Defines what frameworks exist
└── mappings/
    ├── soc2_mappings.json       # SOC 2 control definitions
    └── nist800_53_mappings.json # NIST control definitions

deployment/
├── main.bicep                   # Infrastructure definition
└── config/
    └── mappings.json            # Initial mappings to copy
```

### Phase 3: Running Your First Test

Testing verifies your code works before deploying to production.

```bash
# Run all tests
pytest azure_grc_platform/src/ --cov=azure_grc_platform/src --cov-report=term-missing

# Run just one test file
pytest azure_grc_platform/src/tests/test_app.py -v

# Run tests matching a pattern
pytest azure_grc_platform/src/tests/ -k "soc2" -v
```

**What's happening**: 
- pytest discovers and runs all test files
- `--cov` generates coverage report (shows which lines of code are tested)
- `-v` gives verbose output
- `-k "pattern"` runs only tests whose names contain the pattern

**Expected output** (if tests pass):
```
===== test session starts =====
collected 15 items

test_app.py::test_analyzer_initialization PASSED     [ 6%]
test_app.py::test_findings_retrieval PASSED          [13%]
...
===== 15 passed in 1.23s =====
```

### Phase 4: Running the Application Locally

#### 4.1 Using Test Script

The project includes a script to test locally without Azure deployment.

```bash
# Run local test
python scripts/testing/test_locally.py
```

This script:
- Simulates an HTTP request to the `/analyze` endpoint
- Uses mock Azure data (not real Azure resources)
- Shows you what findings would be collected
- Displays the mapped controls

#### 4.2 Understanding the Output

The output shows:
1. **Findings collected**: List of security problems found
2. **Controls mapped**: Which compliance controls each finding affects
3. **Analysis generated**: AI-powered insights about compliance impact

### Phase 5: Code Quality Checks

Professional code needs to meet quality standards.

```bash
# Check for critical errors
flake8 azure_grc_platform/src/ --select=E9,F63,F7,F82

# Format code automatically
black azure_grc_platform/src/ scripts/

# Sort imports
isort azure_grc_platform/src/ scripts/ --profile black
```

**What's happening**:
- `flake8`: Checks for bugs and style violations
- `black`: Auto-formats code for consistency
- `isort`: Organizes imports alphabetically

### Phase 6: Deployment to Azure

#### 6.1 Validate Infrastructure Template

```bash
# Build and validate Bicep template
az bicep build --file deployment/main.bicep
```

#### 6.2 Create Deployment Package

```bash
# Package all application code
zip -r function-app.zip azure_grc_platform/src/*

# Deploy (full instructions in docs/DEPLOYMENT_GUIDE.md)
az deployment group create \
  --resource-group your-rg \
  --template-file deployment/main.bicep
```

---

## Detailed Code Review

### Architecture Pattern: The Factory Pattern

The **Factory Pattern** is a design pattern that creates objects based on parameters. It's like a restaurant menu where different meals are created based on what you order.

#### Why Use It?

Instead of this bad approach:
```python
# Bad: Hardcoded logic
if framework == "SOC2":
    mapper = SOC2Mapper()
elif framework == "NIST":
    mapper = NISTMapper()
elif framework == "ISO27001":
    mapper = ISO27001Mapper()
```

We use the factory pattern:
```python
# Good: Factory creates the right mapper
mapper = MapperFactory.get_mapper(framework)
```

**Benefits**:
- **Easy to extend**: Add new frameworks without modifying main code
- **Central location**: All mapper creation logic in one place
- **Testable**: Can mock the factory in tests
- **Maintainable**: Easier to understand at a glance

#### Code Example: mapper_factory.py

```python
from mappers.soc2_mapper import SOC2Mapper
from mappers.nist_mapper import NIST80053Mapper

class MapperFactory:
    """Creates appropriate mapper based on framework ID.
    
    This is the Factory Pattern: instead of the caller deciding
    which class to instantiate, the factory decides for them.
    """
    
    @staticmethod
    def get_mapper(framework_id, mappings_file=None):
        """
        Create a mapper based on framework ID.
        
        Args:
            framework_id: "SOC2", "NIST800-53", or "all"
            mappings_file: Custom mappings JSON file path
            
        Returns:
            FrameworkMapper subclass instance
        """
        mappers = {
            "SOC2": SOC2Mapper,
            "NIST800-53": NIST80053Mapper,
        }
        
        mapper_class = mappers.get(framework_id)
        if mapper_class is None:
            raise ValueError(f"Unknown framework: {framework_id}")
        
        return mapper_class(mappings_file)
```

`★ Insight ─────────────────────────────────────`
The factory pattern separates **object creation** from **object usage**. This is powerful because new frameworks can be added by just registering a new mapper class—no changes needed to the calling code. This pattern scales well as the number of compliance frameworks grows.
`─────────────────────────────────────────────────`

### Component 1: Findings Collection (analyzer.py)

**Purpose**: Retrieve security findings from Azure Defender for Cloud

**What it does**:
1. Authenticates to Azure using DefaultAzureCredential
2. Queries Defender for Cloud API
3. Filters findings by framework and time range
4. Returns structured list of findings

#### Code Breakdown

```python
from azure.identity import DefaultAzureCredential
from azure.mgmt.security import SecurityCenter

class SecurityAnalyzer:
    """Collects security findings from Azure Defender for Cloud."""
    
    def __init__(self, subscription_id):
        """
        Initialize the analyzer.
        
        Args:
            subscription_id: Azure subscription ID
        """
        # DefaultAzureCredential is smart—it tries multiple auth methods:
        # 1. Environment variables (for CI/CD)
        # 2. Azure CLI authentication (for local development)
        # 3. Managed Identity (for Azure Function App)
        self.credential = DefaultAzureCredential()
        
        # Create Azure Security Center client
        self.security_client = SecurityCenter(
            credential=self.credential,
            subscription_id=subscription_id
        )
    
    def get_unhealthy_assessments(self, hours=24):
        """
        Get security findings from Defender for Cloud.
        
        Args:
            hours: Look back this many hours for new findings
            
        Returns:
            List of Finding objects with:
            - title: Problem name (e.g., "MFA should be enabled")
            - severity: "High", "Medium", "Low"
            - description: Detailed explanation
            - resourceId: Which resource has the problem
        """
        # Calculate time range
        cutoff_time = datetime.now(timezone.utc) - timedelta(hours=hours)
        
        # Query the Security API for unhealthy assessments
        assessments = self.security_client.assessments.list()
        
        # Filter to unhealthy ones (assessment.status.code == "Unhealthy")
        findings = []
        for assessment in assessments:
            if assessment.status.code == "Unhealthy":
                if assessment.time_generated >= cutoff_time:
                    findings.append({
                        "title": assessment.display_name,
                        "severity": assessment.metadata.severity,
                        "description": assessment.metadata.description,
                        "resourceId": assessment.resource_details.resource_id,
                    })
        
        return findings
```

**Why this design**:
- **DefaultAzureCredential**: Flexible authentication—works locally and on Azure
- **Time filtering**: Only recent findings to avoid processing old issues
- **Structured output**: Findings are dictionaries with consistent keys

`★ Insight ─────────────────────────────────────`
Azure's `DefaultAzureCredential` is a brilliant design pattern itself. It tries authentication methods in order of specificity: local creds → environment vars → managed identity. This means the same code works in development, CI/CD, and production without changes.
`─────────────────────────────────────────────────`

### Component 2: Framework Mapping (mappers/)

**Purpose**: Map security findings to compliance controls

**Concept**: A finding (e.g., "MFA not enabled") might affect multiple compliance controls. This component defines those relationships.

#### The Base Class: framework_mapper.py

```python
from abc import ABC, abstractmethod
import json

class FrameworkMapper(ABC):
    """
    Abstract base class for compliance framework mappers.
    
    Each compliance framework (SOC2, NIST, ISO 27001) has different controls.
    This class defines the interface; subclasses implement specifics.
    
    Abstract = must be subclassed; can't be used directly.
    """
    
    def __init__(self, framework_name, mappings_file=None):
        """
        Initialize the mapper.
        
        Args:
            framework_name: "SOC2", "NIST800-53", etc.
            mappings_file: Path to JSON with control definitions
        """
        self.framework_name = framework_name
        
        # Load control mappings from JSON
        if mappings_file is None:
            mappings_file = f"config/mappings/{framework_name.lower()}_mappings.json"
        
        with open(mappings_file) as f:
            self.controls = json.load(f)
    
    @abstractmethod
    def map_findings(self, findings):
        """
        Map security findings to compliance controls.
        
        Each subclass must implement this.
        
        Args:
            findings: List of finding dictionaries
            
        Returns:
            List of {finding, controls_affected} tuples
        """
        pass
```

#### Concrete Implementation: soc2_mapper.py

```python
class SOC2Mapper(FrameworkMapper):
    """Maps security findings to SOC 2 controls."""
    
    def map_findings(self, findings):
        """
        Map findings to SOC 2 controls.
        
        SOC 2 has 5 trust principles:
        - CC: Criteria (covering 7 categories)
        - A: Availability & Processing Integrity
        - C: Confidentiality
        - I: Integrity
        
        Example control: "CC6.1 - Logical Access Controls"
        """
        findings_with_controls = []
        
        for finding in findings:
            # Find which SOC 2 controls this finding affects
            # Based on finding.title matching keywords in control mapping
            
            affected_controls = []
            finding_lower = finding["title"].lower()
            
            for control in self.controls:
                # Check if any keywords in the finding match this control
                keywords = control.get("keywords", [])
                
                if any(kw.lower() in finding_lower for kw in keywords):
                    affected_controls.append({
                        "control_id": control["id"],
                        "control_name": control["name"],
                        "category": control["category"],
                    })
            
            if affected_controls:
                findings_with_controls.append({
                    "finding": finding,
                    "controls": affected_controls,
                })
        
        return findings_with_controls
```

**How it works**:
1. Load SOC 2 control definitions from JSON
2. For each finding, search for keyword matches
3. Return finding + list of matched controls

**Example mapping logic**:
```
Finding: "Multi-factor authentication should be enabled"
↓
Keywords match: ["MFA", "authentication", "access"]
↓
Affected SOC 2 Controls:
  - CC6.1 (Logical Access Controls)
  - CC6.2 (User Authentication & Identification)
```

`★ Insight ─────────────────────────────────────`
The mapper uses a **configuration-driven** approach: control definitions are in JSON, not hardcoded. This means GRC professionals can adjust mappings without touching code. This is the power of separating **configuration** (what controls exist) from **logic** (how to match findings).
`─────────────────────────────────────────────────`

### Component 3: Report Generation (reporter.py)

**Purpose**: Generate professional compliance reports and send via email

**Key responsibilities**:
1. Send findings to OpenAI for analysis
2. Generate HTML report
3. Create CSV export
4. Send email via Azure Communication Services

#### Code Example: Generate Report

```python
import json
from openai import OpenAI
from azure.communication.email import EmailClient

class ComplianceReporter:
    """Generates compliance reports and sends via email."""
    
    def __init__(self, openai_key, email_endpoint):
        """Initialize with API credentials."""
        self.openai_client = OpenAI(api_key=openai_key)
        self.email_client = EmailClient.from_connection_string(email_endpoint)
    
    def analyze_finding_compliance_impact(self, finding, framework):
        """
        Use OpenAI to generate human-readable compliance analysis.
        
        Args:
            finding: Finding dictionary
            framework: "SOC2" or "NIST800-53"
            
        Returns:
            String: AI-generated analysis
        """
        # Prepare the prompt for OpenAI
        prompt = f"""
        I have a security finding in my {framework} compliance audit:
        
        Title: {finding['title']}
        Severity: {finding['severity']}
        Description: {finding['description']}
        
        Please explain in 2-3 sentences how this affects {framework} compliance.
        Include recommendations for remediation.
        """
        
        # Call OpenAI API
        response = self.openai_client.chat.completions.create(
            model="gpt-4",
            messages=[
                {"role": "user", "content": prompt}
            ],
            temperature=0.3  # Lower temperature = more focused, less creative
        )
        
        return response.choices[0].message.content
    
    def generate_html_report(self, findings_with_controls, framework):
        """
        Generate professional HTML email report.
        
        Args:
            findings_with_controls: Output from mapper
            framework: "SOC2" or "NIST800-53"
            
        Returns:
            String: HTML content
        """
        html = f"""
        <html>
        <head>
            <style>
                body {{ font-family: Arial, sans-serif; }}
                .finding {{ border: 1px solid #ddd; margin: 10px 0; padding: 10px; }}
                .severity-high {{ color: red; }}
                .severity-medium {{ color: orange; }}
                .severity-low {{ color: green; }}
            </style>
        </head>
        <body>
            <h1>{framework} Compliance Report</h1>
            <p>Generated: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}</p>
            <p>Total Findings: {len(findings_with_controls)}</p>
            
            <h2>Detailed Findings</h2>
        """
        
        for item in findings_with_controls:
            finding = item["finding"]
            controls = item["controls"]
            
            html += f"""
            <div class="finding">
                <h3>{finding['title']}</h3>
                <p class="severity-{finding['severity'].lower()}">
                    Severity: {finding['severity']}
                </p>
                <p>{finding['description']}</p>
                <h4>Affected Controls:</h4>
                <ul>
            """
            
            for control in controls:
                html += f"<li>{control['control_id']}: {control['control_name']}</li>"
            
            html += """
                </ul>
            </div>
            """
        
        html += """
        </body>
        </html>
        """
        
        return html
    
    def send_email(self, recipient_email, subject, html_content, csv_attachment):
        """
        Send report via Azure Communication Services.
        
        Args:
            recipient_email: Who to send to
            subject: Email subject line
            html_content: HTML report body
            csv_attachment: CSV file content
        """
        email_message = {
            "senderAddress": "noreply@yourdomain.com",
            "recipients": {
                "to": [{"address": recipient_email}]
            },
            "content": {
                "subject": subject,
                "html": html_content,
            },
        }
        
        # Send the email
        poller = self.email_client.begin_send(email_message)
        result = poller.result()
        
        return result.status
```

**How it works**:
1. **OpenAI Analysis**: For each finding, generate professional compliance impact text
2. **HTML Report**: Create a styled HTML email with findings and controls
3. **Email Delivery**: Send via Azure Communication Services

`★ Insight ─────────────────────────────────────`
The reporter separates concerns beautifully: **analysis** (OpenAI), **formatting** (HTML generation), and **delivery** (email). This makes testing easier—you can test HTML generation without calling email APIs. It also makes the code reusable: someone could call just the HTML generator or just the OpenAI analyzer.
`─────────────────────────────────────────────────`

### Component 4: Azure Function Entry Points (function_app.py)

**Purpose**: Define how external requests trigger the system

Azure Functions have two types of triggers:
- **HTTP Trigger**: Respond to web requests (on-demand)
- **Timer Trigger**: Run on a schedule (automated)

```python
import azure.functions as func
from analyzer import SecurityAnalyzer
from mapper_factory import MapperFactory
from reporter import ComplianceReporter

# Create the Function App
app = func.FunctionApp()

@app.route(route="analyze", methods=["GET", "POST"])
def http_trigger(req: func.HttpRequest) -> func.HttpResponse:
    """
    HTTP endpoint: POST or GET to /analyze
    
    Query parameters:
    - framework: "SOC2", "NIST800-53", or "all" (default: all)
    - hours: Look back this many hours (default: 24)
    - recipient: Email address to send report to
    
    Example:
    POST /analyze?framework=SOC2&hours=24&recipient=manager@company.com
    """
    try:
        # Parse query parameters
        framework = req.params.get("framework", "all")
        hours = int(req.params.get("hours", "24"))
        recipient = req.params.get("recipient")
        
        # Step 1: Collect findings
        analyzer = SecurityAnalyzer(subscription_id=os.environ["AZURE_SUBSCRIPTION_ID"])
        findings = analyzer.get_unhealthy_assessments(hours=hours)
        
        # Step 2: Map findings
        mapper = MapperFactory.get_mapper(framework)
        findings_with_controls = mapper.map_findings(findings)
        
        # Step 3: Generate report
        reporter = ComplianceReporter(
            openai_key=os.environ["OPENAI_API_KEY"],
            email_endpoint=os.environ["COMMUNICATION_SERVICES_ENDPOINT"]
        )
        html_report = reporter.generate_html_report(findings_with_controls, framework)
        
        # Step 4: Send email
        if recipient:
            reporter.send_email(
                recipient_email=recipient,
                subject=f"{framework} Compliance Report",
                html_content=html_report,
                csv_attachment=None
            )
            return func.HttpResponse(
                "Report generated and sent successfully",
                status_code=200
            )
        
        return func.HttpResponse(html_report, status_code=200, mimetype="text/html")
    
    except Exception as e:
        return func.HttpResponse(
            f"Error: {str(e)}",
            status_code=500
        )


@app.schedule_trigger(schedule="0 */1 * * * *")  # Every hour
def timer_trigger(mytimer: func.TimerRequest) -> None:
    """
    Timer trigger: Runs automatically every hour.
    
    Collects findings, analyzes them, and sends email to configured recipient.
    """
    # Similar to HTTP trigger, but without request parameter parsing
    # Uses environment variables for recipient
    
    analyzer = SecurityAnalyzer(os.environ["AZURE_SUBSCRIPTION_ID"])
    findings = analyzer.get_unhealthy_assessments(hours=1)
    
    # ... rest of analysis pipeline
    
    logging.info(f"Timer trigger completed at {utcnow}")
```

**Key concepts**:
- **Query Parameters**: `?framework=SOC2&hours=24` → extracted from `req.params`
- **Error Handling**: Try/except catches failures and returns 500 error
- **Schedule Expression**: `0 */1 * * * *` = every hour (cron syntax)

`★ Insight ─────────────────────────────────────`
Azure Functions use **decorators** (the `@app.route` syntax) to define triggers. This is elegant Python—the framework discovers function entry points by reading decorators rather than requiring configuration files. This approach scales well and keeps code and configuration together.
`─────────────────────────────────────────────────`

---

## Key Concepts to Remember

### 1. The Compliance Mapping Pattern

**Concept**: Security findings (technical problems) need to be connected to compliance controls (business rules).

**How it works**:
```
Azure Finding                    SOC 2 Control
──────────────              ──────────────────
MFA not enabled      ──→    CC6.2: User 
                            Authentication
```

**Why it matters**: Executives need to understand security problems in business terms. Instead of "EnableMFAOnAdminAccount" (technical), they understand "Affects SOC 2 control CC6.2" (business).

### 2. The Factory Pattern Benefits

Instead of: "If SOC2, use SOC2Mapper. If NIST, use NIST Mapper..."

We use: "Let the factory decide which mapper to use."

**Benefits**:
- Adding a new framework (ISO 27001) requires only creating a new mapper class
- Calling code never changes
- Tests can easily mock the factory

### 3. Configuration Over Code

**Principle**: Important decisions should be in configuration files (JSON), not hardcoded in Python.

**Example**:
```json
{
  "controls": [
    {
      "id": "CC6.1",
      "name": "Logical Access Controls",
      "keywords": ["access", "authentication", "permission"]
    }
  ]
}
```

**Why**:
- Non-developers (GRC professionals) can update mappings
- Changes don't require redeployment
- Easy to version-control different mapping strategies

### 4. Serverless Architecture Benefits

**Traditional approach**: "Pay for server running 24/7, even when not analyzing"

**Serverless approach**: "Pay only when function executes"

**Real cost example**:
- 1 hour analysis per day = pay for 1 hour
- Traditional: pay for 24 hours

### 5. Environment-Based Configuration

**Pattern**: Use environment variables for secrets and deployment-specific values.

```python
openai_key = os.environ["OPENAI_API_KEY"]  # ✓ Good
openai_key = "sk-abc123..."                 # ✗ Bad (exposed!)
```

**Why**:
- Secrets never in code
- Different deployments can have different values
- CI/CD systems can inject secrets securely

### 6. The Testing Safety Net

**Without tests**: Change code → Hope it still works → Deploy to production → 💥

**With tests**: Change code → Run tests → They fail ✓ (caught early) or pass ✓ (confident) → Deploy

**Test types**:
- **Unit tests**: Test one function in isolation
- **Integration tests**: Test multiple components working together
- **Mocking**: Replace real Azure services with fake ones for fast, repeatable tests

---

## Self-Review and Improvements

Here are **five concrete recommendations** for improving this codebase:

### 1. **Add Comprehensive Error Handling & Logging**

**Current Issue**: When something fails (Azure API timeout, OpenAI API error), users see generic "500 Error" messages.

**Recommendation**:
- Log all significant operations (findings retrieval started, mapping complete, email sent)
- Create specific exception types for different failures
- Return meaningful error messages to users

**Example implementation**:
```python
import logging

logger = logging.getLogger(__name__)

class FindingsRetrievalError(Exception):
    """Raised when Azure Defender API call fails."""
    pass

try:
    findings = analyzer.get_unhealthy_assessments()
    logger.info(f"Retrieved {len(findings)} findings")
except Exception as e:
    logger.error(f"Failed to retrieve findings: {str(e)}", exc_info=True)
    raise FindingsRetrievalError("Unable to connect to Azure Defender")
```

**Benefits**:
- Users understand what went wrong
- Operations teams can debug with logs
- Easier to monitor production health

---

### 2. **Implement Caching for Configuration**

**Current Issue**: Every HTTP request reads JSON mapping files from disk.

**Recommendation**:
- Cache loaded mappings in memory with a 1-hour TTL
- Reduce disk I/O and improve response time

**Example implementation**:
```python
from functools import lru_cache
import time

class MapperFactory:
    _cache = {}
    _cache_time = {}
    CACHE_TTL = 3600  # 1 hour
    
    @staticmethod
    def get_mapper(framework_id, mappings_file=None):
        now = time.time()
        
        # Check if cached and not expired
        if framework_id in MapperFactory._cache:
            if now - MapperFactory._cache_time[framework_id] < MapperFactory.CACHE_TTL:
                return MapperFactory._cache[framework_id]
        
        # Not cached, create new mapper
        mapper = _create_mapper(framework_id, mappings_file)
        MapperFactory._cache[framework_id] = mapper
        MapperFactory._cache_time[framework_id] = now
        
        return mapper
```

**Benefits**:
- Faster response times (no disk reads)
- Lower Azure Function CPU usage
- Still picks up configuration changes after 1 hour

---

### 3. **Add Request Validation & Rate Limiting**

**Current Issue**: Anyone can call `/analyze` unlimited times. No validation of inputs.

**Recommendation**:
- Validate input parameters (framework must be valid, hours must be positive)
- Add rate limiting per user/API key
- Prevent abuse and bad data

**Example implementation**:
```python
from azure.functions import HttpRequest

class RequestValidator:
    VALID_FRAMEWORKS = {"SOC2", "NIST800-53", "all"}
    MAX_HOURS = 90
    MAX_REQUESTS_PER_HOUR = 10
    
    @staticmethod
    def validate(req: HttpRequest) -> tuple[bool, str]:
        """
        Validate request parameters.
        
        Returns:
            (is_valid, error_message)
        """
        framework = req.params.get("framework", "all")
        if framework not in RequestValidator.VALID_FRAMEWORKS:
            return False, f"Invalid framework. Must be one of {RequestValidator.VALID_FRAMEWORKS}"
        
        hours = req.params.get("hours", "24")
        try:
            hours_int = int(hours)
            if hours_int <= 0 or hours_int > RequestValidator.MAX_HOURS:
                return False, f"Hours must be between 1 and {RequestValidator.MAX_HOURS}"
        except ValueError:
            return False, "Hours must be a number"
        
        return True, ""

# In HTTP trigger:
is_valid, error_msg = RequestValidator.validate(req)
if not is_valid:
    return func.HttpResponse(error_msg, status_code=400)
```

**Benefits**:
- Prevents invalid data from corrupting reports
- Protects against accidental abuse
- Better error messages for users

---

### 4. **Implement Report Caching & Versioning**

**Current Issue**: Generating the same report twice costs time and money (OpenAI API calls).

**Recommendation**:
- Cache generated reports (findings + controls mapping) for 30 minutes
- Generate new report only if findings actually changed
- Include report version/timestamp for audit trails

**Example implementation**:
```python
import hashlib
import json
from datetime import datetime, timedelta

class ReportCache:
    def __init__(self, ttl_minutes=30):
        self._cache = {}
        self.ttl = timedelta(minutes=ttl_minutes)
    
    def get_cache_key(self, findings, framework):
        """Generate cache key from findings hash and framework."""
        findings_json = json.dumps(findings, sort_keys=True)
        findings_hash = hashlib.sha256(findings_json.encode()).hexdigest()
        return f"{framework}:{findings_hash}"
    
    def get(self, cache_key):
        """Get cached report if exists and not expired."""
        if cache_key in self._cache:
            cached_data, timestamp = self._cache[cache_key]
            if datetime.now() - timestamp < self.ttl:
                return cached_data
        return None
    
    def set(self, cache_key, report_data):
        """Cache a report."""
        self._cache[cache_key] = (report_data, datetime.now())

# In reporter:
cache_key = report_cache.get_cache_key(findings, framework)
cached_report = report_cache.get(cache_key)

if cached_report:
    logger.info("Using cached report")
    html_report = cached_report
else:
    logger.info("Generating new report")
    html_report = generate_report_with_openai(findings, framework)
    report_cache.set(cache_key, html_report)
```

**Benefits**:
- Saves money (fewer OpenAI API calls)
- Faster response times (cached reports return instantly)
- Reduced load on Azure services

---

### 5. **Add Telemetry & Monitoring Dashboard**

**Current Issue**: No visibility into system health. Is it working? How many findings per day?

**Recommendation**:
- Send metrics to Azure Monitor (built-in to Azure)
- Track: findings per framework, email delivery success, processing time
- Create dashboard for operations team

**Example implementation**:
```python
from azure.monitor.opentelemetry import AzureMonitorTracer
from opentelemetry import metrics

# Initialize tracer
tracer = AzureMonitorTracer()
meter = metrics.get_meter(__name__)

# Define metrics
findings_counter = meter.create_counter(
    "grc.findings.count",
    unit="1",
    description="Total findings retrieved"
)
email_counter = meter.create_counter(
    "grc.email.sent",
    unit="1",
    description="Emails sent successfully"
)
processing_time = meter.create_histogram(
    "grc.processing.duration_ms",
    unit="ms",
    description="Time to process findings"
)

# In analyzer:
findings_counter.add(len(findings), {"framework": framework})

# In reporter:
email_counter.add(1, {"status": "success"})
processing_time.record(processing_duration_ms)
```

**Dashboard would show**:
- Graph: Findings over time
- Graph: Email delivery success rate
- Alert: If no findings for 48 hours (system might be broken)

**Benefits**:
- Early detection of problems
- Understand usage patterns
- Prove value to stakeholders ("Blocked 15 compliance violations this month")

---

## Conclusion

You now have a comprehensive understanding of the Azure GRC Platform:

1. **Architecture**: How findings flow from Azure Defender → mapped to controls → reported to stakeholders
2. **Design Patterns**: Factory pattern for extensibility, configuration-driven mappings
3. **Technology Stack**: Why each technology was chosen
4. **Testing**: How to verify the system works
5. **Improvements**: Concrete ways to enhance the codebase

**Next steps**:
- Implement the 5 improvements above (start with error handling)
- Deploy to Azure and monitor real-world usage
- Gather feedback from GRC professionals
- Add more compliance frameworks (ISO 27001, HIPAA)

This project demonstrates **professional software engineering**:
- Clear separation of concerns (analyzer, mapper, reporter)
- Testability through mocking and dependency injection
- Configuration-driven decisions
- Scalable architecture (serverless)
- Extensibility (add frameworks without changing core code)

Good luck building your GRC platform! 🚀
