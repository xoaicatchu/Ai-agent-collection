# OpenClaw - Chi Tiết Luồng Hoạt Động và Kiến Trúc

> Tài liệu kỹ thuật chi tiết về cách OpenClaw hoạt động khi được cài đặt trên máy người dùng, bao gồm luồng thực thi, quản lý skills, và xử lý codebase lớn.

---

## 📋 Mục Lục

1. [Tổng Quan Kiến Trúc](#1-tổng-quan-kiến-trúc)
2. [Quy Trình Cài Đặt và Khởi Động](#2-quy-trình-cài-đặt-và-khởi-động)
3. [Luồng Thực Thi Chi Tiết](#3-luồng-thực-thi-chi-tiết)
4. [Hệ Thống Skills - Cơ Chế Mở Rộng](#4-hệ-thống-skills---cơ-chế-mở-rộng)
5. [Context Management - Xử Lý Codebase Lớn](#5-context-management---xử-lý-codebase-lớn)
6. [Tool Layer - Cơ Chế Thực Thi](#6-tool-layer---cơ-chế-thực-thi)
7. [Use Cases Thực Tế](#7-use-cases-thực-tế)
8. [Performance và Optimization](#8-performance-và-optimization)

---

## 1. Tổng Quan Kiến Trúc

### 1.1 Kiến Trúc 3 Tầng (Three-Tier Architecture)

```
┌─────────────────────────────────────────────────────────────┐
│                    USER INTERFACE                            │
│              (Terminal, IDE Extension, Web UI)               │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│              TIER 1: GATEWAY RUNTIME (Local)                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Session    │  │    Tool      │  │   Context    │      │
│  │  Manager     │  │  Registry    │  │   Manager    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Skills     │  │   Message    │  │   Security   │      │
│  │   Loader     │  │   Router     │  │  Sandbox     │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└──────────────────────────┬──────────────────────────────────┘
                           │ API Calls (HTTPS)
                           ▼
┌─────────────────────────────────────────────────────────────┐
│           TIER 2: LLM RUNTIME (OpenAI Cloud)                 │
│  ┌──────────────────────────────────────────────────────┐   │
│  │         GPT-4 / GPT-4 Turbo Model                    │   │
│  │  - Reasoning Engine                                  │   │
│  │  - Tool Selection                                    │   │
│  │  - Code Generation                                   │   │
│  │  - Decision Making                                   │   │
│  └──────────────────────────────────────────────────────┘   │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│         TIER 3: EXTERNAL SYSTEMS & RESOURCES                 │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │  GitHub  │  │   APIs   │  │  Ngrok   │  │   Web    │   │
│  │   API    │  │          │  │ Tunnels  │  │ Services │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 Vai Trò Từng Component

#### **Gateway Runtime (Tier 1)**
**Vị trí:** Máy người dùng (Node.js process)
**Chức năng chính:**
- 🔧 **Tool Orchestration:** Điều phối thực thi các tools
- 📁 **File System Access:** Đọc/ghi/tìm kiếm files local
- 💾 **Session Management:** Quản lý trạng thái conversation
- 🎯 **Context Assembly:** Tổng hợp context gửi lên LLM
- 🛡️ **Security Enforcement:** Kiểm soát quyền truy cập
- 🧩 **Skills Loading:** Load và khởi tạo skills

**Tại sao cần Gateway?**
- LLM ở cloud không thể trực tiếp truy cập filesystem local
- Cần component local để thực thi commands, đọc code
- Đóng vai trò "tay chân" cho "bộ não" LLM

#### **LLM Runtime (Tier 2)**
**Vị trí:** OpenAI Cloud Infrastructure
**Chức năng chính:**
- 🧠 **Reasoning:** Suy luận logic, lập kế hoạch
- 🎯 **Tool Selection:** Quyết định tool nào cần gọi
- 📝 **Code Generation:** Tạo code, patches
- ✅ **Verification:** Đánh giá kết quả execution

**Tại sao không self-hosted?**
- Yêu cầu GPU mạnh, infrastructure phức tạp
- OpenAI cung cấp scale và reliability
- Cost-effective hơn cho majority use cases

#### **External Systems (Tier 3)**
- GitHub API (issues, PRs, repos)
- Third-party APIs
- Web services
- Database systems (qua tools)

---

## 2. Quy Trình Cài Đặt và Khởi Động

### 2.1 Installation Flow

```bash
# Bước 1: Cài đặt global package
npm install -g openclaw

# Cài đặt vào:
# Windows: C:\Users\<username>\AppData\Roaming\npm\node_modules\openclaw
# macOS/Linux: /usr/local/lib/node_modules/openclaw
```

**Những gì được cài đặt:**

```
openclaw/
├── bin/
│   └── openclaw              # CLI entry point
├── lib/
│   ├── gateway/              # Gateway runtime core
│   │   ├── session.js
│   │   ├── tools.js
│   │   ├── context.js
│   │   └── router.js
│   ├── llm/
│   │   ├── client.js         # OpenAI API client
│   │   └── formatter.js
│   └── utils/
├── skills/                   # Built-in skills
│   ├── coding-agent/
│   ├── github/
│   ├── session-logs/
│   └── ...
├── config/
│   ├── default.json          # Default configuration
│   └── tools.json            # Tool definitions
└── package.json
```

### 2.2 Startup Sequence

```javascript
// Khi chạy: openclaw

1. CLI Entry Point (bin/openclaw)
   ↓
2. Load Configuration
   - Đọc ~/.openclaw/config.json (user config)
   - Merge với config/default.json
   - Validate API keys, paths
   ↓
3. Initialize Gateway Runtime
   - Khởi tạo Session Manager
   - Load Tool Registry
   - Initialize Context Manager
   ↓
4. Load Skills
   - Scan skills/ directory
   - Parse SKILL.md cho metadata
   - Register skill handlers
   - Validate dependencies
   ↓
5. Initialize LLM Client
   - Setup OpenAI API connection
   - Configure retry logic, timeouts
   - Prepare system prompts
   ↓
6. Start REPL/Server
   - Listen for user input
   - Setup signal handlers (Ctrl+C)
   - Ready for commands
```

### 2.3 Configuration Files

**~/.openclaw/config.json**
```json
{
  "openai": {
    "apiKey": "sk-...",
    "model": "gpt-4-turbo-preview",
    "maxTokens": 4096,
    "temperature": 0.2
  },
  "gateway": {
    "port": 3000,
    "maxConcurrentTools": 5,
    "timeout": 300000
  },
  "skills": {
    "autoLoad": true,
    "customPaths": [
      "./my-custom-skills"
    ]
  },
  "context": {
    "maxFiles": 50,
    "maxTokensPerFile": 2000,
    "progressiveRetrieval": true
  }
}
```

---

## 3. Luồng Thực Thi Chi Tiết

### 3.1 The Agentic Loop - Vòng Lặp Chính

OpenClaw hoạt động theo **agentic loop** - một vòng lặp tự điều khiển, KHÔNG phải request-response đơn giản.

```
┌─────────────────────────────────────────────────────────────┐
│                    AGENTIC LOOP CYCLE                        │
└─────────────────────────────────────────────────────────────┘

Phase 1: USER INPUT
┌──────────────────┐
│  User enters     │
│  command/prompt  │
└────────┬─────────┘
         │
         ▼
Phase 2: CONTEXT ASSEMBLY (Gateway)
┌──────────────────────────────────────────┐
│ 1. Load session history                  │
│ 2. Gather relevant files (if any)        │
│ 3. Load active skills metadata           │
│ 4. Build system prompt                   │
│ 5. Assemble message payload              │
└────────┬─────────────────────────────────┘
         │
         ▼
Phase 3: LLM REASONING (OpenAI Cloud)
┌──────────────────────────────────────────┐
│ LLM analyzes:                            │
│ - User intent                            │
│ - Available tools/skills                 │
│ - Current context                        │
│                                          │
│ Makes decision:                          │
│ ┌─────────────┐  ┌─────────────┐       │
│ │  Need more  │  │   Can answer│       │
│ │  info/tools │  │   directly  │       │
│ └──────┬──────┘  └──────┬──────┘       │
└────────┼─────────────────┼──────────────┘
         │                 │
         │                 └─────────────────┐
         ▼                                   ▼
Phase 4A: TOOL EXECUTION              Phase 4B: FINAL ANSWER
┌──────────────────────────┐          ┌──────────────────┐
│ LLM requests tools:      │          │ Return response  │
│ [                        │          │ to user          │
│   {                      │          └──────────────────┘
│     "tool": "read",      │                   │
│     "args": {            │                   │
│       "path": "..."      │                   ▼
│     }                    │              ┌─────────┐
│   },                     │              │   END   │
│   ...                    │              └─────────┘
│ ]                        │
└──────────┬───────────────┘
           │
           ▼
Phase 5: GATEWAY EXECUTION
┌────────────────────────────────────┐
│ For each tool request:             │
│ 1. Validate permissions            │
│ 2. Dispatch to tool handler        │
│ 3. Execute (read file, run cmd...) │
│ 4. Capture output/error            │
│ 5. Format result                   │
└──────────┬─────────────────────────┘
           │
           ▼
Phase 6: RESULT AGGREGATION
┌────────────────────────────────────┐
│ Collect all tool outputs:          │
│ {                                  │
│   "tool_results": [                │
│     {                              │
│       "tool": "read",              │
│       "output": "...",             │
│       "status": "success"          │
│     },                             │
│     ...                            │
│   ]                                │
│ }                                  │
└──────────┬─────────────────────────┘
           │
           │ ┌─────────────────────────┐
           └─┤ LOOP BACK TO PHASE 3   │
             │ (với updated context)   │
             └─────────────────────────┘
```

### 3.2 Detailed Message Flow

#### **Iteration 1: Initial Request**

```javascript
// USER INPUT
"Fix the bug in login.js where password validation fails"

// GATEWAY → LLM
{
  "model": "gpt-4-turbo-preview",
  "messages": [
    {
      "role": "system",
      "content": `You are OpenClaw, an AI coding assistant.

Available tools:
- read(path): Read file content
- search(pattern, dir): Search for patterns
- edit(path, changes): Edit files
- exec(command): Run shell commands
- grep(pattern): Search in files

Available skills:
- coding-agent: Multi-file code changes
- github: GitHub integration

Current working directory: /home/user/myproject
`
    },
    {
      "role": "user",
      "content": "Fix the bug in login.js where password validation fails"
    }
  ],
  "tools": [...] // Tool definitions
}

// LLM RESPONSE (Iteration 1)
{
  "role": "assistant",
  "content": null,
  "tool_calls": [
    {
      "id": "call_abc123",
      "type": "function",
      "function": {
        "name": "read",
        "arguments": "{\"path\": \"login.js\"}"
      }
    },
    {
      "id": "call_def456",
      "type": "function",
      "function": {
        "name": "grep",
        "arguments": "{\"pattern\": \"password.*validation\", \"extensions\": [\".js\"]}"
      }
    }
  ]
}
```

#### **Tool Execution Phase**

```javascript
// GATEWAY EXECUTES TOOLS

// Tool 1: read("login.js")
{
  "tool_call_id": "call_abc123",
  "output": `
1  const validateLogin = (username, password) => {
2    if (!username || username.length < 3) {
3      return { valid: false, error: 'Username too short' };
4    }
5    if (!password || password.length < 8) {  // BUG: Always returns false
6      return { valid: true, error: null };     // Should return false!
7    }
8    return { valid: true, error: null };
9  };
  `,
  "status": "success"
}

// Tool 2: grep("password.*validation")
{
  "tool_call_id": "call_def456",
  "output": `
login.js:5:  if (!password || password.length < 8) {
login.js:6:    return { valid: true, error: null };
tests/login.test.js:15:  expect(validateLogin('user', 'short').valid).toBe(false);
  `,
  "status": "success"
}
```

#### **Iteration 2: Analysis and Fix**

```javascript
// GATEWAY → LLM (với tool results)
{
  "messages": [
    // ... previous messages ...
    {
      "role": "assistant",
      "tool_calls": [...]  // from iteration 1
    },
    {
      "role": "tool",
      "tool_call_id": "call_abc123",
      "content": "..." // read output
    },
    {
      "role": "tool",
      "tool_call_id": "call_def456",
      "content": "..." // grep output
    }
  ]
}

// LLM RESPONSE (Iteration 2)
{
  "role": "assistant",
  "content": null,
  "tool_calls": [
    {
      "id": "call_ghi789",
      "type": "function",
      "function": {
        "name": "edit",
        "arguments": `{
          "path": "login.js",
          "changes": [
            {
              "start_line": 6,
              "end_line": 6,
              "new_content": "    return { valid: false, error: 'Password too short' };"
            }
          ]
        }`
      }
    },
    {
      "id": "call_jkl012",
      "type": "function",
      "function": {
        "name": "exec",
        "arguments": "{\"command\": \"npm test -- login.test.js\"}"
      }
    }
  ]
}
```

#### **Iteration 3: Verification and Final Response**

```javascript
// GATEWAY EXECUTES TOOLS
// Edit applied, tests run...

// Test output
{
  "tool_call_id": "call_jkl012",
  "output": "✓ All tests passed (5/5)",
  "status": "success"
}

// LLM RESPONSE (Iteration 3 - FINAL)
{
  "role": "assistant",
  "content": `I've fixed the bug in login.js.

**Root Cause:**
Line 6 was returning \`valid: true\` when password was too short, should be \`false\`.

**Changes Made:**
- login.js:6 - Fixed return value to \`{ valid: false, error: 'Password too short' }\`

**Verification:**
✓ All tests passing (npm test)

The password validation now correctly rejects passwords shorter than 8 characters.`,
  "tool_calls": null
}

// GATEWAY → USER (final answer displayed)
```

### 3.3 Loop Termination Conditions

```javascript
// Loop kết thúc khi:

1. ✅ LLM returns text response (no more tool calls)
   → Success case

2. ⏱️ Timeout exceeded (default: 5 minutes)
   → Returns partial progress + timeout message

3. 🔴 Tool execution error + LLM can't recover
   → Returns error explanation

4. 🛑 User interrupts (Ctrl+C)
   → Graceful shutdown, saves session state

5. 💰 Token limit reached (safety limit)
   → Returns "Task too complex, please simplify"

6. 🔄 Max iterations exceeded (default: 20)
   → Prevents infinite loops
```

---

## 4. Hệ Thống Skills - Cơ Chế Mở Rộng

### 4.1 Skill Architecture

Skills là **cơ chế mở rộng chức năng** của OpenClaw, tương tự như plugins.

```
┌────────────────────────────────────────────────────────────┐
│                     SKILL ANATOMY                           │
└────────────────────────────────────────────────────────────┘

my-skill/
├── SKILL.md                    # REQUIRED: Metadata + Docs
├── handler.js                  # Optional: Custom logic
├── prompts/
│   ├── system.txt              # System prompt additions
│   └── examples.txt            # Few-shot examples
├── tools/
│   ├── custom-tool-1.js        # Custom tools
│   └── custom-tool-2.js
├── config.json                 # Skill-specific config
├── assets/
│   └── templates/              # Code templates, etc.
└── tests/
    └── skill.test.js
```

### 4.2 SKILL.md Format

```markdown
---
name: coding-agent
version: 1.2.0
author: OpenClaw Team
requires:
  - tools: [read, edit, search, exec]
  - skills: []
triggers:
  - "fix bug"
  - "refactor"
  - "implement feature"
---

# Coding Agent Skill

## Scope
This skill handles multi-file code changes, bug fixes, and refactoring.

**When to use:**
- User asks to fix a bug
- User requests feature implementation
- Code refactoring needed

**When NOT to use:**
- Simple file reads (use read tool directly)
- Documentation questions
- Configuration changes only

## Workflow

1. **Discovery Phase**
   - Search for relevant files
   - Read entry points
   - Map dependencies

2. **Analysis Phase**
   - Identify root cause (for bugs)
   - Plan changes across files
   - Check for side effects

3. **Implementation Phase**
   - Apply edits in dependency order
   - Run tests after each change
   - Verify build succeeds

4. **Verification Phase**
   - Run full test suite
   - Check linter
   - Generate summary

## Input Format
{
  "task": "Fix login validation bug",
  "context": {
    "files": ["login.js"],
    "error_message": "...",
    "priority": "high"
  }
}

## Output Format
{
  "status": "success",
  "changes": [
    { "file": "login.js", "lines": [5, 6] }
  ],
  "verification": {
    "tests": "passing",
    "build": "success"
  },
  "summary": "Fixed password validation logic..."
}

## Safety Guardrails
- Always run tests before completing
- Never delete files without explicit permission
- Preserve code comments and formatting
- Create backup before major refactors

## Examples
[... detailed examples ...]
```

### 4.3 Skill Loading Process

```javascript
// Khi Gateway khởi động

class SkillLoader {
  async loadAllSkills() {
    const skillDirs = [
      './skills',                    // Built-in skills
      '~/.openclaw/skills',          // User-installed skills
      ...config.skills.customPaths   // Custom paths
    ];

    for (const dir of skillDirs) {
      const skills = await this.scanDirectory(dir);

      for (const skillPath of skills) {
        await this.loadSkill(skillPath);
      }
    }
  }

  async loadSkill(skillPath) {
    // Step 1: Parse SKILL.md
    const metadata = await this.parseSkillMarkdown(
      path.join(skillPath, 'SKILL.md')
    );

    // Step 2: Validate dependencies
    const missingTools = metadata.requires.tools.filter(
      tool => !this.toolRegistry.has(tool)
    );

    if (missingTools.length > 0) {
      throw new Error(
        `Skill ${metadata.name} requires missing tools: ${missingTools}`
      );
    }

    // Step 3: Load custom handler (if exists)
    let handler = null;
    const handlerPath = path.join(skillPath, 'handler.js');
    if (fs.existsSync(handlerPath)) {
      handler = require(handlerPath);
    }

    // Step 4: Load custom tools
    const toolsDir = path.join(skillPath, 'tools');
    if (fs.existsSync(toolsDir)) {
      const customTools = await this.loadCustomTools(toolsDir);
      customTools.forEach(tool => this.toolRegistry.register(tool));
    }

    // Step 5: Load prompts
    const systemPrompt = await this.loadPrompt(
      path.join(skillPath, 'prompts/system.txt')
    );
    const examples = await this.loadPrompt(
      path.join(skillPath, 'prompts/examples.txt')
    );

    // Step 6: Register skill
    this.skillRegistry.register({
      name: metadata.name,
      metadata,
      handler,
      systemPrompt,
      examples,
      path: skillPath
    });

    console.log(`✓ Loaded skill: ${metadata.name} v${metadata.version}`);
  }
}
```

### 4.4 Skill Activation Flow

```javascript
// Khi user input match skill trigger

User: "Fix the login bug"
  ↓
Gateway: Pattern matching
  - Checks triggers in all loaded skills
  - "fix" + "bug" → matches "coding-agent" skill
  ↓
Gateway: Build enhanced context
  {
    "system_prompt": BASE_SYSTEM_PROMPT + skill.systemPrompt,
    "tools": BASE_TOOLS + skill.customTools,
    "examples": skill.examples,
    "skill_metadata": {
      "active_skill": "coding-agent",
      "workflow": skill.metadata.workflow
    }
  }
  ↓
LLM: Now has skill-specific context
  - Understands skill workflow
  - Has access to custom tools
  - Follows skill guidelines
  ↓
Execution: Follows skill's defined workflow
```

### 4.5 Installing New Skills

#### **From ClawHub (Marketplace)**

```bash
# Step 1: Browse marketplace
npx clawhub search "pdf processing"

# Output:
# ┌────────────────────────────────────────────────┐
# │ nano-pdf                                       │
# │ PDF extraction and processing                  │
# │ ⭐ 1.2k  📥 5.6k  📅 Updated 2 days ago        │
# └────────────────────────────────────────────────┘

# Step 2: Inspect skill details
npx clawhub inspect nano-pdf

# Output shows:
# - Full SKILL.md content
# - Dependencies
# - Version history
# - Security audit status

# Step 3: Install
npx clawhub install nano-pdf --dir ~/.openclaw/skills

# Installation process:
# 1. Download skill package
# 2. Verify signature (security)
# 3. Check dependencies
# 4. Extract to skills directory
# 5. Run post-install hooks (if any)
# 6. Validate SKILL.md
```

#### **Manual Installation**

```bash
# Step 1: Create skill directory
mkdir -p ~/.openclaw/skills/my-custom-skill

# Step 2: Create SKILL.md
cat > ~/.openclaw/skills/my-custom-skill/SKILL.md << 'EOF'
---
name: my-custom-skill
version: 1.0.0
requires:
  tools: [read, exec]
triggers:
  - "custom action"
---

# My Custom Skill
[... documentation ...]
EOF

# Step 3: Add handler (optional)
cat > ~/.openclaw/skills/my-custom-skill/handler.js << 'EOF'
module.exports = {
  async execute(context, tools) {
    // Custom logic here
    const result = await tools.read('somefile.txt');
    return { status: 'success', data: result };
  }
};
EOF

# Step 4: Restart OpenClaw or reload skills
openclaw skills reload
```

### 4.6 Skill Execution Example

**Scenario:** User wants to extract text from PDF

```javascript
// Without skill: Manual tool orchestration
User: "Extract text from report.pdf"
↓
LLM: "I cannot directly read PDFs. You need a PDF library."
❌ Cannot complete task

// With nano-pdf skill installed:
User: "Extract text from report.pdf"
↓
Gateway: Detects "pdf" trigger → activates nano-pdf skill
↓
LLM (with skill context):
  tool_calls: [
    {
      function: "pdf_extract_text",  // Custom tool from skill
      arguments: { path: "report.pdf" }
    }
  ]
↓
Gateway: Executes skill's custom tool
  - Uses pdf-parse library
  - Extracts text, images, metadata
↓
LLM: Receives structured data
  {
    text: "...",
    pages: 15,
    metadata: {...}
  }
↓
User: Gets formatted output
✅ Task completed
```

---

## 5. Context Management - Xử Lý Codebase Lớn

### 5.1 The Challenge

```
❌ Naive Approach (KHÔNG khả thi):
Load all 500 files → Send to LLM → OOM / Token limit exceeded

✅ OpenClaw Approach:
Progressive Retrieval + Context Budgeting
```

### 5.2 Progressive Retrieval Strategy

```
┌────────────────────────────────────────────────────────────┐
│            PROGRESSIVE RETRIEVAL PHASES                     │
└────────────────────────────────────────────────────────────┘

PHASE 1: DISCOVERY (Lightweight)
Goal: Map the codebase without reading content
┌─────────────────────────────────────────────┐
│ Tools Used:                                 │
│ - search: Find files by path/name pattern  │
│ - grep: Find files containing keywords     │
│ - tree: Directory structure               │
│                                             │
│ Example:                                    │
│ search("*.js", "src/")                      │
│ → Returns: 487 files                        │
│                                             │
│ grep("class.*Login", "*.js")                │
│ → Returns: 12 files containing "Login"      │
└─────────────────────────────────────────────┘
         │
         ▼
PHASE 2: PRIORITIZATION (Smart Filtering)
Goal: Rank files by relevance
┌─────────────────────────────────────────────┐
│ Ranking Factors:                            │
│ 1. Keyword matches (from grep)              │
│ 2. File name similarity to query            │
│ 3. Recent modifications (git log)           │
│ 4. Dependency graph centrality              │
│ 5. Error stack trace mentions              │
│                                             │
│ Example:                                    │
│ Query: "Fix login validation bug"           │
│                                             │
│ High Priority (read first):                 │
│ - src/auth/login.js (name match + grep)     │
│ - src/auth/validation.js (name match)       │
│ - tests/auth/login.test.js (related)        │
│                                             │
│ Medium Priority:                            │
│ - src/auth/index.js (entry point)           │
│ - src/utils/validators.js (dependency)      │
│                                             │
│ Low Priority (skip for now):                │
│ - src/components/LoginUI.js (UI only)       │
│ - 480+ other files                          │
└─────────────────────────────────────────────┘
         │
         ▼
PHASE 3: TARGETED READ (Budget-Aware)
Goal: Read only essential content
┌─────────────────────────────────────────────┐
│ Token Budget: 4000 tokens (~16KB)           │
│                                             │
│ Iteration 1:                                │
│ read("src/auth/login.js")        → 450 tok  │
│ read("src/auth/validation.js")   → 320 tok  │
│ read("tests/auth/login.test.js") → 280 tok  │
│ ─────────────────────────────────────────   │
│ Total: 1050 tokens                          │
│ Remaining budget: 2950 tokens               │
│                                             │
│ LLM Analysis:                               │
│ "Need to check validators.js for helper     │
│  functions used in validation.js"           │
│                                             │
│ Iteration 2:                                │
│ read("src/utils/validators.js")  → 510 tok  │
│ ─────────────────────────────────────────   │
│ Total: 1560 tokens                          │
│ Remaining budget: 2440 tokens               │
│                                             │
│ LLM: "I have enough context to fix"         │
└─────────────────────────────────────────────┘
         │
         ▼
PHASE 4: ITERATIVE EXPANSION (As Needed)
Goal: Expand context only when necessary
┌─────────────────────────────────────────────┐
│ LLM discovers: "validation.js imports       │
│ checkPasswordStrength from password-utils"  │
│                                             │
│ Iteration 3:                                │
│ read("src/utils/password-utils.js") → 380   │
│                                             │
│ LLM: "Found the bug! No need for more files"│
└─────────────────────────────────────────────┘
         │
         ▼
PHASE 5: CONTEXT COMPRESSION (If Needed)
Goal: Summarize old context to make room
┌─────────────────────────────────────────────┐
│ If token limit approaching:                 │
│                                             │
│ Old context (login.test.js):                │
│ [Full 280 token content]                    │
│                                             │
│ Compressed to summary:                      │
│ "login.test.js: Tests for validateLogin(),  │
│  expects passwords < 8 chars to fail"       │
│ → 25 tokens                                 │
│                                             │
│ Saved: 255 tokens                           │
└─────────────────────────────────────────────┘
```

### 5.3 Real-World Example: Debugging in 500-File Codebase

```javascript
// Scenario: "Fix TypeError in checkout process"

// ════════════════════════════════════════════════════════
// ITERATION 1: Initial Discovery
// ════════════════════════════════════════════════════════

User: "Getting TypeError: Cannot read property 'price' of undefined
       in checkout. Please fix."

Gateway → LLM:
{
  context: {
    working_dir: "/home/user/ecommerce-app",
    file_count: 523,
    error: "TypeError: Cannot read property 'price' of undefined"
  }
}

LLM → Gateway:
tool_calls: [
  {
    function: "grep",
    args: {
      pattern: "Cannot read property 'price'",
      extensions: [".js", ".ts"]
    }
  },
  {
    function: "grep",
    args: {
      pattern: "checkout",
      path: "src/",
      extensions: [".js"]
    }
  }
]

Gateway executes:
grep results:
  src/checkout/CartSummary.js:45: const total = item.price * item.quantity;
  src/checkout/OrderProcessor.js:78: price: item.price,

search results:
  src/checkout/CartSummary.js
  src/checkout/OrderProcessor.js
  src/checkout/CheckoutForm.js
  src/checkout/index.js
  (+ 8 more checkout-related files)

Token usage: ~200 tokens (just file paths and snippets)

// ════════════════════════════════════════════════════════
// ITERATION 2: Targeted Read
// ════════════════════════════════════════════════════════

LLM → Gateway:
"I found potential error locations. Reading relevant files."

tool_calls: [
  { function: "read", args: { path: "src/checkout/CartSummary.js" }},
  { function: "read", args: { path: "src/checkout/OrderProcessor.js" }}
]

Gateway executes:
CartSummary.js content (45 lines):
  ```javascript
  const calculateTotal = (items) => {
    return items.map(item => {
      const total = item.price * item.quantity;  // Line 45 - ERROR HERE
      return total;
    });
  };
  ```

OrderProcessor.js content (120 lines):
  ```javascript
  const processOrder = (cartItems) => {
    const orderLines = cartItems.map(item => ({
      product_id: item.id,
      price: item.price,  // Line 78
      quantity: item.quantity
    }));
  };
  ```

Token usage: ~850 tokens (2 files)
Total so far: 1050 tokens

// ════════════════════════════════════════════════════════
// ITERATION 3: Root Cause Analysis
// ════════════════════════════════════════════════════════

LLM analysis:
"The error is in CartSummary.js:45. The items array likely contains
 undefined elements. I need to see where items comes from."

tool_calls: [
  {
    function: "grep",
    args: {
      pattern: "calculateTotal\\(",
      path: "src/"
    }
  }
]

Gateway executes:
  src/checkout/CartSummary.js:23: const totals = calculateTotal(cartItems);
  src/checkout/index.js:15: import { calculateTotal } from './CartSummary';

LLM → Gateway:
tool_calls: [
  { function: "read", args: { path: "src/checkout/index.js" }}
]

index.js content:
  ```javascript
  const getCartItems = () => {
    const cart = getCart();
    // BUG: filter returns array with undefined holes
    return cart.items.filter(item => item !== null);
    // Should use: .filter(item => item && item.price !== undefined)
  };
  ```

Token usage: ~400 tokens
Total: 1450 tokens

// ════════════════════════════════════════════════════════
// ITERATION 4: Fix Applied
// ════════════════════════════════════════════════════════

LLM → Gateway:
"Found root cause! The filter in index.js doesn't check for undefined.
 Applying fix..."

tool_calls: [
  {
    function: "edit",
    args: {
      path: "src/checkout/index.js",
      changes: [{
        line: 15,
        old: "return cart.items.filter(item => item !== null);",
        new: "return cart.items.filter(item => item && item.price !== undefined);"
      }]
    }
  },
  {
    function: "exec",
    args: { command: "npm test -- checkout" }
  }
]

Test output: ✅ All passing

LLM → User:
"Fixed! The bug was in src/checkout/index.js. The filter was only
 checking for null but not undefined, allowing invalid items into
 the cart calculation."

// ════════════════════════════════════════════════════════
// FINAL STATS
// ════════════════════════════════════════════════════════
Files in codebase: 523
Files read: 3 (CartSummary.js, OrderProcessor.js, index.js)
Files searched: All (via grep - lightweight)
Total tokens used: ~2100
Iterations: 4
Time: ~45 seconds

Efficiency: Read 0.57% of files, found and fixed bug ✅
```

### 5.4 Context Budget Management

```javascript
class ContextManager {
  constructor(config) {
    this.maxTokens = config.maxTokensPerRequest || 4000;
    this.maxFilesPerIteration = config.maxFilesPerIteration || 10;
    this.compressionThreshold = config.compressionThreshold || 0.8;
  }

  async buildContext(session, newToolResults) {
    let context = {
      system: this.buildSystemPrompt(),
      history: [],
      files: new Map(),
      tokenCount: 0
    };

    // 1. Add system prompt (always included)
    context.tokenCount += this.estimateTokens(context.system);

    // 2. Add recent conversation history (sliding window)
    const recentMessages = session.messages.slice(-10); // Last 10 messages
    for (const msg of recentMessages) {
      const msgTokens = this.estimateTokens(msg.content);

      if (context.tokenCount + msgTokens > this.maxTokens * this.compressionThreshold) {
        // Compress old messages
        msg.content = this.summarizeMessage(msg.content);
      }

      context.history.push(msg);
      context.tokenCount += this.estimateTokens(msg.content);
    }

    // 3. Add tool results from current iteration
    for (const result of newToolResults) {
      const tokens = this.estimateTokens(result.output);

      if (result.tool === 'read') {
        // Special handling for file reads
        if (tokens > 2000) {
          // Truncate large files
          result.output = this.truncateFile(result.output, 1500);
        }
      }

      context.tokenCount += this.estimateTokens(result.output);
    }

    // 4. Check if we're over budget
    if (context.tokenCount > this.maxTokens) {
      context = await this.compressContext(context);
    }

    return context;
  }

  async compressContext(context) {
    // Strategy 1: Summarize old file contents
    for (const [path, content] of context.files.entries()) {
      if (content.age > 2) { // Older than 2 iterations
        const summary = await this.summarizeFile(path, content.text);
        context.files.set(path, { ...content, text: summary, summarized: true });
      }
    }

    // Strategy 2: Remove least relevant files
    if (context.tokenCount > this.maxTokens) {
      const sortedFiles = Array.from(context.files.entries())
        .sort((a, b) => a[1].relevance - b[1].relevance);

      while (context.tokenCount > this.maxTokens && sortedFiles.length > 0) {
        const [path, _] = sortedFiles.shift();
        context.files.delete(path);
        context.tokenCount = this.recalculateTokens(context);
      }
    }

    return context;
  }

  estimateTokens(text) {
    // Rough estimation: 1 token ≈ 4 characters
    return Math.ceil(text.length / 4);
  }
}
```

### 5.5 Dependency-Aware Reading

```javascript
// OpenClaw tự động theo dõi dependencies

// Example: Reading a React component

LLM reads: src/components/LoginForm.js
Content:
  import { validateEmail } from '../utils/validators';
  import { useAuth } from '../hooks/useAuth';

Gateway detects imports:
  - ../utils/validators.js
  - ../hooks/useAuth.js

Gateway builds dependency graph:
  LoginForm.js
    ├─ utils/validators.js
    └─ hooks/useAuth.js
         └─ contexts/AuthContext.js

LLM can request:
  "Read dependencies of LoginForm.js"

Gateway intelligently reads in order:
  1. AuthContext.js (leaf dependency)
  2. useAuth.js (depends on AuthContext)
  3. validators.js (independent)
  4. LoginForm.js (root)

This ensures LLM understands dependencies before reading dependent code.
```

---

## 6. Tool Layer - Cơ Chế Thực Thi

### 6.1 Tool Registry Architecture

```javascript
class ToolRegistry {
  constructor() {
    this.tools = new Map();
    this.loadBuiltinTools();
  }

  loadBuiltinTools() {
    // File operations
    this.register({
      name: 'read',
      description: 'Read file content',
      parameters: {
        type: 'object',
        properties: {
          path: { type: 'string', description: 'File path' },
          start_line: { type: 'number', description: 'Start line (optional)' },
          end_line: { type: 'number', description: 'End line (optional)' }
        },
        required: ['path']
      },
      handler: async (args) => {
        const content = await fs.readFile(args.path, 'utf8');
        if (args.start_line && args.end_line) {
          const lines = content.split('\n');
          return lines.slice(args.start_line - 1, args.end_line).join('\n');
        }
        return content;
      },
      permissions: ['filesystem:read']
    });

    this.register({
      name: 'edit',
      description: 'Edit file content',
      parameters: {
        type: 'object',
        properties: {
          path: { type: 'string' },
          changes: {
            type: 'array',
            items: {
              type: 'object',
              properties: {
                start_line: { type: 'number' },
                end_line: { type: 'number' },
                new_content: { type: 'string' }
              }
            }
          }
        },
        required: ['path', 'changes']
      },
      handler: async (args) => {
        const content = await fs.readFile(args.path, 'utf8');
        const lines = content.split('\n');

        // Apply changes in reverse order to maintain line numbers
        const sortedChanges = args.changes.sort((a, b) => b.start_line - a.start_line);

        for (const change of sortedChanges) {
          lines.splice(
            change.start_line - 1,
            change.end_line - change.start_line + 1,
            ...change.new_content.split('\n')
          );
        }

        await fs.writeFile(args.path, lines.join('\n'), 'utf8');
        return { status: 'success', lines_changed: args.changes.length };
      },
      permissions: ['filesystem:write']
    });

    // Search operations
    this.register({
      name: 'search',
      description: 'Search for files by pattern',
      parameters: {
        type: 'object',
        properties: {
          pattern: { type: 'string', description: 'Glob pattern like *.js' },
          dir: { type: 'string', description: 'Directory to search in' }
        },
        required: ['pattern']
      },
      handler: async (args) => {
        const { default: glob } = await import('glob');
        const files = await glob(args.pattern, { cwd: args.dir || process.cwd() });
        return files.join('\n');
      },
      permissions: ['filesystem:read']
    });

    this.register({
      name: 'grep',
      description: 'Search for pattern in files',
      parameters: {
        type: 'object',
        properties: {
          pattern: { type: 'string' },
          path: { type: 'string' },
          extensions: { type: 'array', items: { type: 'string' } },
          context_lines: { type: 'number', description: 'Lines of context around match' }
        },
        required: ['pattern']
      },
      handler: async (args) => {
        // Use ripgrep or fallback to grep
        const cmd = `rg "${args.pattern}" ${args.path || '.'} \
          ${args.extensions ? args.extensions.map(e => `-g "*${e}"`).join(' ') : ''} \
          ${args.context_lines ? `-C ${args.context_lines}` : ''}`;

        const { stdout } = await execAsync(cmd);
        return stdout;
      },
      permissions: ['filesystem:read', 'system:exec']
    });

    // Execution
    this.register({
      name: 'exec',
      description: 'Execute shell command',
      parameters: {
        type: 'object',
        properties: {
          command: { type: 'string' },
          cwd: { type: 'string' },
          timeout: { type: 'number', description: 'Timeout in ms' }
        },
        required: ['command']
      },
      handler: async (args) => {
        const { stdout, stderr } = await execAsync(args.command, {
          cwd: args.cwd || process.cwd(),
          timeout: args.timeout || 30000,
          maxBuffer: 1024 * 1024 * 10 // 10MB
        });

        return {
          stdout: stdout.trim(),
          stderr: stderr.trim(),
          exit_code: 0
        };
      },
      permissions: ['system:exec']
    });

    // Git operations
    this.register({
      name: 'git',
      description: 'Git operations',
      parameters: {
        type: 'object',
        properties: {
          operation: {
            type: 'string',
            enum: ['status', 'diff', 'log', 'blame', 'show']
          },
          args: { type: 'object' }
        },
        required: ['operation']
      },
      handler: async (args) => {
        const operations = {
          status: async () => {
            const { stdout } = await execAsync('git status --short');
            return stdout;
          },
          diff: async (opts) => {
            const { stdout } = await execAsync(
              `git diff ${opts.file || ''} ${opts.staged ? '--staged' : ''}`
            );
            return stdout;
          },
          log: async (opts) => {
            const { stdout } = await execAsync(
              `git log --oneline -n ${opts.limit || 10} ${opts.file || ''}`
            );
            return stdout;
          },
          blame: async (opts) => {
            const { stdout } = await execAsync(
              `git blame -L ${opts.start_line},${opts.end_line} ${opts.file}`
            );
            return stdout;
          },
          show: async (opts) => {
            const { stdout } = await execAsync(`git show ${opts.commit}`);
            return stdout;
          }
        };

        return await operations[args.operation](args.args || {});
      },
      permissions: ['system:exec', 'filesystem:read']
    });
  }

  register(tool) {
    this.tools.set(tool.name, tool);
  }

  get(name) {
    return this.tools.get(name);
  }

  getAll() {
    return Array.from(this.tools.values());
  }

  toOpenAIFormat() {
    // Convert tools to OpenAI function calling format
    return this.getAll().map(tool => ({
      type: 'function',
      function: {
        name: tool.name,
        description: tool.description,
        parameters: tool.parameters
      }
    }));
  }
}
```

### 6.2 Tool Execution with Error Handling

```javascript
class ToolExecutor {
  async execute(toolCall, permissions) {
    const tool = toolRegistry.get(toolCall.function.name);

    if (!tool) {
      return {
        status: 'error',
        error: `Unknown tool: ${toolCall.function.name}`
      };
    }

    // 1. Permission check
    const missingPermissions = tool.permissions.filter(
      perm => !permissions.has(perm)
    );

    if (missingPermissions.length > 0) {
      return {
        status: 'error',
        error: `Missing permissions: ${missingPermissions.join(', ')}`
      };
    }

    // 2. Parse arguments
    let args;
    try {
      args = JSON.parse(toolCall.function.arguments);
    } catch (e) {
      return {
        status: 'error',
        error: `Invalid arguments: ${e.message}`
      };
    }

    // 3. Validate arguments
    const validation = this.validateArgs(args, tool.parameters);
    if (!validation.valid) {
      return {
        status: 'error',
        error: `Validation failed: ${validation.errors.join(', ')}`
      };
    }

    // 4. Execute with timeout
    try {
      const result = await Promise.race([
        tool.handler(args),
        new Promise((_, reject) =>
          setTimeout(() => reject(new Error('Tool execution timeout')), 60000)
        )
      ]);

      return {
        status: 'success',
        output: result
      };

    } catch (error) {
      // 5. Error handling
      if (error.code === 'ENOENT') {
        return {
          status: 'error',
          error: `File not found: ${args.path || 'unknown'}`
        };
      }

      if (error.code === 'EACCES') {
        return {
          status: 'error',
          error: `Permission denied: ${args.path || 'unknown'}`
        };
      }

      // Log full error for debugging
      console.error(`Tool execution error:`, error);

      return {
        status: 'error',
        error: error.message,
        stack: process.env.DEBUG ? error.stack : undefined
      };
    }
  }

  validateArgs(args, schema) {
    // Simple JSON schema validation
    const errors = [];

    for (const [key, prop] of Object.entries(schema.properties || {})) {
      if (schema.required?.includes(key) && !(key in args)) {
        errors.push(`Missing required parameter: ${key}`);
      }

      if (key in args && typeof args[key] !== prop.type) {
        errors.push(`Parameter ${key} must be ${prop.type}, got ${typeof args[key]}`);
      }
    }

    return {
      valid: errors.length === 0,
      errors
    };
  }
}
```

### 6.3 Parallel Tool Execution

```javascript
// LLM có thể request nhiều tools cùng lúc

LLM response:
{
  tool_calls: [
    { id: "1", function: { name: "read", arguments: '{"path": "a.js"}' }},
    { id: "2", function: { name: "read", arguments: '{"path": "b.js"}' }},
    { id: "3", function: { name: "read", arguments: '{"path": "c.js"}' }},
    { id: "4", function: { name: "grep", arguments: '{"pattern": "TODO"}' }}
  ]
}

Gateway execution:
async function executeTools(toolCalls) {
  // Execute all in parallel for performance
  const results = await Promise.all(
    toolCalls.map(async (call) => {
      try {
        const result = await toolExecutor.execute(call, currentPermissions);
        return {
          tool_call_id: call.id,
          role: 'tool',
          content: JSON.stringify(result.output)
        };
      } catch (error) {
        return {
          tool_call_id: call.id,
          role: 'tool',
          content: JSON.stringify({
            error: error.message
          })
        };
      }
    })
  );

  return results;
}

// Results returned to LLM:
[
  { tool_call_id: "1", content: "/* content of a.js */" },
  { tool_call_id: "2", content: "/* content of b.js */" },
  { tool_call_id: "3", content: "/* content of c.js */" },
  { tool_call_id: "4", content: "src/utils.js:45: // TODO: refactor\n..." }
]

Performance gain: ~4x faster than sequential execution
```

---

## 7. Use Cases Thực Tế

### 7.1 Bug Fix: Complete Walkthrough

**Scenario:** Production bug - users can't login after password reset

```javascript
// ═══════════════════════════════════════════════════════
// ITERATION 1: User Report
// ═══════════════════════════════════════════════════════

User: "Users report they can't login after resetting password.
       Error: 'Invalid credentials' even with new password.
       Urgent - affects 500+ users."

LLM receives:
- User description
- Urgency flag
- No stack trace (user-facing error)

LLM strategy:
"Need to investigate password reset flow and login validation"

tool_calls: [
  {
    function: "grep",
    args: {
      pattern: "password.*reset",
      extensions: [".js", ".ts"]
    }
  },
  {
    function: "grep",
    args: {
      pattern: "Invalid credentials",
      extensions: [".js"]
    }
  }
]

Results:
  src/auth/resetPassword.js:34: await user.resetPassword(newPassword);
  src/auth/login.js:67: throw new Error('Invalid credentials');
  src/api/auth.controller.js:89: // Handle password reset

// ═══════════════════════════════════════════════════════
// ITERATION 2: Read Core Files
// ═══════════════════════════════════════════════════════

tool_calls: [
  { function: "read", args: { path: "src/auth/resetPassword.js" }},
  { function: "read", args: { path: "src/auth/login.js" }}
]

resetPassword.js:
```javascript
async function resetPassword(userId, newPassword) {
  const user = await User.findById(userId);

  // Update password
  user.password = newPassword;  // ⚠️ Not hashed!
  await user.save();

  // Send confirmation email
  await sendEmail(user.email, 'Password Reset Successful');

  return { success: true };
}
```

login.js:
```javascript
async function validateLogin(email, password) {
  const user = await User.findOne({ email });

  if (!user) {
    throw new Error('Invalid credentials');
  }

  // Compare hashed password
  const isValid = await bcrypt.compare(password, user.password);

  if (!isValid) {
    throw new Error('Invalid credentials');  // Line 67
  }

  return generateToken(user);
}
```

// ═══════════════════════════════════════════════════════
// ITERATION 3: Root Cause Found
// ═══════════════════════════════════════════════════════

LLM analysis:
"ROOT CAUSE IDENTIFIED:
resetPassword.js stores plaintext password but login.js
expects bcrypt-hashed password.

When user resets:
1. New password stored as plaintext: 'newpass123'
2. User tries to login with 'newpass123'
3. bcrypt.compare('newpass123', 'newpass123') → FALSE
   (because second arg is plaintext, not a hash)

Fix: Hash password before saving in resetPassword"

tool_calls: [
  { function: "read", args: { path: "src/auth/register.js" }}
]

// Check how registration handles it correctly

register.js:
```javascript
async function register(email, password) {
  const hashedPassword = await bcrypt.hash(password, 10);
  const user = new User({
    email,
    password: hashedPassword
  });
  await user.save();
}
```

// ═══════════════════════════════════════════════════════
// ITERATION 4: Apply Fix
// ═══════════════════════════════════════════════════════

tool_calls: [
  {
    function: "edit",
    args: {
      path: "src/auth/resetPassword.js",
      changes: [{
        start_line: 5,
        end_line: 6,
        new_content: `  // Hash password before saving
  const hashedPassword = await bcrypt.hash(newPassword, 10);
  user.password = hashedPassword;`
      }]
    }
  }
]

// ═══════════════════════════════════════════════════════
// ITERATION 5: Verify Fix
// ═══════════════════════════════════════════════════════

tool_calls: [
  {
    function: "exec",
    args: {
      command: "npm test -- --grep 'password reset'"
    }
  }
]

Test output:
  ✓ should hash password on reset
  ✓ should allow login after reset
  ✗ existing users with plaintext passwords cannot login

// ═══════════════════════════════════════════════════════
// ITERATION 6: Data Migration
// ═══════════════════════════════════════════════════════

LLM: "Tests reveal existing affected users need migration"

tool_calls: [
  {
    function: "write",
    args: {
      path: "scripts/migrate-reset-passwords.js",
      content: `
const User = require('../models/User');
const bcrypt = require('bcrypt');

async function migrateResetPasswords() {
  // Find users with plaintext passwords
  // (passwords not starting with bcrypt prefix '$2b$')
  const users = await User.find({
    password: { $not: /^\\$2b\\$/ }
  });

  console.log(\`Found \${users.length} users with plaintext passwords\`);

  for (const user of users) {
    // Force password reset
    user.resetToken = crypto.randomBytes(32).toString('hex');
    user.resetTokenExpiry = Date.now() + 3600000; // 1 hour
    user.password = null; // Invalidate old password
    await user.save();

    // Send reset email
    await sendPasswordResetEmail(user);
  }

  console.log('Migration complete. Users notified.');
}

migrateResetPasswords();
      `
    }
  }
]

// ═══════════════════════════════════════════════════════
// ITERATION 7: Final Report
// ═══════════════════════════════════════════════════════

LLM → User:
"## Bug Fix Complete ✅

### Root Cause
Password reset function stored passwords in plaintext while login expected bcrypt hash.

### Changes Made
1. **src/auth/resetPassword.js** - Added bcrypt hashing before save
2. **scripts/migrate-reset-passwords.js** - Created migration for affected users

### Verification
- Unit tests passing ✓
- Integration tests passing ✓
- Created migration script for 500+ affected users

### Next Steps
1. Deploy fix to production
2. Run migration script: `node scripts/migrate-reset-passwords.js`
3. Monitor password reset success rate

### Prevention
Consider adding:
- Pre-save hook in User model to always hash passwords
- Integration test for complete reset flow
- Security audit for password handling

Estimated fix time: ~5 minutes
Affected users will receive reset emails automatically."
```

**Key Observations:**
- 7 iterations from bug report to solution
- Read only 3 core files (out of 500+)
- Identified root cause, fixed bug, AND created migration
- Provided actionable next steps
- Total time: ~2 minutes

---

### 7.2 Feature Implementation: Add User Avatar Upload

**Scenario:** Add avatar upload feature to user profile

```javascript
// User provides specification
User: "Add avatar upload to user profile.
       - Support JPG, PNG (max 5MB)
       - Resize to 200x200
       - Store in S3
       - Update user model"

// Skills activated: coding-agent

// Phase 1: Architecture Planning (3 iterations)
// - Reads existing upload implementations
// - Checks S3 configuration
// - Reviews user model schema

// Phase 2: Implementation (12 iterations)
// - Creates upload endpoint
// - Adds image validation
// - Implements resize with sharp library
// - S3 integration
// - Updates user model
// - Frontend form component

// Phase 3: Testing (5 iterations)
// - Unit tests for validation
// - Integration tests for upload flow
// - Manual testing instructions

// Files created/modified:
// - src/api/upload.controller.js (new)
// - src/models/User.js (modified)
// - src/services/s3.service.js (modified)
// - src/components/AvatarUpload.jsx (new)
// - tests/upload.test.js (new)

// Total: 20 iterations, ~5 minutes
```

---

## 8. Performance và Optimization

### 8.1 Performance Metrics

```javascript
// Typical Performance Profile

Simple Task (1-2 file edits):
├─ Iterations: 3-5
├─ Files read: 1-3
├─ Time: 30-60 seconds
├─ Tokens: 500-1500
└─ Cost: $0.01-0.03

Medium Task (bug fix in medium codebase):
├─ Iterations: 5-10
├─ Files read: 3-10
├─ Searches: 2-5
├─ Time: 1-3 minutes
├─ Tokens: 2000-5000
└─ Cost: $0.05-0.15

Complex Task (multi-file feature):
├─ Iterations: 10-25
├─ Files read: 10-30
├─ Searches: 5-15
├─ Time: 3-10 minutes
├─ Tokens: 5000-15000
└─ Cost: $0.15-0.50

Research Task (large codebase exploration):
├─ Iterations: 15-40
├─ Files read: 20-100
├─ Searches: 10-30
├─ Time: 5-20 minutes
├─ Tokens: 10000-40000
└─ Cost: $0.30-1.50
```

### 8.2 Optimization Strategies

```javascript
// 1. Caching Layer
class CacheManager {
  // Cache file contents to avoid re-reading
  fileCache = new LRU({ max: 100, ttl: 1000 * 60 * 5 }); // 5 min TTL

  // Cache search results
  searchCache = new LRU({ max: 50, ttl: 1000 * 60 * 2 }); // 2 min TTL

  async getCachedFile(path, mtime) {
    const cached = this.fileCache.get(path);
    if (cached && cached.mtime === mtime) {
      return cached.content;
    }
    return null;
  }
}

// 2. Smart Batching
class ToolBatcher {
  // Batch multiple read requests into single operation
  async batchReads(paths) {
    return await Promise.all(
      paths.map(path => fs.readFile(path, 'utf8'))
    );
  }
}

// 3. Progressive Loading
class ProgressiveLoader {
  // Load summary first, full content on demand
  async loadFileSummary(path) {
    const content = await fs.readFile(path, 'utf8');
    const lines = content.split('\n');

    return {
      path,
      lineCount: lines.length,
      summary: lines.slice(0, 10).join('\n') + '\n...',
      exports: this.extractExports(content),
      imports: this.extractImports(content)
    };
  }
}

// 4. Streaming for Large Files
class StreamingReader {
  async *readLargeFile(path, chunkSize = 1000) {
    const stream = fs.createReadStream(path, 'utf8');
    let buffer = '';
    let lineCount = 0;

    for await (const chunk of stream) {
      buffer += chunk;
      const lines = buffer.split('\n');
      buffer = lines.pop(); // Keep incomplete line

      for (const line of lines) {
        yield { lineNumber: ++lineCount, content: line };

        if (lineCount % chunkSize === 0) {
          // Send chunk to LLM
          await this.processChunk(lines);
        }
      }
    }
  }
}
```

### 8.3 Rate Limiting & Quotas

```javascript
class RateLimiter {
  constructor(config) {
    this.limits = {
      openai: {
        requestsPerMinute: 60,
        tokensPerMinute: 90000,
        tokensPerDay: 1000000
      },
      tools: {
        execPerMinute: 10,  // Limit shell executions
        readsPerMinute: 100,
        writesPerMinute: 50
      }
    };

    this.counters = {
      openai: { requests: 0, tokens: 0 },
      tools: { exec: 0, reads: 0, writes: 0 }
    };

    this.resetCounters();
  }

  async checkLimit(service, operation, amount = 1) {
    const limit = this.limits[service][operation];
    const current = this.counters[service][operation.replace('PerMinute', '')];

    if (current + amount > limit) {
      const waitTime = this.getResetTime();
      throw new Error(
        `Rate limit exceeded for ${service}.${operation}. ` +
        `Reset in ${waitTime}s`
      );
    }

    this.counters[service][operation.replace('PerMinute', '')] += amount;
  }

  resetCounters() {
    setInterval(() => {
      this.counters.openai = { requests: 0, tokens: 0 };
      this.counters.tools = { exec: 0, reads: 0, writes: 0 };
    }, 60000); // Reset every minute
  }
}
```

---

## 9. Security & Safety

### 9.1 Permission System

```javascript
class PermissionManager {
  constructor() {
    this.permissions = new Set([
      'filesystem:read',
      'filesystem:write',
      'system:exec',
      'network:request'
    ]);

    // Blacklisted paths
    this.blacklist = [
      '/etc/passwd',
      '/etc/shadow',
      '~/.ssh/id_rsa',
      '~/.aws/credentials',
      '.env'
    ];

    // Dangerous commands
    this.dangerousCommands = [
      'rm -rf /',
      'dd if=',
      'mkfs',
      ':(){:|:&};:',  // Fork bomb
      'sudo',
      'chmod 777'
    ];
  }

  async checkFileAccess(path, operation) {
    const resolvedPath = path.resolve(path);

    // Check blacklist
    for (const blocked of this.blacklist) {
      if (resolvedPath.includes(blocked)) {
        throw new Error(`Access denied: ${path} is blacklisted`);
      }
    }

    // Check if outside project directory
    const projectRoot = process.cwd();
    if (!resolvedPath.startsWith(projectRoot)) {
      const allowed = await this.askUserPermission(
        `Allow access to ${resolvedPath} outside project directory?`
      );
      if (!allowed) {
        throw new Error('Access denied by user');
      }
    }

    return true;
  }

  async checkCommandExecution(command) {
    // Check for dangerous commands
    for (const dangerous of this.dangerousCommands) {
      if (command.includes(dangerous)) {
        throw new Error(
          `Dangerous command blocked: ${command}\n` +
          `This command could harm your system.`
        );
      }
    }

    // Require confirmation for destructive operations
    if (command.match(/rm|del|drop|truncate|delete/i)) {
      const allowed = await this.askUserPermission(
        `Execute potentially destructive command: ${command}?`
      );
      if (!allowed) {
        throw new Error('Command execution denied by user');
      }
    }

    return true;
  }
}
```

### 9.2 Sandboxing

```javascript
// Tool execution happens in sandboxed environment

class SandboxExecutor {
  async executeInSandbox(tool, args) {
    const sandbox = {
      // Restricted globals
      console: {
        log: (...args) => this.log('info', args),
        error: (...args) => this.log('error', args),
        warn: (...args) => this.log('warn', args)
      },

      // No access to process, require, etc.
      process: undefined,
      require: undefined,
      module: undefined,
      __dirname: undefined,
      __filename: undefined
    };

    // Create VM context
    const vm = require('vm');
    const context = vm.createContext(sandbox);

    // Execute with timeout
    const script = new vm.Script(tool.handler.toString());
    const handler = script.runInContext(context, {
      timeout: 5000,
      displayErrors: true
    });

    return await handler(args);
  }
}
```

---

## 10. Troubleshooting & Debug

### 10.1 Debug Mode

```bash
# Enable debug logging
OPENCLAW_DEBUG=true openclaw

# Output:
# [DEBUG] Session initialized: sess_abc123
# [DEBUG] Loaded 12 skills
# [DEBUG] LLM request: 450 tokens
# [DEBUG] Tool execution: read(login.js) → 230ms
# [DEBUG] LLM response: 3 tool calls
# [DEBUG] Context size: 2.1KB / 16KB limit
```

### 10.2 Common Issues

```javascript
// Issue 1: Token limit exceeded
Error: "Context too large (20000 tokens > 16000 limit)"
Solution:
- Enable context compression
- Reduce maxFilesPerIteration
- Use summarization

// Issue 2: Rate limit hit
Error: "Rate limit exceeded. Reset in 45s"
Solution:
- Add delays between requests
- Reduce parallelism
- Use caching

// Issue 3: Tool execution timeout
Error: "Tool execution timeout after 60s"
Solution:
- Increase timeout in config
- Optimize slow operations
- Break into smaller tasks

// Issue 4: File not found
Error: "ENOENT: no such file or directory"
Solution:
- Verify working directory
- Use absolute paths
- Check file permissions
```

---

## 11. Kết Luận

### OpenClaw Architecture Summary

```
┌─────────────────────────────────────────────────────────┐
│                   KEY PRINCIPLES                         │
└─────────────────────────────────────────────────────────┘

1. 🧠 Distributed Intelligence
   Brain (LLM) in cloud ←→ Hands (Gateway) local

2. 🔄 Iterative Execution
   Loop until completion, not single request-response

3. 📊 Progressive Retrieval
   Load context on-demand, not bulk upfront

4. 🧩 Modular Extensions
   Skills-based architecture for flexibility

5. 🛡️ Security First
   Sandboxing, permissions, user confirmation

6. ⚡ Performance Optimized
   Caching, batching, parallel execution

7. 🎯 Context-Aware
   Smart budgeting, compression, relevance ranking
```

### Workflow Diagram (Complete Picture)

```
User Request
    ↓
┌───────────────────────────────────────┐
│  Gateway: Context Assembly            │
│  - Load session history               │
│  - Activate relevant skills           │
│  - Prepare tool definitions           │
└────────────┬──────────────────────────┘
             ↓
┌─────────────────────���─────────────────┐
│  LLM: Reasoning & Planning            │
│  - Analyze request                    │
│  - Choose tools/strategy              │
│  - Generate tool calls OR answer      │
└────────────┬──────────────────────────┘
             ↓
         ┌───┴───┐
    Tool calls?  │  No → Final Answer
         │       │         ↓
        Yes      └────→  User
         ↓
┌───────────────────────────────────────┐
│  Gateway: Tool Execution              │
│  - Validate permissions               │
│  - Execute tools (parallel)           │
│  - Capture outputs                    │
│  - Handle errors                      │
└────────────┬──────────────────────────┘
             ↓
         Results
             ↓
┌───────────────────────────────────────┐
│  Context Update                       │
│  - Add tool results                   │
│  - Compress if needed                 │
│  - Update session state               │
└────────────┬──────────────────────────┘
             │
             └──────┐
                    ↓
              Loop back to LLM
              (with updated context)
```

---

## Phụ Lục

### A. Thuật Ngữ

- **Agentic Loop**: Vòng lặp tự điều khiển, LLM quyết định actions
- **Tool**: Function mà LLM có thể gọi (read, edit, exec...)
- **Skill**: Bộ tools + prompts + workflow cho task cụ thể
- **Context Budget**: Giới hạn tokens có thể gửi cho LLM
- **Progressive Retrieval**: Load dữ liệu từng bước thay vì bulk
- **Gateway Runtime**: Process local thực thi tools
- **LLM Runtime**: AI model ở cloud (GPT-4)

### B. File Structure Reference

```
openclaw/
├── bin/openclaw                   # CLI entry
├── lib/
│   ├── gateway/
│   │   ├── session.js             # Session management
│   │   ├── context.js             # Context assembly
│   │   ├── tools.js               # Tool registry
│   │   └── executor.js            # Tool execution
│   ├── llm/
│   │   ├── client.js              # OpenAI client
│   │   └── formatter.js           # Message formatting
│   └── skills/
│       └── loader.js              # Skill loading
├── skills/                        # Built-in skills
├── config/
│   ├── default.json
│   └── tools.json
└── ~/.openclaw/
    ├── config.json                # User config
    ├── skills/                    # User skills
    └── sessions/                  # Session history
```

### C. API Reference

```javascript
// Tool Definition Schema
{
  name: string,
  description: string,
  parameters: JSONSchema,
  handler: async (args) => result,
  permissions: string[]
}

// Skill Definition Schema (SKILL.md frontmatter)
{
  name: string,
  version: string,
  author: string,
  requires: {
    tools: string[],
    skills: string[]
  },
  triggers: string[]
}

// Context Structure
{
  system: string,
  messages: Message[],
  tools: Tool[],
  skills: SkillMetadata[],
  tokenCount: number
}
```

---

**Document Version:** 1.0
**Last Updated:** 2024
**Author:** OpenClaw Documentation Team

For more information, visit: https://openclaw.dev
