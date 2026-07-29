# Class 10: Langchain tools — function schemas and tool binding

## Why Tools Exist: The “Brain Without Hands” Problem

- LLM-based bots (e.g., the CineBot movie-booking assistant) can produce structured output but cannot act on the world
- Without tools, the model either hallucinates answers (inventing show times) or admits it has no access
- Tools give the agent “hands”: the ability to fetch real-time data, query databases, execute code, or call APIs
- Core mental model: tools are glorified API calls or functions; if you can’t write a proper function, you can’t write a good tool

## Defining a Tool with @tool

- Decorate any Python function with @tool from langchain_core.tools
- The decorator wraps the function into a tool-compatible object:
  - Function name → tool name
  - Docstring → tool description (sent to the AI; make it informative and concise)
  - Type-hinted arguments → input schema
  - Return type → output type
- Name and description can be overridden inside @tool(name=..., description=...):
  - Useful when the function name is already used elsewhere (polymorphism)
  - Useful when wrapping third-party functions you cannot rename
- Without @tool, a plain function has no .name, .description, or .args attributes; the decorator adds all of these

## Argument Schema with Pydantic (args_schema)

- Pass a Pydantic BaseModel as args_schema to give the tool rich input validation
- Benefits over bare type hints:
  - Field-level descriptions guide the model on expected values
  - Constraints (e.g., Literal["front", "middle", "back"] for preferred row) are enforced
  - Reduces model errors: sending 20–30 extra tokens for a precise schema beats multiple failed calls
- Inspect a tool’s full schema via tool.args; without args_schema the attribute is absent

## Reserved Parameter Names: config and runtime

- config and runtime are reserved by LangChain and **cannot** be used as tool parameter names
- The function definition will not error at decoration time; the error surfaces only at runtime when the tool is called
- Key takeaway: rename any parameter that clashes with these keywords before deploying

## Binding vs. Execution (model.bind_tools ≠ calling the tool)

- model.bind_tools([tool_a, tool_b]) makes the model aware of available tools
- When invoked, the model returns an AIMessage with tool_calls populated but **content empty**: it is a request, not an execution
- The tool is never called by the model itself; only an agent (or explicit code) executes it
- Flow summary:
  1. Query → agent
  2. Agent → LLM (decides which tool + args)
  3. LLM returns tool call request
  4. Agent executes the tool
  5. Tool result → LLM again (summarizes/formats)
  6. Final answer → user
- LLM is used **at least twice**: once to select the tool, once to process its output

## Inbuilt and Third-Party Tools

- LangChain provides prebuilt tools (e.g. TavilySearch)
  - Install via pip, then import and use directly
  - Tavily: 1,000 free calls, then charged per API key
- Prebuilt tools are still just classes/functions under the hood
- Can wrap inbuilt tools with @tool to override name or description
- Server-side tools (e.g. ChatGPT web search) run on the provider’s infrastructure, not locally

### return_direct: Skipping the Final LLM Step

- Setting return_direct=True on a tool causes the tool’s raw output to be returned directly, bypassing step 6
- Use case: legal text, refund policies, or any content that must not be paraphrased or hallucinated
  - Real-world cautionary example: Air Canada chatbot hallucinated a bereavement refund policy; a judge ruled the airline liable
- When return_direct=True, the tool message is the final response; no further LLM call is made
- Trade-off: cannot chain further tool calls after a return_direct tool in the same turn

## Tool Runtime: Context-Aware Tools

- Tools can accept a special runtime: RunnableConfig argument (invisible to the model)
- The model only sees declared parameters; runtime opens a hidden layer of context:
  - runtime.state — current conversation state and short-term memory
  - runtime.context — immutable config set at invocation (e.g., user tier: free vs. paid)
  - runtime.store — long-term persistent memory (survives across conversations)
  - runtime.stream — emit live progress updates during execution
- Mirror analogy: the model sees only its own reflection (declared args); runtime is the world behind the mirror
- Demo built: save_favorite_genre and recall_favorite_genre tools that read/write to an InMemoryStore via runtime.store
  - Agent correctly called save_favorite_genre when a user mentioned liking sci-fi, storing the preference
  - Preference retrievable in subsequent turns without re-stating it

## Dynamic Tool Loading and What’s Ahead

- Problem 1: loading all tools upfront is expensive and exposes capabilities to wrong users
  - Example: a free user should never have access to book_vip_lounge; the LLM could call it by mistake
- Problem 2: tools may need to be loaded mid-session (e.g., connecting to an MCP server at runtime)
- Problem 3: even a smart LLM cannot be 100% trusted to always pick the right tool
- Solution direction: **middleware** (covered next class) — intercepts and controls tool calls before execution
- MCP (Model Context Protocol) framed as a collection/registry of tools; more detail deferred
- Broader learning arc: Models → Messages → Structured Output → Tools → **Agents** (next module, where everything converges)

### Next Steps

- **Complete and upload the assignment**
  - Mayank will finalize and post the practical assignment by evening (27th July).
- **Revise today's session before the next class**
  - Run all code cells end-to-end; agents module builds directly on tools concepts covered today.