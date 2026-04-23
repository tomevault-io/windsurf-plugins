---
trigger: always_on
description: RULE NUMBER 1 (NEVER EVER EVER FORGET THIS RULE!!!): YOU ARE NEVER ALLOWED TO DELETE A FILE WITHOUT EXPRESS PERMISSION FROM ME OR A DIRECT COMMAND FROM ME. EVEN A NEW FILE THAT YOU YOURSELF CREATED, SUCH AS A TEST CODE FILE. YOU HAVE A HORRIBLE TRACK RECORD OF DELETING CRITICALLY IMPORTANT FILES OR OTHERWISE THROWING AWAY TONS OF EXPENSIVE WORK THAT I THEN NEED TO PAY TO REPRODUCE. AS A RESULT, YOU HAVE PERMANENTLY LOST ANY AND ALL RIGHTS TO DETERMINE THAT A FILE OR FOLDER SHOULD BE DELETED. YOU
---

RULE NUMBER 1 (NEVER EVER EVER FORGET THIS RULE!!!): YOU ARE NEVER ALLOWED TO DELETE A FILE WITHOUT EXPRESS PERMISSION FROM ME OR A DIRECT COMMAND FROM ME. EVEN A NEW FILE THAT YOU YOURSELF CREATED, SUCH AS A TEST CODE FILE. YOU HAVE A HORRIBLE TRACK RECORD OF DELETING CRITICALLY IMPORTANT FILES OR OTHERWISE THROWING AWAY TONS OF EXPENSIVE WORK THAT I THEN NEED TO PAY TO REPRODUCE. AS A RESULT, YOU HAVE PERMANENTLY LOST ANY AND ALL RIGHTS TO DETERMINE THAT A FILE OR FOLDER SHOULD BE DELETED. YOU MUST **ALWAYS** ASK AND *RECEIVE* CLEAR, WRITTEN PERMISSION FROM ME BEFORE EVER EVEN THINKING OF DELETING A FILE OR FOLDER OF ANY KIND!!!

### IRREVERSIBLE GIT & FILESYSTEM ACTIONS — DO-NOT-EVER BREAK GLASS

1. **Absolutely forbidden commands:** `git reset --hard`, `git clean -fd`, `rm -rf`, or any command that can delete or overwrite code/data must never be run unless the user explicitly provides the exact command and states, in the same message, that they understand and want the irreversible consequences.
2. **No guessing:** If there is any uncertainty about what a command might delete or overwrite, stop immediately and ask the user for specific approval. “I think it’s safe” is never acceptable.
3. **Safer alternatives first:** When cleanup or rollbacks are needed, request permission to use non-destructive options (`git status`, `git diff`, `git stash`, copying to backups) before ever considering a destructive command.
4. **Mandatory explicit plan:** Even after explicit user authorization, restate the command verbatim, list exactly what will be affected, and wait for a confirmation that your understanding is correct. Only then may you execute it—if anything remains ambiguous, refuse and escalate.
5. **Document the confirmation:** When running any approved destructive command, record (in the session notes / final response) the exact user text that authorized it, the command actually run, and the execution time. If that record is absent, the operation did not happen.

We only use uv in this project, NEVER pip. And we use a venv. And we ONLY target Python 3.14 (we don't care about compatibility with earlier python versions), and we ONLY use pyproject.toml (not requirements.txt) for managing the project.

In general, you should try to follow all suggested best practices listed in the file `third_party_docs/PYTHON_FASTMCP_BEST_PRACTICES.md`

You can also consult `third_party_docs/fastmcp_distilled_docs.md` for any questions about the fastmcp library, or `third_party_docs/mcp_protocol_specs.md` for any questions about the MCP protocol in general. For anything relating to Postgres, be sure to read `third_party_docs/POSTGRES18_AND_PYTHON_BEST_PRACTICES.md`.

We load all configuration details from the existing .env file (even if you can't see this file, it DOES exist, and must NEVER be overwritten!). We NEVER use os.getenv() or dotenv or other methods to get variables from our .env file other than using python-decouple in this very specific pattern of usage (this is just an example but it always follows the same basic pattern):

```
from decouple import Config as DecoupleConfig, RepositoryEnv

# Initialize decouple config
decouple_config = DecoupleConfig(RepositoryEnv(".env"))

# Configuration
API_BASE_URL = decouple_config("API_BASE_URL", default="http://localhost:8007")
```

We use SQLmodel (which uses SQLalchemy ORM under the hood) for various database related functions. Here are some important guidelines to keep in mind when working with the database with these libraries:

Do:

- Create your engine with create_async_engine() and sessions via async_sessionmaker(...), then use async with AsyncSession(...) (or async_scoped_session) so sessions are closed automatically.
- Await every database operation that’s defined as async: await session.execute(...), await session.scalars(...), await session.stream(...), await session.commit(), await session.rollback(), etc.
- Keep one AsyncSession per request/task; don’t share it across concurrently running coroutines.
- Use session.scalars(select(...)), .first(), .one(), or .unique() to get ORM entities; use session.stream(...)/stream_scalars(...) when you need server-side streaming.
- Wrap sync-only work inside await session.run_sync(...) when you must call a synchronous SQLAlchemy helper.
- Explicitly load relationships (e.g., selectinload, joinedload) or use await obj.awaitable_attrs.<rel>; otherwise, lazy loads happen synchronously and will error in async code.
- On shutdown, call await engine.dispose() to close the async engine’s pool.

Don’t:

- Don’t reuse a single AsyncSession across multiple concurrent tasks or requests.
- Don’t rely on implicit IO from attribute access (lazy loads) inside async code; always load eagerly or use the awaitable-attrs API.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/mcp_agent_mail](https://github.com/Dicklesworthstone/mcp_agent_mail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
