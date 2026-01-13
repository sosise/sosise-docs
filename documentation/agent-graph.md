# Agent Graph

## Quick Start

Agent Graph enables you to build multi-agent LLM-powered workflows where multiple AI agents collaborate to solve complex tasks. Each agent has a specific role and can pass work to other agents!

### Simple Example

```bash
# Initialize multi-agent system
./artisan ma:init

# Run the example
./artisan ma:run
```

That's it! The Worker agent extracts the data, the Validator agent checks it, and you get structured JSON output.

## Introduction

Agent Graph implements the multi-agent orchestration pattern, where multiple specialized AI agents work together in a coordinated workflow. Think of it as an assembly line where each station (agent) performs a specific task and passes the result to the next station.

Sosise Agent Graph provides:
- **AgentGraphService** - Main orchestrator managing agent execution flow
- **Worker-Validator pattern** - Ready-to-use template for validation workflows
- **Three-tier memory system** - Agent-private, shared, and global memory
- **Two storage drivers** - InMemory (development) and File (production)
- **Visual logging** - Beautiful console output with colors and icons
- **LLM integrations** - OpenAI and Groq support out of the box

## Getting Started

### Initialize Multi-Agent System

```bash
./artisan ma:init [ServiceName]
```

This command creates all necessary files and directories:

```
src/app/
├── Services/MA/
│   ├── MultiagentService.ts      # Main service
│   └── Agents/
│       ├── WorkerAgent.ts        # Example worker agent
│       └── ValidatorAgent.ts     # Example validator agent
├── Repositories/
│   ├── LLM/
│   │   ├── LLMRepositoryInterface.ts
│   │   ├── OpenAICompletionRepository.ts
│   │   └── GroqCompletionRepository.ts
│   └── Prompt/
│       ├── PromptRepositoryInterface.ts
│       └── PromptRepository.ts
├── Types/LLM/
│   ├── ChatMessageType.ts
│   └── CompletionResponseType.ts
├── Enums/LLM/
│   └── CompletionModelsEnum.ts
└── Console/Commands/
    └── MACommand.ts              # Run command

storage/ma/
└── prompts/
    ├── worker-agent.txt          # Worker prompt template
    └── validator-agent.txt       # Validator prompt template
```

### Create Custom Agent

```bash
./artisan ma:create MyCustomAgent
```

This creates a new agent file in `src/app/Services/MA/Agents/MyCustomAgent.ts`.

## Core Concepts

### Agents

Each agent implements the `BaseAgentInterface` with two required properties and one method:

```typescript
import AbstractBaseAgent from 'sosise-core/build/Services/AgentGraph/AbstractBaseAgent';
import { AgentContextType, AgentResponseType } from 'sosise-core/build/Types/AgentGraph/AgentGraphTypes';
import AgentMessageTypeEnum from 'sosise-core/build/Enums/AgentGraph/AgentMessageTypeEnum';

export default class MyAgent extends AbstractBaseAgent {
    // Unique identifier for routing between agents
    public readonly id: string = 'my-agent';

    // Human-readable name for logs
    public readonly name: string = 'My Agent';

    // Main processing method
    public async process(context: AgentContextType): Promise<AgentResponseType> {
        // Your agent logic here

        return {
            content: 'Result from my agent',
            type: AgentMessageTypeEnum.RESULT,
            next: 'next-agent-id', // or 'END' to finish
        };
    }
}
```

### Message Types

Agents communicate using three message types:

| Type | Description | Use Case |
|------|-------------|----------|
| `TASK` | Initial task or new work request | User input, delegating work |
| `RESULT` | Completed work output | Returning processed data |
| `FEEDBACK` | Correction or improvement request | Validation failures, retry requests |

```typescript
import AgentMessageTypeEnum from 'sosise-core/build/Enums/AgentGraph/AgentMessageTypeEnum';

// In agent response
return {
    content: 'Task completed successfully',
    type: AgentMessageTypeEnum.RESULT,  // TASK, RESULT, or FEEDBACK
    next: 'END',
};
```

