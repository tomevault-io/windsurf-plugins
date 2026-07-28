---
trigger: always_on
description: class AgentRepresentation()
---



# src.uagents.agent

Agent



## AgentRepresentation Objects

```python
class AgentRepresentation()
```

Represents an agent in the context of a message.

**Attributes**:

- `_address` _str_ - The address of the agent.
- `_name` _str | None_ - The name of the agent.
- `_identity` _Identity_ - The identity of the agent.
  
  Properties:
- `name` _str_ - The name of the agent.
- `address` _str_ - The address of the agent.
- `identifier` _str_ - The agent's address and network prefix.
- `identity` _Identity_ - The identity of the agent.



#### __init__
```python
def __init__(address: str, name: str | None, identity: Identity,
             prefix: AddressPrefix) -> None
```

Initialize the AgentRepresentation instance.

**Arguments**:

- `address` _str_ - The address of the context.
- `name` _str | None_ - The optional name associated with the context.
- `identity` _Identity_ - The identity of the agent.
- `prefix` _AddressPrefix_ - The address prefix for the agent's network.



#### name
```python
@property
def name() -> str
```

Get the name associated with the context or a truncated address if name is None.

**Returns**:

- `str` - The name or truncated address.



#### address
```python
@property
def address() -> str
```

Get the address of the context.

**Returns**:

- `str` - The address of the context.



#### identifier
```python
@property
def identifier() -> str
```

Get the address of the agent used for communication including the network prefix.

**Returns**:

- `str` - The agent's address and network prefix.



#### identity
```python
@property
def identity() -> Identity
```

Get the identity of the agent.

**Returns**:

- `Identity` - The identity of the agent.



## Agent Objects

```python
class Agent(Sink)
```

An agent that interacts within a communication environment.

**Attributes**:

- `_name` _str_ - The name of the agent.
- `_port` _int_ - The port on which the agent's server runs.
- `_background_tasks` _set[asyncio.Task]_ - Set of background tasks associated with the agent.
- `_resolver` _Resolver_ - The resolver for agent communication.
- `_loop` _asyncio.AbstractEventLoop_ - The asyncio event loop used by the agent.
- `_logger` - The logger instance for logging agent activities.
- `_endpoints` _list[AgentEndpoint]_ - List of endpoin[↗](https://github.com/fetchai/uAgents/blob/main/python/src/uagents/agent.py#L38)
ts at which the agent is reachable.
- `_use_mailbox` _bool_ - Indicates if the agent uses a mailbox for communication.
- `_agentverse` _AgentverseConfig_ - Agentverse configuration settings.
- `_mailbox_client` _MailboxClient_ - The client for interacting with the agentverse mailbox.
- `_ledger` - The client for interacting with the blockchain ledger.
- `_almanac_contract` - The almanac contract for registering agent addresses to endpoints.
- `_storage` - Key-value store for agent data storage.
- `_interval_handlers` _list[tuple[IntervalCallback, float]]_ - List of interval
  handlers and their periods.
- `_interval_messages` _set[str]_ - Set of message digests that may be sent by interval tasks.
- `_signed_message_handlers` _dict[str, MessageCallback]_ - Handlers for signed messages.
- `_unsigned_message_handlers` _dict[str, MessageCallback]_ - Handlers for
  unsigned messages.
- `_message_history` _EnvelopeHistory_ - History of messages received by the agent.
- `_models` _dict[str, type[Model]]_ - Dictionary mapping supported message digests to messages.
- `_replies` _dict[str, dict[str, type[Model]]]_ - Dictionary of allowed replies for each type
  of incoming message.
- `_queries` _dict[str, asyncio.Future]_ - Dictionary mapping query senders to their response
  Futures.
- `_dispatcher` - The dispatcher for internal handling/sorting of messages.
- `_dispenser` - The dispatcher for external message handling.
- `_message_queue` - Asynchronous queue for incoming messages.
- `_message_tasks` - A set for storing message handler tasks
  to prevent the GC from deleting them.
- `_handle_messages_concurrently` _bool_ - Whether to handle incoming messages concurrently.
- `_on_startup` _list[Callable]_ - List of functions to run on agent startup.
- `_on_shutdown` _list[Callable]_ - List of functions to run on agent shutdown.
- `_version` _str_ - The version of the agent.
- `_protocol` _Protocol_ - The internal agent protocol consisting of all interval and message
  handlers assigned with agent decorators.
- `protocols` _dict[str, Protocol]_ - Dictionary mapping all supported protocol digests to their
  corresponding protocols.
- `_ctx` _Context_ - The context for agent interactions.
- `_network` _str_ - The network to use for the agent ('mainnet' or 'testnet').
- `_prefix` _str_ - The address prefix for the agent (determined by the network).
- `_enable_agent_inspector` _bool_ - Enable the agent inspector REST endpoints.
- `_metadata` _dict[str, Any]_ - Metadata associated with the agent.
- `_readme` _str | None_ - The agent's README file.
- `_avatar_url` _str | None_ - The URL for the agent's avatar image on Agentverse.
- `_banner_url` _str | None_ - The URL for the agent's profile banner image on Agentverse.
  
  Properties:
- `name` _str_ - The name of the agent.
- `address` _str_ - The address of the agent used for communication.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fetchai/uAgents](https://github.com/fetchai/uAgents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
