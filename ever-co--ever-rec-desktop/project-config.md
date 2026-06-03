---
trigger: always_on
description: provideServerRendering(withRoutes(serverRoutes)),
---


<big style="margin-top: 2em">
Angular is a web framework that empowers developers to build fast, reliable applications.
</big>

Maintained by a dedicated team at Google, Angular provides a broad suite of tools, APIs, and
libraries to simplify and streamline your development workflow. Angular gives you
a solid platform on which to build fast, reliable applications that scale with both the size of
your team and the size of your codebase.

**Want to see some code?** Jump over to our [Essentials](mdc:essentials) for a quick overview of
what it's like to use Angular, or get started in the [Tutorial](mdc:tutorials/learn-angular) if you
prefer following step-by-step instructions.

## Features that power your development
## Develop applications faster than ever
## Ship with confidence
## Works at any scale
## Open-source first
## A thriving community
Get started with Angular quickly with online starters or locally with your terminal.

## Play Online

If you just want to play around with Angular in your browser without setting up a project, you can use our online sandbox:
## Set up a new project locally

If you're starting a new project, you'll most likely want to create a local project so that you can use tooling such as Git.

### Prerequisites

- **Node.js** - [v20.11.1 or newer](mdc:reference/versions)
- **Text editor** - We recommend [Visual Studio Code](mdc:https:/code.visualstudio.com)
- **Terminal** - Required for running Angular CLI commands
- **Development Tool** - To improve your development workflow, we recommend the [Angular Language Service](mdc:tools/language-service)

### Instructions

The following guide will walk you through setting up a local Angular project.

#### Install Angular CLI

Open a terminal (if you're using [Visual Studio Code](mdc:https:/code.visualstudio.com), you can open an [integrated terminal](mdc:https:/code.visualstudio.com/docs/editor/integrated-terminal)) and run the following command:

```
// npm
npm install -g @angular/cli
```
```
// pnpm
pnpm install -g @angular/cli
```
```
// yarn
yarn global add @angular/cli
```
```
// bun
bun install -g @angular/cli
```
If you are having issues running this command in Windows or Unix, check out the [CLI docs](mdc:tools/cli/setup-local#install-the-angular-cli) for more info.

#### Create a new project

In your terminal, run the CLI command `ng new` with the desired project name. In the following examples, we'll be using the example project name of `my-first-angular-app`.

```shell
ng new <project-name>
```
You will be presented with some configuration options for your project. Use the arrow and enter keys to navigate and select which options you desire.

If you don't have any preferences, just hit the enter key to take the default options and continue with the setup.

After you select the configuration options and the CLI runs through the setup, you should see the following message:

```shell
✔ Packages installed successfully.
    Successfully initialized git.
```

At this point, you're now ready to run your project locally!

#### Running your new project locally

In your terminal, switch to your new Angular project.

```shell
cd my-first-angular-app
```
All of your dependencies should be installed at this point (which you can verify by checking for the existent for a `node_modules` folder in your project), so you can start your project by running the command:

```shell
npm start
```
If everything is successful, you should see a similar confirmation message in your terminal:

```shell
Watch mode enabled. Watching for file changes...
NOTE: Raw file sizes do not reflect development server per-request transformations.
  ➜  Local:   http://localhost:4200/
  ➜  press h + enter to show help
```

And now you can visit the path in `Local` (e.g., `http://localhost:4200`) to see your application. Happy coding! 🎉

## Next steps

Now that you've created your Angular project, you can learn more about Angular in our [Essentials guide](mdc:essentials) or choose a topic in our in-depth guides!
# Angular coding style guide

## Introduction

This guide covers a range of style conventions for Angular application code. These recommendations
are not required for Angular to work, but instead establish a set of coding practices that promote
consistency across the Angular ecosystem. A consistent set of practices makes it easier to share
code and move between projects.

This guide does _not_ cover TypeScript or general coding practices unrelated to Angular. For
TypeScript, check
out [Google's TypeScript style guide](mdc:https:/google.github.io/styleguide/tsguide.html).

### When in doubt, prefer consistency

Whenever you encounter a situation in which these rules contradict the style of a particular file,
prioritize maintaining consistency within a file. Mixing different style conventions in a single
file creates more confusion than diverging from the recommendations in this guide.

## Naming

### Separate words in file names with hyphens

Separate words within a file name with hyphens (`-`). For example, a component named `UserProfile`
has a file name `user-profile.ts`.

### Use the same name for a file's tests with `.spec` at the end

For unit tests, end file names with `.spec.ts`. For example, the unit test file for

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ever-co/ever-rec-desktop](https://github.com/ever-co/ever-rec-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
