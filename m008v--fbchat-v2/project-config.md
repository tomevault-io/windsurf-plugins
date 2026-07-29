---
trigger: always_on
description: File này cung cấp context triển khai. Tài liệu cho người dùng nằm ở `README.md`, `README_EN.md` và `DOCS.md`.
---

# CLAUDE.md - Hướng dẫn codebase fbchat-v2 cho contributor và coding agent

File này cung cấp context triển khai. Tài liệu cho người dùng nằm ở `README.md`, `README_EN.md` và `DOCS.md`.

## 📌 Tóm tắt bắt buộc

- Repository dùng kiến trúc 3 tầng: `_core`, `_features`, `_messaging`.
- Public I/O API mới là async-first và không có hậu tố `_async`.
- HTTP async phải dùng `httpx.AsyncClient` thật.
- Helper blocking còn lại phải có hậu tố `_blocking` và chỉ nằm ở boundary.
- `dataFB`, cookie, credential, token, TOTP và E2EE device state là secret.
- `src/config.json` là local-only; chỉ track `src/config.example.json`.
- Bot mẫu dùng E2EE listener làm receive/send transport chính.
- Mọi thay đổi public behavior phải cập nhật cả tài liệu Việt và Anh.
- Commit dùng Conventional Commits có scope.

## 📂 Cấu trúc repository

```text
fbchat-v2/
├── src/
│   ├── main.py
│   ├── config.example.json
│   ├── _core/
│   │   ├── _http.py
│   │   ├── _session.py
│   │   ├── _storage.py
│   │   ├── _utils.py
│   │   └── _facebookLogin.py
│   ├── _features/
│   │   ├── _facebook/
│   │   └── _thread/
│   └── _messaging/
│       ├── _send.py
│       ├── _attachments.py
│       ├── _listening.py
│       ├── _listening_e2ee.py
│       ├── _bridge_actions.py
│       ├── _send_e2ee.py
│       ├── _editMessage.py
│       ├── _changeTheme.py
│       ├── _createNotes.py
│       ├── _reactions.py
│       ├── _unsend.py
│       └── _message_requests.py
├── bridge-e2ee/
│   ├── main.go
│   ├── bridge/
│   ├── meta/
│   └── go.mod
├── tests/
├── README.md
├── README_EN.md
├── DOCS.md
├── FLOWCHART.md
└── pyproject.toml
```

## 🏗️ Kiến trúc 3 tầng

### Tầng 1: `_core`

Sở hữu:

- HTTP transport.
- Session bootstrap.
- Storage abstraction.
- Cookie/form/parser/ID utilities.
- Credential login và 2FA.

Không được import `_features` hoặc `_messaging`.

### Tầng 2: `_features`

Sở hữu nghiệp vụ Facebook và thread:

- Account/profile/Marketplace.
- Search, notification, block/unblock.
- Thread list, rename, emoji, nickname, admin.

Phụ thuộc `_core`, không sở hữu listener hoặc bot lifecycle.

### Tầng 3: `_messaging`

Sở hữu:

- Send và attachment HTTP.
- MQTT listener thường.
- E2EE bridge listener/action.
- Reaction, edit, unsend, theme, notes, message requests.

Phụ thuộc `_core`. Chỉ bot/application mới phối hợp `_features` với `_messaging`.

## 📋 Hợp đồng `dataFB`

Schema tối thiểu:

```python
{
    "fb_dtsg": "...",
    "jazoest": "...",
    "sessionID": "...",
    "FacebookID": "100012345678",
    "clientRevision": "...",
    "cookieFacebook": "c_user=...; xs=...; fr=...; datr=...;",
}
```

Nguồn duy nhất:

```python
data_fb = await dataGetHome(cookie_or_storage)
```

Không tự tạo dict giả ở runtime. Tests dùng fixture synthetic trong `tests/conftest.py`.

Không log object. Khi validate, chỉ báo tên field thiếu:

```python
missing = [name for name in REQUIRED if not data_fb.get(name)]
logger.error("Missing dataFB fields: %s", missing)
```

## ⚡ Hợp đồng async

### Naming

Public coroutine dùng tên domain ngắn:

```python
await dataGetHome(...)
await module.func(...)
await sender.send(...)
await listener.connect_mqtt()
```

Không thêm:

```python
func_async = func
func_sync = func
```

Blocking compatibility helper phải rõ nghĩa:

```python
send_blocking(...)
connect_mqtt_blocking(...)
call_blocking(...)
```

### Native async và thread adapter

Native async bắt buộc cho HTTP:

```python
response = await send_request_async(req, client=client)
```

Thread adapter chỉ hợp lệ cho thư viện blocking không thể thay:

```python
await asyncio.to_thread(self.connect_mqtt_blocking)
await asyncio.to_thread(self.call_blocking, method, params, timeout)
```

Không bọc code `requests` mới trong `to_thread` chỉ để gắn nhãn async. Chỉ giữ boundary legacy có lý do rõ ràng và test.

### Client injection

Feature HTTP nên có keyword-only client:

```python
async def func(
    dataFB: dict[str, Any],
    feature_value: str,
    *,
    client: httpx.AsyncClient | None = None,
) -> dict[str, Any]:
    ...
```

Caller-owned client không được đóng bên trong feature. Owned temporary client phải được context manager đóng.

## 🌐 HTTP implementation

### Transport flow

```text
feature._build_request
  -> _core._utils helper
    -> _core._http.post_async/get_async
      -> httpx.AsyncClient
```

`_core._http._clean_kwargs()` copy input trước khi pop. Giữ behavior này; mutate request dict của caller sẽ tạo race khi retry hoặc concurrent use.

### GraphQL flow

```python
form = formAll(
    data_fb,
    FBApiReqFriendlyName="FriendlyName",
    docID="123",
)
payload = await post_form_json_async(
    GRAPHQL_URL,
    form,
    data_fb["cookieFacebook"],
    client=client,
)
```

Parser phải kiểm tra:

- HTTP status.
- JSON decode.
- Top-level `error`.
- GraphQL `errors`.
- Nested `data`/`payload` và field bắt buộc.

HTTP 200 không đồng nghĩa mutation success.

### TLS và timeout

- Không dùng `verify=False`.
- Không dùng `ssl.CERT_NONE` cho MQTT chứa cookie.
- Mọi request có timeout hữu hạn.
- Retry chỉ áp dụng lỗi transient, có backoff và giới hạn.
- Không retry mutation mù nếu request có thể đã được server áp dụng.

## 💾 Session và storage

`dataGetHome()`:

1. Resolve cookie từ tham số hoặc storage.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [m008v/fbchat-v2](https://github.com/m008v/fbchat-v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
