---
trigger: always_on
description: Bare-metal x86_64 operating system built on pure RPN principles.
---

# Simplicity OS - Project Directives

## Vision
Bare-metal x86_64 operating system built on pure RPN principles.
Everything is a WORD. Hardware is directly composable.

## Core Philosophy
1. **Everything is a WORD** - No APIs, no system calls, just RPN words
2. **Stack-based interface** - Query returns values, -SET consumes values
3. **Direct hardware access** - No abstraction layers
4. **Lego composability** - Complex from simple, always
5. **Introspectable** - See and modify everything at runtime

## CRITICAL ARCHITECTURAL GUARDRAILS

### Pure Data Model (NEVER VIOLATE)
1. **ONLY `.` prints** - No other operation produces output
2. **Everything pushes to stack** - All operations return data
3. **No side-effect output** - Operations manipulate, don't display

### Object Model (MANDATORY)
1. **No fixed-size allocations** - Everything grows dynamically
2. **All data is objects** - With type headers
3. **Scalable to petabytes** - 1TB apps, 2PB data supported
4. **Type-tagged** - Every object knows its type

### Consistency Rules (STRICT)
1. **Pure RPN** - All operations including meta (tick + operator)
2. **No prefix operations** - Always: data then operation
3. **Tick for references** - ~word gets reference, word executes
4. **Objects, not primitives** - Strings are objects, not char arrays

### Error Handling (MANDATORY)
1. **No crashes** - All operations must validate input
2. **Return error objects** - Push STRING error descriptions
3. **Graceful degradation** - Invalid refs → "(invalid reference)"
4. **Stack safety** - Check bounds before pop/push

### Stack Discipline (STRICT)
1. **Document stack effects** - Every word shows ( in -- out )
2. **Validate critical operations** - @ and ! check address validity
3. **Correct order** - `!` is ( value addr -- ) so use: `100 [x] !` not `[x] 100 !`
4. **Type awareness** - Operations should check when possible

### Register Conventions (DOCUMENTED)
```
R14 = Top of stack (TOS) - Cached for performance
R15 = Data stack pointer (second-on-stack and below)
RBP = Return stack pointer (grows down: push = [RBP], RBP-=8)
RSI = Instruction pointer during definition execution
RSP = Machine stack (for function calls, preserved)

Why TOS register: Reduces memory accesses, improves performance
Stack layout: R14 = TOS, [R15-8] = second, [R15-16] = third, ...

Scratch registers (caller-saved):
RAX, RBX, RCX, RDX = Temporary, not preserved across operations
R8, R9, R10, R11, R12, R13 = Available for complex operations

Preserved across word calls:
R14, R15, RBP = Stack pointers (never corrupted)
RSI = Only modified during definition execution
```

### Memory Model (ENFORCED)
1. **Heap starts at 2MB** - Grows upward, unbounded
2. **No hardcoded addresses** - All pointers allocated
3. **Object headers** - [type:8][size:8][data:N]
4. **Page tables map as needed** - Currently 0-4MB, expand on demand

### Type System (REQUIRED)
```
TYPE_INT = 0        (immediate, no header, value < 0x100000)
TYPE_STRING = 1     (null-terminated text)
TYPE_REF = 2        (execution token)
TYPE_ARRAY = 3      (count + elements)
TYPE_USER_BASE = 4  (user-defined types start here)
```

**User-Defined Types:**
- `type-new` allocates next available tag (4, 5, 6...)
- `type-name` associates a STRING name with a tag
- `type-set` changes an object's type tag
- User types are structurally arrays with different type tags
- `.` displays `[typename: data...]` for named user types
- Up to 256 user-defined types supported

**Creating Custom Types:**
```
type-new                        ( -- 4 )
"point" 4 type-name             ( )
"point" [ swap , , 4 type-set ] define   ( x y -- point )
10 20 point .                   ( ) [point: 10 20 ]
```

## Design Constraints
- BIOS boot (simpler than UEFI)
- x86_64 protected mode → long mode
- Minimal kernel, pure RPN
- QEMU primary test platform
- Real hardware secondary target

## Naming Conventions
- `DEVICE` - Query device state, returns stack values
- `DEVICE-SET` - Configure device, consumes stack values
- `DEVICE-READ` - Read from device
- `DEVICE-WRITE` - Write to device

**Examples:**
```
SCREEN          ( -- x y brightness type manufacturer )
SCREEN-SET      ( x y brightness -- )
DISK-READ       ( sector -- addr len )
DISK-WRITE      ( addr len sector -- )
KEYBOARD-READ   ( -- scancode )
```

## Development Workflow
1. Write assembly in `kernel/`
2. Build with `make`
3. User tests in QEMU via `make run` in their terminal
4. Verify functionality works
5. Commit with clear message
6. Push to GitHub

**IMPORTANT: Do NOT start QEMU processes** - The user runs `make run` themselves in a separate terminal. Only build with `make`, never spawn QEMU.

## Testing Protocol
- Every feature tested in QEMU before commit
- Boot test: Does it boot and show prompt?
- Functionality test: Does new WORD work as specified?
- Integration test: Do existing WORDs still work?
- Manual verification required before marking complete

## VNC Self-Testing (for Claude)
When debugging requires iterative testing, Claude can use VNC:

```bash
# Start QEMU with VNC (headless)
qemu-system-x86_64 -drive file=build/simplicity.img,format=raw -vnc :1 -daemonize

# Capture screenshot
vncdotool -s localhost:1 capture /tmp/screenshot.png

# Send keystrokes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [isene/SimplicityOS](https://github.com/isene/SimplicityOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
