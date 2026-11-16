# 1. Project Goal

Build a support ticket routing system for TaskNest using LangGraph. An OpenAI language model must classify each customer ticket into billing, technical, or general. The graph must then forward the ticket to the appropriate team and generate a fixed, short customer reply.

# 2. State Definition

Use a `TypedDict` with exactly these four keys:

- `ticket: str` — original customer message
- `category: str` — LLM classification
- `team: str` — selected support team
- `reply: str` — fixed response to the customer

# 3. Graph Nodes

Define exactly four nodes:

1. `classify`:
   - Read `state["ticket"]`.
   - Call the OpenAI LLM exactly once.
   - Classify the ticket as `"billing"`, `"technical"`, or `"general"`.
   - Return only `{"category": selected_category}`.

2. `billing_team`:
   - Return `team = "Billing"`.
   - Return `reply = "Your ticket has been forwarded to our billing team."`.

3. `tech_team`:
   - Return `team = "Technical"`.
   - Return `reply = "Your ticket has been forwarded to our technical team."`.

4. `general_team`:
   - Return `team = "General"`.
   - Return `reply = "Your ticket has been forwarded to our general support team."`.

The three team nodes must use plain Python and must not call the LLM.

# 4. Conditional Routing

- The graph starts at the `classify` node.
- Define a routing function named `route(state)`.
- The route function must only read `state["category"]` and return its value.
- The route function must not call the LLM.
- Use exactly one `add_conditional_edges` call after `classify`.
- Map categories as follows:
  - `"billing"` -> `"billing_team"`
  - `"technical"` -> `"tech_team"`
  - `"general"` -> `"general_team"`
- Each team node must connect directly to `END`.
- Do not add loops or additional nodes.

# 5. LLM Configuration

- Use `ChatOpenAI` from `langchain_openai`.
- Use a low-cost OpenAI text model. Start with `gpt-6-luna`, as suggested in the homework, and verify that it is available before running.
- Selected model: `gpt-6-luna`, matching `MODEL_NAME` in the notebook. It was chosen as the homework's suggested low-cost option for a simple three-category classification task, to limit the cost of testing without using a larger model. No specific prices or verified pricing comparison are claimed here.
- Store the model name in one variable so it can be changed easily.
- Set `temperature=0` for consistent classification.
- Use a Pydantic `BaseModel` with a `category` field typed as `Literal["billing", "technical", "general"]`.
- Use `with_structured_output(...)` to enforce the expected classification format.
- The `classify` node must call the LLM exactly once per ticket.
- The classification instruction should consider the main problem described in the ticket, not just individual keywords.
- Billing covers payments, invoices, refunds, and subscriptions.
- Technical covers bugs, crashes, error messages, and login problems.
- General covers feature suggestions, general questions, and other messages.
- For mixed messages, choose the team responsible for solving the main problem.
- Return only the selected category into the graph state.

# 6. API Key Handling

- Store the OpenAI API key in Google Colab Secrets under the exact name `OPENAI_API_KEY`.
- Enable Notebook access for this secret.
- Load the key using:

  ```python
  import os
  from google.colab import userdata
  os.environ["OPENAI_API_KEY"] = userdata.get("OPENAI_API_KEY")
  ```

- `ChatOpenAI` should use the key from the environment variable.
- Never hardcode the API key.
- Never print or expose the key.
- Do not use `getpass`.
- Do not include any real key in the notebook, `prompt.md`, or GitHub repository.
- Include clear setup instructions for Google Colab Secrets in the notebook.

# 7. Testing Requirements

1. Create a Google Colab notebook using LangGraph.
2. Include an installation cell:

   ```python
   !pip -q install langgraph langchain-openai
   ```

3. Display the graph using `draw_mermaid_png()`.
4. Run at least 7 test tickets:
   - "I was charged twice this month."
   - "I want a refund for my subscription."
   - "I cannot log in to my account."
   - "The app keeps crashing."
   - "Can you add a dark mode?"
   - "Thank you for the great app!"
   - "The app crashes every time I try to pay."
5. For every test, print the original ticket, category, team, and reply.
6. Verify that the result contains exactly the four required state keys.
7. Use exactly one LLM call per ticket.
8. Include no loops or memory in the graph.
9. Make sure the notebook runs from top to bottom in a fresh Google Colab session after setting `OPENAI_API_KEY` in Colab Secrets.
10. Include clear instructions for running and testing the notebook.

# Prompt Revision History

- Started with the seven section headings, then added the TaskNest goal and the exact four-key state definition.
- Specified the four nodes, fixed team replies, and the single conditional routing call with three branches ending at `END`.
- Added structured OpenAI classification, a configurable model name, temperature zero, and rules based on the main problem rather than isolated keywords.
- Added Colab Secrets instructions and restrictions against exposing keys or using `getpass`.
- Added Colab installation, graph visualization, seven test tickets including the mixed payment-crash case, and checks for the four state keys and one LLM call per ticket.
- After reviewing the generated notebook, documented its actual `gpt-6-luna` model choice and cost rationale and added this revision history. These entries reflect the development steps recorded in this chat and the current project files; they do not claim a Git commit history or successful live execution.

# Reflection

All seven tickets were classified as I expected. The most interesting example was "The app crashes every time I try to pay," which the LLM correctly routed to the Technical team rather than Billing. Compared with fixed rules in Dana's Bakery, an LLM can understand the meaning and context of a message instead of relying only on keywords. However, using an LLM introduces API costs, requires more testing, and may produce less predictable results.
