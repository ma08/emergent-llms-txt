# Emergent Platform - Quick Reference Guide

> **Quick access guide for LLM agents working on Emergent platform**

---

## 🎯 Tech Stack (Current Support)

| Component | Supported | NOT Supported |
|-----------|-----------|---------------|
| **Frontend** | React + Tailwind CSS | Vue, Angular, Svelte, Next.js |
| **Backend** | FastAPI (Python) | Express, Django, Flask, Rails |
| **Database** | MongoDB | PostgreSQL, MySQL, SQLite |
| **Package Manager** | yarn (frontend), pip (backend) | npm ❌ BREAKING CHANGE |
| **Process Manager** | Supervisor | Docker Compose, custom setups |

---

## 🔐 Critical Environment Variables (NEVER MODIFY)

```bash
# Frontend: /app/frontend/.env
REACT_APP_BACKEND_URL=<production-url>  # Use: import.meta.env.REACT_APP_BACKEND_URL

# Backend: /app/backend/.env
MONGO_URL=<mongodb-connection>  # Use: os.environ.get('MONGO_URL')
```

**Rule**: NEVER hardcode URLs or ports. Always use environment variables.

---

## 🌐 URL & Port Configuration

```
Internal Ports (DO NOT MODIFY):
- Frontend: 3000
- Backend: 8001
- MongoDB: 27017

Kubernetes Ingress Routing:
- /api/* → Backend (port 8001)
- /* → Frontend (port 3000)
```

### ✅ CORRECT API Usage
```javascript
// All backend routes MUST have /api prefix
fetch(`${REACT_APP_BACKEND_URL}/api/users`)
fetch(`${REACT_APP_BACKEND_URL}/api/auth/login`)
```

### ❌ WRONG API Usage
```javascript
// Missing /api prefix - will route to frontend!
fetch(`${REACT_APP_BACKEND_URL}/users`)  // WRONG
```

---

## 📂 Directory Structure

```
/app/
├── backend/
│   ├── server.py          # FastAPI app
│   ├── requirements.txt   # Python deps
│   └── .env              # MONGO_URL
├── frontend/
│   ├── src/              # React code
│   ├── package.json      # Node deps
│   └── .env              # REACT_APP_BACKEND_URL
└── test_result.md        # Testing state
```

---

## 🛠️ Essential Commands

### Service Management
```bash
# Restart services
sudo supervisorctl restart backend
sudo supervisorctl restart frontend
sudo supervisorctl restart all

# Check status
sudo supervisorctl status

# View logs
tail -n 100 /var/log/supervisor/backend.err.log
tail -n 100 /var/log/supervisor/frontend.err.log
```

### Dependency Management
```bash
# Python (always update requirements.txt)
pip install <package> && pip freeze > /app/backend/requirements.txt

# JavaScript (use yarn, NEVER npm)
cd /app/frontend
yarn add <package>
```

### When to Restart Services
- ✅ After installing dependencies
- ✅ After modifying .env files
- ❌ After code changes (hot reload enabled)

---

## 🔑 Emergent Universal LLM Key

**What it supports:**
- ✅ OpenAI (text + gpt-image-1)
- ✅ Anthropic Claude (text only)
- ✅ Google Gemini (text + Nano Banana images)
- ❌ Audio, video, other modalities

**How to get it:**
```python
# Use emergent_integrations_manager tool
# DO NOT ask user for this key
key = emergent_integrations_manager()
```

**NOT for:** Stripe, Fal.ai, SendGrid, Twilio, etc.

---

## 🔌 Third-Party Integration Process

1. **Call Integration Expert**
   ```python
   integration_playbook_expert_v2(
       task="INTEGRATION: [Service Name]"
   )
   ```

2. **ASK USER for API keys** (mandatory!)
   ```python
   ask_human(
       question="Please provide [Service] API key..."
   )
   ```

3. **Implement exactly as playbook specifies**
4. **Test before proceeding**

**Rule**: NEVER implement integrations from memory. Always use playbook.

---

## 🧪 Testing Protocol

### 1. Backend Testing
```python
# Always update test_result.md first
deep_testing_backend_v2(
    task="Test endpoints: POST /api/auth/login, GET /api/users..."
)
```

### 2. Frontend Testing
```python
# ASK USER first if they want to test manually
ask_human(question="Should I test frontend or will you test manually?")

# If yes, then:
auto_frontend_testing_agent(
    task="Test user login flow, dashboard loading..."
)
```

### 3. When Stuck - Troubleshoot Agent

**Call immediately when:**
- ❌ 3+ failed attempts at same thing
- ❌ Same error appears twice
- ❌ 10 tool calls without progress
- ❌ User says "nothing works"
- ❌ Service unavailability

```python
troubleshoot_agent(
    task="""
    ISSUE: [Description]
    COMPONENT: Backend/Frontend
    ERROR_MESSAGES: [Errors]
    RECENT_ACTIONS: [What changed]
    PREVIOUS_FIX_ATTEMPTS: [What you tried]
    RELEVANT_FILES: [Paths]
    """
)
```

---

## 📋 Development Workflow Checklist

### Start Every Task
- [ ] Read `/app/test_result.md`
- [ ] Understand existing codebase
- [ ] Plan implementation
- [ ] Identify needed integrations

### Backend Development
- [ ] Use UUIDs (NOT MongoDB ObjectID)
- [ ] Prefix all routes with `/api`
- [ ] Update `requirements.txt` after installs
- [ ] Use `os.environ.get('MONGO_URL')`
- [ ] Restart backend after .env changes

### Frontend Development
- [ ] Use `REACT_APP_BACKEND_URL` for API calls
- [ ] Install with `yarn add` (NEVER npm)
- [ ] Follow existing design patterns
- [ ] Import React at top (especially with Context)
- [ ] Hot reload handles restarts

