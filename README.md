# Jira MCP Server for Cursor

A TypeScript-based MCP server that integrates with Jira, allowing Cursor to interact with Jira tickets.

## Features

- List Jira tickets
- Get ticket details
- Get ticket comments
- Create new tickets
- Add comments to tickets
- Update ticket status
- Full MCP protocol support for Cursor integration

## Setup

1. Install dependencies:

```bash
npm install
```

2. Create a `.env` file based on `.env.example` and fill in your Jira credentials:

```
JIRA_HOST=https://your-domain.atlassian.net
JIRA_EMAIL=your-email@example.com
JIRA_API_TOKEN=your-api-token
PORT=3000
```

To get your Jira API token:

1. Log in to <https://id.atlassian.com/manage/api-tokens>
2. Click "Create API token"
3. Copy the token and paste it in your `.env` file

## Development

Run the development server:

```bash
npm run dev
```

## Build and Run

Build the project:

```bash
npm run build
```

Start the server:

```bash
npm start
```

## Cursor Integration

To use this MCP server with Cursor, you have two options:

### Option 1: Command-based Integration (Recommended)

1. Build the project:

```bash
npm run build
```

2. Open Cursor's settings:
   - Click on the Cursor menu
   - Select "Settings" (or use the keyboard shortcut)
   - Navigate to the "Extensions" or "Integrations" section

3. Add the MCP configuration:

```json
{
  "mcps": {
    "jira": {
      "command": "node",
      "args": ["/path/to/jira-mcp-cursor/dist/server.js"]
    }
  }
}
```

Replace `/path/to/jira-mcp-cursor` with the absolute path to your project.

### Option 2: HTTP-based Integration (Alternative)

1. Start the MCP server (if not already running):

```bash
npm start
```

2. Open Cursor's settings:
   - Click on the Cursor menu
   - Select "Settings" (or use the keyboard shortcut)
   - Navigate to the "Extensions" or "Integrations" section

3. Add the MCP configuration:

```json
{
  "mcps": {
    "jira": {
      "url": "http://localhost:3000",
      "capabilities": [
        "list_tickets",
        "get_ticket",
        "get_comments",
        "create_ticket",
        "update_status",
        "add_comment"
      ]
    }
  }
}
```

## Using Jira in Cursor

After configuring the MCP server, you can use Jira commands directly in Cursor:

- `/jira list` - List your tickets
- `/jira view TICKET-123` - View ticket details
- `/jira comments TICKET-123` - Get ticket comments
- `/jira create` - Create a new ticket
- `/jira comment TICKET-123` - Add a comment
- `/jira status TICKET-123` - Update ticket status

## MCP Protocol Support

The server implements the Model-Client-Protocol (MCP) required by Cursor:

- Stdio communication for command-based integration
- Tool registration for Jira operations

## API Endpoints

### List Tickets

Retrieves a list of Jira tickets, optionally filtered by a JQL query.

**Endpoint:** `GET /api/tickets`

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| jql | string | No | Jira Query Language (JQL) string to filter tickets |

**Example Request:**

```http
GET /api/tickets?jql=project=TEST+AND+status=Open
```

**Example Response:**

```text
TEST-123: Example ticket (Open)
TEST-124: Another ticket (In Progress)
```

### Get Ticket

Retrieves detailed information about a specific ticket.

**Endpoint:** `GET /api/tickets/:id`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | Yes | The Jira ticket ID (e.g., TEST-123) |

**Example Request:**

```http
GET /api/tickets/TEST-123
```

**Example Response:**

```text
Key: TEST-123
Summary: Example ticket
Status: Open
Type: Task
Description:
Detailed ticket description
```

### Get Ticket Comments

Retrieves all comments for a specific ticket.

**Endpoint:** `GET /api/tickets/:id/comments`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | Yes | The Jira ticket ID (e.g., TEST-123) |

**Example Request:**

```http
GET /api/tickets/TEST-123/comments
```

**Example Response:**

```text
[3/20/2024, 10:00:00 AM] John Doe:
Comment text
---

[3/20/2024, 9:30:00 AM] Jane Smith:
Another comment
---
```

### Create Ticket

Creates a new Jira ticket.

**Endpoint:** `POST /api/tickets`

**Request Body:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| summary | string | Yes | The ticket summary |
| description | string | Yes | The ticket description |
| projectKey | string | Yes | The project key (e.g., TEST) |
| issueType | string | Yes | The type of issue (e.g., Task, Bug) |

**Example Request:**

```http
POST /api/tickets
Content-Type: application/json

{
  "summary": "New feature request",
  "description": "Implement new functionality",
  "projectKey": "TEST",
  "issueType": "Task"
}
```

**Example Response:**

```text
Created ticket: TEST-124
```

### Add Comment

Adds a new comment to an existing ticket.

**Endpoint:** `POST /api/tickets/:id/comments`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | Yes | The Jira ticket ID (e.g., TEST-123) |

**Request Body:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| body | string | Yes | The comment text |

**Example Request:**

```http
POST /api/tickets/TEST-123/comments
Content-Type: application/json

{
  "body": "This is a new comment"
}
```

**Example Response:**

```text
Added comment to TEST-123
```

### Update Status

Updates the status of an existing ticket.

**Endpoint:** `POST /api/tickets/:id/status`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | Yes | The Jira ticket ID (e.g., TEST-123) |

**Request Body:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| transitionId | string | Yes | The ID of the transition to perform |

**Example Request:**

```http
POST /api/tickets/TEST-123/status
Content-Type: application/json

{
  "transitionId": "21"
}
```

**Example Response:**

```text
Updated status of TEST-123
```

### Search Tickets

Searches for tickets across specified projects using text search.

**Endpoint:** `GET /api/tickets/search`

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| searchText | string | Yes | Text to search for in tickets |
| projectKeys | string | Yes | Comma-separated list of project keys to search in |
| maxResults | number | No | Maximum number of results to return (default: 50) |

**Example Request:**

```http
GET /api/tickets/search?searchText=login+bug&projectKeys=TEST,PROD&maxResults=10
```

**Example Response:**

```text
Found 2 tickets matching "login bug"

[TEST] TEST-123: Login page bug
Status: Open (Updated: 3/20/2024, 10:00:00 AM)
Description:
Users unable to login using SSO
----------------------------------------

[PROD] PROD-456: Fix login performance
Status: In Progress (Updated: 3/19/2024, 3:30:00 PM)
Description:
Login page taking too long to load
----------------------------------------
```
