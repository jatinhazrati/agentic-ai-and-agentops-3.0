# Class 03: Python fundamentals — type validation, pydantic, and AI agents

### Why Type Validation Matters

- Python and JavaScript are loosely typed: no enforcement on variable types
- Assigning a string to an int-typed variable raises no error in Python
- Production systems need both field type validation and data validation
  - Bad data can break operations (e.g. 2026 - "age" throws TypeError)
  - Malicious inputs (e.g. age = 1000) pass silently without constraints
- TypeScript exists precisely because JavaScript needed static typing for production use

### Why Pydantic

- Used by Anthropic, OpenAI, NVIDIA, Google, Amazon, and most Python-heavy companies
- Downloaded millions of times per month via PyPI
- FastAPI uses it natively for request/response modeling and Swagger docs
- AI outputs (JSON responses) are mapped to Pydantic models to avoid runtime errors
- Even if AI writes your code, you need to understand Pydantic to read and fix it

### Plain Class vs. Data Class vs. Base Model

- Plain class: requires manual __init__; no type enforcement whatsoever
- Data class (@dataclass): removes boilerplate constructor; still no validation
- Pydantic BaseModel: inherits validation automatically
  - Declare fields with type hints; Pydantic enforces them on instantiation
  - Raises ValidationError immediately if types don’t match

### Core Pydantic Features

- Type coercion: Pydantic converts "28" → 28 (int), but not "2 8" or 28.5
- Optional fields: provide a default value (e.g. is_interested: bool = False)
  - Use None for unknown defaults; Python has no null
- Required fields: any field without a default must be supplied
- Serialization: model.model_dump() → dict; model.model_dump_json() → JSON string
- Inbuilt special types: EmailStr, HttpUrl, SecretStr, IPvAnyAddress
  - SecretStr masks sensitive values like API keys in output

### Field Constraints and Annotated Syntax

- Field() adds data-level constraints beyond type checking:
  - min_length, max_length for strings
  - gt, lt, ge, le for numbers (e.g. years_of_experience: int = Field(gt=0, lt=50))
- Annotated syntax is an alternative; identical behavior, more verbose
  - AI-generated code often uses Annotated; both are equivalent
- Use Field() when you need to constrain what values are valid, not just what type

### Validators: Field, Model, and Computed

- @field_validator("field_name"): runs custom logic on a single field
  - Example: block disposable email domains like ymail.com
  - Cannot access other fields inside the validator
- @model_validator: runs after all field validators; receives the full model via self
  - Use for cross-field logic (e.g. password == confirm_password)
  - Execution order is fixed: field validators always run before model validator
- @computed_field + @property: derives a value from existing fields
  - Not stored by the user; calculated on the fly (e.g. BMI from height/weight, experience tier from years)
  - Property method name must match the field name used in the model

### Nested Models

- A Pydantic model can use another model as a field type (e.g. address: Address)
- Validators defined on the nested model apply automatically when used inside a parent
- Maps directly to nested JSON structures; useful for any API input/output

### AI Foundations: Key Terms

- LLM (Large Language Model): next-token predictor trained on vast text data
  - Examples: ChatGPT, Claude, Gemini
  - Predicts the statistically most likely next token, not “understanding” meaning
- Token: roughly ¾ of a word; the unit LLMs process (input and output)
  - Token is the currency of AI: pricing is per million input/output tokens
  - Output tokens are always more expensive than input tokens
- Vector embedding: each word/token mapped to coordinates in high-dimensional space
  - Similar words sit closer together (e.g. “cat” and “dog” are near; “cat” and “happy” are far)
  - Dimensions are user-defined; more dimensions = finer resolution
- Context window: the total tokens a model can “see” at once (e.g. GPT-4o: 400k tokens)
  - Once full, oldest content is dropped silently
  - Memory is separate from context window (persistent; covered later)
- Parameters: billions of small numerical weights set during training
  - No single parameter means anything; capability emerges from their combined state
  - More parameters ≈ higher potential capacity, not guaranteed quality

## References

- [Pydantic, explained — instructor's interactive site](https://pydantic-with-mayank.netlify.app)
