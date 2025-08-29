# ServiceNow MCP Change Management Tools

This document provides information about the change management tools available in the ServiceNow MCP server.

## Overview

The change management tools allow Claude to interact with ServiceNow's change management functionality, enabling users to create, update, and manage change requests through natural language conversations.

**Note:** The tools use different schemas for creation (basic fields) and updates (comprehensive fields including planning and communication details) to match typical ServiceNow workflows.

## Change Request Identifiers

**Important:** The change management tools now support both ServiceNow identifier formats:

### Display Number (Recommended)
- **Format**: Human-readable format like `CHG0030001`, `CHG0001234`
- **Use Case**: Primary identifier for user input and communication
- **Example**: "Update change request CHG0030001 to add implementation notes"

### Sys ID
- **Format**: 32-character hexadecimal identifier like `a4fb2cae47a7261065fda464116d43c2`
- **Use Case**: Internal ServiceNow identifier, typically used in API responses
- **Example**: "Get details for change request a4fb2cae47a7261065fda464116d43c2"

### Automatic Detection
All change management functions automatically detect the identifier format and use the appropriate ServiceNow API endpoint:
- **Display Number**: Uses query endpoint with `?sysparm_query=number={display_number}`
- **Sys ID**: Uses direct endpoint with `/{sys_id}`

This means you can use either format interchangeably when calling any change management tool.

## Schema Differences

### Create vs Update Schemas
The change management tools use different schemas for creation and updates to match typical ServiceNow workflows:

**Create Schema (Basic Fields):**
- Essential fields for initial change request creation
- Focuses on basic information needed to start the change process
- Fields: short_description, type, description, risk, impact, category, requested_by, assignment_group, start_date, end_date

**Update Schema (Comprehensive Fields):**
- All fields including planning and communication details
- Allows comprehensive updates as the change progresses through its lifecycle
- Additional fields: risk_impact_analysis, comments, work_notes, justification, implementation_plan, backout_plan, test_plan

### Planning and Communication Fields (Update Only)

#### Risk and Impact Analysis
- **Field**: `risk_impact_analysis`
- **Location**: Planning tab in ServiceNow UI
- **Purpose**: Detailed analysis of risks and impacts associated with the change
- **Visibility**: Internal staff only
- **Usage**: Available only during updates, not during creation

#### Additional Planning Fields
- **Field**: `justification`
  - **Location**: Planning tab in ServiceNow UI
  - **Purpose**: Justification for the change request
  - **Visibility**: Internal staff only

- **Field**: `implementation_plan`
  - **Location**: Planning tab in ServiceNow UI
  - **Purpose**: Detailed implementation plan for the change
  - **Visibility**: Internal staff only

- **Field**: `backout_plan`
  - **Location**: Planning tab in ServiceNow UI
  - **Purpose**: Plan to rollback the change if needed
  - **Visibility**: Internal staff only

- **Field**: `test_plan`
  - **Location**: Planning tab in ServiceNow UI
  - **Purpose**: Testing strategy and plan for the change
  - **Visibility**: Internal staff only

#### Comments and Work Notes
- **Comments Field**: `comments`
  - **Location**: Notes tab in ServiceNow UI
  - **Purpose**: Public comments visible to customers
  - **Use Case**: Customer communication, status updates
  
- **Work Notes Field**: `work_notes`
  - **Location**: Notes tab in ServiceNow UI (when "Work notes" checkbox is checked)
  - **Purpose**: Internal notes visible only to staff
  - **Use Case**: Internal communication, technical details, troubleshooting notes

### Field Usage Examples
```python
# Create change request (basic fields only)
{
    "short_description": "Server maintenance",
    "type": "normal",
    "description": "Routine server maintenance and updates",
    "risk": "low",
    "impact": "medium"
}

# Update change request with planning and communication fields
{
    "change_id": "CHG0030007",
    "risk_impact_analysis": "Updated risk assessment based on testing",
    "comments": "Testing completed successfully",
    "work_notes": "Internal: All test cases passed, ready for production",
    "justification": "Security patches required for compliance",
    "implementation_plan": "Apply patches during maintenance window",
    "backout_plan": "Rollback to previous version if issues arise",
    "test_plan": "Test in staging environment before production"
}
```

### Important Note on Field Names
The field names used in the API correspond to ServiceNow's internal field names. If you experience issues with certain fields not being populated, please verify the exact field names in your ServiceNow instance's table schema or contact your ServiceNow administrator.

## Available Tools

The ServiceNow MCP server provides the following change management tools:

### Core Change Request Management

1. **create_change_request** - Create a new change request in ServiceNow
   - Parameters:
     - `short_description` (required): Short description of the change request
     - `description`: Detailed description of the change request
     - `type` (required): Type of change (normal, standard, emergency)
     - `risk`: Risk level of the change
     - `impact`: Impact of the change
     - `category`: Category of the change
     - `requested_by`: User who requested the change
     - `assignment_group`: Group assigned to the change
     - `start_date`: Planned start date (YYYY-MM-DD HH:MM:SS)
     - `end_date`: Planned end date (YYYY-MM-DD HH:MM:SS)

