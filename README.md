## Cheshire Cat AI (ccat) Framework and the `cat` Object

The Cheshire Cat AI (ccat) is a framework designed to create and manage AI agents. Plugins are a core component, allowing developers to extend the cat's capabilities. The `cat` object is central to plugin development, as it provides access to the AI's internal state, tools, and memory.

### Key Components and Concepts:

* **Hooks:** These are specific points in the AI's execution pipeline where a plugin can intervene. They allow you to modify data, trigger actions, or interact with the AI's internal processes.
* **Tools:** These are functions that the AI can decide to use to perform specific actions, often involving external APIs or data sources.
* **Rabbit Hole:** This is the component responsible for ingesting and processing documents (text, web pages, etc.) into the AI's memory. It handles chunking, embedding, and storing information.
* **Memory:** The ccat uses different types of memory:
    * **Working Memory:** Holds short-term information relevant to the current conversation.
    * **Declarative Memory (Vector Memory):** Stores factual information and knowledge learned from documents or conversations, typically using vector embeddings for semantic search.
    * **Procedural Memory:** Stores skills and procedures (though less explicitly detailed in the initial links for plugin interaction in the same way as declarative/working memory).
* **`cat` object:** This object is passed to plugin hooks and tools. It acts as an interface to the AI's core functionalities.

### Simplified `cat` Object Structure (Conceptual)

Based on the documentation, the `cat` object likely provides access to methods and attributes related to:

* **`cat.llm`**: Interacting with the Large Language Model (e.g., `cat.llm.summmarize_text()`, `cat.llm.chat_completion()`).
* **`cat.memory`**: Accessing and manipulating the AI's memory.
    * `cat.memory.vectors`: Interacting with the vector memory (declarative memory).
        * `cat.memory.vectors.get_declarative_memory_instance()`
        * `cat.memory.vectors.get_procedural_memory_instance()`
        * `cat.memory.vectors.get_working_memory_instance()`
        * Methods to add, search, or delete memories.
    * `cat.memory.long_term`: Potentially an alias or specific methods for long-term storage.
* **`cat.working_memory`**: Direct access or methods for the current conversation's context.
* **`cat.rabbit_hole`**: Interacting with the document ingestion pipeline (e.g., `cat.rabbit_hole.store_file()`, `cat.rabbit_hole.store_url()`).
* **`cat.mad_hatter`**: Access to settings and configurations.
* **`cat.wolf_manager`**: Manages underlying services and infrastructure.
* **`cat.plugins`**: Accessing other plugins or plugin-specific functionalities.
* **`cat.send_ws_message()`**: Sending messages over WebSocket, often for real-time updates or notifications.
* **`cat.get_tool_config(tool_name)`**: Retrieving the configuration for a specific tool.
* **`cat.get_plugin_config(plugin_id)`**: Retrieving the configuration for the current or another plugin.

It's important to note that the exact attributes and methods available on the `cat` object might vary slightly depending on the context (i.e., which hook is being executed) and the ccat version. The documentation provides the most accurate details for specific methods.

---

Now, let's outline the document for the IA and junior user.

## Cheshire Cat AI Plugin Development: Functions and the `cat` Object Lifecycle

**Audience:** AI Language Model, Junior Developer
**Goal:** Understand ccat plugin functions, see examples of their use, and track how the `cat` object is populated and utilized throughout different stages of the AI's operation.

---

### 1. Introduction to ccat Plugins

Cheshire Cat AI (ccat) plugins extend the core functionalities of the AI. They allow you to customize how the AI processes information, interacts with users, and learns new things. At the heart of plugin development is the `cat` object, which provides access to the AI's brain and tools.

---

### 2. The `cat` Object: Your Interface to the AI

The `cat` object is an instance that gets passed to your plugin's functions (hooks and tools). Think of it as your remote control and information panel for the AI.

**Key things you can do with `cat`:**

* **Access LLM:** Use the Large Language Model for tasks like summarization, translation, or generating text.
    * *Example:* `cat.llm.chat_completion(prompt="What is the capital of France?")`