### Agent Context

Every agent receives a context object with all the information needed to process:

```typescript
interface AgentContextType {
    threadId: string;                    // Current thread identifier
    message: AgentMessageType;           // Current message to process
    history: AgentMessageType[];         // All previous messages
    memory: ScopedMemoryInterface;       // Memory access (see Memory System)
    initialMessage: string;              // Original user input
    currentAgentIterations: number;      // How many times THIS agent ran
    totalIterations: number;             // Total iterations across all agents
}
```

Example usage:

```typescript
public async process(context: AgentContextType): Promise<AgentResponseType> {
    // Access the current message content
    const input = context.message.content;

    // Check who sent the message
    const sender = context.message.from.name;

    // Check message type
    if (context.message.type === AgentMessageTypeEnum.FEEDBACK) {
        // Handle feedback - retry with corrections
    }

    // Access original user request
    const originalTask = context.initialMessage;

    // Prevent infinite loops
    if (context.currentAgentIterations >= 3) {
        return { content: 'Max retries exceeded', type: AgentMessageTypeEnum.RESULT, next: 'END' };
    }

    // Use memory to store/retrieve data
    await context.memory.set('lastProcessed', input);
    const previousData = await context.memory.get('lastProcessed');

    // ...
}
```

### Memory System

Agent Graph provides three levels of memory:

#### Agent Memory (Private)
Data accessible only by the specific agent within a thread.

```typescript
// Store private data
await context.memory.set('myKey', { data: 'value' });

// Retrieve private data
const data = await context.memory.get('myKey');
```

#### Shared Memory (Thread-Level)
Data accessible by all agents within the same thread.

```typescript
// Store shared data (accessible by all agents in thread)
await context.memory.setShared('extractedData', { name: 'John' });

// Retrieve shared data from any agent
const shared = await context.memory.getShared('extractedData');
```

#### Global Memory
Data accessible across all threads and agents.

```typescript
// Store global data
await context.memory.setGlobal('config', { maxRetries: 3 });

// Retrieve global data from anywhere
const config = await context.memory.getGlobal('config');
```

### Storage Drivers

#### InMemoryStorageService
Fast, ephemeral storage for development and testing. Data is lost when the process ends.

```typescript
import InMemoryStorageService from 'sosise-core/build/Services/AgentGraph/Memory/InMemoryStorageService';

const memory = new InMemoryStorageService();
```

#### FileStorageService
Persistent storage for production. Saves data to the filesystem.

```typescript
import FileStorageService from 'sosise-core/build/Services/AgentGraph/Memory/FileStorageService';

const memory = new FileStorageService('storage/ma');
```

## Worker-Validator Pattern

The Worker-Validator pattern is a common multi-agent workflow where:
1. **Worker** processes the task
2. **Validator** checks the result
3. If invalid, **Validator** sends feedback to **Worker**
4. **Worker** retries with the feedback
5. Repeat until valid or max iterations reached

### Flow Diagram

```
User Input
    │
    ▼
┌─────────┐
│ Worker  │◄────────────────┐
└────┬────┘                 │
     │ RESULT               │ FEEDBACK
     ▼                      │
┌───────────┐               │
│ Validator │───────────────┘
└─────┬─────┘
      │ RESULT (approved)
      ▼
     END
```

### Example: Data Extraction

**WorkerAgent** extracts structured data:

```typescript
public async process(context: AgentContextType): Promise<AgentResponseType> {
    const prompt = this.promptRepository.getPrompt('worker-agent.txt');

    // Include feedback if this is a retry
    const isFeedback = context.message.type === AgentMessageTypeEnum.FEEDBACK;
    const fullPrompt = prompt
        .replace('%message%', context.initialMessage)
        .replace('%feedback%', isFeedback ? context.message.content : 'None');

    const response = await this.llm.groq.getCompletion(
        [{ role: 'user', content: fullPrompt }],
        CompletionModelsEnum.GPT_OSS_20B,
        0
    );

    return {
        content: response.responseText,
        type: AgentMessageTypeEnum.RESULT,
        next: 'validator-agent',
        metadata: { promptFile: 'worker-agent.txt' },
    };
}
```

