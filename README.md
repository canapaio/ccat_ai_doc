# CheshireCat Framework: Complete Guide to Intervention Points

The CheshireCat framework provides a sophisticated system of intervention points (hooks) that allow developers to customize and extend the AI agent's behavior at specific moments in its execution pipeline. This guide explains how to effectively use each intervention point with practical examples and best practices.

## Overview of Intervention Categories

The framework organizes intervention points into five main categories:
- **Flow**: Core execution pipeline hooks
- **Agent**: AI agent behavior and prompt management
- **Rabbit Hole**: Document processing and memory ingestion
- **Plugin**: Plugin lifecycle management
- **Factory**: Component configuration and settings

---

## Flow Intervention Points

Flow points control the main execution pipeline of the CheshireCat system.

### Before Cat Bootstrap
**Purpose**: Modify or prepare the system before the Cat initializes its core components.

**Use Cases**:
- Custom configuration loading
- Environment validation
- Pre-initialization logging
- Component dependency checks

**Example**:
```python
@hook
def before_cat_bootstrap(cat):
    # Validate required environment variables
    if not os.getenv('CUSTOM_API_KEY'):
        raise ValueError("CUSTOM_API_KEY not found")
    
    # Initialize custom logging
    setup_custom_logger()
    
    return cat
```

### After Cat Bootstrap
**Purpose**: Execute actions after all Cat components are initialized and ready.

**Use Cases**:
- Post-initialization validation
- Custom component registration
- System health checks
- Welcome message preparation

**Example**:
```python
@hook
def after_cat_bootstrap(cat):
    # Verify all components are working
    cat.memory.working_memory.update({
        "system_status": "initialized",
        "startup_time": datetime.now()
    })
    
    return cat
```

### Before Cat Reads Message
**Purpose**: Intercept and potentially modify incoming WebSocket messages.

**Use Cases**:
- Message validation and sanitization
- User authentication checks
- Rate limiting implementation
- Message preprocessing

**Example**:
```python
@hook
def before_cat_reads_message(user_message_json, cat):
    # Add user session info
    user_message_json["session_id"] = generate_session_id()
    
    # Sanitize message content
    user_message_json["text"] = sanitize_input(user_message_json["text"])
    
    return user_message_json
```

### Cat Recall Query
**Purpose**: Modify the query before it's embedded for memory search.

**Use Cases**:
- Query enhancement and expansion
- Language normalization
- Context addition
- Query preprocessing

**Example**:
```python
@hook
def cat_recall_query(user_message, cat):
    # Add context to improve recall
    enhanced_query = f"User context: {cat.working_memory.get('user_context', '')} Query: {user_message}"
    
    return enhanced_query
```

### Before Cat Recalls Memories
**Purpose**: Intervene before the system searches through all memory types.

**Use Cases**:
- Memory search strategy customization
- Performance optimization
- Access control implementation

**Example**:
```python
@hook
def before_cat_recalls_memories(query, cat):
    # Log memory access for audit
    log_memory_access(query, cat.user_id)
    
    # Modify search parameters based on user type
    if cat.working_memory.get("user_type") == "premium":
        cat.memory_config.max_results = 20
    
    return query
```

### Memory-Specific Recall Hooks

#### Before Cat Recalls Episodic Memories
**Purpose**: Control episodic memory (conversation history) retrieval.

**Use Cases**:
- Conversation history filtering
- Privacy controls
- Context window management

#### Before Cat Recalls Declarative Memories
**Purpose**: Control document memory retrieval.

**Use Cases**:
- Document access permissions
- Content filtering
- Knowledge base selection

#### Before Cat Recalls Procedural Memories
**Purpose**: Control tool/action memory retrieval.

**Use Cases**:
- Tool permissions management
- Action filtering based on context
- Security restrictions

### After Cat Recalls Memories
**Purpose**: Process and potentially modify recalled memories before use.

**Use Cases**:
- Memory content filtering
- Relevance scoring adjustment
- Content summarization

**Example**:
```python
@hook
def after_cat_recalls_memories(memory_results, cat):
    # Filter sensitive information
    filtered_results = filter_sensitive_content(memory_results)
    
    # Add relevance metadata
    for result in filtered_results:
        result["relevance_score"] = calculate_relevance(result, cat.working_memory)
    
    return filtered_results
```

