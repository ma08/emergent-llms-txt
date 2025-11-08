# Emergent Platform - Comprehensive LLM Context Guide

## Table of Contents
1. [Platform Overview](#platform-overview)
2. [Supported Tech Stack](#supported-tech-stack)
3. [Environment Architecture](#environment-architecture)
4. [Service Configuration](#service-configuration)
5. [Development Workflow](#development-workflow)
6. [Integration Capabilities](#integration-capabilities)
7. [Deployment Guide](#deployment-guide)
8. [Testing Protocol](#testing-protocol)
9. [Best Practices](#best-practices)
10. [Common Issues & Solutions](#common-issues--solutions)

---

## Platform Overview

Emergent is a cloud-based development platform that enables rapid application development using AI agents (E1, E1.1, E1.5). The platform runs in a Kubernetes container environment with fully configured development tools and services.

### Key Features
- **AI-Powered Development**: LLM agents (E1 series) that can build, modify, and debug full-stack applications
- **Pre-configured Environment**: Ready-to-use FastAPI + React + MongoDB stack
- **Integrated Services**: Frontend, backend, and database services with hot reload
- **Universal LLM Key**: Single key for OpenAI, Anthropic, and Google integrations
- **Automated Testing**: Built-in testing agents for frontend and backend
- **Version Control**: Git integration with GitHub connectivity
- **Native Deployment**: One-click deployment to production

---

## Supported Tech Stack

### ✅ Currently Supported

#### Frontend
- **Framework**: React (with Create React App or Vite-compatible setup)
- **Styling**: 
  - Tailwind CSS (pre-configured)
  - CSS/SCSS
  - CSS-in-JS
- **Package Manager**: Yarn (npm is NOT supported - breaking change)
- **Port**: 3000 (internal)
- **Build Tool**: Create React App configuration

#### Backend
- **Framework**: FastAPI (Python)
- **ASGI Server**: Uvicorn (managed by supervisor)
- **Port**: 8001 (internal)
- **Package Manager**: pip with requirements.txt
- **Python Version**: Python 3.x

#### Database
- **Primary Database**: MongoDB
- **Connection**: Pre-configured via MONGO_URL environment variable
- **Important**: Use UUIDs instead of MongoDB ObjectID (JSON serialization issues)

#### Process Management
- **Supervisor**: Manages frontend and backend services
- **Hot Reload**: Enabled for both frontend and backend
- **Service Commands**:
  ```bash
  sudo supervisorctl restart frontend
  sudo supervisorctl restart backend
  sudo supervisorctl restart all
  sudo supervisorctl status
  ```

### ❌ Not Currently Supported

- **Other Frontend Frameworks**: Vue.js, Angular, Svelte, Next.js (as primary framework)
- **Other Backend Frameworks**: Express.js, Django, Flask, Ruby on Rails
- **Other Databases**: PostgreSQL, MySQL, SQLite (as primary database)
- **Other Package Managers**: npm for frontend (use yarn instead)
- **Docker Compose**: Native Kubernetes environment
- **Custom Port Configuration**: Ports are pre-configured

---

## Environment Architecture

### Container Environment
```
Kubernetes Pod
├── Linux Container (Ubuntu-based)
├── Bash Command Line Access
├── Git Version Control
├── Supervisor Process Manager
└── Pre-installed Development Tools
```

### Directory Structure
```
/app/
├── backend/                 # FastAPI backend
│   ├── server.py           # Main FastAPI application
│   ├── requirements.txt    # Python dependencies
│   └── .env               # Backend environment variables
│       └── MONGO_URL      # MongoDB connection string
│
├── frontend/               # React frontend
│   ├── src/               # Source code
│   │   ├── index.js       # Entry point
│   │   ├── App.js         # Main component
│   │   ├── App.css        # Component styles
│   │   └── index.css      # Global styles
│   ├── public/            # Static assets
│   ├── package.json       # Node dependencies
│   ├── tailwind.config.js # Tailwind configuration
│   ├── postcss.config.js  # PostCSS configuration
│   └── .env              # Frontend environment variables
│       └── REACT_APP_BACKEND_URL  # Backend API URL
│
├── tests/                 # Test directory
├── scripts/              # Utility scripts
├── test_result.md        # Testing state and history
└── README.md             # Project documentation
```

### Service Ports

| Service | Internal Port | External Access |
|---------|--------------|-----------------|
| Frontend | 3000 | Via Kubernetes ingress |
| Backend | 8001 | Via `/api` prefix routing |
| MongoDB | 27017 | Internal only |

---

## Service Configuration

### Critical Environment Variables

#### Frontend `.env` (PROTECTED)
```bash
REACT_APP_BACKEND_URL=<production-configured-url>
```
- **Usage in Code**: `import.meta.env.REACT_APP_BACKEND_URL` or `process.env.REACT_APP_BACKEND_URL`
- **DO NOT MODIFY**: Pre-configured for production
- **Purpose**: All API calls from frontend

#### Backend `.env` (PROTECTED)
```bash
MONGO_URL=<mongodb-connection-string>
```
- **Usage in Code**: `os.environ.get('MONGO_URL')`
- **DO NOT MODIFY**: Pre-configured for MongoDB access
- **Purpose**: Database connectivity

### URL Usage Rules

#### ✅ CORRECT Usage
```python
# Backend - Database connection
mongo_url = os.environ.get('MONGO_URL')
client = MongoClient(mongo_url)
```

```javascript
// Frontend - API calls
const backendUrl = import.meta.env.REACT_APP_BACKEND_URL;
const response = await fetch(`${backendUrl}/api/endpoint`);
```

#### ❌ INCORRECT Usage
```python
# DO NOT hardcode URLs
mongo_url = "mongodb://localhost:27017"  # WRONG
```

```javascript
// DO NOT hardcode URLs
const backendUrl = "http://localhost:8001";  # WRONG
```

### Kubernetes Ingress Rules

**CRITICAL**: All backend API routes MUST be prefixed with `/api`

```javascript
// ✅ CORRECT - Will route to port 8001
fetch(`${backendUrl}/api/users`)
fetch(`${backendUrl}/api/auth/login`)

// ❌ WRONG - Will route to port 3000 (frontend)
fetch(`${backendUrl}/users`)
fetch(`${backendUrl}/auth/login`)
```

### Service Restart Protocol

**When to Restart**:
- ✅ Installing new dependencies
- ✅ Modifying `.env` files
- ✅ Configuration file changes

**When NOT to Restart**:
- ❌ Code changes (hot reload enabled)
- ❌ CSS/styling updates
- ❌ Component modifications

```bash
# Restart specific service
sudo supervisorctl restart backend

# Restart all services
sudo supervisorctl restart all

# Check service status
sudo supervisorctl status

# View logs
tail -n 100 /var/log/supervisor/backend.err.log
tail -n 100 /var/log/supervisor/backend.out.log
tail -n 100 /var/log/supervisor/frontend.err.log
```

---

## Development Workflow

### Standard Development Process

1. **Initial Analysis**
   - Read `/app/test_result.md` for context
   - Understand existing codebase
   - Plan implementation approach

2. **Backend Development** (if applicable)
   - View backend files using `mcp_view_bulk`
   - Implement MongoDB models (use UUIDs, not ObjectID)
   - Create API endpoints with `/api` prefix
   - Update `requirements.txt`: `pip install <package> && pip freeze > /app/backend/requirements.txt`
   - Restart backend: `sudo supervisorctl restart backend`

3. **Frontend Development**
   - Follow existing design principles
   - Use Tailwind CSS for styling
   - Make API calls to `REACT_APP_BACKEND_URL`
   - Install packages: `yarn add <package>` (NOT npm)
   - Auto-restart via hot reload

4. **Testing**
   - Backend: Use `deep_testing_backend_v2` agent
   - Frontend: Use `auto_frontend_testing_agent` (after asking user)
   - Always update `test_result.md` before testing

5. **Documentation**
   - Update README.md
   - Document API endpoints
   - Add code comments for complex logic

### File Operations

#### Creating Files
```python
# Use mcp_create_file for new files
mcp_create_file(
    path="/app/backend/models.py",
    file_text="# Model code here"
)
```

#### Editing Files
```python
# Use mcp_search_replace for precise edits
mcp_search_replace(
    path="/app/backend/server.py",
    old_str="exact text to replace",
    new_str="new text"
)
```

#### Bulk Operations
```python
# Use mcp_bulk_file_writer for multiple files
mcp_bulk_file_writer(
    files=[
        {"path": "/app/backend/file1.py", "content": "..."},
        {"path": "/app/backend/file2.py", "content": "..."}
    ]
)
```

### Dependency Management

#### Python Dependencies
```bash
# Install and update requirements.txt
pip install numpy pandas && pip freeze > /app/backend/requirements.txt

# Never manually edit requirements.txt
```

#### JavaScript Dependencies
```bash
# Always use yarn (NOT npm)
cd /app/frontend
yarn add react-router-dom

# Never use npm - it's a breaking change
```

---

## Integration Capabilities

### Emergent Universal LLM Key

**What It Is**: A single API key that works across multiple LLM providers

**Supported Services**:
- ✅ OpenAI (text generation + image generation with gpt-image-1)
- ✅ Anthropic Claude (text generation only)
- ✅ Google Gemini (text generation + Nano Banana image generation)

**NOT Supported**:
- ❌ Audio generation
- ❌ Video generation
- ❌ Other modalities

**How to Access**:
```python
# Use emergent_integrations_manager tool
# Do NOT ask user for this key
key = emergent_integrations_manager()
```

**Important Notes**:
- Available to all users
- Budget management via Profile → Universal Key → Add Balance
- Auto top-up available
- Do NOT use for non-LLM services (Stripe, Fal.ai, SendGrid, etc.)

### Third-Party Integrations

**Process**:
1. Call `integration_playbook_expert_v2` agent
2. Receive VERIFIED or UNVERIFIED playbook
3. **ASK USER for API keys** (mandatory step)
4. Implement exactly as specified in playbook
5. Test integration before proceeding

**Example Integration Request**:
```
INTEGRATION: Stripe Payment Gateway
CONSTRAINTS: Need subscription billing support
```

**Common Integrations**:
- Payment: Stripe, PayPal
- Authentication: Auth0, Firebase, Google OAuth
- AI/ML: OpenAI, Anthropic (via Emergent key), Fal.ai
- Email/SMS: SendGrid, Twilio
- Storage: AWS S3, Google Cloud Storage

**Critical Rules**:
- ❌ NEVER implement integrations from your knowledge alone
- ✅ ALWAYS use `integration_playbook_expert_v2` first
- ✅ ALWAYS ask user for API keys before implementation
- ✅ Follow playbook instructions exactly (model names, configurations)
- ✅ For Emergent-supported LLMs, use Emergent LLM key

---

## Deployment Guide

### Pre-Deployment Checklist

1. **Environment Variables**
   - All API keys in `.env` files
   - No hardcoded URLs or secrets
   - REACT_APP_BACKEND_URL properly configured

2. **Dependencies**
   - `requirements.txt` up to date
   - `package.json` up to date
   - All packages installed

3. **API Routes**
   - All backend routes prefixed with `/api`
   - Frontend uses `REACT_APP_BACKEND_URL`

4. **Testing**
   - Backend tested with `deep_testing_backend_v2`
   - Frontend tested (if applicable)
   - All critical flows verified

### Deployment Process

**Native Deployment** (Recommended):
1. User clicks "Deploy" in Emergent UI
2. Platform handles build and deployment
3. Automatic URL generation
4. Zero-downtime deployment

**GitHub Integration**:
1. User clicks "Save to GitHub" in chat input
2. Code pushed to connected repository
3. Can deploy from GitHub to external platforms

**Important Notes**:
- DO NOT handle git operations programmatically
- Inform users to use "Save to GitHub" feature
- For deployment questions, call `support_agent`

### Common Deployment Issues

**Issue**: Services not starting
```bash
# Check supervisor status
sudo supervisorctl status

# Check logs
tail -n 100 /var/log/supervisor/backend.err.log
tail -n 100 /var/log/supervisor/frontend.err.log
```

**Issue**: API routes not working
- Verify `/api` prefix on all backend routes
- Check REACT_APP_BACKEND_URL in frontend
- Confirm backend is running on port 8001

**Issue**: Database connection failed
- Verify MONGO_URL in backend/.env
- Check MongoDB service status
- Test connection in backend code

For persistent deployment issues, call `deployment_agent`:
```python
deployment_agent(
    task="""
    ISSUE: Deployment failing with error X
    COMPONENT: Backend/Frontend
    ERROR_MESSAGES: [paste errors]
    """
)
```

---

## Testing Protocol

### Testing Agent Types

1. **Backend Testing**: `deep_testing_backend_v2`
   - Tests API endpoints with curl
   - Validates database operations
   - Checks error handling

2. **Frontend Testing**: `auto_frontend_testing_agent`
   - Uses Playwright for UI testing
   - Tests user interactions
   - Validates UI rendering

3. **Troubleshooting**: `troubleshoot_agent`
   - Root cause analysis (RCA)
   - Persistent error investigation
   - System diagnostics

### Testing Workflow

**Before Testing**:
```yaml
# Update /app/test_result.md
user_problem_statement: "User's request"
backend:
  - task: "User authentication"
    implemented: true
    working: "NA"
    needs_retesting: true
    file: "server.py"

agent_communication:
  - agent: "main"
    message: "Implemented user auth, please test login and registration"
```

**Call Testing Agent**:
```python
deep_testing_backend_v2(
    task="""
    Test the user authentication system:
    1. Test registration endpoint: POST /api/auth/register
    2. Test login endpoint: POST /api/auth/login
    3. Verify JWT token generation
    4. Test protected routes with token
    
    Database: MongoDB at MONGO_URL
    Backend: Running on port 8001
    """
)
```

**After Testing**:
- Review test results in `test_result.md`
- Fix issues identified by testing agent
- Do NOT fix issues already resolved by testing agent
- Call `troubleshoot_agent` if stuck after 3 attempts

### When to Call Troubleshoot Agent

**Mandatory Calls**:
- After 3 consecutive failed attempts at same operation
- After 5 tool calls focused on same sub-problem
- Same error message appears twice
- Every 10 tool calls without clear progress
- User reports "nothing is working" or "no changes showing"
- ANY service unavailability or preview failures

**Input Format**:
```python
troubleshoot_agent(
    task="""
    ISSUE: Backend service not starting after auth implementation
    COMPONENT: Backend
    ERROR_MESSAGES: 
    - ModuleNotFoundError: No module named 'jwt'
    - Service failed to start
    RECENT_ACTIONS:
    - Added JWT authentication to server.py
    - Installed PyJWT package
    PREVIOUS_FIX_ATTEMPTS:
    - Tried pip install PyJWT
    - Restarted backend service
    - Checked requirements.txt
    RELEVANT_FILES:
    - /app/backend/server.py
    - /app/backend/requirements.txt
    - /app/backend/.env
    """
)
```

### Screenshot Testing

**When to Take Screenshots**:
- Before finishing task (verify UI works)
- After major UI changes
- When user reports UI broken
- Design review (padding, alignment, colors)
- Image verification (relevant and not broken)

**Example**:
```python
screenshot_tool(
    page_url="http://localhost:3000",
    script="""
# Set viewport
await page.set_viewport_size({"width": 1920, "height": 800})

# Navigate and wait
await page.goto("http://localhost:3000")
await page.wait_for_load_state("networkidle")

# Take screenshot
await page.screenshot(path="homepage.png", quality=20, full_page=False)
    """
)
```

---

## Best Practices

### Code Quality

1. **Use Environment Variables**
   - Never hardcode URLs, ports, or API keys
   - Use .env files for all configuration
   - Import variables correctly (see Service Configuration)

2. **API Endpoint Naming**
   - Always prefix with `/api`: `/api/users`, `/api/auth/login`
   - Use RESTful conventions: GET, POST, PUT, DELETE
   - Include version if needed: `/api/v1/users`

3. **Database Operations**
   - Use UUIDs instead of MongoDB ObjectID
   - Implement proper error handling
   - Use connection pooling
   - Close connections properly

4. **Frontend Code**
   - Import React at top of files (especially with Context)
   - Use functional components and hooks
   - Implement proper error boundaries
   - Show loading states

5. **File Operations**
   - View files before editing (match indentation exactly)
   - Use `mcp_search_replace` for precise edits
   - Use `mcp_bulk_file_writer` for multiple files
   - Run linters after edits when appropriate

### Performance

1. **Hot Reload**
   - Don't restart services for code changes
   - Only restart for dependencies or .env changes

2. **Dependency Management**
   - Keep dependencies minimal
   - Update requirements.txt properly
   - Use specific versions when stability matters

3. **File Uploads**
   - Use chunked uploads for large files
   - Store in persistent locations
   - Implement progress indicators
   - Handle errors at each phase

### Security

1. **Environment Variables**
   - Never commit .env files to git
   - Use .gitignore properly
   - Rotate API keys regularly

2. **API Security**
   - Implement authentication
   - Use HTTPS in production
   - Validate all inputs
   - Implement rate limiting

3. **Database Security**
   - Use connection strings from environment
   - Implement proper access controls
   - Sanitize user inputs
   - Use parameterized queries

### Error Handling

1. **Backend Errors**
   - Use try-except blocks
   - Return appropriate HTTP status codes
   - Log errors for debugging
   - Provide helpful error messages

2. **Frontend Errors**
   - Implement error boundaries
   - Show user-friendly messages
   - Handle network failures gracefully
   - Log errors to console

3. **Testing Errors**
   - Fix one issue at a time
   - Don't get stuck in loops
   - Call troubleshoot_agent after 3 attempts
   - Document solutions in test_result.md

---

## Common Issues & Solutions

### Service Issues

**Backend Not Starting**
```bash
# Check logs
tail -n 100 /var/log/supervisor/backend.err.log

# Common causes:
# 1. Missing dependency - install and update requirements.txt
# 2. Syntax error - check Python linting
# 3. Import error - verify all imports are installed
# 4. Port conflict - check if port 8001 is free

# Solution
pip install <missing-package> && pip freeze > /app/backend/requirements.txt
sudo supervisorctl restart backend
```

**Frontend Not Starting**
```bash
# Check logs
tail -n 100 /var/log/supervisor/frontend.err.log

# Common causes:
# 1. Missing dependency - install with yarn
# 2. Syntax error - check JS linting
# 3. Import error - verify all imports
# 4. Port conflict - check if port 3000 is free

# Solution
cd /app/frontend
yarn add <missing-package>
sudo supervisorctl restart frontend
```

### API Issues

**404 Not Found on API Routes**
```javascript
// Problem: Missing /api prefix
fetch(`${backendUrl}/users`)  // ❌ Routes to frontend

// Solution: Add /api prefix
fetch(`${backendUrl}/api/users`)  // ✅ Routes to backend
```

**CORS Errors**
```python
# Add CORS middleware to FastAPI
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # Configure appropriately
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

### Database Issues

**Connection Failed**
```python
# Problem: Hardcoded connection string
client = MongoClient("mongodb://localhost:27017")  # ❌

# Solution: Use environment variable
import os
mongo_url = os.environ.get('MONGO_URL')
client = MongoClient(mongo_url)  # ✅
```

**ObjectID Serialization Error**
```python
# Problem: Using MongoDB ObjectID
from bson import ObjectId
user_id = ObjectId()  # ❌ Not JSON serializable

# Solution: Use UUID
import uuid
user_id = str(uuid.uuid4())  # ✅ JSON serializable
```

### Integration Issues

**API Key Not Working**
```bash
# 1. Verify key is in .env file
cat /app/backend/.env | grep API_KEY

# 2. Restart backend to load new env vars
sudo supervisorctl restart backend

# 3. Verify import in code
import os
api_key = os.environ.get('API_KEY')
print(f"API Key loaded: {api_key is not None}")
```

**Integration Failing**
```python
# 1. Get fresh playbook
integration_playbook_expert_v2(
    task="INTEGRATION: [Service Name]"
)

# 2. Ask user for correct API key
ask_human(
    question="Please provide your [Service] API key. You can get it from: [URL from playbook]"
)

# 3. Implement exactly as playbook specifies
# 4. Test integration before proceeding
```

### Git and Deployment Issues

**Need to Push to GitHub**
```
DO NOT use git commands programmatically.
Instead, tell user:

"To save your code to GitHub, please use the 'Save to GitHub' feature 
in the chat input area at the bottom of the screen."
```

**Deployment Questions**
```python
# Call support agent for platform questions
support_agent(
    task="User is asking about: [deployment/github/rollback/etc]"
)
```

### Testing Issues

**Tests Keep Failing**
```python
# After 3 attempts, call troubleshoot agent
troubleshoot_agent(
    task="""
    ISSUE: [Description]
    COMPONENT: [Backend/Frontend]
    ERROR_MESSAGES: [Errors]
    PREVIOUS_FIX_ATTEMPTS: [What you tried]
    RELEVANT_FILES: [File paths]
    """
)
```

**Testing Agent Not Testing Correctly**
```yaml
# Update test_result.md with clear instructions
test_plan:
  current_focus:
    - "Specific task to test"
  test_all: false

agent_communication:
  - agent: "main"
    message: "Please test: [specific scenario with exact steps]"
```

### Development Loop Issues

**Stuck in Fix Loop**
```
Signs you're stuck:
- Same error after 3+ attempts
- Trying variations of same approach
- No progress after 10 tool calls
- Thinking "just one more try"

ACTION: STOP and call troubleshoot_agent immediately
```

**User Reports Nothing Working**
```python
# Immediately call troubleshoot agent
troubleshoot_agent(
    task="""
    ISSUE: User reports application not working
    COMPONENT: [Best guess]
    ERROR_MESSAGES: [Any errors seen]
    RECENT_ACTIONS: [What was changed recently]
    """
)
```

---

## Quick Reference Commands

### Service Management
```bash
# Restart services
sudo supervisorctl restart backend
sudo supervisorctl restart frontend
sudo supervisorctl restart all
sudo supervisorctl status

# View logs
tail -n 100 /var/log/supervisor/backend.err.log
tail -n 100 /var/log/supervisor/backend.out.log
tail -n 100 /var/log/supervisor/frontend.err.log
tail -n 100 /var/log/supervisor/frontend.out.log
```

### Dependency Management
```bash
# Python
pip install <package> && pip freeze > /app/backend/requirements.txt

# JavaScript (use yarn, NOT npm)
cd /app/frontend
yarn add <package>
yarn remove <package>
```

### File Operations
```bash
# View directory structure
ls -la /app/backend
ls -la /app/frontend/src

# Search for patterns
grep -r "pattern" /app/backend
grep -r "pattern" /app/frontend/src

# Check file content
cat /app/backend/.env
cat /app/frontend/.env
```

### Process Monitoring
```bash
# Check if services are running
ps aux | grep python  # Backend
ps aux | grep node    # Frontend

# Check port usage
netstat -tulpn | grep 8001  # Backend port
netstat -tulpn | grep 3000  # Frontend port
```

---

## Support and Resources

### When to Call Support Agent

Call `support_agent` for:
- Platform capability questions (What can E1 do?)
- Deployment options and processes
- GitHub integration questions
- Version differences (E1 vs E1.1 vs E1.5)
- Billing or subscription issues
- Refund requests or complaints
- Any non-development platform questions

### When to Call Troubleshoot Agent

Call `troubleshoot_agent` for:
- Persistent technical errors (3+ attempts)
- Service failures or unavailability
- Complex debugging scenarios
- Root cause analysis
- When stuck in fix loops

### When to Call Integration Playbook Expert

Call `integration_playbook_expert_v2` for:
- ANY third-party service integration
- Payment processors (Stripe, PayPal)
- Authentication services (Auth0, Firebase, OAuth)
- AI/ML services (OpenAI, Perplexity, Fal.ai)
- Email/SMS services (SendGrid, Twilio)
- Cloud storage (S3, Google Cloud)

### When to Ask Human

Call `ask_human` for:
- Clarification on requirements
- API keys for third-party services
- Design preferences or choices
- Confirmation before critical actions
- Feature priority decisions

---

## Version Information

- **Document Version**: 1.0
- **Last Updated**: July 2025
- **Platform**: Emergent (E1 Agent System)
- **Tech Stack**: FastAPI + React + MongoDB
- **Environment**: Kubernetes Container

---

## Important Reminders

### DO ✅
- Always check `test_result.md` first
- Use environment variables for ALL URLs and keys
- Prefix backend routes with `/api`
- Use yarn for JavaScript dependencies
- Call troubleshoot_agent after 3 failed attempts
- Ask user for API keys before integration
- Test backend before frontend
- Update requirements.txt and package.json properly
- Follow existing code patterns and style
- Document your changes

### DON'T ❌
- Modify .env files unnecessarily
- Hardcode URLs or API keys
- Use npm (use yarn instead)
- Start your own servers (use supervisor)
- Run long tasks in foreground
- Use MongoDB ObjectID (use UUID)
- Implement integrations from memory
- Get stuck in fix loops
- Handle git operations programmatically
- Forget `/api` prefix on backend routes

---

## Contact and Feedback

For platform support, feature requests, or issues:
- Use `support_agent` within your development session
- Access help documentation in Emergent UI
- Contact Emergent support team through platform

---

**End of Emergent Platform Context Guide**

This document is designed to be comprehensive yet practical for LLM agents working on the Emergent platform. Use it as a reference throughout your development process.
