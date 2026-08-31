---
trigger: always_on
description: X/Twitter automation with no X API key: a 56-command CLI, a Node library, 152
---

# XActions for agents

X/Twitter automation with no X API key: a 56-command CLI, a Node library, 152
MCP tools, 50 agent skills, 95 browser console scripts, and a web dashboard.
Apache-2.0, by nichxbt.

This file is for the agent, not the human. It answers one question first,
because getting it wrong costs the whole session: **do you shell out to the
CLI, or do you load the MCP server?**

---

## Two lanes into the same engine

XActions ships both lanes from one install, and they call the same code. The
difference is what each costs you before you have read a single tweet.

| | CLI lane | MCP lane |
|---|---|---|
| How you call it | one Bash call, `xactions <cmd> --compact` | a tool call, after the client connects the server |
| What loads into context up front | nothing | the whole tool list |
| What comes back | the rows you asked for, one per line | a JSON result object |
| Writes | reads, plus `engage` and `bulk` | every write tool, with an approval gate |
| Usable without client configuration | yes, it is just a process | no, the client must be configured first |

The tool list is not small. The server advertises 152 tools, and the
`tools/list` payload it serves is about 60 KB of JSON before you have done any
work at all. Measure it yourself:

```bash
node -e "import('./src/mcp/server.js').then(m => console.log(JSON.stringify(m.TOOLS).length, 'bytes,', m.TOOLS.length, 'tools'))"
```

That cost is worth paying when you are going to make many calls in a session,
keep state between them, or write. It is not worth paying to answer "how many
followers does this account have."

### The rule

**Reading a handful of things? Shell out. Working a long session, or writing?
Load the server.**

Concretely, prefer the CLI when:

- You need one or two facts and then you are done.
- You are inside a larger task where X is a detail, not the subject.
- You want to pipe, filter, or count the result with `jq`, `grep`, `sort`,
  `wc`, or feed it to another command in the same Bash call.
- The client has no MCP configuration and you are not going to add one.

Prefer the MCP server when:

- You are posting, replying, following, muting, or deleting, and you want the
  draft-approval gate to hold each write for a human.
- The session will make many calls and the per-call schema cost amortises.
- You want the structured error objects and the session state the server keeps
  across calls.

Nothing stops you from using both. `xactions mcp-config` writes the client
config, and the CLI keeps working next to it.

---

## The CLI lane

### The output contract

Two flags, and they are global, so every read command takes them:

- `--compact` prints one record per line as tab-separated `key=value` pairs,
  essential fields only, no colours and no spinners.
- `--fields <list>` narrows that to exactly the columns you name, in the order
  you name them.

`--json` is per command and prints the full structured object.

**`--compact` wins when both are passed.** They are alternatives, not a pair:
pick `--compact` when you are going to read the answer yourself, and `--json`
when you are going to pipe it into `jq`.

Field names are shared across commands, so `--fields likes` means the same
thing everywhere. The defaults per record kind:

| Record kind | Default columns |
|---|---|
| tweet | `id username date likes retweets replies views text` |
| profile | `id username name followers following tweets verified bio` |
| user | `id username name followers verified bio` |
| media | `type url tweetUrl` |
| report | `username followers following postsPerDay engagementRate medianEngagement mediaShare bestHourUTC bestWeekday` |

Anything else the record carried can still be named in `--fields`.

### Recipes

Read a profile:

```bash
xactions profile NASA --compact
xactions profile NASA --fields username,followers,verified --compact
```

Read someone's posts:

```bash
xactions tweets NASA --limit 50 --compact
xactions tweets NASA --limit 200 --fields id,date,likes,text --compact
xactions tweets NASA --limit 50 --json | jq '[.[] | select(.likes > 1000)] | length'
```

Search:

```bash
xactions search "agent skills" --limit 40 --compact
xactions search "mcp server" --filter top --limit 25 --fields username,likes,text --compact
```

`--filter` takes `latest` (the default), `top`, `people`, `photos`, `videos`.

Followers, and who does not follow back:

```bash
xactions followers nichxbt --limit 500 --compact
xactions following nichxbt --limit 500 --fields username,followers --compact
xactions non-followers nichxbt --limit 500 --compact
```

Account report, and comparisons:

```bash
xactions analyze NASA --compact
xactions analyze NASA SpaceX --fields username,followers,engagementRate,bestHourUTC --compact
```

`analyze` takes several usernames and samples `--limit` posts from each
(default 50, clamped to the 5 to 200 range).

The rest of the read surface, same flags:

```bash
xactions hashtag ai --limit 50 --compact
xactions thread https://x.com/NASA/status/1234567890 --compact
xactions media NASA --limit 30 --compact
```

### One Bash call, whole answer

The point of the lane is that the shell does the second half of the work, so
nothing intermediate has to pass through the context window:

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nirholas/XActions](https://github.com/nirholas/XActions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
