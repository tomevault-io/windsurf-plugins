---
trigger: always_on
description: Instructions for coding agents. Two audiences:
---

# AGENTS.md

Instructions for coding agents. Two audiences:

- **[Installing livechat into a Rails app](#installing-into-a-rails-app)** — you are working in a host app and were asked to add support chat, live chat, or an in-app inbox.
- **[Working on the gem itself](#working-on-the-gem-itself)** — you are working in this repository.

Requirements: Ruby >= 3.2, Rails >= 7.1 and < 9. Active Storage only for file attachments. **No Redis and no Action Cable** — the transport is polling unless you opt in.

If you are in a host app and this file is not in front of you, it ships inside the gem: `cat "$(bundle show livechat)/AGENTS.md"`.

---

## Installing into a Rails app

### 1. Install

```bash
bundle add livechat
bin/rails generate livechat:install
bin/rails db:migrate
```

The generator writes `config/initializers/livechat.rb`, one migration (`livechat_conversations`, `livechat_messages`), and `mount_livechat at: "/livechat"` into `config/routes.rb`. Read the initializer it wrote — every option is documented there in comments, and it is the source of truth over any summary of it, including this file.

Every `config.…` line below belongs inside the `Livechat.configure do |config|` block in that initializer. Uncomment and edit in place rather than appending a second `configure` block.

### 2. Wire the three things the generator cannot

**a. The widget tag.** Nothing appears until this is on the page:

```erb
<%# app/views/layouts/application.html.erb, before </body> %>
<%= livechat_tag %>
```

The helper is injected into ActionView by the engine — no include, no import, no asset pipeline entry. It renders the launcher bubble bottom-right.

**b. `authorize_agent` — do this before deploying.** The inbox at `/livechat` defaults to **development only**. It fails closed, so shipping without this is not an open inbox — it is a 403 reading "Forbidden. Set Livechat.config.authorize_agent to grant access."

```ruby
config.authorize_agent = ->(request) { request.env["warden"]&.user&.admin? }
```

**c. Visitor identity**, if the app has users. Without it every visitor is a cookie-tracked guest, and nobody in the inbox has a name.

```ruby
config.current_user  = ->(request) { request.env["warden"]&.user }
config.visitor_label = ->(user) { user.name }        # what the inbox shows
config.agent_label   = ->(user) { user.name }        # signed onto each reply
```

> **`current_user`, `enabled` and `authorize_agent` receive the raw `request`, not a controller.** Writing `->(request) { current_user }` is the most common mistake here — that method does not exist in this scope. Resolve the user *from the request*: Warden env, a signed cookie, `Current.user` if middleware already set it. Note the different shapes: `visitor_label` and `agent_label` receive the **user**, while `agent_display_name` receives the already-stored **label string**.

Rails 8 built-in auth:

```ruby
config.current_user = lambda do |request|
  token = request.cookies["session_token"]
  Session.find_signed(token)&.user if token
end
```

### 3. Verify

```bash
bin/rails routes | grep livechat     # engine mounted
bin/rails livechat:seed_demo         # optional sample conversations, idempotent
```

Then in the running app: load any page, confirm the bubble appears bottom-right, send a message, and answer it at `/livechat`.

### Opening the widget

| Way | How |
| --- | --- |
| The launcher bubble | On by default. `config.show_launcher = false` to remove it |
| Your own element | `<%= livechat_button %>`, or any element with `data-livechat-open` |
| JavaScript | `window.Livechat.open()` |

A visitor has **one conversation**, not a queue of tickets — writing again reopens the same thread. Signed-in visitors keep it across devices (keyed by user id); guests are tracked by cookie.

### Email notifications need two settings, not one

```ruby
config.mailer_from = "support@example.com"   # required, or nothing sends
config.agent_emails = ["team@example.com"]   # array, or a callable returning one
```

Setting `agent_emails` alone sends nothing: `mailer_from` is what switches email on (`Livechat.config.emails_enabled?` is `mailer_from.present?`). Notification is one email per unread stretch, not one per message.

### Realtime is opt-in

Polling is the default transport, on purpose — a host with no Action Cable works untouched. Turning on push requires the host to actually mount a cable:

```ruby
config.action_cable = true
config.action_cable_url = "/cable"   # keep in sync with the mount in routes.rb
```

Leave it off unless the app already has Action Cable working. Polling is not a degraded mode here.

### Attachments

`config.attach_files` is on by default but **silently inert without Active Storage** in the host app (`rails active_storage:install`) — the widget keeps working, just without a paperclip. Caps: `max_attachments` (5), `max_attachment_size` (10 MB), `allowed_attachment_types` (nil = any). Files are served through the engine at `/livechat/attachments/:id`, gated per request — never a public blob URL. Do not build your own blob links.

Every authorized agent can read every conversation; visitor reads are scoped to
the signed-in id or guest cookie. There is no tenant-isolated agent inbox in
1.x. No retention job runs automatically. Hosts should schedule their own

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yshmarov/livechat](https://github.com/yshmarov/livechat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
