# Class 07: LangChain fundamentals — models, agents, and harness engineering

## The LangChain / LangGraph Family

- Four offerings in the “Lang family”:
  - LangGraph: lowest-level orchestration, maximum control
  - LangChain: built on LangGraph, the primary focus of this course
  - Deep Agents: batteries-included harness on top of LangChain agents (released March 2025)
  - LangSmith: observability and tracing only, not for building
- LangFuse: has “lang” in the name but is a separate open-source platform, not part of the family

## Hierarchy: LangGraph, LangChain, Deep Agents

- LangGraph: load-bearing base, full control, most effort
  - Analogous to buying vegetables and cooking from scratch
  - Low-level = minimal abstraction, direct control over execution
- LangChain: built on LangGraph, pre-built scaffolding with customizable harness
  - Analogous to a full kitchen where you control every ingredient
- Deep Agents: highest abstraction, least configuration
  - Analogous to ordering food via an app: fast, but little control
  - “Batteries included”: context compression, virtual file system, sub-agent spawning

## Harness Engineering

- Model alone = raw engine: stateless, no tools, no memory, can’t send emails or browse
- Agent = model + harness
- Harness components: system prompt, tools, middleware, guardrails, checkpoints, memory, sub-agents
- LangChain’s role: provides the infrastructure to harness any LLM effectively
- Changing the model tomorrow requires only one line change in LangChain

## LangChain Timeline

- Oct 2022: LangChain launched with LLM abstraction and chains (sequential computation steps)
- Dec 2022: ReAct agent introduced (Reason + Act), first general-purpose agent
- Feb 2024: LangGraph released as the low-level orchestration layer
- Oct 2024: LangGraph becomes the preferred approach; old LangChain agents deprecated
- Oct 2025: LangChain v1.0 released, built on LangGraph; pre-v1 code moved to langchain-classic
- Mar 2026: Deep Agents released

## LangSmith: Observability

- Traces the full agent lifecycle: LLM calls, tool calls, memory saves, timing, token usage
- Analogous to a flight’s black box recorder
- Cannot infer agent behavior from code alone; traces make it visible

## Environment Setup

- Using uv for virtual environment (uv init langchain-course)
- Key libraries installed via uv add; more added as needed
- .env file stores all API keys (OpenAI, Groq, Anthropic, etc.)
- .gitignore includes .env so keys are never pushed to GitHub
- .env.example committed as a placeholder template
- Google Colab used for all teaching/demos; VS Code used for projects with multiple files
- All notebooks pushed to the shared GitHub repo

## Models: Types, Providers, Free vs. Paid

- Model quality directly impacts agent performance
- LangChain unifies all model calls: one interface for OpenAI, Anthropic, Gemini, Groq, Ollama, etc.
  - Without LangChain: each provider has a different API shape (client.chat.completions.create vs client.messages.create vs models.generate_content)
- Free vs. paid:
  - Paid: OpenAI, Anthropic, Gemini (beyond free tier)
  - Free-tier aggregators: OpenRouter (50 req/day, 20 req/min on free tier; $10 credit unlocks 2,000/day)
  - Truly free (self-hosted): open-source models via Ollama or LM Studio
- Closed source vs. open source:
  - Closed: GPT, Claude, Gemini; weights not released; cannot self-host
  - Open source: DeepSeek, Llama, Mistral; weights available; can run locally via Ollama
- OpenRouter routes to multiple providers automatically; free model variant IDs end in /free
- init_chat_model is the recommended LangChain approach for model-agnostic code

## Code Walkthrough: Calling Models

- Basic invoke pattern:
  - from langchain.chat_models import init_chat_model
  - Define model, call .invoke() with a message string or message list
  - Print response.content for the answer
- Using SystemMessage + HumanMessage from langchain_core.messages for prompt control
- Response object contains: content, content_blocks, id, tool_calls, usage_metadata, response_metadata
- Same response structure regardless of provider (OpenAI, Anthropic, OpenRouter)
- Environment key loading: load_dotenv() + os.environ.get("OPENAI_API_KEY")
- Colab equivalent: Secrets panel + userdata.get("OPENAI_API_KEY")

