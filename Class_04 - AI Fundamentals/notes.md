# Class 04: AI fundamentals - LLMs, tokens, embeddings, and context windows

### LLMs: Core Concept

- Trained on vast text data; predicts the next token, not looking up facts
- ChatGPT is the canonical example: “ChatGPT is to AI what Google is to internet”
- All major models (ChatGPT, Claude, Gemini) are LLMs at their core
- Biases and errors stem from training patterns, not malice

### Tokens: The Currency of AI

- AI processes input and output in tokens, not words
- “Unbelievable things happen” = 5 tokens (OpenAI tokenizer demo)
- Tokenization algorithms (GPT-4o, o1, etc.) are pre-built; no need to implement
- Both input and output tokens are charged
  - Output always costs more: model must “think” to generate
  - GPT-5.5: $5/M input, $30/M output (6x ratio)
- Tokens are the unit of context, cost, and limits across all AI providers

### Embeddings and Vectorization

- Vector embeddings predate AI; a core NLP concept
- Process: word/text → numerical representation → position in multi-dimensional space
- Words with similar meaning cluster near each other in that space
- Model quality determines accuracy of clustering
  - Bad model: assigns arbitrary numbers
  - Good model: places “football”, “golf”, “tennis” near each other
- Used in RAG pipelines and model training; not tied to LLMs directly

### Context Windows

- Everything the model can “see” at once: your messages, its replies, any documents
- Hard limit per model; oldest content drops off when limit is hit
- LLM calls are stateless: the model remembers nothing between calls
  - All previous messages are re-sent with every new call
  - Longer chats = more tokens sent = slower, more expensive, more forgetting
- Live demo on Claude: context grew from ~4k to 46k tokens as chat progressed
- Context window is per chat session, not per account session
- Model limits:
  - GPT-5.5: 1M token context window
  - GPT-5.4 mini: 400k tokens
- Practical tip: start a new chat to reset context and avoid degraded responses

### Parameters

- Internal values tuned during training; the model’s actual “settings”
- No single parameter means anything; capability emerges from billions combined
- GPT-3: ~175B parameters; GPT-4: ~1T (estimated)
- More parameters = greater learning capacity
- Analogous to neurons in the brain; values are fixed post-training until a new version
- Weights, parameters, values: same concept for this level of discussion

### Web vs. API

- Web (e.g., chat.openai.com): controlled buffet
  - Fixed subscription price; token limits enforced per session
  - Claude Pro ($20/month) allows ~3M tokens per session
  - Companies subsidize web to drive adoption
- API: à la carte
  - Charged per input and output token
  - Required for building applications and agents
  - OpenRouter aggregates models from OpenAI, Anthropic, Google, etc. in one place
- Cannot use the ChatGPT web interface inside an application; API is the only path

### AI Agents: Anatomy

- Agent = Brain + Memory + Tools
- Brain: any LLM (OpenAI, Claude, Gemini, open-source)
  - Stateless by itself; only the brain can make decisions
- Memory: stores chat history (SQL, Mem0, etc.)
  - Simple memory: retrieves previous messages and injects them into each call
  - Long-term memory: persists across sessions (e.g., remembering a user’s name)
- Tools: capabilities given to the agent (web search, calculator, calendar, etc.)
  - Each tool has a description; the brain reads descriptions to decide which to use
  - Without tools, an agent cannot act on the world
- Agentic flow for a multi-step query:
  1. Retrieve memory (previous messages)
  2. Call brain with full context + tool descriptions
  3. Brain decides which tool(s) to invoke
  4. Tool(s) execute and return results
  5. Brain summarizes results
  6. Save new messages back to memory
- Brain may be called multiple times per user message (once to plan, once to summarize)
- Frameworks (LangChain, LangGraph, CrewAI) are just wrappers around this same pattern

### System Prompts and API Mechanics

- Every model has a system prompt defining its behavior (e.g., “be a helpful assistant”)
- System prompt is sent with every single API call, even for a one-token “hi”
  - Explains why a single “hi” can consume 4,000+ tokens
- Default API message structure: system → user → assistant → user → …
- All prior messages in a conversation are re-sent on each new call
  - Demonstrated live: sending “who am I?” required sending 3 messages total
- Usage tracked per API key; visible in OpenAI/OpenRouter dashboards
  - Live demo: 2 messages = 45 input tokens, 82 output tokens