* **Manage Memory:** Read from and write to the AI's working (short-term) and declarative (long-term vector) memory.
    * *Example (Adding to declarative memory):* `cat.memory.vectors.get_declarative_memory_instance().add_point(text="Paris is the capital of France.", metadata={"source": "plugin_info"})`
    * *Example (Recalling from declarative memory):* `cat.memory.vectors.get_declarative_memory_instance().recall_memories_from_text(text="capital of France")`
* **Ingest Information (Rabbit Hole):** Tell the AI to learn from files, web pages, or text.
    * *Example:* `cat.rabbit_hole.store_url(url="https://en.wikipedia.org/wiki/France")`
* **Use Tools:** Access built-in or other plugin-defined tools.
* **Send Messages:** Communicate with the user or other systems.
    * *Example:* `cat.send_ws_message(content="I've just learned about France from Wikipedia!", msg_type="notification")`
* **Get Configuration:** Access settings for your plugin or specific tools.
    * *Example:* `my_api_key = cat.get_plugin_config(plugin_id="my_awesome_plugin")["api_key"]`

---

### 3. Core Plugin Functions: Hooks

Hooks are entry points where your plugin can tap into the AI's lifecycle. The `cat` object is your primary tool within these hooks.

Let's look at common hooks and what `cat` might look like or be used for at each stage:

#### A. `agent_init(cat)`

* **When it runs:** Once, when the ccat agent starts up and loads the plugin.
* **Purpose:** Initialize your plugin, set up connections, load configurations, or perform one-time setup tasks.
* **`cat` at this stage:**
    * `cat` is available with access to core functionalities like `cat.mad_hatter` for settings or `cat.log` for logging.
    * Memory might be empty or contain data from previous sessions.
* **Example:**
    ```python
    # In your plugin's main.py
    def agent_init(cat):
        cat.log("My Awesome Plugin is initializing!")
        # Potentially load API keys from plugin config
        config = cat.get_plugin_config(plugin_id="my_awesome_plugin")
        api_key = config.get("my_api_key")
        if api_key:
            cat.log("API Key loaded.")
        # You could also pre-load some general information into memory if needed
        # cat.memory.vectors.get_declarative_memory_instance().add_point(text="General knowledge point for my plugin.", metadata={"source": "plugin_init"})
    ```
    * **`cat` population change:** No direct change to `cat`'s *structure*, but you *use* `cat` to access configurations or potentially populate memory.

#### B. `after_cat_bootstrap(cat)`

* **When it runs:** After the cat core systems (LLM, memory, etc.) are fully initialized and ready.
* **Purpose:** Perform setup that depends on the cat's core being fully operational. For example, registering dynamic tools or interacting with the LLM for initial setup.
* **`cat` at this stage:**
    * All core components of `cat` (LLM, memory, rabbit\_hole) are fully functional.
* **Example:**
    ```python
    # In your plugin's main.py
    def after_cat_bootstrap(cat):
        cat.log("Cat is fully bootstrapped. My plugin can now use the LLM for setup.")
        summary = cat.llm.summarize_text("This plugin enhances user interaction by providing real-time stock quotes.")
        cat.memory.vectors.get_declarative_memory_instance().add_point(
            text=f"Plugin purpose: {summary}",
            metadata={"source": "plugin_bootstrap_summary"}
        )
    ```
    * **`cat` population change:** Again, no change to `cat`'s structure, but you *use* its fully initialized components. Declarative memory is populated with a summary.

#### C. `before_llm_execution(user_message, cat, **kwargs)`

* **When it runs:** Just before the user's message is sent to the LLM.
* **Purpose:** Modify the user's message, add context, or inject information.
* **`cat` at this stage:**
    * Contains the current state of working memory and access to declarative memory.
    * `user_message` (string) is the direct input.
* **Example:**
    ```python
    # In your plugin's main.py
    def before_llm_execution(user_message, cat, **kwargs):
        # Add context from a specific memory point
        # relevant_info = cat.memory.vectors.get_declarative_memory_instance().recall_memories_from_text("user preferences")
        # if relevant_info:
        #     user_message = f"{user_message} (User preference: {relevant_info[0]['text']})"

        # Modify the user message
        modified_message = user_message + " - Please answer in a friendly tone."
        cat.log(f"Original message: {user_message}, Modified: {modified_message}")
        return {"user_message": modified_message} # Return the modified message
    ```
    * **`cat` population change:** You might use `cat.memory` to retrieve context. The primary change is to the `user_message` that will be sent to the LLM.

