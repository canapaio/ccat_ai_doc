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
