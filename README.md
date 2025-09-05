# EC2 AI Agent Enhancement System - ServiceNow Implementation

## System Overview

I successfully enhanced Netflix's existing manual EC2 remediation system (WL2) with AI Agent capabilities that enable DevOps engineers to use conversational AI for incident response and EC2 remediation. This enhancement addresses the bottleneck experienced during peak streaming hours when multiple EC2 instances fail simultaneously and engineers must manually parse incident descriptions, identify instance IDs, and navigate to the correct records.

The enhanced system maintains the proven manual remediation workflow while adding an intelligent conversational interface powered by ServiceNow AI Agent Studio. DevOps engineers can now choose between the traditional UI Action approach or natural language conversations with the AI Agent. Both approaches utilize the same AWS Integration Server API and create identical remediation logs, ensuring consistency and audit trails.

The AI Agent enhancement was implemented after the launch incident where 5 EC2 instances failed within 10 minutes across different regions. Manual response took 15 minutes, but the AI Agent can analyze all incidents simultaneously and provide intelligent remediation guidance through natural conversation, reducing response time to under 3 minutes while maintaining human approval workflows for safety.

## Implementation Steps

### 1. AI Agent Creation

**AI Agent Configuration:**

* **Name:** "EC2 Remediation Assistant"
* **Description:** "Helps DevOps engineers restart EC2 instances from incident tickets by reading incident descriptions and executing remediation scripts with human approval"
* **Role:** EC2 remediation specialist for DevOps operations
* **Instructions:** Configured to read incident descriptions, identify EC2 instance IDs, request human approval, and execute remediation using existing API integration

### 2. Script Tool Development

**Script Tool Configuration:**

* Adapted existing `EC2RemediationHelper` logic for conversational interface
* **Input Schema:** Single `instance_id` parameter with LLM-friendly description
* **Execution Mode:** Supervised (requires human approval before execution)
* **Integration:** Maintains identical API calls and logging to existing manual system
* **Key Adaptation:** Translates between natural language input and database structure while preserving all existing functionality

### 3. Agent Tool Integration

**Tool Relationship Configuration:**

* System automatically created `sn_aia_agent_tool_m2m` relationship records
* Configured tool parameters for instance_id input and human-readable responses
* Verified agent can successfully call script tool and request human approval
* Tested integration with existing AWS Integration Server API

### 4. Conversational Testing

**Test Scenarios Completed:**

* **Direct Instance Requests:** "Restart instance i-09ae69f1cb71f622e"
* **Incident-Based Requests:** "Help me solve incident INC0001234"
* **Invalid Input Handling:** Tested malformed instance IDs and error responses
* **Approval Workflow:** Verified human approval requirements before execution

### 5. Integration Verification

**System Consistency Validation:**

* Confirmed AI Agent creates identical entries in Remediation Log table as manual system
* Verified same AWS Integration Server API calls for both approaches
* Tested both manual UI Action and AI Agent on same instances
* Validated execution plan tracking and task completion logging

### 6. Update Set Component Collection

**Components Captured:**

* AI Agent definition from `sn_aia_agent` table
* Agent-tool relationships from `sn_aia_agent_tool_m2m` table
* Execution plans from `sn_aia_execution_plan` table
* Associated execution tasks demonstrating successful remediation
* All existing data layer and LLM integration components from WL2

## Architecture Diagram

![EC2 AI Agent Enhancement Architecture]()

The enhanced architecture diagram shows both the existing manual remediation workflow and the new AI Agent conversational interface. Both approaches converge at the AWS Integration Server API, ensuring identical remediation outcomes and logging. The diagram illustrates decision points for when DevOps engineers should use manual UI Actions versus conversational AI remediation based on incident complexity and urgency.

Key architectural components:

* **Manual Path:** EC2 Instance record → UI Action → RemediationHelper → AWS API
* **AI Agent Path:** Natural language input → AI Agent → Script Tool → AWS API
* **Shared Components:** AWS Integration Server, Remediation Log table, incident creation workflow
* **Decision Logic:** Simple single-instance failures use manual approach, complex multi-instance scenarios leverage AI Agent

## Optimization

### Script Comparison Analysis