# Class 08: Lang chain models, messages, and structured output — in depth

## LangChain Ecosystem Recap

- Lang family: LangChain, LangGraph, LangSmith, DeepAgents, LangFlow
  - LangGraph provides foundation; LangChain snaps agents together
  - DeepAgents: mid-way between control and automation
- Agent = Model + Harness
  - Claude Web is an agent (has memory, tools, model)
  - Pure AI is just the model; everything around it is built
- Focus: LangChain v1.0 (latest); prior-version content is widely available but shallow

## Model Initialization and Parameters

- Two init approaches: ChatModel class and init_chat_model
- Key parameters:
  - model: required
  - api_key: optional if env var is set (e.g. OPENAI_API_KEY)
  - temperature: randomness control (low = deterministic, high = creative)
  - max_tokens: caps output length; source of many open-router errors
  - timeout: max wait before canceling request
  - max_retries: default 6
- Model metadata exposes LangChain version, max input/output tokens, supported modalities
  - Use this to check if a model supports image output before wasting time testing

## Model Providers and Free Tier Limits

- Paid: OpenAI, Anthropic, Gemini
- Free via OpenRouter: 20 req/min, 50 req/day (even for free-tier models like Nvidia Nemotron)
- Groq: hosts open-source models on its own infra; free but less powerful
  - Install: pip install langchain-groq; set Groq API key
- Trade-off: free models train on your data; paid models do not

## Three Ways to Call a Model

- invoke: single message or list of messages; most straightforward
- stream: yields output chunks as generated; much better UX for long responses
  - Returns AIMessageChunk objects; collect with a loop
  - Use time.sleep() to visualize token-by-token output in demos
  - Each chunk ≈ one output token
- batch: sends multiple independent requests in parallel
  - Reduces cost and time vs. sequential calls
  - batch_as_completed: streams results as each finishes, no waiting for slowest
  - Combine batch + stream for parallel streaming

## Message Types in Depth

- Four core types in langchain_core.messages:
  - SystemMessage: primes model behavior; set by developer, not overridable by user
  - HumanMessage: user input; can be text, image, audio, file, or multimodal
  - AIMessage: model output; can include content, tool calls, usage metadata
  - ToolMessage: wraps tool result with tool_call_id; required for LangChain to process tool output
- Messages can also carry optional metadata: name, id, session context
  - Useful for multi-user chatbots, filtering logs, debugging errors
- Chunk variants exist: AIMessageChunk, HumanMessageChunk
  - Always handle voice/audio apps in chunks, never wait for full message
- Dictionary format also accepted: {"role": "user", "content": "..."} — class form preferred

## Tool Binding and Tool Calls

- Bind tools to model: model.with_tools([get_weather])
  - LangChain accepts raw Python functions directly (no schema needed)
- On invoke, model returns AIMessage with empty content but populated tool_calls
  - Model tells you _which_ tool to call and with _which_ arguments
  - Model does NOT call the tool itself; that is always the developer’s responsibility
- After calling the tool manually, wrap result in ToolMessage:
  - Include content (tool result) and tool_call_id (from AI message)
- Full message sequence: HumanMessage → AIMessage (tool call) → ToolMessage (result) → final AIMessage
- Never pass raw tool output directly; LangChain requires one of the four message types

## Structured Output

- model.with_structured_output(EmailClass): forces model to return a Pydantic object
- Define schema with BaseModel and Field(description=...)
- Response type is the Pydantic class, not AIMessage
- Downstream logic depending on the class will never fail due to unexpected format
- Provider/tool-calling strategy for structured output: covered after tools section

## Roadmap and Projects

- Next topics: Tools, Agents, Middleware, RAG
- Four projects planned for LangChain completion:
  - SQL Agent, RAG Agent (mini projects)
  - Multi-modal agent (mini)
  - Fraud Detection System (major): FastAPI, LangChain agents, GCP Cloud Run, TypeScript/Tailwind frontend, CI/CD
- Deployment: starting with GCP (Cloud Run)
- Interview questions for LangChain to be sourced from real job postings (not AI-generated Q&A) and shared via support channel