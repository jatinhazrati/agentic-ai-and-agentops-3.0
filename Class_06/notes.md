# Class 06: Agents in Python — memory, tools, and agentic loops

### Revision: AI Models, Chatbots, and Agents

- Three-tier hierarchy: AI model (single response), chatbot (maintains memory), agent (memory + tools + loop)
- Agent = LLM + tools + memory
- All major providers (OpenAI, Grok, Anthropic, OpenRouter) follow OpenAI’s API structure as the de facto standard
  - Grok and OpenRouter are OpenAI-compatible: just swap base_url and API key, no other code changes
  - Anthropic is the exception: its own API structure, no chat.completions.create

### Structured Output with Pydantic

- AI communicates purely via strings/tokens; structured output requires explicit parsing on the Python side
- Workflow: define a Pydantic model (e.g. WeatherQuestion) → pass schema in prompt → extract fields from response
- Structured output enables clean argument passing to tools (e.g. extracting city to call get_weather)

### Tool Calling Mechanics

- Tool schema tells the LLM what tools exist, their inputs, and what they do
  - Description is optional but strongly recommended; without it, the LLM guesses and may hallucinate
  - Future shortcut: @tool decorator auto-generates schema from docstrings
- LLM returns one of two things: a plain text reply, or a structured tool call with function name + arguments
  - Tool call ID is assigned by the LLM; must be echoed back in the result message so the LLM can match responses to requests
- tools_by_name dict maps string function names (returned by LLM) to actual Python callables

### The Agentic Loop

- Loop is required because a single LLM call may return a tool call, not a final answer
- Loop flow per turn:
  1. Call LLM with full message history + tool schemas
  2. If no tool calls: return message.content directly
  3. If tool calls: execute each tool, append results with matching tool call IDs, loop again
  4. On next iteration, LLM reads tool results and produces final natural language answer
- max_turns (default 4–5) prevents infinite loops and runaway hallucination
  - Claude and other providers also enforce their own hard stop; users sometimes see “maximum tool calls reached, click continue”
- Multiple tool calls can be requested in a single LLM response; each gets a unique ID

### Memory Management

- Memory = a running list of message dicts (role + content)
- Message roles: system, user, assistant, tool
- Memory persists within a session; cleared on restart unless persisted to a DB or file
- Demonstrated live: after “Hi, I’m Mayank” → “Who am I?” correctly answered because prior messages were in context

### Live Demo: Mini Agent (Files 6 & 7)

- File 6: single-tool agent (get_weather) built from scratch in Python, run via terminal
  - Showed full message history in JSON after each turn; traced tool call ID matching end-to-end
  - Google Colab notebook version created during break for step-by-step visual walkthrough
- File 7 (Streamlit app): three tools: get_weather (mocked), calculator, convert_currency (live Frankfurter API)
  - Demonstrated real USD→INR and EUR→INR conversion via live API
  - Showed two tool calls in one query: “What is the weather in Tokyo and the USD to INR rate?”
- Caching note: cache TTL should match data volatility (weather: ~1 day; currency: ~10 min; stock prices: never)

### LangChain Introduction

- LangChain = framework (harness) around the LLM; handles loop, memory, tool dispatch automatically
- Analogy: deep agent = Swiggy order (no control); LangChain = home-cooked meal (configurable); LangGraph = raw ingredients (lowest-level orchestration)
- Using LangChain v1 (latest: ~1.3.x); legacy “classic” version is pre-v1, widely found on YouTube but not what’s being taught
- Key install: uv add langchain langchain-openai langchain-groq
- create_react_agent(model, tools, prompt) replaces the entire hand-rolled Python loop
- Printed raw output showed LangChain internally doing exactly what was built manually: tool call ID, two loop iterations, token counts
- Upcoming path: LangChain → LangGraph → DeepAgent (LangSmith for monitoring covered separately)

### Project Setup and Best Practices

- Use uv for environment management: uv init, uv add <package>, uv sync; no manual venv activation needed
- .env for secrets; .gitignore must exclude .env and .venv
- Commit .env.example (placeholder keys) so collaborators know what variables are needed
- pyproject.toml replaces requirements.txt in the uv workflow
- Core advice: frameworks abstract the loop, but understanding the internals (memory, tool IDs, loop logic) is what matters for building production software and passing senior engineering interviews