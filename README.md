# Agentic Workflow Automation & Tool Governance (Bedrock AgentCore + MCP)

[![AWS Bedrock](https://img.shields.io/badge/AWS-Bedrock_AgentCore-FF9900?logo=amazonaws)](https://aws.amazon.com/bedrock/)
[![MCP](https://img.shields.io/badge/Protocol-MCP-green)](https://modelcontextprotocol.io/)
[![Step Functions](https://img.shields.io/badge/AWS-Step_Functions-FF9900?logo=amazonaws)](https://aws.amazon.com/step-functions/)
[![Framework](https://img.shields.io/badge/Methodology-PMI--CPMAI-blue)](https://www.pmi.org/)

An enterprise agentic workflow architecture built on **Amazon Bedrock AgentCore** and **Model Context Protocol (MCP)** servers, featuring deterministic **Human-in-the-Loop (HITL)** governance via **AWS Step Functions**.

---

## 1. CPMAI Phase I: Matching AI to Business Needs

Following the **PMI Certified Professional in Managing AI (CPMAI) Phase I (Business Understanding)** methodology, this project evaluates the business feasibility and safety constraints of autonomous AI agents before enterprise deployment.

### 1.1 Business Objective & ROI Feasibility
* **Target Audience**: Enterprise Operations & Customer Support Operations.
* **Problem Statement**: Multi-step operational workflows (e.g., refunds, account modifications, order updates) require manual representative handling, leading to high operational friction and resolution delays. However, granting unconstrained write access to AI models creates extreme financial and security risks from prompt injection or parameter hallucination.
* **Projected Financial ROI**: Automating 40% of routine multi-step tasks while routing high-risk actions through deterministic human approval projects **\$1.2M in annual cost avoidance** and reduces average resolution time by 60%.

### 1.2 Cognitive vs. Non-Cognitive Justification
* **Why AI is Required (Probabilistic Need)**: Multi-step task planning, intent classification, and parameter extraction from unstructured customer interactions require probabilistic natural language processing that rigid, rule-based scripts cannot handle.
* **Non-Cognitive Integration (Deterministic Boundary)**: State-changing database actions (refunds, cancellations, privilege grants) are explicitly **decoupled** from model execution. Bedrock AgentCore proposes tool parameters via Model Context Protocol (MCP), but an **AWS Step Functions state machine** deterministically enforces Human-in-the-Loop (HITL) approval before any database write is committed.

### 1.3 AI Pattern Mapping
* **Primary Pattern**: **Autonomous Systems / Goal-Driven Systems** (dynamic multi-step task decomposition and tool selection via MCP).
* **Secondary Pattern**: **Conversational & Human Interaction** paired with **Patterns & Anomalies** (detecting unauthorized prompt injection attempts or parameter drift).

### 1.4 DIKUW Pyramid Alignment
* **Data (Base Facts)**: Unstructured customer support tickets, order records in DynamoDB, and transaction ledgers in Aurora.
* **Information (Structured Schemas)**: Tool definitions and API contracts standardized using Pydantic JSON schemas exposed over Model Context Protocol (MCP) servers.
* **Knowledge (Agentic Planning)**: Amazon Bedrock AgentCore mapping customer intents to dynamic tool execution plans.
* **Understanding & Governance (HITL Determinism)**: AWS Step Functions risk-evaluation engine enforcing human authorization boundaries whenever tool risk profiles transition from Read-Only to High-Risk Write.

### 1.5 CPMAI Go/No-Go Assessment (3x3 Feasibility Matrix)

| Feasibility Pillar | Assessment Criteria | Status | Strategic Justification |
| :--- | :--- | :---: | :--- |
| **Business Feasibility** | Problem Definition | 🟢 **GO** | High operational cost with clear safety boundary requirements defined. |
| | Sponsor Commitment | 🟢 **GO** | Operations leadership approves autonomous read-only tasks with mandatory HITL on writes. |
| | Sufficient ROI | 🟢 **GO** | Projected \$1.2M annual savings with significant cycle-time reduction. |
| **Data Feasibility** | Data Availability | 🟢 **GO** | Account, order, and ticket data accessible via secure microservice APIs. |
| | Access & Security | 🟢 **GO** | Strict IAM role policies isolate read tools from write tools. |
| | Data Quality | 🟢 **GO** | OpenAPI and MCP JSON schemas enforce strict input/output validation. |
| **Execution Feasibility** | Technology & Skills | 🟢 **GO** | Bedrock AgentCore, MCP, and Step Functions provide mature infrastructure. |
| | Implementation Timeline | 🟢 **GO** | Phased rollout: Read-only automated tools in Sprint 1; HITL writes in Sprint 2. |
| | Operational Context | 🟢 **GO** | Integrates seamlessly into existing admin dashboards via API Gateway & Webhooks. |

*Overall Assessment*: **ALL GREEN (GO)** — Project approved for technical implementation.


---

## 2. Target System Architecture

```mermaid
graph TD
    subgraph ClientAndTrigger ["1. Client & Trigger Tier"]
        AgentUser["Support Agent / System Event"]
        APIGW["AWS API Gateway"]
    end

    subgraph AgenticOrchestration ["2. Agentic Core & MCP Layer"]
        BedrockAgent["Amazon Bedrock AgentCore\n(Supervisor Agent)"]
        MCPServer["Model Context Protocol (MCP) Server\n(AWS Fargate / Lambda)"]
    end

    subgraph GovernanceAndHITL ["3. Security & Human-in-the-Loop Gating"]
        ToolRouter{"Tool Risk Assessment\n(Read-Only vs High-Risk Write)"}
        StepFunctions["AWS Step Functions\n(Deterministic HITL State Machine)"]
        SNSApproval["AWS SNS / Admin Email Approval"]
    end

    subgraph ExecutionAndStorage ["4. Enterprise Data & Execution Tier"]
        DynamoDB[("Amazon DynamoDB\n(Order / Ticket State)")]
        AuroraDB[("Amazon Aurora PostgreSQL\n(Customer Accounts)")]
    end

    subgraph TelemetryAndAudit ["5. Telemetry & Governance Logging"]
        CloudWatch["Amazon CloudWatch\n(Agent Action Logs & Cost Metrics)"]
        AuditTrail["S3 Immutable Audit Bucket\n(Prompt Injection & Tool Traces)"]
    end

    %% Execution Flow
    AgentUser -->|1. Submit Task Request| APIGW
    APIGW -->|2. Route to Supervisor| BedrockAgent
    BedrockAgent -->|3. Query Available Tools via MCP| MCPServer
    MCPServer -->|4. Propose Tool Execution| ToolRouter

    %% Risk Decision Routing
    ToolRouter -->|Low-Risk: Read-Only Query| DynamoDB
    ToolRouter -->|High-Risk: Refund / Status Change| StepFunctions

    %% HITL Approval Flow
    StepFunctions -->|Pause & Send Approval Request| SNSApproval
    SNSApproval -->|Human Approves Action| StepFunctions
    StepFunctions -->|Execute Authorized Write| AuroraDB
    ToolRouter -.->|Blocked / Prompt Injection Detected| AuditTrail

    %% Logging
    BedrockAgent -.->|Log Token Spend & Agent Trace| CloudWatch