**ValidatorAgent** validates the result:

```typescript
public async process(context: AgentContextType): Promise<AgentResponseType> {
    // Prevent infinite loops
    if (context.currentAgentIterations >= this.MAX_ITERATIONS) {
        return {
            content: { error: 'Max iterations exceeded', lastAttempt: context.message.content },
            type: AgentMessageTypeEnum.RESULT,
            next: 'END',
        };
    }

    // Validate using LLM
    const validationResult = await this.validate(context);

    if (validationResult.approved) {
        // Success - return final result
        return {
            content: JSON.parse(context.message.content),
            type: AgentMessageTypeEnum.RESULT,
            next: 'END',
        };
    }

    // Not approved - send feedback back to worker
    return {
        content: validationResult.feedback,
        type: AgentMessageTypeEnum.FEEDBACK,
        next: context.message.from.id,  // Return to the sender
    };
}
```

## Running the System

### Creating the Service

```typescript
import AgentGraphService from 'sosise-core/build/Services/AgentGraph/AgentGraphService';
import FileStorageService from 'sosise-core/build/Services/AgentGraph/Memory/FileStorageService';

export default class MultiagentService {
    private graph: AgentGraphService;

    constructor() {
        const memory = new FileStorageService('storage/ma');
        const promptRepository = new PromptRepository();

        // Create LLM providers
        const llmProviders = {
            openai: new OpenAICompletionRepository(),
            groq: new GroqCompletionRepository(),
        };

        // Create agents
        const workerAgent = new WorkerAgent(llmProviders, promptRepository);
        const validatorAgent = new ValidatorAgent(llmProviders, promptRepository);

        // Create graph
        this.graph = new AgentGraphService({
            agents: [workerAgent, validatorAgent],
            start: workerAgent,
            memory: memory,
        });
    }

    public async run(input: string, threadId?: string): Promise<GraphResultType> {
        const id = threadId ?? `thread-${Date.now()}`;
        return this.graph.run(input, { threadId: id });
    }
}
```

### Graph Configuration

```typescript
interface AgentGraphConfigType {
    agents: BaseAgentInterface[];  // All agents in the graph
    start: BaseAgentInterface;     // Starting agent
    memory?: GraphStorageInterface; // Optional custom storage (default: InMemory)
}
```

### Running and Getting Results

```typescript
const service = new MultiagentService();

const result = await service.run('Extract name and email from: John Smith, john@example.com');

console.log(result.status);   // 'completed' or 'error'
console.log(result.content);  // Final output from the last agent
console.log(result.threadId); // Thread identifier
console.log(result.error);    // Error message if status is 'error'
```

## Configuration

### LLM Repositories

The system supports multiple LLM providers through repository interfaces:

#### OpenAI

```typescript
export default class OpenAICompletionRepository implements LLMRepositoryInterface {
    public async getCompletion(
        messages: ChatMessageType[],
        model: CompletionModelsEnum,
        temperature: number
    ): Promise<CompletionResponseType> {
        // Implementation using OpenAI API
    }
}
```

#### Groq

```typescript
export default class GroqCompletionRepository implements LLMRepositoryInterface {
    public async getCompletion(
        messages: ChatMessageType[],
        model: CompletionModelsEnum,
        temperature: number
    ): Promise<CompletionResponseType> {
        // Implementation using Groq API
    }
}
```

#### Creating Custom LLM Repository

```typescript
import LLMRepositoryInterface from './LLMRepositoryInterface';

export default class AnthropicRepository implements LLMRepositoryInterface {
    public async getCompletion(
        messages: ChatMessageType[],
        model: CompletionModelsEnum,
        temperature: number
    ): Promise<CompletionResponseType> {
        // Your Anthropic Claude implementation
        return {
            responseText: 'Response from Claude',
            usage: { promptTokens: 100, completionTokens: 50, totalTokens: 150 },
        };
    }
}
```

### Prompts

