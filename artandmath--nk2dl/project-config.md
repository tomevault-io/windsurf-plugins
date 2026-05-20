---
trigger: always_on
description: - The files in /project_root/thinkbox are for reference and not for modification
---

# Thinkbox reference code

- The files in /project_root/thinkbox are for reference and not for modification
- They contain the nuke submission code that ships with thinkbox deadline
- This project aims to write a replacement for the functionality of the thinkbox code
- We don't modify the following files unless explicitely asked to do so.
- These are the files:
```
Client:
[DeadlineNukeClient.py](mdc:thinkbox/Client/DeadlineNukeClient.py)
[DeadlineNukeFrameServerClient.py](mdc:thinkbox/Client/DeadlineNukeFrameServerClient.py)
[DeadlineNukeVrayStandaloneClient.py](mdc:thinkbox/Client/DeadlineNukeVrayStandaloneClient.py)
[menu.py](mdc:thinkbox/Client/menu.py)

Main:
[DeadlineFRGlobals.py](mdc:thinkbox/Main/DeadlineFRGlobals.py)
[DeadlineGlobals.py](mdc:thinkbox/Main/DeadlineGlobals.py)
[DeadlineVRayGlobals.py](mdc:thinkbox/Main/DeadlineVRayGlobals.py)
[DeadlineNukeFrameServerClient.py](mdc:thinkbox/Client/DeadlineNukeFrameServerClient.py)
[SubmitNukeToDeadline.py](mdc:thinkbox/Main/SubmitNukeToDeadline.py)
[SubmitNukeVRayStandalone.py](mdc:thinkbox/Main/SubmitNukeVRayStandalone.py)

Plugin:
[Nuke.py](mdc:deadline/plugins/nuke/Nuke.py)
[Nuke.param](mdc:deadline/plugins/nuke/Nuke.param)
[Nuke.options](mdc:deadline/plugins/nuke/Nuke.options)
```

# Documentation from the thinkbox website. the following is reference only, not a set of rules. The following is documentation for the codebase we are replacing.

Nuke
Render Job Submission
You can submit jobs from within Nuke by installing the integrated submission script, or you can submit them from the Monitor. The instructions for installing the integrated submission script can be found further down this page.

To submit from within Nuke, select Submit To Deadline from the Thinkbox menu.

int mon1

Submission Options
The general Deadline options are explained in the Job Submission documentation, and the Draft/Integration options are explained in the Draft and Integration documentation. The Nuke specific options are:

Render With NukeX: Enable this option if you want to render with NukeX instead of Nuke.

Use Batch Mode: If enabled, Deadline will keep the Nuke file loaded in memory between tasks.

Render Threads: The number of threads to use for rendering.

Use The GPU For Rendering: If Nuke should also use the GPU for rendering (Nuke 7 and later only).

Use Specific GPU Override: If Nuke should use the specified GPU, otherwise the Workers GPU Affinity will be used ( Nuke 8 and later only ).

Maximum RAM Usage: The maximum RAM usage (in MB) to be used for rendering.

Enforce Write Node Render Order: Forces Nuke to obey the render order of Write nodes.

Minimum Stack Size: The minimum stack size (in MB) to be used for rendering. Set to 0 to not enforce a minimum stack size.

Continue On Error: If enabled, Nuke will attempt to keep rendering if an error occurs.

Reload Plugin Between Tasks: If enabled, Nuke will be restarted between tasks. This can add overhead, but ensures that Nuke’s memory is flushed between tasks.

Use Performance Profiler: If enabled, Nuke will profile the performance of the Nuke script while rendering and create a xml file per task for later analysis (Nuke 9 and later only).

XML Directory: If Use Performance Profiler is enabled, this is the directory on the network where the performance profile xml files will be saved.

Render Mode: Use scene settings, the proxy file paths, or force Nuke to use full resolution when rendering.

Choose Views To Render: Enable this option to choose which view(s) to render. By default, all views are rendered.

Submit Write Nodes As Separate Jobs: Each write node is submitted as a separate job.

Use Node’s Frame List: If submitting each write node as a separate job or task, enable this to pull the frame range from the write node, instead of using the global frame range.

Set Dependencies Based on Write Node Render Order: When submitting write nodes as separate jobs, this option will make the separate jobs dependent on each other based on write node render order.

Submit Write Nodes As Separate Tasks For The Same Job: Enable to submit a job where each task for the job represents a different write node, and all frames for that write node are rendered by its corresponding task.

Selected Nodes Only: If enabled, only the selected Write nodes will be rendered.

Nodes With ‘Read File’ Enabled Only: If enabled, only the Write nodes that have the ‘Read File’ option enabled will be rendered.

Render Precomp Nodes First: If enabled, all write nodes in precomp nodes will be rendered before the main job.

Only Render Precomp Nodes: If enabled, only the Write nodes that are in precomp nodes will be rendered.

Only Simulate Eddy Nodes: If enabled, only the Eddy Nodes will be simulated.

The Submit Each Write Node As A Separate Task option can be useful if you have a bunch of write nodes in a Nuke script to output different Quicktime movies. You can enable this option, and bump up the Concurrent Tasks value to allow machines to process multiple write nodes concurrently. Since Quicktime generation only uses a single thread, you can get much better throughput with this option on multi-core machines.

Nuke Script Job
To run a Nuke Script Job, select the Nuke Script Job type.

../_images/nuke_script_job.png

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [artandmath/nk2dl](https://github.com/artandmath/nk2dl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