### Before Cat Stores Episodic Memories
**Purpose**: Modify conversation data before storage.

**Use Cases**:
- Data sanitization
- Privacy filtering
- Metadata addition
- Storage optimization

### Before Cat Sends Message
**Purpose**: Final message processing before delivery.

**Use Cases**:
- Response filtering
- Format conversion
- Analytics tracking
- Rate limiting

---

## Agent Intervention Points

Agent hooks control the AI agent's behavior and prompt construction.

### Before Agent Starts
**Purpose**: Prepare agent input and configuration.

**Use Cases**:
- Dynamic prompt modification
- Context injection
- Agent configuration tuning

**Example**:
```python
@hook
def before_agent_starts(agent_input, cat):
    # Add current time context
    agent_input["current_time"] = datetime.now().isoformat()
    
    # Add user preferences
    agent_input["user_preferences"] = cat.working_memory.get("preferences", {})
    
    return agent_input
```

### Agent Fast Reply
**Purpose**: Provide immediate responses without full pipeline execution.

**Use Cases**:
- Cached response delivery
- Simple query handling
- Emergency responses
- Performance optimization

**Example**:
```python
@hook
def agent_fast_reply(fast_reply, cat):
    query = cat.working_memory.get("user_message", "")
    
    # Handle simple greetings
    if query.lower() in ["hi", "hello", "hey"]:
        return {
            "output": "Hello! How can I help you today?",
            "return_direct": True
        }
    
    # Check for cached responses
    cached_response = check_cache(query)
    if cached_response:
        return {
            "output": cached_response,
            "return_direct": True
        }
    
    return fast_reply
```

### Agent Prompt Components

#### Agent Prompt Prefix
**Purpose**: Customize the personality and system prompt.

**Use Cases**:
- Dynamic personality adjustment
- Context-aware behavior modification
- Role-based prompting

#### Agent Prompt Suffix
**Purpose**: Modify the prompt suffix with memories and conversation history.

**Use Cases**:
- Memory presentation formatting
- Context organization
- Instruction clarification

#### Agent Prompt Instructions
**Purpose**: Customize reasoning and instruction prompts.

**Use Cases**:
- Task-specific instruction modification
- Reasoning strategy adjustment
- Output format specification

### Agent Allowed Tools
**Purpose**: Control which tools are available to the agent.

**Use Cases**:
- Security restrictions
- Context-based tool filtering
- User permission management

**Example**:
```python
@hook
def agent_allowed_tools(tools, cat):
    user_role = cat.working_memory.get("user_role", "guest")
    
    # Filter tools based on user role
    if user_role == "admin":
        return tools  # Admin gets all tools
    elif user_role == "user":
        # Regular users get limited tools
        allowed_tool_names = ["search", "calculator", "weather"]
        return [tool for tool in tools if tool.name in allowed_tool_names]
    else:
        # Guests get minimal tools
        return [tool for tool in tools if tool.name == "search"]
```

---

## Rabbit Hole Intervention Points

Rabbit Hole hooks control document processing and memory ingestion.

### Before Rabbit Hole Insert Memory
**Purpose**: Modify documents before insertion into declarative memory.

**Use Cases**:
- Content validation
- Metadata addition
- Access control setup

### Text Processing Hooks

#### Before Rabbit Hole Splits Text
**Purpose**: Modify text before chunking.

**Use Cases**:
- Text preprocessing
- Format normalization
- Content enhancement

#### After Rabbit Hole Splitted Text
**Purpose**: Process text chunks after splitting.

**Use Cases**:
- Chunk validation
- Metadata addition
- Quality filtering

**Example**:
```python
@hook
def after_rabbit_hole_splitted_text(chunks, cat):
    processed_chunks = []
    
    for chunk in chunks:
        # Add metadata to each chunk
        chunk.metadata.update({
            "processed_at": datetime.now().isoformat(),
            "chunk_quality": assess_chunk_quality(chunk.text),
            "language": detect_language(chunk.text)
        })
        
        # Filter out low-quality chunks
        if chunk.metadata["chunk_quality"] > 0.5:
            processed_chunks.append(chunk)
    
    return processed_chunks
```

### Document Storage Hooks

#### Before Rabbit Hole Stores Documents
**Purpose**: Intervene before starting the document ingestion pipeline.