2. **update_change_request** - Update an existing change request
   - Parameters:
     - `change_id` (required): Change request display number (e.g., CHG0030001) or sys_id (32-character hex)
     - `short_description`: Short description of the change request
     - `description`: Detailed description of the change request
     - `state`: State of the change request
     - `risk`: Risk level of the change
     - `impact`: Impact of the change
     - `category`: Category of the change
     - `assignment_group`: Group assigned to the change
     - `start_date`: Planned start date (YYYY-MM-DD HH:MM:SS)
     - `end_date`: Planned end date (YYYY-MM-DD HH:MM:SS)
     - `risk_impact_analysis`: Risk and impact analysis for the change
     - `comments`: Public comments visible to customers
     - `work_notes`: Internal work notes visible only to staff
     - `justification`: Justification for the change request
     - `implementation_plan`: Implementation plan for the change
     - `backout_plan`: Backout plan for the change
     - `test_plan`: Test plan for the change

3. **list_change_requests** - List change requests with filtering options
   - Parameters:
     - `limit`: Maximum number of records to return (default: 10)
     - `offset`: Offset to start from (default: 0)
     - `state`: Filter by state
     - `type`: Filter by type (normal, standard, emergency)
     - `category`: Filter by category
     - `assignment_group`: Filter by assignment group
     - `timeframe`: Filter by timeframe (upcoming, in-progress, completed)
     - `query`: Additional query string

4. **get_change_request_details** - Get detailed information about a specific change request
   - Parameters:
     - `change_id` (required): Change request display number (e.g., CHG0030001) or sys_id (32-character hex)

5. **add_change_task** - Add a task to a change request
   - Parameters:
     - `change_id` (required): Change request display number (e.g., CHG0030001) or sys_id (32-character hex)
     - `short_description` (required): Short description of the task
     - `description`: Detailed description of the task
     - `assigned_to`: User assigned to the task
     - `planned_start_date`: Planned start date (YYYY-MM-DD HH:MM:SS)
     - `planned_end_date`: Planned end date (YYYY-MM-DD HH:MM:SS)

### Change Approval Workflow

1. **submit_change_for_approval** - Submit a change request for approval
   - Parameters:
     - `change_id` (required): Change request display number (e.g., CHG0030001) or sys_id (32-character hex)
     - `approval_comments`: Comments for the approval request

2. **approve_change** - Approve a change request
   - Parameters:
     - `change_id` (required): Change request display number (e.g., CHG0030001) or sys_id (32-character hex)
     - `approver_id`: ID of the approver
     - `approval_comments`: Comments for the approval

3. **reject_change** - Reject a change request
   - Parameters:
     - `change_id` (required): Change request display number (e.g., CHG0030001) or sys_id (32-character hex)
     - `approver_id`: ID of the approver
     - `rejection_reason` (required): Reason for rejection

## Example Usage with Claude

Once the ServiceNow MCP server is configured with Claude Desktop, you can ask Claude to perform actions like:

### Creating and Managing Change Requests

- "Create a change request for server maintenance to apply security patches tomorrow night"
- "Schedule a database upgrade for next Tuesday from 2 AM to 4 AM"
- "Create an emergency change to fix the critical security vulnerability in our web application"

### Adding Tasks and Implementation Details

- "Add a task to the server maintenance change for pre-implementation checks"
- "Add a task to verify system backups before starting the database upgrade"
- "Update the implementation plan for the network change to include rollback procedures"

### Approval Workflow

- "Submit the server maintenance change for approval"
- "Show me all changes waiting for my approval"
- "Approve the database upgrade change with comment: implementation plan looks thorough"
- "Reject the network change due to insufficient testing"

### Querying Change Information

- "Show me all emergency changes scheduled for this week"
- "What's the status of the database upgrade change?"
- "List all changes assigned to the Network team"
- "Show me the details of change CHG0010001"

## Example Code

Here's an example of how to use the change management tools programmatically:

```python
from servicenow_mcp.auth.auth_manager import AuthManager
from servicenow_mcp.tools.change_tools import create_change_request
from servicenow_mcp.utils.config import ServerConfig

# Create server configuration
server_config = ServerConfig(
    instance_url="https://your-instance.service-now.com",
)

# Create authentication manager
auth_manager = AuthManager(
    auth_type="basic",
    username="your-username",
    password="your-password",
    instance_url="https://your-instance.service-now.com",
)

# Create a change request
create_params = {
    "short_description": "Server maintenance - Apply security patches",
    "description": "Apply the latest security patches to the application servers.",
    "type": "normal",
    "risk": "moderate",
    "impact": "medium",
    "category": "Hardware",
    "start_date": "2023-12-15 01:00:00",
    "end_date": "2023-12-15 03:00:00",
}

result = create_change_request(auth_manager, server_config, create_params)
print(result)
```

For a complete example, see the [change_management_demo.py](../examples/change_management_demo.py) script.

## Integration with Claude Desktop

To configure the ServiceNow MCP server with change management tools in Claude Desktop:

1. Edit the Claude Desktop configuration file at `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or the appropriate path for your OS:

```json
{
  "mcpServers": {
    "ServiceNow": {
      "command": "/Users/yourusername/dev/servicenow-mcp/.venv/bin/python",
      "args": [
        "-m",
        "servicenow_mcp.cli"
      ],
      "env": {
        "SERVICENOW_INSTANCE_URL": "https://your-instance.service-now.com",
        "SERVICENOW_USERNAME": "your-username",
        "SERVICENOW_PASSWORD": "your-password",
        "SERVICENOW_AUTH_TYPE": "basic"
      }
    }
  }
}
```

2. Restart Claude Desktop to apply the changes

## Customization

The change management tools can be customized to match your organization's specific ServiceNow configuration:

- State values may need to be adjusted based on your ServiceNow instance configuration
- Additional fields can be added to the parameter models if needed
- Approval workflows may need to be modified to match your organization's approval process

To customize the tools, modify the `change_tools.py` file in the `src/servicenow_mcp/tools` directory. 