#### D. `after_llm_execution(message, cat, **kwargs)`

* **When it runs:** After the LLM has processed the input and generated a response.
* **Purpose:** Modify the LLM's response, extract information, or trigger actions based on the response.
* **`cat` at this stage:**
    * `message` (usually a dictionary) contains the LLM's raw response.
    * `cat` allows you to save this response or related extracted info to memory.
* **Example:**
    ```python
    # In your plugin's main.py
    def after_llm_execution(message, cat, **kwargs):
        llm_response_text = message["content"]
        cat.log(f"LLM responded: {llm_response_text}")

        # If LLM mentions a specific keyword, store it
        if "Cheshire Cat" in llm_response_text:
            cat.memory.vectors.get_declarative_memory_instance().add_point(
                text=f"LLM mentioned 'Cheshire Cat' in response: {llm_response_text}",
                metadata={"source": "llm_response_trigger"}
            )

        # Modify the response before sending to user
        modified_response = llm_response_text + " - Meow! 🐾"
        message["content"] = modified_response
        return message # Return the modified message object
    ```
    * **`cat` population change:** You might populate `cat.memory` with insights from the LLM's response. The `message` content is modified.

#### E. `before_cat_sends_message(message, cat, **kwargs)`

* **When it runs:** Just before the final message is sent to the user.
* **Purpose:** Last chance to modify the message, add attachments, or log the final output.
* **`cat` at this stage:**
    * `message` (usually a dictionary) is the final response object to be sent.
* **Example:**
    ```python
    # In your plugin's main.py
    def before_cat_sends_message(message, cat, **kwargs):
        final_text = message["content"]
        cat.log(f"Final message to user: {final_text}")
        # Ensure a friendly closing
        if not final_text.endswith("🐾"):
            final_text += " Have a purrfect day! 🐾"
        message["content"] = final_text
        return message
    ```
    * **`cat` population change:** The `message` content can be altered.

#### F. `after_cat_reads_message(message, cat, **kwargs)`

* **When it runs:** After the user's message is received by the ccat but before any significant processing (like `before_llm_execution`).
* **Purpose:** Early interception of user input, perhaps for logging, sanitization, or routing before it hits the main LLM chain.
* **`cat` at this stage:**
    * `message` is the raw incoming message from the user.
* **Example:**
    ```python
    # In your plugin's main.py
    def after_cat_reads_message(message, cat, **kwargs):
        user_input = message["content"]
        cat.log(f"User said: {user_input}")
        # Example: If user says "help", inject a known good help query to memory for context
        if "help" in user_input.lower():
            cat.working_memory.append({"role": "system", "content": "User is asking for help. Provide general assistance."})
        return message # Must return the message
    ```
    * **`cat` population change:** `cat.working_memory` might be updated with system messages or context based on the user's raw input.

#### G. `before_rabbit_hole_stores_file(file_bytes, cat, **kwargs)` / `before_rabbit_hole_stores_url(url, cat, **kwargs)` / `before_rabbit_hole_stores_text(text, cat, **kwargs)`

* **When it runs:** Before a file, URL, or text is processed and stored by the Rabbit Hole.
* **Purpose:** Modify the content before ingestion, prevent storage, or add metadata.
* **`cat` at this stage:**
    * Provides access to memory if you need to check something before storing.
    * The respective content (`file_bytes`, `url`, `text`) is available.
* **Example (`before_rabbit_hole_stores_url`):**
    ```python
    # In your plugin's main.py
    def before_rabbit_hole_stores_url(url, cat, **kwargs):
        cat.log(f"Rabbit Hole is about to store URL: {url}")
        if "internal-docs.mycompany.com" in url:
            # Allow storing
            # You could also modify the URL or add specific metadata instructions here
            # kwargs["metadata"] = {"source_type": "internal"}
            return {"url": url, "kwargs": kwargs} # Pass it on
        else:
            cat.log(f"Skipping non-internal URL: {url}")
            cat.send_ws_message(content=f"Sorry, I can only process URLs from internal-docs.mycompany.com", msg_type="error")
            return None # Returning None prevents storage
    ```
    * **`cat` population change:** No direct change to `cat`'s structure. You control *what* goes into `cat.memory` via the Rabbit Hole.