#### After Rabbit Hole Stores Documents
**Purpose**: Execute actions after document ingestion completion.

### Component Instantiation Hooks

#### Rabbit Hole Instantiates Parsers
**Purpose**: Hook available parsers for file ingestion.

**Use Cases**:
- Custom parser registration
- Parser configuration
- Format-specific handling

#### Rabbit Hole Instantiates Splitter
**Purpose**: Hook the text splitter component.

**Use Cases**:
- Custom splitting strategies
- Splitter configuration
- Content-aware chunking

---

## Plugin Intervention Points

Plugin hooks manage the plugin lifecycle and configuration.

### Plugin Lifecycle Hooks

#### Activated
**Purpose**: Execute actions when a plugin is enabled.

**Use Cases**:
- Plugin initialization
- Resource allocation
- Configuration validation

#### Deactivated
**Purpose**: Execute actions when a plugin is disabled.

**Use Cases**:
- Cleanup operations
- Resource deallocation
- State preservation

### Plugin Settings Hooks

#### Settings Schema
**Purpose**: Override plugin settings schema retrieval.

#### Settings Model
**Purpose**: Override plugin settings model retrieval.

#### Load Settings
**Purpose**: Customize how plugin settings are loaded.

#### Save Settings
**Purpose**: Customize how plugin settings are saved.

**Example**:
```python
@hook
def plugin_save_settings(settings, plugin_id, cat):
    # Add validation before saving
    validated_settings = validate_plugin_settings(settings, plugin_id)
    
    # Add audit trail
    audit_log = {
        "timestamp": datetime.now().isoformat(),
        "user_id": cat.user_id,
        "action": "settings_updated",
        "plugin_id": plugin_id
    }
    
    # Save to custom location
    save_to_custom_storage(validated_settings, audit_log)
    
    return validated_settings
```

---

## Factory Intervention Points

Factory hooks control component configuration and settings retrieval.

### Factory Allowed LLMs
**Purpose**: Intervene before retrieving LLM settings.

**Use Cases**:
- LLM selection based on context
- Cost optimization
- Performance tuning

### Factory Allowed Embedders
**Purpose**: Intervene before retrieving embedder settings.

**Use Cases**:
- Embedder selection optimization
- Multi-language support
- Performance considerations

### Factory Allowed AuthHandlers
**Purpose**: Intervene before retrieving authentication handler settings.

**Use Cases**:
- Dynamic authentication strategy
- Security policy enforcement
- Multi-tenant support

**Example**:
```python
@hook
def factory_allowed_llms(llm_settings, cat):
    user_tier = cat.working_memory.get("user_tier", "free")
    
    if user_tier == "premium":
        # Premium users get access to advanced models
        return llm_settings
    else:
        # Free users get limited model access
        filtered_settings = [
            setting for setting in llm_settings 
            if setting.get("tier", "free") == "free"
        ]
        return filtered_settings
```

---

## Best Practices and Tips

### 1. Hook Implementation Guidelines
- Always return the expected data type from hooks
- Handle exceptions gracefully to prevent system crashes
- Use logging for debugging and monitoring
- Keep hook functions lightweight and efficient

### 2. Performance Considerations
- Minimize processing time in frequently called hooks
- Use caching where appropriate
- Avoid blocking operations in critical path hooks
- Monitor hook execution times

### 3. Security Best Practices
- Validate all inputs in hooks
- Implement proper access controls
- Sanitize data before processing
- Log security-relevant events

### 4. Development Workflow
- Test hooks in isolation before integration
- Use version control for hook implementations
- Document hook behavior and dependencies
- Implement rollback strategies for critical hooks

### 5. Common Patterns
- **Middleware Pattern**: Use hooks to implement cross-cutting concerns
- **Chain of Responsibility**: Allow multiple hooks to process the same event
- **Observer Pattern**: Use hooks for event notification and logging
- **Decorator Pattern**: Enhance existing functionality without modification

---

## Conclusion

The CheshireCat framework's intervention points provide powerful customization capabilities for AI agent behavior. By understanding and effectively utilizing these hooks, developers can create sophisticated, context-aware, and highly customized AI applications.

Remember to always consider the impact of your interventions on system performance and stability. Start with simple implementations and gradually add complexity as needed. The modular nature of these hooks allows for incremental development and easy maintenance of your customizations.