**Manual System vs AI Agent Approach:**

| Aspect                     | Manual System                 | AI Agent Enhancement            |
| -------------------------- | ----------------------------- | ------------------------------- |
| **Input Method**     | sys_id from form context      | instance_id from conversation   |
| **Lookup Direction** | Direct GlideRecord get        | Query by instance_id field      |
| **Return Format**    | JavaScript object             | JSON string for LLM consumption |
| **Error Handling**   | Form-based user feedback      | Conversational error messages   |
| **Human Approval**   | Implicit (user clicks button) | Explicit approval workflow      |

**When to Use Each Approach:**

**Manual UI Action - Best For:**

* Single instance failures with known record location
* Routine maintenance and planned restarts
* Engineers familiar with ServiceNow navigation
* Simple remediation scenarios requiring quick action

**AI Agent Conversation - Best For:**

* Multiple instance failures across regions
* Incident-driven scenarios requiring instance ID identification
* Complex troubleshooting requiring conversational guidance
* New team members learning remediation procedures

### Performance Optimizations

**AI Agent Efficiency:**

* Configured supervised execution to balance automation with safety
* Implemented efficient instance ID validation before API calls
* Designed conversational flow to minimize back-and-forth interactions
* Optimized script tool to reuse existing RemediationHelper logic

**Integration Performance:**

* Maintained identical AWS API timeout and retry configurations
* Preserved all existing error handling and logging mechanisms
* Ensured no performance degradation to manual workflow
* Created execution plan tracking for conversation audit trails

### System Reliability

**Enhanced Error Handling:**

* AI Agent provides conversational error explanations
* Maintains all existing API error handling from manual system
* Execution plans track conversation success/failure status
* Human approval prevents unauthorized remediation attempts

## DevOps Usage

### For Netflix DevOps Engineers:

### Manual Remediation (Existing Workflow)

**Use When:** Single instance failure, routine maintenance, quick action needed

1. Navigate to EC2 Instance record showing "Off" status
2. Click "Trigger EC2 Remediation" button
3. Review confirmation and remediation log entry
4. Monitor instance status update from AWS Integration Server

### AI Agent Conversational Remediation (New Enhancement)

**Use When:** Multiple failures, incident-driven scenarios, need guidance

**Starting a Conversation:**

1. Access AI Agent Studio or designated chat interface
2. Begin with incident context: "Help me with incident INC0001234"
3. Or direct instance request: "Restart instance i-09ae69f1cb71f622e"

**Conversation Flow:**

1. **Agent Analysis:** AI reads incident description and identifies instance IDs
2. **Human Verification:** Agent requests confirmation before remediation
3. **Approval Process:** Engineer approves or modifies remediation plan
4. **Execution:** Agent executes same remediation API as manual system
5. **Results:** Agent provides conversational status updates and next steps

**Monitoring Results:**

* Check Remediation Log table for identical logging as manual approach
* Review execution plans in `sn_aia_execution_plan` for conversation tracking
* Monitor execution tasks for detailed step-by-step remediation audit trail
* Verify incident updates and Slack notifications continue as normal

### Decision Matrix for Approach Selection

| Scenario                 | Recommended Approach  | Reason                       |
| ------------------------ | --------------------- | ---------------------------- |
| Single instance restart  | Manual UI Action      | Fastest for known instances  |
| Multiple region failures | AI Agent Conversation | Handles complexity better    |
| Incident-driven response | AI Agent Conversation | Parses incident context      |
| Routine maintenance      | Manual UI Action      | Simple, direct approach      |
| Training new engineers   | AI Agent Conversation | Provides guided assistance   |
| Emergency response       | Both (situational)    | Use fastest available method |

### Escalation Procedures

**For Both Approaches:**

* If remediation fails, review Remediation Log error details
* Check execution plans (AI Agent) or system logs (manual) for troubleshooting
* Escalate to AWS infrastructure team if multiple attempts fail
* Use existing incident management procedures for complex failures

**AI Agent Specific:**

* Review conversation history for context understanding
* Check execution plan objectives for proper incident interpretation
* Verify human approval workflow completed successfully
* Escalate to ServiceNow team if agent tool integration fails