#### H. `after_rabbit_hole_digestion(docs, cat, **kwargs)`

* **When it runs:** After the Rabbit Hole has processed content (chunked, embedded) and stored it in declarative memory.
* **Purpose:** Access the processed documents (chunks), perform actions based on successful ingestion (e.g., notify user, trigger further analysis).
* **`cat` at this stage:**
    * `docs` is a list of document chunks that were just ingested.
    * `cat.memory` now contains these new chunks.
* **Example:**
    ```python
    # In your plugin's main.py
    def after_rabbit_hole_digestion(docs, cat, **kwargs):
        cat.log(f"Rabbit Hole finished digesting. {len(docs)} document chunks were added to memory.")
        # Example: Send a notification to the user
        source_name = "the provided content"
        if docs and docs[0].metadata and "source" in docs[0].metadata:
            source_name = docs[0].metadata["source"]

        cat.send_ws_message(content=f"I've finished learning from {source_name}! Ask me anything.", msg_type="notification")

        # You could also trigger another process, e.g., summarize the new docs
        # combined_text = " ".join([doc.page_content for doc in docs])
        # summary = cat.llm.summarize_text(combined_text)
        # cat.memory.vectors.get_declarative_memory_instance().add_point(
        # text=f"Summary of recently ingested content ({source_name}): {summary}",
        # metadata={"source": "post_digestion_summary"}
        # )
        return docs # Must return the docs
    ```
    * **`cat` population change:** `cat.memory` (specifically declarative memory) is now populated with the new information from the digested `docs`. You might further populate memory with summaries or analyses derived from these docs using `cat.llm`.

---

### 4. Core Plugin Functions: Tools

Tools are functions your plugin exposes that the AI can decide to use to accomplish a task. The LLM, when it deems appropriate, will "call" your tool.

#### `tool_example(tool_input, cat, **kwargs)`

* **When it runs:** When the LLM decides this tool is needed to answer a user query or perform an action.
* **Purpose:** Execute a specific action (e.g., fetch data from an API, perform a calculation, interact with a device).
* **`cat` at this stage:**
    * Fully available. You can use `cat.llm` for sub-tasks, `cat.memory` to store results or get context, `cat.send_ws_message` to give feedback.
    * `tool_input` is the string or structured data the LLM provides to your tool.
* **Defining a Tool:**
    You define tools in your plugin's `main.py` by decorating a function with `@tool`.
    ```python
    from cat.mad_hatter.decorators import tool

    @tool(
        name="GetStockPrice", # How the LLM refers to it
        description="Fetches the current stock price for a given company ticker symbol. Input should be the ticker symbol (e.g., AAPL).",
        # 'json_schema' can be used for more complex input structures
    )
    def get_stock_price_tool(ticker_symbol: str, cat):
        cat.log(f"Tool 'GetStockPrice' called with ticker: {ticker_symbol}")
        # In a real scenario, you'd call an API here
        # For example: price = some_stock_api.get_price(ticker_symbol)
        if ticker_symbol == "AAPL":
            price_info = f"The current price of AAPL is $170.34."
        elif ticker_symbol == "GOOGL":
            price_info = f"The current price of GOOGL is $155.72."
        else:
            price_info = f"Sorry, I don't have data for {ticker_symbol}."

        # Optionally, store this interaction in memory
        cat.memory.vectors.get_declarative_memory_instance().add_point(
            text=f"Stock price for {ticker_symbol} was queried. Result: {price_info}",
            metadata={"source": "GetStockPriceTool", "ticker": ticker_symbol}
        )
        # The tool should return a string that the LLM can use in its response to the user
        return price_info
    ```