Prompts are stored in `storage/ma/prompts/` and loaded via `PromptRepository`:

```typescript
// Load prompt from file
const prompt = this.promptRepository.getPrompt('worker-agent.txt');

// Replace placeholders
const fullPrompt = prompt
    .replace('%message%', userInput)
    .replace('%feedback%', feedback || 'None');
```

Example prompt file (`storage/ma/prompts/worker-agent.txt`):

```
You are a data extraction agent. Extract structured information from the text.

Input text:
%message%

Previous feedback (if any):
%feedback%

Return a JSON object with the extracted data.
```

## Real-World Examples

### Data Extraction Pipeline

```typescript
// Agents: Extractor -> Validator -> Enricher -> END

const extractorAgent = new ExtractorAgent(llm, prompts);  // Extracts raw data
const validatorAgent = new ValidatorAgent(llm, prompts);  // Validates structure
const enricherAgent = new EnricherAgent(llm, prompts);    // Adds metadata

const graph = new AgentGraphService({
    agents: [extractorAgent, validatorAgent, enricherAgent],
    start: extractorAgent,
});
```

### Content Generation with Review

```typescript
// Agents: Writer -> Reviewer -> (feedback loop) -> END

const writerAgent = new WriterAgent(llm, prompts);
const reviewerAgent = new ReviewerAgent(llm, prompts);

// Writer generates content, Reviewer approves or sends feedback
```

### Multi-Step Analysis

```typescript
// Agents: Analyzer -> Summarizer -> Reporter -> END

const analyzerAgent = new AnalyzerAgent(llm, prompts);   // Deep analysis
const summarizerAgent = new SummarizerAgent(llm, prompts); // Create summary
const reporterAgent = new ReporterAgent(llm, prompts);    // Format report
```

## Best Practices

### 1. Use Clear Agent IDs and Names

```typescript
// Good: Clear and descriptive
public readonly id: string = 'data-extractor';
public readonly name: string = 'Data Extractor Agent';

// Avoid: Vague or confusing
public readonly id: string = 'agent1';
public readonly name: string = 'A1';
```

### 2. Limit Iterations

Always protect against infinite loops:

```typescript
private readonly MAX_ITERATIONS: number = 3;

public async process(context: AgentContextType): Promise<AgentResponseType> {
    if (context.currentAgentIterations >= this.MAX_ITERATIONS) {
        return {
            content: { error: 'Max iterations exceeded' },
            type: AgentMessageTypeEnum.RESULT,
            next: 'END',
        };
    }
    // ...
}
```

### 3. Handle Errors Gracefully

```typescript
public async process(context: AgentContextType): Promise<AgentResponseType> {
    try {
        const result = await this.llm.groq.getCompletion(...);
        return { content: result, type: AgentMessageTypeEnum.RESULT, next: 'validator' };
    } catch (error) {
        return {
            content: { error: error.message },
            type: AgentMessageTypeEnum.RESULT,
            next: 'END',
        };
    }
}
```

### 4. Use Metadata for Context

```typescript
return {
    content: result,
    type: AgentMessageTypeEnum.RESULT,
    next: 'validator-agent',
    metadata: {
        promptFile: 'worker-agent.txt',
        model: 'gpt-4',
        processingTime: Date.now() - startTime,
    },
};
```

### 5. Use Shared Memory for Cross-Agent Data

```typescript
// In WorkerAgent
await context.memory.setShared('extractedEntities', entities);

// In ValidatorAgent
const entities = await context.memory.getShared('extractedEntities');
```

## Summary

Agent Graph provides a powerful framework for building multi-agent LLM workflows:

- **Orchestration** - Manage complex agent workflows with AgentGraphService
- **Flexibility** - Create custom agents for any task
- **Memory** - Three-tier memory system for data sharing
- **Persistence** - File storage for production deployments
- **Validation** - Built-in Worker-Validator pattern
- **Logging** - Beautiful visual console output
- **LLM Support** - OpenAI and Groq out of the box

Use Agent Graph to build sophisticated AI applications where multiple specialized agents collaborate to solve complex problems!
