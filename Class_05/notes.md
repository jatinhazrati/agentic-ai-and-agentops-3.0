# Class 5 - Python and Agents

### Overview

- Focus: pure vanilla Python implementation of AI concepts, no frameworks
- Goal: understand internals before moving to LangChain (starting next class)
- Files covered: 01 through 06 in the class repo (GitHub link shared in chat)

### Simple things

- Groq is repository of open source models

### AI Model vs. Chatbot vs. Agent

- AI model: stateless, single-shot prediction
  - Takes a question, returns an answer, remembers nothing
  - Has a context window, output token limit, and a knowledge cutoff date
- Chatbot: AI model + conversation history
  - Appends every user message and assistant reply to a running list
  - Full history sent on every call, which grows token cost over time
- Agent: chatbot + tools + a decision loop
  - Can call external tools (web search, calculator, weather API, etc.)
  - Loops until it has a final answer to return
  
### System Messages and Message Roles

- Three message roles: system, user, assistant
- System message sets the AI’s persona and context
  - Sent on every call, not just the first one (AI is stateless)
  - Optional but good practice; adds tokens to every request
- Even sending one token to Claude costs ~19k tokens due to its system prompt
- Claude’s system prompt used as a live example of how real products configure AI

### Calling Real AI APIs

- Two categories of providers:
  - Paid: OpenAI (GPT), Anthropic (Claude)
  - Free/cheaper: Groq, OpenRouter
- OpenAI call structure: client.chat.completions.create(model, max_tokens, messages)
  - Response object includes choices, message content, and token usage
  - response.choices[0].message.content extracts the answer
- Groq is OpenAI-compatible: swap the base_url and API key, code stays the same
  - Has some free open-source models (Llama 3.3 70B used in demo)
  - Usage and token counts visible in the Groq dashboard
- API keys stored in .env; load_dotenv() loads them at runtime

### Structured Output with Pydantic

- Raw AI replies are always strings, even when they look like JSON
- Structured output matters when the reply feeds downstream application logic
  - Example: email reply needs subject and body fields, not a prose paragraph
- Approach: instruct the model to reply as a JSON object with a defined shape
  - Instruction sent as a user message (not a system message in the demo)
  - Strip any extra prefix/suffix (e.g. json ... ) before parsing
- Parse the cleaned string → json.loads() → cast into a Pydantic model
- Pydantic benefits over raw prompt instructions: default values, field descriptions, validation, code reuse
- LangChain’s structured output (passing a Pydantic schema as response_format) handles this automatically; shown briefly as a preview

### Tool Calling: Manual vs. AI-Directed

- Manual approach (File 04): developer decides which tool to call based on keyword matching
  - Simple but doesn’t scale beyond a handful of tools
- AI-directed approach (File 05): pass a tool schema to the model; the model decides
  - Schema fields: type, name, description, parameters (with types and required)
  - Good description = better tool selection; treat it like documentation for the AI
  - AI returns a tool_calls object with the function name and arguments, not the answer
  - Developer code then executes the named function with those arguments
- Key rule: the AI never calls the tool itself; it only tells you which tool to call and with what parameters
- If the question doesn’t need a tool, the AI returns a plain text answer (demonstrated with “Hi, how are you?”)
- Multiple tools can be registered; AI picks the right one per query (weather vs. calculator demo)

### The Agentic Loop (File 06)

- Agent components: brain (LLM client), memory (message list), tools (schemas + functions)
- Loop logic:
  1. Call AI with current message history and tool schemas
  2. If response has no tool calls → return text answer to user
  3. If response has tool calls → execute the tool, append result to history, loop again
- Max iterations set to 4 (prevents infinite loops)
- This is the same pattern all frameworks (LangChain, LangGraph) implement internally
- Understanding this loop is why pure-Python first matters: frameworks abstract it, but you need to control it

### Next Steps

- Next class: first 30 minutes to recap and demo the agentic loop live, then start LangChain
- LangChain, LangGraph, and deep agents are the upcoming topics

### Action Items

- **Revise all six Python files from today's class**
  - Required before the next session; LangChain builds directly on these concepts.