### Testing
- [ ] Update `test_result.md` before testing
- [ ] Test backend first
- [ ] Ask user before frontend testing
- [ ] Call troubleshoot agent if stuck
- [ ] Take screenshot before finishing

### Integration
- [ ] Call `integration_playbook_expert_v2`
- [ ] Ask user for API keys
- [ ] Add keys to `.env`
- [ ] Restart services
- [ ] Test integration works

---

## 🚨 Common Mistakes to Avoid

| Mistake | Correct Approach |
|---------|------------------|
| Using npm | Use yarn |
| Hardcoding URLs | Use environment variables |
| Forgetting `/api` prefix | All backend routes need `/api` |
| Using MongoDB ObjectID | Use UUID (JSON serialization) |
| Starting own server | Use supervisor |
| Modifying .env URLs | Never change REACT_APP_BACKEND_URL or MONGO_URL |
| Implementing integrations from memory | Always use integration_playbook_expert_v2 |
| Staying stuck in fix loops | Call troubleshoot_agent after 3 attempts |
| Not testing backend | Always test backend with deep_testing_backend_v2 |

---

## 🤖 Agent Tools Quick Reference

| Use Case | Tool | When |
|----------|------|------|
| **Get LLM integration** | `integration_playbook_expert_v2` | Before LLM integration |
| **Get Emergent LLM key** | `emergent_integrations_manager` | For OpenAI/Claude/Gemini |
| **Third-party integration** | `integration_playbook_expert_v2` | Any external service |
| **Backend testing** | `deep_testing_backend_v2` | After backend changes |
| **Frontend testing** | `auto_frontend_testing_agent` | After frontend changes (ask user first) |
| **Stuck/errors** | `troubleshoot_agent` | After 3 failed attempts |
| **Platform questions** | `support_agent` | Deployment, GitHub, platform features |
| **Ask user** | `ask_human` | Clarification, API keys, confirmation |
| **Take screenshot** | `screenshot_tool` | Before finishing, UI verification |

---

## 🎨 File Operations

```python
# View files
mcp_view_file(path="/app/backend/server.py")
mcp_view_bulk(paths=["/app/backend/server.py", "/app/backend/models.py"])

# Create new file
mcp_create_file(path="/app/backend/new.py", file_text="...")

# Edit existing file
mcp_search_replace(
    path="/app/backend/server.py",
    old_str="exact text with correct indentation",
    new_str="replacement text"
)

# Multiple files at once
mcp_bulk_file_writer(
    files=[
        {"path": "/app/backend/file1.py", "content": "..."},
        {"path": "/app/frontend/src/file2.js", "content": "..."}
    ]
)
```

---

## 💾 Database Best Practices

```python
# ✅ CORRECT - Use UUID
import uuid
user_id = str(uuid.uuid4())

# ❌ WRONG - MongoDB ObjectID not JSON serializable
from bson import ObjectId
user_id = ObjectId()  # Don't use this

# ✅ CORRECT - Environment variable
import os
mongo_url = os.environ.get('MONGO_URL')
client = MongoClient(mongo_url)

# ❌ WRONG - Hardcoded
client = MongoClient("mongodb://localhost:27017")
```

---

## 🚀 Deployment Quick Facts

- **Native Deployment**: User clicks "Deploy" in UI (zero config)
- **GitHub**: User clicks "Save to GitHub" in chat input
- **Git Operations**: DO NOT handle programmatically - tell user to use UI
- **Deployment Issues**: Call `deployment_agent`

---

## 📞 When to Call Which Agent

```
Integration needed → integration_playbook_expert_v2
Backend testing → deep_testing_backend_v2
Frontend testing → auto_frontend_testing_agent (ask user first)
Stuck/errors (3+ attempts) → troubleshoot_agent
Platform questions → support_agent
User clarification → ask_human
Deployment issues → deployment_agent
```

---

## ⚡ Hot Tips

1. **Always read test_result.md first** - Contains task context
2. **Use environment variables** - Never hardcode
3. **Call troubleshoot early** - Don't stay stuck
4. **Ask for API keys** - Before any integration
5. **Test backend first** - Then frontend
6. **Use yarn, not npm** - npm breaks things
7. **/api prefix required** - All backend routes
8. **UUID not ObjectID** - JSON serialization
9. **Hot reload enabled** - Only restart for deps/.env
10. **Update test_result.md** - Before calling testing agents

---

## 🎯 Success Checklist Before Finishing

- [ ] All features implemented and working
- [ ] Backend tested with `deep_testing_backend_v2`
- [ ] Frontend tested (if applicable, after asking user)
- [ ] Screenshot taken to verify UI
- [ ] Environment variables properly set
- [ ] No hardcoded URLs or secrets
- [ ] All routes have `/api` prefix
- [ ] Dependencies in requirements.txt and package.json
- [ ] Services restart successfully
- [ ] test_result.md updated

---

## 🆘 Emergency Commands

```bash
# Service crashed - check logs
tail -n 100 /var/log/supervisor/backend.err.log
tail -n 100 /var/log/supervisor/frontend.err.log

# Force restart everything
sudo supervisorctl restart all

# Check if processes running
ps aux | grep python  # Backend
ps aux | grep node    # Frontend

# Check ports
netstat -tulpn | grep 8001
netstat -tulpn | grep 3000
```

---

**Remember**: This is an MVP development platform. Focus on working functionality over perfect code. Use troubleshoot_agent early and often. Test thoroughly. Ask users for clarification when needed.

---

**Document Version**: 1.0 | **Updated**: July 2025 | **Platform**: Emergent E1
