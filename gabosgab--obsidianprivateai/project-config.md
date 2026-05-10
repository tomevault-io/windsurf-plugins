---
trigger: always_on
description: Avoid using the global app object, app (or window.app). Instead, use the reference provided by your plugin instance, this.app.
---

Avoid using the global app object, app (or window.app). Instead, use the reference provided by your plugin instance, this.app.

The global app object is intended for debugging purposes and might be removed in the future.

Please avoid unnecessary logging. In it's default configuration, the developer console should only show error messages, debug messages should not be shown.

Consider organizing your code base using folders 
If your plugin uses more than one .ts file, consider organizing them into folders to make it easier to review and maintain.

The Node.js API, and the Electron API aren't available on mobile devices. Any calls to these libraries made by your plugin or it's dependencies can cause your plugin to crash.

Lookbehind in regular expressions is only supported on iOS 16.4 and above, and some iPhone and iPad users may still use earlier versions. To implement a fallback for iOS users, either refer to Platform-specific features, or use a JavaScript library to detect specific browser versions.

Section headings don't have "settings" in the heading text.

Use Sentence case in UI.

For more information on writing and formatting text for Obsidian, refer to our Style guide.

Only use headings under settings if you have more than one section. 
Avoid adding a top-level heading in the settings tab, such as "General", "Settings", or the name of your plugin.

If you have more than one section under settings, and one contains general settings, keep them at the top without adding a heading.

For example, look at the settings under Settings → Appearance.

Avoid "settings" in settings headings 
In the settings tab, you can add headings to organize settings. Avoid including the word "settings" to these headings. Since everything in under the settings tab is settings, repeating it for every heading becomes redundant.

Prefer "Advanced" over "Advanced settings".
Prefer "Templates" over "Settings for templates".
Use sentence case in UI 
Any text in UI elements should be using Sentence case instead of Title Case, where only the first word in a sentence, and proper nouns, should be capitalized.

Prefer "Template folder location" over "Template Folder Location".
Prefer "Create new note" over "Create New Note".
Use setHeading instead of a <h1>, <h2> 
Using the heading elements from HTML will result in inconsistent styling between different plugins.
Instead you should prefer the following:

new Setting(containerEl).setName('your heading title').setHeading();
Security 
Avoid innerHTML, outerHTML and insertAdjacentHTML 
Building DOM elements from user-defined input, using innerHTML, outerHTML and insertAdjacentHTML can pose a security risk.

The following example builds a DOM element using a string that contains user input, ${name}. name can contain other DOM elements, such as <script>alert()</script>, and can allow a potential attacker to execute arbitrary code on the user's computer.

function showName(name: string) {
  let containerElement = document.querySelector('.my-container');
  // DON'T DO THIS
  containerElement.innerHTML = `<div class="my-class"><b>Your name is: </b>${name}</div>`;
}
Instead, use the DOM API or the Obsidian helper functions, such as createEl(), createDiv() and createSpan() to build the DOM element programmatically. For more information, refer to HTML elements.

To cleanup a HTML elements contents use el.empty();

Resource management 
Clean up resources when plugin unloads 
Any resources created by the plugin, such as event listeners, must be destroyed or released when the plugin unloads.

When possible, use methods like registerEvent() or addCommand() to automatically clean up resources when the plugin unloads.

export default class MyPlugin extends Plugin {
  onload() {
    this.registerEvent(this.app.vault.on('create', this.onCreate));
  }

  onCreate: (file: TAbstractFile) => {
    // ...
  }
}
Note
You don't need to clean up resources that are guaranteed to be removed when your plugin unloads. For example, if you register a mouseenter listener on a DOM element, the event listener will be garbage-collected when the element goes out of scope.

Don't detach leaves in onunload 
When the user updates your plugin, any open leaves will be reinitialized at their original position, regardless of where the user had moved them.

Commands 
Avoid setting a default hotkey for commands 
Setting a default hotkey may lead to conflicts between plugins and may override hotkeys that the user has already configured.

It's also difficult to choose a default hotkey that is available on all operating systems.

Use the appropriate callback type for commands 
When you add a command in your plugin, use the appropriate callback type.

Use callback if the command runs unconditionally.
Use checkCallback if the command only runs under certain conditions.
If the command requires an open and active Markdown editor, use editorCallback, or the corresponding editorCheckCallback.

Workspace 
Avoid accessing workspace.activeLeaf directly 
If you want to access the active view, use getActiveViewOfType() instead:

const view = this.app.workspace.getActiveViewOfType(MarkdownView);


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gabosgab/ObsidianPrivateAI](https://github.com/gabosgab/ObsidianPrivateAI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
