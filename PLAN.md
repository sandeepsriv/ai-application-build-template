# Project Management MVP - Detailed Plan

## Part 1: Plan Enrichment ✓

**Objective**: Create detailed execution plan with checklists, tests, and success criteria for all 10 parts. Document existing frontend code.

**Subtasks**:
- [ ] Review AGENTS.md to understand existing frontend structure
- [ ] Create frontend/AGENTS.md documenting existing code (architecture, components, data model, limitations)
- [ ] Enhance PLAN.md with detailed steps for each part
- [ ] Define success criteria for each part
- [ ] Define testing approach for each part
- [ ] Get user approval on plan before proceeding

**Testing Approach**:
- Manual review of frontend code
- Documentation clarity and completeness

**Success Criteria**:
- frontend/AGENTS.md exists and clearly describes all components, data structures, and integration points
- PLAN.md is enriched with 5-10 subtasks per part
- Each part has defined test strategy and success criteria
- User has reviewed and approved plan

---

## Part 2: Docker & Backend Scaffolding

**Objective**: Set up Docker infrastructure, FastAPI backend, and server control scripts. Verify basic "hello world" and API connectivity.

**Subtasks**:
- [ ] Create Dockerfile with Python 3.12, uv package manager, FastAPI
- [ ] Create docker-compose.yml orchestrating frontend and backend services
- [ ] Create backend/ directory structure:
  - [ ] backend/main.py with FastAPI app and example routes
  - [ ] backend/requirements.txt with FastAPI, uvicorn, python-dotenv, openai (for OpenRouter)
  - [ ] backend/pyproject.toml or env file for uv configuration
- [ ] Create .env.example with OPENROUTER_API_KEY placeholder
- [ ] Create start.sh (Mac/Linux) script that builds and runs Docker container
- [ ] Create stop.sh script that stops and removes container
- [ ] Create start.bat and stop.bat for Windows (optional for MVP)
- [ ] FastAPI serves hello-world HTML at GET /
- [ ] FastAPI has example API endpoint at GET /api/example that returns JSON: `{"message": "Hello from API", "status": "ok"}`
- [ ] Frontend on port 3000, backend on port 8000, accessible via localhost

**Testing Approach**:
- Docker build succeeds without errors
- Container runs and logs show no startup errors
- curl/fetch to http://localhost:8000/ returns HTML
- curl/fetch to http://localhost:8000/api/example returns JSON

**Success Criteria**:
- Docker container builds and runs without errors
- http://localhost:8000/ displays hello-world HTML in browser
- http://localhost:8000/api/example returns valid JSON
- Backend logs show "Application startup complete"
- Both start.sh and stop.sh work as expected

---

## Part 3: Integrate Next.js Frontend

**Objective**: Build and serve the Next.js frontend from backend. Verify Kanban board displays at /.

