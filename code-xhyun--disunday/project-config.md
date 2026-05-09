---
trigger: always_on
description: <!-- This AGENTS.md file is generated. Look for an agents.md package.json script to see what files to update instead. -->
---

<!-- This AGENTS.md file is generated. Look for an agents.md package.json script to see what files to update instead. -->

# restarting the discord bot

ONLY restart the discord bot if the user explicitly asks for it.

To restart the discord bot process so it uses the new code, send a SIGUSR2 signal to it.

1. Find the process ID (PID) of the disunday discord bot (e.g., using `ps aux | grep disunday` or searching for "disunday" in process list).
2. Send the signal: `kill -SIGUSR2 <PID>`

The bot will wait 1000ms and then restart itself with the same arguments.

## sqlite

this project uses sqlite to preserve state between runs. the database should never have breaking changes, new disunday versions should keep working with old sqlite databases created by an older disunday version. if this happens specifically ask the user how to proceed, asking if it is ok adding migration in startup so users with existing db can still use disunday and will not break.

## errore

errore is a submodule. should always be in main. make sure it is never in detached state.

it is a package for using errors as values in ts.

## opencode

if I ask you questions about opencode you can opensrc it from anomalyco/opencode

# core guidelines

when summarizing changes at the end of the message, be super short, a few words and in bullet points, use bold text to highlight important keywords. use markdown.

please ask questions and confirm assumptions before generating complex architecture code.

NEVER run commands with & at the end to run them in the background. this is leaky and harmful! instead ask me to run commands in the background using tmux if needed.

NEVER commit yourself unless asked to do so. I will commit the code myself.

NEVER use git to revert files to previous state if you did not create those files yourself! there can be user changes in files you touched, if you revert those changes the user will be very upset!

## files

always use kebab case for new filenames. never use uppercase letters in filenames

never write temporary files to /tmp. instead write them to a local ./tmp folder instead. make sure it is in .gitignore too

## see files in the repo

use `git ls-files | tree --fromfile` to see files in the repo. this command will ignore files ignored by git

## handling unexpected file contents after a read or write

if you find code that was not there since the last time you read the file it means the user or another agent edited the file. do not revert the changes that were added. instead keep them and integrate them with your new changes

IMPORTANT: NEVER commit your changes unless clearly and specifically asked to!

## opening me files in zed to show me a specific portion of code

you can open files when i ask me "open in zed the line where ..." using the command `zed path/to/file:line`

# typescript

- ALWAYS use normal imports instead of dynamic imports, unless there is an issue with es module only packages and you are in a commonjs package (this is rare).
- when throwing errors always use clause instead of error inside message: `new Error("wrapping error", { cause: e })` instead of `new Error(\`wrapping error ${e}\`)`

- use a single object argument instead of multiple positional args: use object arguments for new typescript functions if the function would accept more than one argument, so it is more readable, ({a,b,c}) instead of (a,b,c). this way you can use the object as a sort of named argument feature, where order of arguments does not matter and it's easier to discover parameters.

- always add the {} block body in arrow functions: arrow functions should never be written as `onClick={(x) => setState('')}`. NEVER. instead you should ALWAYS write `onClick={() => {setState('')}}`. this way it's easy to add new statements in the arrow function without refactoring it.

- in array operations .map, .filter, .reduce and .flatMap are preferred over .forEach and for of loops. For example prefer doing `.push(...array.map(x => x.items))` over mutating array variables inside for loops. Always think of how to turn for loops into expressions using .map, .filter or .flatMap if you ever are about to write a for loop.

- if you encounter typescript errors like "undefined | T is not assignable to T" after .filter(Boolean) operations: use a guarded function instead of Boolean: `.filter(isTruthy)`. implemented as `function isTruthy<T>(value: T): value is NonNullable<T> { return Boolean(value) }`

- minimize useless comments: do not add useless comments if the code is self descriptive. only add comments if requested or if this was a change that i asked for, meaning it is not obvious code and needs some inline documentation. if a comment is required because the part of the code was result of difficult back and forth with me, keep it very short.

- ALWAYS add all information encapsulated in my prompt to comments: when my prompt is super detailed and in depth, all this information should be added to comments in your code. this is because if the prompt is very detailed it must be the fruit of a lot of research. all this information would be lost if you don't put it in the code. next LLM calls would misinterpret the code and miss context.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [code-xhyun/disunday](https://github.com/code-xhyun/disunday) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