* **`cat` population change:**
    * You might use `cat` to fetch credentials for an API call (via `cat.get_plugin_config`).
    * You can store the result of the tool's execution or metadata about its usage in `cat.memory`.
    * The tool's *output* (the return value) is sent back to the LLM, which then incorporates it into its response to the user. The `cat` object itself isn't directly "populated" by the tool in terms of its structure, but its *memories can be updated by the tool's actions*.

---

### 5. Timeline Example: What Happens to `cat`?

Let's trace a simple interaction:

1.  **Startup:**
    * `agent_init(cat)` runs: `cat` used to load plugin config.
    * `after_cat_bootstrap(cat)` runs: `cat.llm` used to summarize plugin purpose, result stored in `cat.memory.vectors`.
        * *`cat.memory` now contains initial bootstrap info.*

2.  **User uploads a document ("my_company_policy.pdf"):**
    * `before_rabbit_hole_stores_file(file_bytes, cat, **kwargs)` might run: Plugin could check file type or add metadata.
    * Rabbit Hole processes the PDF.
    * `after_rabbit_hole_digestion(docs, cat, **kwargs)` runs:
        * `docs` contains chunks from "my\_company\_policy.pdf".
        * `cat.send_ws_message` used to notify user of completion.
        * Optionally, `cat.llm` used to summarize the new docs and store summary in `cat.memory.vectors`.
        * *`cat.memory.vectors` is significantly populated with content from "my\_company\_policy.pdf" and potentially its summary.*

3.  **User asks: "What is the vacation policy according to my_company_policy.pdf?"**
    * `after_cat_reads_message(message, cat, **kwargs)` runs: Logs the input. `cat.working_memory` might be updated if specific keywords trigger actions.
    * `before_llm_execution(user_message, cat, **kwargs)` runs:
        * The original `user_message` is "What is the vacation policy according to my\_company\_policy.pdf?".
        * Plugin might add context from `cat.memory` if relevant (e.g., "Remember to check the most recent version of the policy.").
        * Returns modified `user_message` to LLM.
    * **LLM Processing:**
        * The LLM receives the (possibly modified) user query.
        * The ccat framework automatically performs a similarity search in `cat.memory.vectors` using the user's query. Relevant chunks from "my\_company\_policy.pdf" are retrieved and provided to the LLM as context.
        * LLM formulates an answer based on the query and the retrieved context.
    * `after_llm_execution(message, cat, **kwargs)` runs:
        * `message` contains the LLM's answer (e.g., "The vacation policy states...").
        * Plugin might log this answer or store a fact from it in `cat.memory.vectors`.
        * Returns (possibly modified) `message`.
    * `before_cat_sends_message(message, cat, **kwargs)` runs:
        * Final chance to tweak the `message` before it goes to the user.
        * Adds a friendly closing.
    * **User receives the answer.**

4.  **User asks: "What's the stock price for AAPL?"**
    * (Similar `after_cat_reads_message`, `before_llm_execution` flow)
    * **LLM Processing:**
        * LLM recognizes the need for real-time data not in its memory.
        * It identifies the `GetStockPrice` tool as suitable.
        * LLM decides to call `GetStockPrice` with input "AAPL".
    * **Tool Execution:** `get_stock_price_tool("AAPL", cat)` runs:
        * `cat.log` used.
        * Tool simulates API call.
        * `cat.memory.vectors.get_declarative_memory_instance().add_point(...)` stores info about this query.
        * Returns "The current price of AAPL is $170.34."
        * *`cat.memory.vectors` now contains a record of this stock query.*
    * **LLM Processing resumes:**
        * LLM receives "The current price of AAPL is $170.34." from the tool.
        * LLM incorporates this into a natural language response.
    * (Similar `after_llm_execution`, `before_cat_sends_message` flow)
    * **User receives the stock price.**

Throughout this timeline, the `cat` object itself doesn't fundamentally change its *structure* (the methods and attributes available, like `cat.llm`, `cat.memory`, remain consistent). However, the *data it manages*, particularly within `cat.memory` (both working and declarative) and the information passed through it (user messages, LLM responses), is constantly changing and being populated by the different stages and plugin functions. Your plugin uses the stable interface of the `cat` object to interact with this dynamic flow of information.

---

This document should provide a good starting point for both an AI and a junior developer to understand how to work with ccat plugins and the `cat` object!