**Subtasks**:
- [ ] Add frontend build step to Dockerfile: `npm run build` in frontend/
- [ ] Configure Dockerfile to copy next/.next/ output to backend serving directory
- [ ] Update backend main.py to serve static files from /frontend/.next at /
- [ ] Remove hardcoded hello-world HTML from Part 2
- [ ] Modify FastAPI to preserve /api/* routes for backend API calls
- [ ] Test frontend build succeeds (no TS errors, no build warnings)
- [ ] Test Kanban board renders at http://localhost:8000/
- [ ] Test drag-and-drop works in browser
- [ ] Test column renaming works
- [ ] Test add/delete cards work
- [ ] Verify no console errors in browser DevTools

**Testing Approach**:
- Manual browser testing of all interactive features
- Check for console errors
- Verify static assets load (CSS, JS bundles)

**Success Criteria**:
- http://localhost:8000/ shows fully interactive Kanban board
- All drag-and-drop, rename, add, delete features work
- No console errors in browser
- Frontend CSS loads correctly (styled, not broken)
- Container starts and app is immediately usable

---

## Part 4: Authentication - Fake Sign-in

**Objective**: Add login/logout flow with hardcoded credentials ("user" / "password"). Protect Kanban board behind auth.

**Subtasks**:
- [ ] Create new frontend component: LoginPage.tsx with email/password form
- [ ] Update frontend app/page.tsx to show LoginPage if not authenticated
- [ ] Implement frontend auth state using React context or hooks (localStorage for session token)
- [ ] Add logout button to Kanban header
- [ ] Create backend endpoint: POST /api/auth/login with hardcoded validation
  - [ ] Accept JSON `{ username, password }`
  - [ ] Validate username == "user" AND password == "password"
  - [ ] Return JSON `{ token: "fake-jwt", username }`
  - [ ] Return 401 if credentials invalid
- [ ] Create backend endpoint: POST /api/auth/logout (optional, just clears frontend state)
- [ ] Frontend stores token in localStorage/sessionStorage
- [ ] Frontend includes Authorization header in all Kanban API calls (once Part 6 is complete)
- [ ] Test login with correct credentials redirects to Kanban
- [ ] Test login with wrong credentials shows error message
- [ ] Test logout clears auth state and returns to login
- [ ] Test direct URL access to / redirects to login if no token

**Testing Approach**:
- Manual browser testing of login/logout flow
- Test valid and invalid credentials
- Test page redirects and state persistence
- E2E tests for auth flows

**Success Criteria**:
- http://localhost:8000/ shows login page when not authenticated
- Login with "user"/"password" displays Kanban board
- Login with wrong credentials shows error
- Logout button clears auth and returns to login
- Refresh page preserves auth session (token in localStorage)
- No auth token → cannot access Kanban

---

## Part 5: Database Schema & Architecture

**Objective**: Define database schema, document approach, get user approval. No implementation yet.

**Subtasks**:
- [ ] Define SQLite schema in docs/DATABASE.md:
  - [ ] users table: id (PK), username, password_hash (or hardcoded for MVP), created_at
  - [ ] boards table: id (PK), user_id (FK), title, created_at, updated_at
  - [ ] columns table: id (PK), board_id (FK), title, position, created_at, updated_at
  - [ ] cards table: id (PK), column_id (FK), title, details, position, created_at, updated_at
- [ ] Document relationship diagram (1 user → 1 board → 5 columns → N cards)
- [ ] Document data file path: ./data/kanban.db
- [ ] Document initialization: backend creates DB and schema if doesn't exist
- [ ] Document JSON export of board state (for AI context in Part 9)
- [ ] Create Python models/schemas using SQLAlchemy ORM
- [ ] Document migration strategy (none needed for MVP, schema created on startup)
- [ ] Get user sign-off on schema

**Testing Approach**:
- Design review by user
- No code testing yet (design approval phase)

**Success Criteria**:
- Database schema is documented and clear
- Relationships are well-defined
- User has approved schema design
- Schema supports all MVP features (multi-user ready)
- Initialization strategy is clear

---

## Part 6: Backend CRUD API

**Objective**: Implement API routes for board read/write. Create database, run migrations, thoroughly test.

**Subtasks**:
- [ ] Set up SQLAlchemy ORM with SQLite connection
- [ ] Create models (User, Board, Column, Card)
- [ ] Implement database initialization: `init_db()` creates tables on startup if needed
- [ ] Test database file created at ./data/kanban.db
- [ ] Implement API endpoint: GET /api/board - returns full board data (columns + cards) for authenticated user
  - [ ] Include all columns and their cards, ready for frontend rendering
  - [ ] Return 401 if not authenticated
- [ ] Implement API endpoint: POST /api/columns/{columnId}/cards - create new card
  - [ ] Accept JSON `{ title, details }`
  - [ ] Return new card object with id
  - [ ] Return 404 if column doesn't exist
- [ ] Implement API endpoint: DELETE /api/cards/{cardId} - delete card
  - [ ] Return 204 No Content on success
  - [ ] Return 404 if card doesn't exist
- [ ] Implement API endpoint: PATCH /api/columns/{columnId} - rename column
  - [ ] Accept JSON `{ title }`
  - [ ] Return updated column
- [ ] Implement API endpoint: PATCH /api/cards/{cardId}/move - move card to another column
  - [ ] Accept JSON `{ targetColumnId, position }`
  - [ ] Update card's column and order in database
  - [ ] Return updated card
- [ ] Test all endpoints with curl/Postman before frontend integration
- [ ] Test database persists data across app restarts
- [ ] Test error cases (404, 401, invalid input)
- [ ] Create backend unit tests in backend/tests/ directory (pytest)

**Testing Approach**:
- Pytest unit tests for each endpoint
- Test with invalid inputs, missing fields, auth failures
- Manual curl testing to verify responses
- Test database persistence
- Test initialization creates schema correctly

**Success Criteria**:
- All 6 API endpoints work correctly
- Database file is created on first startup
- Data persists across container restart
- All endpoints return correct status codes and data
- 80%+ test coverage for backend code
- No database errors in logs

---

## Part 7: Frontend ↔ Backend Integration

**Objective**: Replace in-memory state with API calls. Verify persistent, client-side working Kanban.

**Subtasks**:
- [ ] Update KanbanBoard.tsx to use API instead of useState:
  - [ ] On mount, call GET /api/board and populate state
  - [ ] Include Authorization header with token from auth context
- [ ] Update handleAddCard to call POST /api/columns/{columnId}/cards
  - [ ] Optimistic UI update, then sync response
  - [ ] Handle errors (show toast/alert if fail)
- [ ] Update handleDeleteCard to call DELETE /api/cards/{cardId}
  - [ ] Optimistic update, then confirm/revert on error
- [ ] Update handleRenameColumn to call PATCH /api/columns/{columnId}
- [ ] Update handleDragEnd to call PATCH /api/cards/{cardId}/move when card position changes
  - [ ] Send new columnId and position
  - [ ] Handle race conditions (multiple drags in flight)
- [ ] Add loading state and error handling to Kanban component
- [ ] Add loading spinner while initial board loads
- [ ] Test signup, login, see persistent board
- [ ] Test board data persists across page refresh
- [ ] Test add/delete/rename/move actions persist to database
- [ ] Test error handling (network down, API errors)
- [ ] Create E2E tests with backend (playwright) for full flows

**Testing Approach**:
- E2E tests: login → add card → refresh → card still there
- E2E tests: drag card → refresh → position persisted
- Error scenario tests (API down, permission denied)
- Load testing with concurrent operations
- Browser DevTools network tab to verify API calls

**Success Criteria**:
- All CRUD operations persist to database
- Page refresh shows same board data
- Drag-and-drop sends API calls and updates back-end
- Error handling works gracefully
- 100+ E2E tests passing
- No "data loss" on errors (rollback or user notification)

---

## Part 8: AI Connectivity Test

**Objective**: Verify OpenRouter API integration works. Test with simple "2+2" query.

**Subtasks**:
- [ ] Load OPENROUTER_API_KEY from .env in backend
- [ ] Create utility module: backend/ai.py with function `call_ai(prompt: str) → str`
- [ ] Use openai.OpenAI client with base_url="https://openrouter.ai/api/v1"
- [ ] Set model to `openai/gpt-oss-120b`
- [ ] Create test endpoint: POST /api/ai/test with JSON `{ prompt }`
  - [ ] Call AI with the prompt
  - [ ] Return JSON `{ response }`
- [ ] Test from command line: POST with prompt "What is 2+2?"
  - [ ] Verify response is correct (4 or similar)
  - [ ] Check response time (should be ~1-5 seconds)
- [ ] Test error handling (invalid API key, rate limit, timeout)
- [ ] Log all AI calls with prompt + response for debugging
- [ ] Verify no errors in backend logs

**Testing Approach**:
- Manual curl test of /api/ai/test endpoint
- Test with API key (should work) and blank key (should error)
- Verify response format is JSON
- Time the API call (should complete under 10 seconds)

**Success Criteria**:
- POST /api/ai/test returns correct response for "2+2"
- API key is properly loaded from .env
- OpenRouter API connectivity confirmed
- Error cases handled gracefully (no crash)
- Logs show request and response details

---

## Part 9: AI with Kanban Context & Structured Outputs

**Objective**: AI receives full board JSON + user message + conversation history. Returns message + optional board updates.

**Subtasks**:
- [ ] Define AI response schema (Pydantic model):
  ```
  AIResponse {
    message: str (response to user)
    boardUpdates: [ (optional)
      { action: "createCard", columnId: str, title: str, details: str },
      { action: "deleteCard", cardId: str },
      { action: "renameColumn", columnId: str, title: str },
      { action: "moveCard", cardId: str, targetColumnId: str, position: int }
    ]
  }
  ```
- [ ] Update backend/ai.py `call_ai()` to:
  - [ ] Accept board JSON as context
  - [ ] Accept user message (string)
  - [ ] Accept conversation history (list of messages)
  - [ ] Construct prompt with board context + history + new message
  - [ ] Use OpenRouter API with structured output schema (if supported) or parse JSON response
  - [ ] Return AIResponse object
- [ ] Create backend endpoint: POST /api/chat with JSON:
  ```
  { message: str, conversationHistory: [...] }
  ```
  - [ ] Fetch current board via GET /api/board
  - [ ] Call call_ai() with board + message + history
  - [ ] If boardUpdates in response, apply each update to database
  - [ ] Return AIResponse with updated board state
  - [ ] Allow up to 50 message history (limit for performance)
- [ ] Test AI correctly reads board state
- [ ] Test AI can request card creation with specific details
- [ ] Test AI can move cards or rename columns via structured outputs
- [ ] Test invalid AI responses don't crash backend
- [ ] Test conversation history is preserved across calls
- [ ] Create pytest tests for all scenarios

**Testing Approach**:
- Pytest unit tests for AI response parsing
- Integration tests: user message → board update → database persisted
- Example prompts:
  - "Create a card in backlog titled 'Test card'"
  - "Move all cards from Backlog to In Progress"
  - "What's the total number of cards?"
  - "Rename the Done column to Complete"
- Verify board state matches AI intent after each call

**Success Criteria**:
- POST /api/chat returns proper AIResponse JSON
- AI-requested board updates persist to database
- Conversation history is tracked and sent with each call
- Invalid responses handled gracefully (user sees error, no board corruption)
- All test scenarios pass
- No duplicate operations (idempotency)
- API calls succeed within 5-10 seconds

---

## Part 10: AI Chat Sidebar UI

**Objective**: Add beautiful chat sidebar to frontend. Support streaming responses and live board updates.

**Subtasks**:
- [ ] Create new frontend component: ChatSidebar.tsx
  - [ ] Collapsible sidebar on right side of Kanban
  - [ ] Message list showing user + AI messages
  - [ ] Auto-scroll to latest message
  - [ ] Timestamp on messages
- [ ] Create new frontend component: ChatInput.tsx
  - [ ] Text input + send button
  - [ ] Disable send while loading
  - [ ] Clear input after send
- [ ] Create new frontend component: MessageBubble.tsx
  - [ ] Different styles for user vs AI messages
  - [ ] Code blocks for JSON/structured responses
- [ ] Integrate ChatSidebar into main layout
- [ ] Add state management for conversation history
  - [ ] Store conversation in context or state
  - [ ] Persist to localStorage for session continuity
- [ ] Implement sending messages:
  - [ ] POST /api/chat with message + history
  - [ ] Show loading spinner while waiting
  - [ ] Display AI response in chat
- [ ] Implement live board updates:
  - [ ] If AI response includes boardUpdates, apply them to frontend state
  - [ ] Trigger re-render of Kanban board automatically
  - [ ] Show toast notification of changes (e.g., "Card created: Design flow")
- [ ] Handle AI errors gracefully:
  - [ ] Show error message in chat if API fails
  - [ ] Retry button
  - [ ] Don't lose conversation history
- [ ] Add keyboard shortcut to focus chat input (e.g., Cmd+K on Mac)
- [ ] Responsive design: hide chat sidebar on mobile, show toggle button
- [ ] Test all AI chat flows end-to-end
- [ ] Test concurrent operations (user dragging card while AI updates board)

**Testing Approach**:
- E2E tests: user message → AI response → board update visible
- E2E tests: multiple messages in sequence
- E2E tests: AI error handling and retry
- Manual browser testing of UI/UX
- Test sidebar responsiveness on mobile
- Test UI performance with large message history (100+ messages)

**Success Criteria**:
- Chat sidebar displays cleanly on right side of Kanban
- Messages appear instantly for user, within 3-5 seconds for AI
- AI can successfully update board; changes appear on Kanban immediately
- Conversation history is preserved on page refresh
- All error cases show user-friendly messages
- E2E tests confirm full chat → board update flow works
- No lag or performance issues on Kanban while chatting
- Mobile experience is good (responsive)

---

## Summary of Key Technical Decisions

| Decision | Value |
|----------|-------|
| Frontend | Next.js 16, React 19, Tailwind CSS 4 |
| Backend | Python FastAPI, uvicorn |
| Database | SQLite at ./data/kanban.db |
| AI Provider | OpenRouter (model: openai/gpt-oss-120b) |
| Package Manager (Python) | uv |
| Containerization | Docker + docker-compose |
| Auth | Hardcoded credentials ("user", "password") for MVP |
| Ports | Frontend: 3000 (dev), Backend: 8000 |
| Board Structure | 1 user → 1 board → 5 columns → N cards |

---

## Success Checklist by End of Project

- [ ] User can sign in with "user" / "password"
- [ ] Kanban board displays with 5 columns and example cards
- [ ] Drag-and-drop works; card position persists
- [ ] Cards can be added, deleted, renamed (via AI or manual)
- [ ] Columns can be renamed
- [ ] AI chat sidebar is accessible and functional
- [ ] User can ask AI to create/move/delete cards
- [ ] AI changes are reflected immediately on board
- [ ] All changes persist across page refresh and container restart
- [ ] No console errors or warnings
- [ ] Docker builds and runs on macOS, Windows, Linux
- [ ] Tests pass: frontend unit, E2E, backend unit
