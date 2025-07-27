# Autonomous Development Pipeline - Complete Appendices Guide

## Table of Contents
- [Appendix A: Task-Kanban-MCP Integration Guide](#appendix-a-task-kanban-mcp-integration-guide)
- [Appendix B: Cursor Integration Specifications](#appendix-b-cursor-integration-specifications)
- [Appendix C: Task Routing Algorithms](#appendix-c-task-routing-algorithms)
- [Appendix D: Task Workflow Templates](#appendix-d-task-workflow-templates)
- [Appendix E: Training Materials](#appendix-e-training-materials)

---

## Appendix A: Task-Kanban-MCP Integration Guide

### A.1 Installation and Configuration

#### Prerequisites
```bash
# System requirements
- Node.js 18+ 
- SQLite3
- Git
- MCP CLI tools
```

#### Installation Steps
```bash
# 1. Clone the task-kanban-mcp repository
git clone https://github.com/AdamManuel-dev/task-kanban-mcp.git
cd task-kanban-mcp

# 2. Install dependencies
npm install

# 3. Build the project
npm run build

# 4. Initialize the database
npm run init-db

# 5. Start the MCP server
npm run start
```

#### Configuration File (config.json)
```json
{
  "server": {
    "name": "task-kanban",
    "version": "1.0.0",
    "port": 3001,
    "host": "localhost",
    "protocol": "websocket"
  },
  "database": {
    "path": "./data/tasks.db",
    "backup": {
      "enabled": true,
      "interval": "hourly",
      "retention": 30
    }
  },
  "security": {
    "authentication": "jwt",
    "encryption": "aes-256-gcm",
    "cors": {
      "enabled": true,
      "origins": ["http://localhost:*", "cursor://*"]
    }
  },
  "logging": {
    "level": "info",
    "file": "./logs/task-kanban.log",
    "maxSize": "10m",
    "maxFiles": 10
  }
}
```

### A.2 MCP Protocol Implementation

#### WebSocket Connection Setup
```typescript
import { MCPClient } from '@modelcontextprotocol/sdk';

class TaskKanbanMCPClient {
  private client: MCPClient;
  private ws: WebSocket;

  async connect() {
    this.ws = new WebSocket('ws://localhost:3001');
    
    this.client = new MCPClient({
      name: 'autonomous-dev-pipeline',
      version: '1.0.0',
      transport: {
        type: 'websocket',
        socket: this.ws
      }
    });

    await this.client.connect();
    
    // Subscribe to real-time updates
    this.client.on('task.updated', this.handleTaskUpdate);
    this.client.on('task.created', this.handleTaskCreated);
    this.client.on('board.changed', this.handleBoardChange);
  }

  private handleTaskUpdate = (data: TaskUpdateEvent) => {
    console.log('Task updated:', data);
    // Propagate to relevant agent
  };
}
```

#### MCP Tool Definitions
```typescript
// Available MCP tools for task management
const taskKanbanTools = {
  // Board Management
  'task-kanban/create-board': {
    description: 'Create a new kanban board',
    inputSchema: {
      type: 'object',
      properties: {
        name: { type: 'string' },
        columns: { 
          type: 'array',
          items: { type: 'string' }
        }
      }
    }
  },

  // Task Operations
  'task-kanban/create-task': {
    description: 'Create a new task',
    inputSchema: {
      type: 'object',
      properties: {
        boardId: { type: 'string' },
        title: { type: 'string' },
        description: { type: 'string' },
        assignee: { type: 'string' },
        priority: { type: 'string' },
        metadata: { type: 'object' }
      }
    }
  },

  // Bulk Operations
  'task-kanban/batch-update': {
    description: 'Update multiple tasks',
    inputSchema: {
      type: 'object',
      properties: {
        taskIds: { type: 'array' },
        updates: { type: 'object' }
      }
    }
  }
};
```

### A.3 Agent Authentication Setup

#### JWT Token Generation
```typescript
import jwt from 'jsonwebtoken';

interface AgentToken {
  agentId: string;
  agentType: 'orchestrator' | 'development' | 'operations';
  permissions: string[];
}

class AgentAuthenticator {
  private secret = process.env.JWT_SECRET;

  generateToken(agent: AgentToken): string {
    return jwt.sign(agent, this.secret, {
      expiresIn: '24h',
      issuer: 'autonomous-dev-pipeline'
    });
  }

  validateToken(token: string): AgentToken {
    return jwt.verify(token, this.secret) as AgentToken;
  }
}

// Agent-specific tokens
const agentTokens = {
  orchestrator: generateToken({
    agentId: 'master-orchestrator',
    agentType: 'orchestrator',
    permissions: ['create', 'read', 'update', 'delete', 'admin']
  }),
  
  development: generateToken({
    agentId: 'dev-quality',
    agentType: 'development',
    permissions: ['read', 'update', 'create:subtask']
  }),
  
  operations: generateToken({
    agentId: 'ops-security',
    agentType: 'operations',
    permissions: ['read', 'update', 'create:security', 'archive']
  })
};
```

### A.4 Board Configuration Templates

#### Main Development Board
```json
{
  "board": {
    "id": "main-dev",
    "name": "Autonomous Development Pipeline",
    "description": "Primary development workflow",
    "columns": [
      {
        "id": "backlog",
        "name": "Backlog",
        "limit": null,
        "color": "#94a3b8"
      },
      {
        "id": "ready",
        "name": "Ready",
        "limit": 10,
        "color": "#60a5fa"
      },
      {
        "id": "in-progress-human",
        "name": "In Progress - Human",
        "limit": 5,
        "color": "#34d399",
        "metadata": {
          "assigneeType": "human",
          "notifyOnEntry": true
        }
      },
      {
        "id": "in-progress-ai",
        "name": "In Progress - AI",
        "limit": 20,
        "color": "#a78bfa",
        "metadata": {
          "assigneeType": "ai",
          "autoProgress": true
        }
      },
      {
        "id": "quality-check",
        "name": "Quality Check",
        "limit": 10,
        "color": "#fbbf24",
        "metadata": {
          "autoCreateSubtasks": true,
          "requiredChecks": ["lint", "test", "security"]
        }
      },
      {
        "id": "review",
        "name": "Review",
        "limit": 5,
        "color": "#f97316"
      },
      {
        "id": "done",
        "name": "Done",
        "limit": null,
        "color": "#10b981",
        "metadata": {
          "archiveAfterDays": 30
        }
      }
    ]
  }
}
```

#### Specialized Boards
```json
{
  "securityBoard": {
    "name": "Security & Compliance",
    "columns": ["New", "Scanning", "Review", "Remediation", "Resolved"]
  },
  
  "maintenanceBoard": {
    "name": "Recurring Maintenance",
    "columns": ["Scheduled", "Active", "Completed"],
    "automation": {
      "recurringTasks": true,
      "autoSchedule": true
    }
  },
  
  "experimentBoard": {
    "name": "Innovation & Experiments",
    "columns": ["Ideas", "Prototype", "Testing", "Evaluation", "Adopted/Rejected"]
  }
}
```

### A.5 Custom Field Definitions

```typescript
// Task metadata schema
interface TaskMetadata {
  // Core fields (all tasks)
  core: {
    id: string;
    title: string;
    description: string;
    createdAt: Date;
    updatedAt: Date;
    createdBy: string;
    assignee: string;
    priority: 'low' | 'medium' | 'high' | 'critical';
  };

  // Agent-specific fields
  agentFields: {
    // Orchestrator fields
    complexity: number;              // 1-10 scale
    estimatedHours: number;
    businessValue: number;           // 1-100 scale
    dependencies: string[];
    assigneeType: 'human' | 'ai' | 'hybrid';
    
    // Development fields
    files: string[];                // Affected files
    branch: string;                 // Git branch
    commits: string[];              // Related commits
    testCoverage: number;           // Percentage
    lintScore: number;              // Quality score
    
    // Operations fields
    securityScore: number;          // Risk assessment
    compliance: string[];           // Regulations met
    deploymentEnv: string;          // Target environment
    approvals: Approval[];          // Required approvals
  };

  // Workflow fields
  workflow: {
    currentColumn: string;
    previousColumn: string;
    transitions: Transition[];
    blockers: Blocker[];
    timeInColumn: number;           // Minutes
    totalTime: number;              // Minutes
  };
}
```

---

## Appendix B: Cursor Integration Specifications

### B.1 MCP Protocol Details for Cursor

#### Cursor Extension Architecture
```typescript
// cursor-extension/src/mcpClient.ts
import * as vscode from 'vscode';
import { MCPClient } from '@modelcontextprotocol/sdk';

export class CursorMCPExtension {
  private client: MCPClient;
  private statusBar: vscode.StatusBarItem;
  private taskPanel: vscode.WebviewPanel;

  async activate(context: vscode.ExtensionContext) {
    // Initialize MCP client
    this.client = new MCPClient({
      name: 'cursor-adp',
      version: '1.0.0',
      capabilities: {
        tools: true,
        prompts: true,
        resources: true
      }
    });

    // Connect to task-kanban server
    await this.connectToTaskServer();
    
    // Register commands
    this.registerCommands(context);
    
    // Initialize UI components
    this.initializeUI(context);
  }

  private async connectToTaskServer() {
    const config = vscode.workspace.getConfiguration('adp');
    const serverUrl = config.get('taskServerUrl', 'ws://localhost:3001');
    
    await this.client.connect({
      transport: {
        type: 'websocket',
        url: serverUrl
      }
    });
  }
}
```

### B.2 Task Panel UI Components

#### Task Panel HTML Template
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>ADP Tasks</title>
    <style>
        .task-panel {
            font-family: var(--vscode-font-family);
            padding: 10px;
        }
        
        .task-card {
            background: var(--vscode-editor-background);
            border: 1px solid var(--vscode-panel-border);
            border-radius: 4px;
            padding: 10px;
            margin-bottom: 10px;
        }
        
        .task-priority-high {
            border-left: 3px solid #f87171;
        }
        
        .task-assignee {
            display: inline-block;
            padding: 2px 8px;
            border-radius: 12px;
            font-size: 12px;
        }
        
        .assignee-human {
            background: #34d399;
            color: #000;
        }
        
        .assignee-ai {
            background: #a78bfa;
            color: #000;
        }
        
        .progress-bar {
            height: 4px;
            background: var(--vscode-progressBar-background);
            border-radius: 2px;
            overflow: hidden;
        }
        
        .progress-fill {
            height: 100%;
            background: var(--vscode-progressBar-foreground);
            transition: width 0.3s ease;
        }
    </style>
</head>
<body>
    <div class="task-panel">
        <h2>My Tasks</h2>
        <div id="task-list"></div>
        
        <h2>AI Tasks in Progress</h2>
        <div id="ai-task-list"></div>
        
        <h2>Blocked Tasks</h2>
        <div id="blocked-task-list"></div>
    </div>
    
    <script>
        const vscode = acquireVsCodeApi();
        
        // Handle messages from extension
        window.addEventListener('message', event => {
            const message = event.data;
            switch (message.type) {
                case 'updateTasks':
                    updateTaskDisplay(message.tasks);
                    break;
                case 'taskProgress':
                    updateTaskProgress(message.taskId, message.progress);
                    break;
            }
        });
        
        function updateTaskDisplay(tasks) {
            const humanTasks = tasks.filter(t => t.assignee === 'human');
            const aiTasks = tasks.filter(t => t.assignee === 'ai');
            const blockedTasks = tasks.filter(t => t.blockers.length > 0);
            
            renderTasks('task-list', humanTasks);
            renderTasks('ai-task-list', aiTasks);
            renderTasks('blocked-task-list', blockedTasks);
        }
        
        function renderTasks(containerId, tasks) {
            const container = document.getElementById(containerId);
            container.innerHTML = tasks.map(task => `
                <div class="task-card task-priority-${task.priority}">
                    <h4>${task.title}</h4>
                    <p>${task.description}</p>
                    <div class="task-meta">
                        <span class="task-assignee assignee-${task.assignee}">
                            ${task.assignee}
                        </span>
                        <span class="task-id">#${task.id}</span>
                    </div>
                    <div class="progress-bar">
                        <div class="progress-fill" style="width: ${task.progress}%"></div>
                    </div>
                    <div class="task-actions">
                        <button onclick="startTask('${task.id}')">Start</button>
                        <button onclick="completeTask('${task.id}')">Complete</button>
                        <button onclick="viewDetails('${task.id}')">Details</button>
                    </div>
                </div>
            `).join('');
        }
        
        function startTask(taskId) {
            vscode.postMessage({
                type: 'startTask',
                taskId: taskId
            });
        }
    </script>
</body>
</html>
```

### B.3 Notification System Setup

```typescript
// Notification manager for Cursor
class TaskNotificationManager {
  private notificationQueue: TaskNotification[] = [];
  
  constructor(private context: vscode.ExtensionContext) {
    this.setupNotificationHandlers();
  }

  private setupNotificationHandlers() {
    // WebSocket listener for real-time updates
    this.mcpClient.on('task.assigned', (data) => {
      if (data.assignee === 'human') {
        this.showTaskAssignedNotification(data);
      }
    });

    this.mcpClient.on('task.blocked', (data) => {
      this.showBlockerNotification(data);
    });

    this.mcpClient.on('ai.handoff', (data) => {
      this.showHandoffNotification(data);
    });
  }

  private showTaskAssignedNotification(task: Task) {
    const message = `New task assigned: ${task.title}`;
    const actions = ['View Task', 'Start Now', 'Dismiss'];
    
    vscode.window.showInformationMessage(message, ...actions)
      .then(selection => {
        switch(selection) {
          case 'View Task':
            this.openTaskDetails(task.id);
            break;
          case 'Start Now':
            this.startTask(task.id);
            break;
        }
      });
  }

  private showBlockerNotification(blocker: Blocker) {
    const message = `Task blocked: ${blocker.taskTitle} - ${blocker.reason}`;
    vscode.window.showWarningMessage(message, 'View Details', 'Resolve')
      .then(selection => {
        if (selection === 'Resolve') {
          this.openBlockerResolution(blocker);
        }
      });
  }

  private configureNotificationPriorities() {
    return {
      critical: {
        type: 'error',
        persistent: true,
        sound: true
      },
      high: {
        type: 'warning',
        persistent: true,
        sound: false
      },
      medium: {
        type: 'information',
        persistent: false,
        sound: false
      },
      low: {
        type: 'information',
        persistent: false,
        sound: false,
        silent: true
      }
    };
  }
}
```

### B.4 Context Synchronization

```typescript
// File context sync with tasks
class ContextSynchronizer {
  private fileWatcher: vscode.FileSystemWatcher;
  private activeTask: Task | null = null;

  initialize() {
    // Watch for file changes
    this.fileWatcher = vscode.workspace.createFileSystemWatcher('**/*');
    
    this.fileWatcher.onDidChange(this.handleFileChange);
    this.fileWatcher.onDidCreate(this.handleFileCreate);
    
    // Track active editor
    vscode.window.onDidChangeActiveTextEditor(this.handleEditorChange);
    
    // Track terminal commands
    vscode.window.onDidOpenTerminal(this.handleTerminalOpen);
  }

  private handleFileChange = async (uri: vscode.Uri) => {
    if (this.activeTask) {
      // Update task context with modified files
      await this.mcpClient.call('task-kanban/add-context', {
        taskId: this.activeTask.id,
        context: {
          modifiedFiles: [uri.fsPath],
          timestamp: new Date().toISOString(),
          editor: 'cursor'
        }
      });
    }
  };

  private syncTaskContext = async (task: Task) => {
    const context = {
      openFiles: vscode.window.visibleTextEditors.map(e => e.document.uri.fsPath),
      workspaceFolder: vscode.workspace.workspaceFolders?.[0]?.uri.fsPath,
      gitBranch: await this.getCurrentGitBranch(),
      cursorPosition: vscode.window.activeTextEditor?.selection.active,
      diagnostics: this.getRelevantDiagnostics()
    };

    await this.mcpClient.call('task-kanban/update-context', {
      taskId: task.id,
      context
    });
  };

  private getRelevantDiagnostics() {
    const diagnostics = vscode.languages.getDiagnostics();
    return Array.from(diagnostics).map(([uri, diags]) => ({
      file: uri.fsPath,
      issues: diags.map(d => ({
        severity: d.severity,
        message: d.message,
        line: d.range.start.line,
        column: d.range.start.character
      }))
    }));
  }
}
```

### B.5 API Documentation

#### Cursor Extension API
```typescript
// Public API for other extensions
export interface CursorADPAPI {
  // Task operations
  getCurrentTask(): Promise<Task | null>;
  startTask(taskId: string): Promise<void>;
  completeTask(taskId: string, results?: any): Promise<void>;
  
  // AI assistance
  requestAIHelp(context: string): Promise<AIResponse>;
  getAISuggestions(code: string): Promise<Suggestion[]>;
  
  // Collaboration
  handoffToAI(taskId: string, context: any): Promise<void>;
  requestHumanReview(taskId: string, reason: string): Promise<void>;
  
  // Events
  onTaskAssigned: vscode.Event<Task>;
  onTaskCompleted: vscode.Event<Task>;
  onAIHandoff: vscode.Event<HandoffEvent>;
}

// Extension activation
export function activate(context: vscode.ExtensionContext): CursorADPAPI {
  const extension = new CursorMCPExtension();
  extension.activate(context);
  
  // Return public API
  return {
    getCurrentTask: () => extension.getCurrentTask(),
    startTask: (id) => extension.startTask(id),
    completeTask: (id, results) => extension.completeTask(id, results),
    // ... other API methods
  };
}
```

---

## Appendix C: Task Routing Algorithms

### C.1 Complexity Scoring Formulas

```typescript
// Task complexity analyzer
class TaskComplexityAnalyzer {
  private weights = {
    codeComplexity: 0.3,
    domainKnowledge: 0.2,
    creativity: 0.2,
    uncertainty: 0.15,
    dependencies: 0.15
  };

  calculateComplexity(task: TaskRequest): ComplexityScore {
    const scores = {
      codeComplexity: this.analyzeCodeComplexity(task),
      domainKnowledge: this.analyzeDomainRequirements(task),
      creativity: this.analyzeCreativityNeeds(task),
      uncertainty: this.analyzeUncertainty(task),
      dependencies: this.analyzeDependencies(task)
    };

    const totalScore = Object.entries(scores).reduce(
      (total, [key, score]) => total + score * this.weights[key],
      0
    );

    return {
      total: Math.round(totalScore),
      breakdown: scores,
      recommendation: this.getRecommendation(totalScore)
    };
  }

  private analyzeCodeComplexity(task: TaskRequest): number {
    const indicators = {
      algorithmDesign: /algorithm|optimize|performance|complex/i,
      systemDesign: /architect|design|pattern|structure/i,
      concurrency: /parallel|concurrent|async|thread/i,
      dataStructures: /tree|graph|heap|advanced/i
    };

    let score = 1; // Base score

    // Check description for complexity indicators
    Object.entries(indicators).forEach(([type, regex]) => {
      if (regex.test(task.description)) {
        score += 2;
      }
    });

    // Check for file count estimate
    if (task.estimatedFiles > 10) score += 2;
    if (task.estimatedFiles > 20) score += 2;

    // Check for cross-system integration
    if (task.tags?.includes('integration')) score += 3;

    return Math.min(score, 10);
  }

  private analyzeCreativityNeeds(task: TaskRequest): number {
    const creativeIndicators = [
      /design|create|invent|innovative/i,
      /user experience|ui|interface/i,
      /novel|unique|custom/i,
      /from scratch|greenfield/i
    ];

    let score = 1;
    
    creativeIndicators.forEach(regex => {
      if (regex.test(task.description)) {
        score += 2.5;
      }
    });

    // Reduce score for well-defined patterns
    if (/boilerplate|standard|template|crud/i.test(task.description)) {
      score = Math.max(1, score - 3);
    }

    return Math.min(score, 10);
  }

  private getRecommendation(score: number): RoutingRecommendation {
    if (score <= 3) {
      return {
        assignee: 'ai',
        confidence: 0.9,
        reason: 'Simple, well-defined task suitable for AI'
      };
    } else if (score <= 5) {
      return {
        assignee: 'ai',
        confidence: 0.7,
        reason: 'Moderate complexity, AI with human review'
      };
    } else if (score <= 7) {
      return {
        assignee: 'hybrid',
        confidence: 0.8,
        reason: 'Complex task requiring human-AI collaboration'
      };
    } else {
      return {
        assignee: 'human',
        confidence: 0.9,
        reason: 'High complexity requiring human expertise'
      };
    }
  }
}
```

### C.2 Assignment Decision Trees

```typescript
// Decision tree for task assignment
class TaskAssignmentDecisionTree {
  private decisionTree: DecisionNode = {
    question: 'Is this a creative or design task?',
    yes: {
      question: 'Does it require novel solutions?',
      yes: { assign: 'human', confidence: 0.95 },
      no: {
        question: 'Can it follow existing patterns?',
        yes: { assign: 'hybrid', confidence: 0.8 },
        no: { assign: 'human', confidence: 0.85 }
      }
    },
    no: {
      question: 'Is this a routine/repetitive task?',
      yes: {
        question: 'Are there clear specifications?',
        yes: { assign: 'ai', confidence: 0.9 },
        no: { assign: 'hybrid', confidence: 0.7 }
      },
      no: {
        question: 'Does it require domain expertise?',
        yes: {
          question: 'Is the expertise codified?',
          yes: { assign: 'ai', confidence: 0.75 },
          no: { assign: 'human', confidence: 0.9 }
        },
        no: {
          question: 'Is complexity score > 6?',
          yes: { assign: 'human', confidence: 0.8 },
          no: { assign: 'ai', confidence: 0.85 }
        }
      }
    }
  };

  assignTask(task: TaskRequest, context: TaskContext): Assignment {
    const complexityScore = this.analyzer.calculateComplexity(task);
    const treeResult = this.traverseTree(this.decisionTree, task, context);
    
    // Combine tree decision with complexity analysis
    const finalDecision = this.combineDecisions(
      treeResult,
      complexityScore.recommendation,
      context
    );

    return {
      ...finalDecision,
      reasoning: this.generateReasoning(task, finalDecision)
    };
  }

  private traverseTree(
    node: DecisionNode,
    task: TaskRequest,
    context: TaskContext
  ): Assignment {
    if ('assign' in node) {
      return {
        assignee: node.assign,
        confidence: node.confidence
      };
    }

    const answer = this.evaluateQuestion(node.question, task, context);
    return this.traverseTree(answer ? node.yes : node.no, task, context);
  }

  private evaluateQuestion(
    question: string,
    task: TaskRequest,
    context: TaskContext
  ): boolean {
    // Question evaluation logic
    const evaluators: Record<string, (t: TaskRequest, c: TaskContext) => boolean> = {
      'Is this a creative or design task?': (t) => 
        /design|create|ui|ux|interface|layout/i.test(t.description),
      
      'Does it require novel solutions?': (t) =>
        /novel|innovative|unique|custom|new approach/i.test(t.description),
      
      'Can it follow existing patterns?': (t, c) =>
        c.hasExistingPatterns && !t.tags?.includes('greenfield'),
      
      'Is this a routine/repetitive task?': (t) =>
        /routine|standard|boilerplate|crud|update|fix/i.test(t.description),
      
      'Are there clear specifications?': (t) =>
        t.hasDetailedSpecs || t.description.length > 200,
      
      'Does it require domain expertise?': (t) =>
        t.tags?.includes('domain-specific') || 
        /expert|specialized|domain/i.test(t.description),
      
      'Is the expertise codified?': (t, c) =>
        c.knowledgeBase.hasDomainExamples(t.domain),
      
      'Is complexity score > 6?': (t, c) =>
        c.complexityScore > 6
    };

    return evaluators[question]?.(task, context) ?? false;
  }
}
```

### C.3 Workload Balancing Logic

```typescript
class WorkloadBalancer {
  private readonly maxHumanTasks = 5;
  private readonly maxAITasks = 20;
  private readonly maxHumanHoursPerDay = 6;

  async balanceWorkload(
    tasks: TaskRequest[],
    currentWorkload: WorkloadState
  ): Promise<TaskAssignment[]> {
    const assignments: TaskAssignment[] = [];
    
    // Sort tasks by priority and complexity
    const sortedTasks = this.prioritizeTasks(tasks);
    
    for (const task of sortedTasks) {
      const optimalAssignee = await this.findOptimalAssignee(
        task,
        currentWorkload
      );
      
      assignments.push({
        taskId: task.id,
        assignee: optimalAssignee.assignee,
        scheduledStart: optimalAssignee.startTime,
        estimatedDuration: task.estimatedHours,
        reason: optimalAssignee.reason
      });
      
      // Update workload state
      this.updateWorkloadState(currentWorkload, optimalAssignee);
    }
    
    return assignments;
  }

  private async findOptimalAssignee(
    task: TaskRequest,
    workload: WorkloadState
  ): Promise<OptimalAssignment> {
    const candidates = [
      {
        type: 'human',
        availability: this.calculateHumanAvailability(workload),
        suitability: this.calculateHumanSuitability(task)
      },
      {
        type: 'ai',
        availability: this.calculateAIAvailability(workload),
        suitability: this.calculateAISuitability(task)
      }
    ];

    // Score each candidate
    const scores = candidates.map(c => ({
      ...c,
      score: c.availability * 0.4 + c.suitability * 0.6
    }));

    // Check for hybrid approach
    if (task.complexity > 6 && task.estimatedHours > 4) {
      scores.push({
        type: 'hybrid',
        availability: Math.min(candidates[0].availability, candidates[1].availability),
        suitability: 0.9,
        score: 0.85
      });
    }

    // Select best option
    const best = scores.reduce((a, b) => a.score > b.score ? a : b);
    
    return {
      assignee: best.type,
      startTime: this.calculateStartTime(best.type, workload),
      confidence: best.score,
      reason: this.generateAssignmentReason(task, best)
    };
  }

  private calculateHumanAvailability(workload: WorkloadState): number {
    const currentTasks = workload.human.activeTasks.length;
    const todayHours = workload.human.hoursWorkedToday;
    
    // Factor in current task load
    const taskLoad = 1 - (currentTasks / this.maxHumanTasks);
    
    // Factor in daily hours
    const hoursAvailable = 1 - (todayHours / this.maxHumanHoursPerDay);
    
    // Factor in time of day (humans less available late)
    const hour = new Date().getHours();
    const timeOfDayFactor = hour < 9 || hour > 17 ? 0.5 : 1;
    
    return taskLoad * 0.5 + hoursAvailable * 0.3 + timeOfDayFactor * 0.2;
  }

  private calculateAIAvailability(workload: WorkloadState): number {
    const currentTasks = workload.ai.activeTasks.length;
    const apiLimitUsage = workload.ai.apiCallsToday / workload.ai.apiLimit;
    
    // Factor in current task load
    const taskLoad = 1 - (currentTasks / this.maxAITasks);
    
    // Factor in API limits
    const apiAvailable = 1 - apiLimitUsage;
    
    // AI is always available time-wise
    return taskLoad * 0.7 + apiAvailable * 0.3;
  }
}
```

### C.4 Priority Calculation Methods

```typescript
class TaskPriorityCalculator {
  private weights = {
    businessValue: 0.35,
    urgency: 0.25,
    dependencies: 0.20,
    complexity: 0.10,
    riskReduction: 0.10
  };

  calculatePriority(task: TaskRequest, context: PriorityContext): Priority {
    const scores = {
      businessValue: this.scoreBusinessValue(task),
      urgency: this.scoreUrgency(task, context),
      dependencies: this.scoreDependencies(task, context),
      complexity: this.scoreComplexity(task),
      riskReduction: this.scoreRiskReduction(task)
    };

    const totalScore = Object.entries(scores).reduce(
      (total, [key, score]) => total + score * this.weights[key],
      0
    );

    return {
      score: totalScore,
      level: this.getPriorityLevel(totalScore),
      factors: scores,
      reasoning: this.generatePriorityReasoning(scores)
    };
  }

  private scoreBusinessValue(task: TaskRequest): number {
    let score = 5; // Base score

    // Revenue impact
    if (task.tags?.includes('revenue')) score += 3;
    if (task.estimatedRevenue > 10000) score += 2;

    // Customer impact
    if (task.tags?.includes('customer-facing')) score += 2;
    if (task.affectedUsers > 1000) score += 1;

    // Strategic importance
    if (task.tags?.includes('strategic')) score += 2;

    return Math.min(score, 10);
  }

  private scoreUrgency(task: TaskRequest, context: PriorityContext): number {
    let score = 5;

    // Deadline proximity
    if (task.deadline) {
      const daysUntilDeadline = this.daysUntil(task.deadline);
      if (daysUntilDeadline < 1) score = 10;
      else if (daysUntilDeadline < 3) score = 8;
      else if (daysUntilDeadline < 7) score = 6;
    }

    // External dependencies
    if (task.externalDeadline) score += 2;
    
    // Blocking others
    if (context.blockedTasks?.includes(task.id)) score += 3;

    return Math.min(score, 10);
  }

  private scoreDependencies(task: TaskRequest, context: PriorityContext): number {
    const blockedCount = context.taskGraph.getBlockedBy(task.id).length;
    const criticalPathImpact = context.taskGraph.getCriticalPathImpact(task.id);
    
    let score = 5;
    score += Math.min(blockedCount, 3);
    score += criticalPathImpact * 2;
    
    return Math.min(score, 10);
  }

  private getPriorityLevel(score: number): PriorityLevel {
    if (score >= 8.5) return 'critical';
    if (score >= 7) return 'high';
    if (score >= 4) return 'medium';
    return 'low';
  }

  private generatePriorityReasoning(scores: PriorityScores): string {
    const topFactors = Object.entries(scores)
      .sort(([, a], [, b]) => b - a)
      .slice(0, 2)
      .map(([factor, score]) => `${factor} (${score}/10)`);
    
    return `Priority driven by ${topFactors.join(' and ')}`;
  }
}
```

### C.5 Learning Mechanisms

```typescript
class TaskRoutingLearner {
  private historicalData: TaskOutcome[] = [];
  private model: RoutingModel;

  async learn(outcome: TaskOutcome): Promise<void> {
    // Store outcome
    this.historicalData.push(outcome);
    
    // Update model if enough data
    if (this.historicalData.length % 100 === 0) {
      await this.updateModel();
    }
    
    // Immediate feedback for significant events
    if (outcome.wasReassigned || outcome.failed) {
      await this.immediateAdjustment(outcome);
    }
  }

  private async updateModel(): Promise<void> {
    const trainingData = this.prepareTrainingData();
    
    // Update routing confidence based on outcomes
    const accuracyByType = this.calculateAccuracyByType();
    
    // Adjust complexity thresholds
    const optimalThresholds = this.findOptimalThresholds();
    
    // Update decision weights
    const newWeights = this.optimizeWeights(trainingData);
    
    this.model = {
      accuracyRates: accuracyByType,
      complexityThresholds: optimalThresholds,
      decisionWeights: newWeights,
      patterns: this.extractPatterns(trainingData)
    };
  }

  private calculateAccuracyByType(): AccuracyRates {
    const outcomes = {
      ai: { success: 0, total: 0 },
      human: { success: 0, total: 0 },
      hybrid: { success: 0, total: 0 }
    };

    this.historicalData.forEach(outcome => {
      const type = outcome.assignedTo;
      outcomes[type].total++;
      if (outcome.successful && !outcome.wasReassigned) {
        outcomes[type].success++;
      }
    });

    return {
      ai: outcomes.ai.total > 0 ? outcomes.ai.success / outcomes.ai.total : 0.5,
      human: outcomes.human.total > 0 ? outcomes.human.success / outcomes.human.total : 0.5,
      hybrid: outcomes.hybrid.total > 0 ? outcomes.hybrid.success / outcomes.hybrid.total : 0.5
    };
  }

  private findOptimalThresholds(): ComplexityThresholds {
    // Analyze misassignments
    const misassigned = this.historicalData.filter(o => o.wasReassigned);
    
    // Group by original assignment and complexity
    const byComplexity = misassigned.reduce((acc, outcome) => {
      const complexity = outcome.task.complexity;
      if (!acc[complexity]) acc[complexity] = { toHuman: 0, toAI: 0 };
      
      if (outcome.reassignedTo === 'human') acc[complexity].toHuman++;
      else if (outcome.reassignedTo === 'ai') acc[complexity].toAI++;
      
      return acc;
    }, {});

    // Find optimal cutoff points
    let aiThreshold = 3;
    let humanThreshold = 7;
    
    // Adjust based on reassignment patterns
    Object.entries(byComplexity).forEach(([complexity, data]) => {
      const comp = parseInt(complexity);
      if (data.toHuman > data.toAI && comp < humanThreshold) {
        aiThreshold = Math.min(aiThreshold, comp - 1);
      }
      if (data.toAI > data.toHuman && comp > aiThreshold) {
        humanThreshold = Math.max(humanThreshold, comp + 1);
      }
    });

    return { aiMax: aiThreshold, humanMin: humanThreshold };
  }

  private extractPatterns(data: TaskOutcome[]): RoutingPattern[] {
    const patterns: RoutingPattern[] = [];
    
    // Find successful AI patterns
    const successfulAI = data.filter(o => 
      o.assignedTo === 'ai' && o.successful && !o.wasReassigned
    );
    
    const aiPatterns = this.findCommonPatterns(successfulAI, 'ai');
    patterns.push(...aiPatterns);
    
    // Find successful human patterns
    const successfulHuman = data.filter(o => 
      o.assignedTo === 'human' && o.successful
    );
    
    const humanPatterns = this.findCommonPatterns(successfulHuman, 'human');
    patterns.push(...humanPatterns);
    
    return patterns;
  }

  private findCommonPatterns(
    outcomes: TaskOutcome[],
    assigneeType: string
  ): RoutingPattern[] {
    const patterns: Map<string, RoutingPattern> = new Map();
    
    outcomes.forEach(outcome => {
      // Extract task characteristics
      const characteristics = [
        outcome.task.type,
        outcome.task.tags?.join(','),
        outcome.task.complexity < 5 ? 'simple' : 'complex',
        outcome.task.hasSpecs ? 'specified' : 'unspecified'
      ].filter(Boolean).join('|');
      
      if (!patterns.has(characteristics)) {
        patterns.set(characteristics, {
          pattern: characteristics,
          assignee: assigneeType,
          confidence: 0,
          occurrences: 0,
          successRate: 0
        });
      }
      
      const pattern = patterns.get(characteristics)!;
      pattern.occurrences++;
      pattern.successRate = 
        (pattern.successRate * (pattern.occurrences - 1) + 1) / pattern.occurrences;
      pattern.confidence = pattern.successRate * Math.min(pattern.occurrences / 10, 1);
    });
    
    return Array.from(patterns.values())
      .filter(p => p.confidence > 0.7)
      .sort((a, b) => b.confidence - a.confidence);
  }
}
```

---

## Appendix D: Task Workflow Templates

### D.1 Feature Development Workflow

```typescript
const featureDevelopmentWorkflow: WorkflowTemplate = {
  name: 'Feature Development',
  description: 'Standard workflow for new feature implementation',
  
  stages: [
    {
      name: 'Planning',
      column: 'Ready',
      tasks: [
        {
          title: 'Feature Analysis',
          assignee: 'human',
          template: {
            description: 'Analyze requirements and create technical design',
            checklist: [
              'Review user requirements',
              'Create technical design document',
              'Identify dependencies',
              'Estimate effort'
            ],
            outputs: ['design-doc', 'estimates']
          }
        }
      ]
    },
    {
      name: 'Implementation',
      column: 'In Progress',
      parallel: true,
      tasks: [
        {
          title: 'Core Implementation',
          assignee: 'dynamic', // Based on complexity
          dependencies: ['Feature Analysis']
        },
        {
          title: 'Unit Tests',
          assignee: 'ai',
          dependencies: ['Core Implementation'],
          autoTrigger: true
        },
        {
          title: 'Documentation',
          assignee: 'ai',
          dependencies: ['Core Implementation'],
          autoTrigger: true
        }
      ]
    },
    {
      name: 'Quality Assurance',
      column: 'Quality Check',
      tasks: [
        {
          title: 'Code Review',
          assignee: 'human',
          dependencies: ['Core Implementation', 'Unit Tests']
        },
        {
          title: 'Integration Tests',
          assignee: 'ai',
          dependencies: ['Code Review']
        },
        {
          title: 'QA Testing',
          assignee: 'human',
          dependencies: ['Integration Tests']
        }
      ]
    },
    {
      name: 'Deployment',
      column: 'Ready to Deploy',
      tasks: [
        {
          title: 'Deploy to Staging',
          assignee: 'ai',
          requiresApproval: true
        },
        {
          title: 'Production Deploy',
          assignee: 'ai',
          requiresApproval: true,
          approvers: ['tech-lead', 'product-owner']
        }
      ]
    }
  ],
  
  automations: [
    {
      trigger: 'task.completed',
      condition: 'task.title === "Core Implementation"',
      actions: [
        'create-subtask:unit-tests',
        'create-subtask:documentation',
        'notify:reviewer'
      ]
    }
  ]
};
```

### D.2 Bug Fix Workflow

```typescript
const bugFixWorkflow: WorkflowTemplate = {
  name: 'Bug Fix',
  description: 'Workflow for fixing reported bugs',
  
  stages: [
    {
      name: 'Triage',
      column: 'Ready',
      tasks: [
        {
          title: 'Bug Analysis',
          assignee: 'ai',
          template: {
            description: 'Analyze bug report and reproduce issue',
            inputs: ['bug-report', 'logs', 'reproduction-steps'],
            outputs: ['root-cause', 'fix-approach']
          },
          estimatedTime: '30m'
        }
      ]
    },
    {
      name: 'Fix Implementation',
      column: 'In Progress',
      tasks: [
        {
          title: 'Implement Fix',
          assignee: 'dynamic',
          assignmentRules: {
            'complexity < 4': 'ai',
            'complexity >= 4': 'human',
            'critical-severity': 'human'
          }
        },
        {
          title: 'Regression Tests',
          assignee: 'ai',
          autoTrigger: true,
          template: {
            description: 'Create tests to prevent regression'
          }
        }
      ]
    },
    {
      name: 'Verification',
      column: 'Quality Check',
      tasks: [
        {
          title: 'Fix Verification',
          assignee: 'human',
          template: {
            checklist: [
              'Verify bug is fixed',
              'Check for side effects',
              'Validate performance impact'
            ]
          }
        }
      ]
    },
    {
      name: 'Release',
      column: 'Ready to Deploy',
      tasks: [
        {
          title: 'Hotfix Deploy',
          assignee: 'ai',
          condition: 'severity === "critical"',
          requiresApproval: true
        }
      ]
    }
  ]
};
```

### D.3 Security Review Workflow

```typescript
const securityReviewWorkflow: WorkflowTemplate = {
  name: 'Security Review',
  description: 'Security assessment and remediation workflow',
  
  stages: [
    {
      name: 'Initial Scan',
      column: 'Security Review',
      tasks: [
        {
          title: 'Automated Security Scan',
          assignee: 'ai',
          tools: ['snyk', 'codeql', 'custom-scanner'],
          parallel: true
        },
        {
          title: 'Dependency Audit',
          assignee: 'ai',
          tools: ['npm-audit', 'license-checker']
        }
      ]
    },
    {
      name: 'Analysis',
      column: 'In Progress',
      tasks: [
        {
          title: 'Vulnerability Assessment',
          assignee: 'human',
          template: {
            inputs: ['scan-results'],
            outputs: ['risk-assessment', 'remediation-plan']
          }
        },
        {
          title: 'False Positive Review',
          assignee: 'human',
          condition: 'scanResults.hasFindings'
        }
      ]
    },
    {
      name: 'Remediation',
      column: 'In Progress',
      tasks: [
        {
          title: 'Apply Security Patches',
          assignee: 'ai',
          condition: 'hasAutoFixAvailable'
        },
        {
          title: 'Manual Remediation',
          assignee: 'human',
          condition: '!hasAutoFixAvailable'
        }
      ]
    },
    {
      name: 'Validation',
      column: 'Quality Check',
      tasks: [
        {
          title: 'Re-scan',
          assignee: 'ai',
          autoTrigger: true
        },
        {
          title: 'Security Sign-off',
          assignee: 'human',
          role: 'security-team'
        }
      ]
    }
  ],
  
  sla: {
    'critical': '4 hours',
    'high': '24 hours',
    'medium': '72 hours',
    'low': '1 week'
  }
};
```

### D.4 Deployment Workflow

```typescript
const deploymentWorkflow: WorkflowTemplate = {
  name: 'Deployment',
  description: 'Production deployment workflow with safety checks',
  
  stages: [
    {
      name: 'Pre-deployment',
      column: 'Ready to Deploy',
      tasks: [
        {
          title: 'Pre-deployment Checks',
          assignee: 'ai',
          parallel: true,
          subtasks: [
            'Verify all tests passing',
            'Check dependency versions',
            'Validate configuration',
            'Review rollback plan'
          ]
        },
        {
          title: 'Change Summary',
          assignee: 'ai',
          template: {
            description: 'Generate deployment change summary',
            outputs: ['changelog', 'risk-assessment']
          }
        }
      ]
    },
    {
      name: 'Staging Deployment',
      column: 'In Progress',
      tasks: [
        {
          title: 'Deploy to Staging',
          assignee: 'ai',
          tools: ['kubernetes', 'terraform']
        },
        {
          title: 'Staging Validation',
          assignee: 'ai',
          subtasks: [
            'Health checks',
            'Smoke tests',
            'Performance baseline'
          ]
        },
        {
          title: 'UAT Sign-off',
          assignee: 'human',
          role: 'product-owner'
        }
      ]
    },
    {
      name: 'Production Deployment',
      column: 'In Progress',
      tasks: [
        {
          title: 'Production Deploy',
          assignee: 'ai',
          requiresApproval: true,
          approvers: ['tech-lead', 'ops-team'],
          strategy: 'blue-green'
        },
        {
          title: 'Production Monitoring',
          assignee: 'ai',
          duration: '2 hours',
          monitors: [
            'Error rates',
            'Response times',
            'Resource usage',
            'Business metrics'
          ]
        }
      ]
    },
    {
      name: 'Post-deployment',
      column: 'Done',
      tasks: [
        {
          title: 'Deployment Report',
          assignee: 'ai',
          autoTrigger: true
        },
        {
          title: 'Stakeholder Notification',
          assignee: 'ai',
          template: 'deployment-complete-notification'
        }
      ]
    }
  ],
  
  rollback: {
    automatic: {
      errorRateThreshold: 0.05,
      responseTimeThreshold: 2000,
      businessMetricDrop: 0.1
    },
    manual: {
      approvers: ['tech-lead', 'ops-team']
    }
  }
};
```

### D.5 Maintenance Workflow

```typescript
const maintenanceWorkflow: WorkflowTemplate = {
  name: 'Recurring Maintenance',
  description: 'Automated maintenance tasks',
  
  recurring: true,
  schedule: {
    dependencyUpdates: 'weekly',
    securityScans: 'daily',
    performanceAudits: 'monthly',
    codeQualityChecks: 'weekly'
  },
  
  tasks: [
    {
      name: 'Dependency Updates',
      assignee: 'ai',
      stages: [
        {
          title: 'Check for Updates',
          tools: ['npm-check', 'pip-audit']
        },
        {
          title: 'Test Compatibility',
          condition: 'hasUpdates',
          autoTrigger: true
        },
        {
          title: 'Apply Updates',
          condition: 'testsPass',
          requiresApproval: 'hasMajorUpdates'
        }
      ]
    },
    {
      name: 'Code Quality',
      assignee: 'ai',
      stages: [
        {
          title: 'Quality Scan',
          tools: ['sonarqube', 'eslint', 'prettier']
        },
        {
          title: 'Auto-fix Issues',
          condition: 'hasAutoFixable'
        },
        {
          title: 'Create Tech Debt Tasks',
          condition: 'hasComplexIssues'
        }
      ]
    },
    {
      name: 'Performance Audit',
      assignee: 'ai',
      stages: [
        {
          title: 'Performance Profiling',
          tools: ['lighthouse', 'custom-profiler']
        },
        {
          title: 'Optimization Opportunities',
          outputs: ['performance-report', 'optimization-tasks']
        }
      ]
    }
  ]
};
```

---

## Appendix E: Training Materials

### E.1 Task-Kanban Interface Guide

#### Getting Started with Task-Kanban

##### 1. Accessing the Task Board
```markdown
## Web Interface
- Navigate to http://localhost:3001/board
- Login with your agent credentials
- Select your primary board

## Cursor IDE
- Open Command Palette (Cmd/Ctrl + Shift + P)
- Type "ADP: Open Task Board"
- Tasks appear in sidebar panel

## CLI Access
```bash
# List your tasks
task-kanban list --assignee=me

# Start a task
task-kanban start TASK-123

# Update progress
task-kanban progress TASK-123 --percent=50
```
```

##### 2. Understanding Task States
```markdown
## Task Columns and Their Meanings

### Backlog
- Tasks not yet ready to work on
- May need more information
- No time pressure

### Ready
- Tasks with complete information
- Prioritized and estimated
- Available to start

### In Progress - Human
- Actively being worked on by human developer
- Shows in Cursor IDE
- Time tracking active

### In Progress - AI
- Being processed by AI agent
- Real-time progress updates
- May create subtasks

### Quality Check
- Automated quality gates running
- Linting, testing, security scans
- May return to In Progress if issues found

### Review
- Human review required
- Code review or approval needed
- Final quality check

### Done
- Task completed successfully
- All checks passed
- Archived after 30 days
```

### E.2 Agent Interaction Patterns

#### Master Orchestrator Patterns
```typescript
// Creating a Feature Epic
const epicPattern = {
  title: "Implement User Authentication",
  type: "epic",
  breakdown: [
    {
      title: "Design Authentication Flow",
      assignee: "human",
      complexity: 7,
      skills: ["architecture", "security"]
    },
    {
      title: "Implement JWT Service",
      assignee: "ai",
      complexity: 4,
      template: "service-implementation"
    },
    {
      title: "Create Login UI",
      assignee: "hybrid",
      complexity: 5,
      humanTasks: ["design", "ux"],
      aiTasks: ["implementation", "testing"]
    }
  ]
};

// Priority Adjustment Pattern
const priorityPattern = {
  trigger: "customer-escalation",
  action: "reprioritize",
  rules: [
    "Move customer-impacting bugs to critical",
    "Defer non-essential features",
    "Assign best available resource"
  ]
};
```

#### Development Agent Patterns
```typescript
// Human-AI Handoff Pattern
const handoffPattern = {
  from: "human",
  to: "ai",
  task: "Complete Unit Tests",
  context: {
    implementedFunctions: ["login", "logout", "refreshToken"],
    testFramework: "jest",
    coverageTarget: 90,
    specialInstructions: "Mock external auth service"
  }
};

// Quality Gate Pattern
const qualityPattern = {
  stage: "quality-check",
  gates: [
    { check: "linting", autoFix: true },
    { check: "tests", threshold: 85 },
    { check: "security", severity: "high" },
    { check: "documentation", required: ["public-api"] }
  ],
  onFailure: "create-fix-tasks"
};
```

### E.3 Best Practices for Task Creation

#### Writing Effective Task Descriptions
```markdown
## Good Task Description Example

**Title**: Implement Redis Caching for User Sessions

**Description**: 
Add Redis-based session caching to improve authentication performance.

**Acceptance Criteria**:
- [ ] Redis connection pool implemented
- [ ] Session read/write methods created
- [ ] TTL configurable via environment
- [ ] Fallback to database if Redis unavailable
- [ ] Performance improvement >50%

**Technical Notes**:
- Use existing Redis client in lib/cache
- Follow session interface in types/auth.ts
- Add monitoring metrics

**Complexity**: 5 (clear requirements, established patterns)
**Estimated Hours**: 4
**Assignee Recommendation**: AI (well-defined, pattern-based)
```

#### Task Decomposition Guidelines
```markdown
## When to Break Down Tasks

### Size Indicators (break down if):
- Estimated time > 8 hours
- Multiple skill sets required
- Unclear requirements
- Multiple deliverables

### Good Decomposition Example:
Original: "Add Search Feature"

Broken down:
1. "Design Search API Schema" (Human, 2h)
2. "Implement Search Index" (AI, 4h)
3. "Create Search UI Component" (Human, 3h)
4. "Add Search Analytics" (AI, 2h)
5. "Performance Test Search" (AI, 1h)

### Dependencies:
- UI depends on API
- Analytics depends on UI
- Performance depends on all
```

### E.4 Troubleshooting Guide

#### Common Issues and Solutions

##### 1. Task Stuck in Column
```markdown
**Problem**: Task not moving from "In Progress" to "Quality Check"

**Diagnostic Steps**:
1. Check task details: `task-kanban show TASK-123`
2. View task history: `task-kanban history TASK-123`
3. Check for blockers: `task-kanban blockers TASK-123`

**Common Solutions**:
- Missing required fields (e.g., branch name)
- Failed automated checks
- Waiting for human approval
- API rate limits hit

**Resolution**:
```bash
# Force transition (with reason)
task-kanban move TASK-123 --to="Quality Check" --reason="Manual override: tests fixed"
```
```

##### 2. AI Agent Not Picking Up Tasks
```markdown
**Problem**: AI tasks sitting in "Ready" column

**Diagnostic Steps**:
1. Check AI agent status: `adp status development-agent`
2. View agent workload: `adp workload --agent=ai`
3. Check error logs: `adp logs --agent=development --level=error`

**Common Causes**:
- Agent at capacity (20 concurrent tasks)
- API rate limits reached
- Missing required context
- Agent crashed/disconnected

**Resolution**:
- Restart agent: `adp restart development-agent`
- Clear stuck tasks: `adp clear-stuck --agent=ai`
- Increase capacity: Update config.json
```

##### 3. Human-AI Handoff Failures
```markdown
**Problem**: Context lost during handoff

**Prevention**:
1. Always use handoff template
2. Include all relevant files
3. Specify output format
4. Add verification criteria

**Template**:
```typescript
{
  handoffType: "human-to-ai",
  task: "TASK-123",
  context: {
    completedWork: ["API design", "Database schema"],
    remainingWork: ["Implementation", "Tests"],
    files: ["src/api/auth.ts", "docs/api.md"],
    constraints: ["Must use existing patterns"],
    expectedOutput: ["Working endpoints", "90% test coverage"]
  }
}
```
```

### E.5 Video Tutorials

#### Available Training Videos

1. **Getting Started (15 min)**
   - System overview
   - First task creation
   - Basic workflow

2. **Task Management Deep Dive (30 min)**
   - Complex task patterns
   - Dependency management
   - Priority optimization

3. **Human-AI Collaboration (25 min)**
   - Effective handoffs
   - Hybrid workflows
   - Best practices

4. **Troubleshooting Common Issues (20 min)**
   - Debugging stuck tasks
   - Performance optimization
   - Error resolution

5. **Advanced Features (45 min)**
   - Custom workflows
   - Automation rules
   - Analytics and reporting

#### Quick Reference Commands
```bash
# Essential Commands Cheatsheet

# Task Operations
task-kanban create --title="Fix login bug" --priority=high
task-kanban assign TASK-123 --to=ai
task-kanban start TASK-123
task-kanban complete TASK-123 --with-results results.json

# Board Management
task-kanban boards list
task-kanban board show main-dev
task-kanban board create --name="Experiments" --columns="Idea,Testing,Done"

#