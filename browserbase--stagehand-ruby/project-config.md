---
trigger: always_on
description: <!-- x-stagehand-custom-start -->
---

# Stagehand Ruby API library

<!-- x-stagehand-custom-start -->
<div id="toc" align="center" style="margin-bottom: 0;">
  <ul style="list-style: none; margin: 0; padding: 0;">
    <a href="https://stagehand.dev">
      <picture>
        <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/browserbase/stagehand/main/media/dark_logo.png" />
        <img alt="Stagehand" src="https://raw.githubusercontent.com/browserbase/stagehand/main/media/light_logo.png" width="200" style="margin-right: 30px;" />
      </picture>
    </a>
  </ul>
</div>
<p align="center">
  <strong>The AI Browser Automation Framework</strong><br>
  <a href="https://docs.stagehand.dev/v3/sdk/ruby">Read the Docs</a>
</p>

<p align="center">
  <a href="https://github.com/browserbase/stagehand/tree/main?tab=MIT-1-ov-file#MIT-1-ov-file">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/browserbase/stagehand/main/media/dark_license.svg" />
      <img alt="MIT License" src="https://raw.githubusercontent.com/browserbase/stagehand/main/media/light_license.svg" />
    </picture>
  </a>
  <a href="https://stagehand.dev/discord">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/browserbase/stagehand/main/media/dark_discord.svg" />
      <img alt="Discord Community" src="https://raw.githubusercontent.com/browserbase/stagehand/main/media/light_discord.svg" />
    </picture>
  </a>
</p>

<p align="center">
	<a href="https://trendshift.io/repositories/12122" target="_blank"><img src="https://trendshift.io/api/badge/repositories/12122" alt="browserbase%2Fstagehand | Trendshift" style="width: 250px; height: 55px;" width="250" height="55"/></a>
</p>

<p align="center">
If you're looking for other languages, you can find them
<a href="https://docs.stagehand.dev/v3/first-steps/introduction"> here</a>
</p>

<div align="center" style="display: flex; align-items: center; justify-content: center; gap: 4px; margin-bottom: 0;">
  <b>Vibe code</b>
  <span style="font-size: 1.05em;"> Stagehand with </span>
  <a href="https://director.ai" style="display: flex; align-items: center;">
    <span>Director</span>
  </a>
  <span> </span>
  <picture>
    <img alt="Director" src="https://raw.githubusercontent.com/browserbase/stagehand/main/media/director_icon.svg" width="25" />
  </picture>
</div>
<!-- x-stagehand-custom-end -->

The Stagehand Ruby library provides convenient access to the Stagehand REST API from any Ruby 3.2.0+ application. It ships with comprehensive types & docstrings in Yard, RBS, and RBI – [see below](https://github.com/browserbase/stagehand-ruby#Sorbet) for usage with Sorbet. The standard library's `net/http` is used as the HTTP transport, with connection pooling via the `connection_pool` gem.

It is generated with [Stainless](https://www.stainless.com/).

## Documentation

Documentation for releases of this gem can be found [on RubyDoc](https://gemdocs.org/gems/stagehand).

The REST API documentation can be found on [docs.stagehand.dev](https://docs.stagehand.dev).

## Installation

To use this gem, install via Bundler by adding the following to your application's `Gemfile`:

<!-- x-release-please-start-version -->

```ruby
gem "stagehand", :git => "git://github.com/browserbase/stagehand-ruby.git"
```

<!-- x-release-please-end -->

## Usage

This mirrors `examples/remote_browser_playwright_example.rb`.

```ruby
require "bundler/setup"
require "stagehand"

require_relative "examples/env"
ExampleEnv.load!

require "playwright"

client = Stagehand::Client.new(
  browserbase_api_key: ENV["BROWSERBASE_API_KEY"],
  browserbase_project_id: ENV["BROWSERBASE_PROJECT_ID"],
  model_api_key: ENV["MODEL_API_KEY"],
  server: "remote"
)

start_response = client.sessions.start(
  model_name: "anthropic/claude-sonnet-4-6",
  browser: { type: :browserbase }
)

session_id = start_response.data.session_id
cdp_url = start_response.data.cdp_url
raise "No CDP URL returned for this session." if cdp_url.to_s.empty?

Playwright.create(playwright_cli_executable_path: "./node_modules/.bin/playwright") do |playwright|
  browser = playwright.chromium.connect_over_cdp(cdp_url)
  context = browser.contexts.first || browser.new_context
  page = context.pages.first || context.new_page

  client.sessions.navigate(session_id, url: "https://news.ycombinator.com")
  page.wait_for_load_state(state: "domcontentloaded")

  observe_stream = client.sessions.observe_streaming(
    session_id,
    instruction: "find the link to view comments for the top post"
  )
  observe_stream.each { |_event| }

  act_stream = client.sessions.act_streaming(
    session_id,
    input: "Click the comments link for the top post"
  )
  act_stream.each { |_event| }

  extract_stream = client.sessions.extract_streaming(
    session_id,
    instruction: "extract the text of the top comment on this page",
    schema: {
      type: "object",
      properties: {
        commentText: {type: "string"},
        author: {type: "string"}
      },
      required: ["commentText"]
    }
  )
  extract_stream.each { |_event| }

  execute_stream = client.sessions.execute_streaming(
    session_id,
    execute_options: {
      instruction: "Click the 'Learn more' link if available",
      max_steps: 3

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [browserbase/stagehand-ruby](https://github.com/browserbase/stagehand-ruby) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
