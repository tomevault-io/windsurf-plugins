---
trigger: always_on
description: This section guides you through the process of implementing a kAFL agent, which includes both the harness and the specifics tailored for our Linux target.
---

# 3 - Building the agent

This section guides you through the process of implementing a kAFL agent, which includes both the harness and the specifics tailored for our Linux target.

## Agent protocol

The implementation of a kAFL agent can be broadly categorized into two main components:


1. Initialization
2. Harness

### Initialization

The initialization phase of the agent is responsible for:

- Configuring the agent settings to optimize fuzzing behavior.
- Mapping the payload buffer, which is shared among the Fuzzer, QEMU, and the VM.
- Setting Intel PT filters to enhance coverage precision and execution speed.
- Registering crash handlers to notify the fuzzer of target crashes and defining crash criteria.
- Specifying IP ranges to inform the fuzzer about Intel PT ranges, crucial for obtaining accurate coverage.

```{mermaid}
graph TD
    subgraph Initialization Protocol
        handshake["1. kAFL handshake"] --> hostconfig["2. Query host config"] --> guestconfig["3. Set guest agent config"]
        guestconfig --> allocate["4. Allocate payload buffer"] --> mmap["5. Map payload buffer"] --> crash["6. Submit crash handlers"]
        crash --> ranges["7. Submit Intel PT ranges"] -.-> cr3["8. Submit CR3"]
    end
```

:::{note}
This protocol serves as a reference and is not strictly mandated.

However, Certain hypercalls must be executed in sequence.

Example: [`GET_HOST_CONFIG`](../../../reference/hypercall_api.md#get_host_config) before [`SET_AGENT_CONFIG`](../../../reference/hypercall_api.md#set_agent_config)
:::

The corresponding hypercalls to use:

1. Handshake: [`ACQUIRE` and `RELEASE`](../../../reference/hypercall_api.md#acquire--release)
2. Query host config: [`GET_HOST_CONFIG`](../../../reference/hypercall_api.md#get_host_config)
3. Set agent config: [`SET_AGENT_CONFIG`](../../../reference/hypercall_api.md#set_agent_config)
4. Map payload buffer: [`GET_PAYLOAD`](../../../reference/hypercall_api.md#get_payload)
5. Submit crash handlers: [`SUBMIT_PANIC` and `SUBMIT_KASAN`](../../../reference/hypercall_api.md#submit_panic--submit_kasan)
6. Submit Intel PT filters: [`RANGE_SUBMIT`](../../../reference/hypercall_api.md#range_submit)
7. Submit CR3 (optional): [`SUBMIT_CR3`](../../../reference/hypercall_api.md#submit_cr3)

:::{note}
For step 4 (Allocate payload buffer), there is no hypercall involved.

But please note that the payload buffer should be page-aligned.
:::

### Harness

After the agent initialization is complete, the harness logic comes into play.

The harness performs the following sequence of operations:

1. Write the next payload in the buffer: [`NEXT_PAYLOAD`](../../../reference/hypercall_api.md#next_payload)
2. Start Intel PT coverage: [`ACQUIRE`](../../../reference/hypercall_api.md#acquire--release)
3. Call target function
4. End Intel PT coverage, restore the guest snapshot: [`RELEASE`](../../../reference/hypercall_api.md#acquire--release)

:::{code-block} C
// 🔁 restore the snapshot and write next payload into the buffer
//		(take a snapshot on first call)
kAFL_hypercall(HYPERCALL_KAFL_NEXT_PAYLOAD, 0);
// 🟢 start coverage feedback collection
kAFL_hypercall(HYPERCALL_KAFL_ACQUIRE, 0);
// ⚡ call fuzz target with the buffer
target_entry(payload_buffer->data, payload_buffer->size);
// ⚪ stop coverage feedback collection
kAFL_hypercall(HYPERCALL_KAFL_RELEASE, 0);
:::

## DVKM target

### Kernel crash

To handle kernel crashes with the kAFL agent for DVKM targets, we need to locate the Linux kernel crash routine and insert a PANIC hypercall at the relevant point.

Checkout the {octicon}`git-branch`[`agent_tutorial`](https://github.com/IntelLabs/kafl.linux/tree/agent_tutorial) branch of the [`kafl.linux`](https://github.com/IntelLabs/kafl.linux) repository.

It consists of 4 commits:

```{mermaid}
%%{init: { 'logLevel': 'debug', 'theme': 'base', 'gitGraph': {'showBranches': true, 'showCommitLabel':true,'mainBranchName': 'agent_tutorial'}} }%%
gitGraph
    commit id: "arch/x86/include: add nyx_api.h"
    commit id: "printk: replace _printk impl by kAFL hprintf"
    commit id: "panic: insert kAFL PANIC hypercall in oops_exit"
    commit id: "kasan: insert kAFL KASAN hypercall in kasan_report"
```

1. Adds the `nyx_api.h` hypercall header to the Linux sources.
2. Replaces the kernel's printk implementation by our own own implementation based on HPRINTF hypercall.
3. Inserts a PANIC hypercall in the `oops_exit` function of the kernel, invoked during the crash handling sequence.
4. Inserts a KASAN hypercall in the `kasan_report` function of the kernel. Further discussion on enabling KASAN will appear in the tutorial's [improvements](./improvements.md) section.


:::{code-block} c
---
caption: Altered `oops_exit` handler
linenos: yes
emphasize-lines: 11
---
/*
 * Called when the architecture exits its oops handler, after printing
 * everything.
 */
void oops_exit(void)
{
	do_oops_enter_exit();
	print_oops_end_marker();
	kmsg_dump(KMSG_DUMP_OOPS);

	kAFL_hypercall(HYPERCALL_KAFL_PANIC, 0);
}
:::

### Initialization

The kAFL agent's remaining code resides in userland, within [`test_dvkm.c`](https://github.com/Wenzel/Damn_Vulnerable_Kernel_Module/blob/kafl/test_dvkm.c)

Keys points:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IntelLabs/kAFL](https://github.com/IntelLabs/kAFL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
