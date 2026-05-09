---
trigger: always_on
description: Terminal SSH desktop com agente IA integrado. A IA executa comandos no dispositivo remoto, lê saídas, raciocina e continua iterando até completar a tarefa.
---

# RB Terminal

Terminal SSH desktop com agente IA integrado. A IA executa comandos no dispositivo remoto, lê saídas, raciocina e continua iterando até completar a tarefa.

## Stack

| Componente      | Tecnologia              |
|-----------------|-------------------------|
| Linguagem       | Python 3.11+            |
| GUI             | PySide6                 |
| LLM Provider    | OpenRouter              |
| SSH             | asyncssh                |
| Terminal        | pyte (emulação VT100)   |
| Criptografia    | cryptography (PBKDF2)   |
| Persistência    | JSON local unificado    |
| Async           | qasync (Qt + asyncio)   |

## Estrutura do Projeto

```
rb-terminal/
├── main.py                     # Entry point (--debug para logs detalhados)
├── core/
│   ├── agent.py                # Agente IA com OpenRouter API
│   ├── ssh_session.py          # Wrapper asyncssh com PTY
│   ├── sftp_manager.py         # Operações SFTP async (download, upload, list)
│   ├── data_manager.py         # Gerenciador unificado de dados (singleton)
│   ├── crypto.py               # Criptografia PBKDF2 + master password
│   ├── hosts.py                # [LEGADO] Mantido para referência
│   ├── settings.py             # [LEGADO] Mantido para referência
│   ├── device_types.py         # Gerenciador de tipos de dispositivos
│   └── web_autologin.py        # Auto-login web (MikroTik, Zabbix, Proxmox)
├── gui/
│   ├── main_window.py          # Janela principal com hosts view + abas + chat
│   ├── terminal_widget.py      # Widget terminal com emulação ANSI
│   ├── file_browser.py         # File browser SFTP lateral
│   ├── tab_session.py          # Dataclass TabSession (estado por aba)
│   ├── chat_widget.py          # Widget chat IA
│   ├── hosts_view.py           # Tela principal de hosts (cards/lista/tabela)
│   ├── host_card.py            # HostCard, HostListItem, HostsTableWidget
│   ├── fields_config_dialog.py # Dialog para configurar campos visíveis
│   ├── tags_widget.py          # Widget de tags com autocomplete
│   ├── hosts_dialog.py         # Dialogs de hosts
│   ├── settings_dialog.py      # Dialog de configurações
│   ├── setup_dialog.py         # Dialog de primeira execução
│   ├── unlock_dialog.py        # Dialog de desbloqueio
│   ├── change_password_dialog.py # Dialog para alterar senha mestra
│   └── export_import_dialogs.py  # Dialogs de exportação/importação
├── config/
│   └── settings.json           # Configurações fallback
└── requirements.txt
```

## Arquivos de Dados

Salvos em `~/.rb-terminal/` (ou `%APPDATA%\.rb-terminal` no Windows):

- `data.json` - Arquivo unificado com hosts, settings e config de segurança
- `pointer.json` - Aponta para localização customizada do data.json (ex: Dropbox)
- `.session` - Cache da chave derivada para sessão atual
- `device_types.json` - Tipos de dispositivos customizados

### Estrutura do data.json

```json
{
  "version": "1.0",
  "security": {
    "has_master_password": true,
    "password_salt": "base64...",
    "password_hash": "base64..."
  },
  "settings": {
    "openrouter_api_key": "sk-...",
    "default_model": "google/gemini-2.5-flash",
    "max_agent_iterations": 10,
    "chat_position": "bottom",
    "sftp_position": "left",
    "available_tags": ["prod", "dev"],
    "hosts_view_mode": "cards",
    "hosts_sort_by": "name",
    "max_conversations_per_host": 10,
    "winbox_path": "C:/path/to/winbox64.exe",
    "card_visible_fields": ["name", "host", "tags", "device_type"],
    "list_visible_fields": ["name", "host", "port", "username", "tags", "device_type", "manufacturer"],
    "list_column_widths": {"host": 200, "port": 80},
    "ai_system_prompt": "",
    "telegram_bot_token": "",
    "telegram_chat_id": "",
    "telegram_backup_enabled": false
  },
  "hosts": [
    {
      "id": "uuid",
      "name": "Router Principal",
      "hosts": ["192.168.1.1", "router.example.com", "2001:db8::1"],
      "host": "192.168.1.1",
      "port": 22,
      "username": "admin",
      "port_knocking": [
        {"protocol": "tcp", "port": 1234},
        {"protocol": "udp", "port": 5678}
      ],
      "winbox_port": 8291,
      "http_port": 80,
      "https_enabled": false,
      "web_username": "admin",
      "web_password_encrypted": "..."
    }
  ],
  "conversations": [
    {
      "id": "uuid",
      "host_id": "host-uuid",
      "title": "Título gerado da primeira mensagem",
      "created_at": "2024-12-04T10:00:00",
      "updated_at": "2024-12-04T10:30:00",
      "messages": [
        {"role": "user", "content": "...", "timestamp": "..."},
        {"role": "assistant", "content": "...", "timestamp": "..."}
      ],
      "prompt_tokens": 0,
      "completion_tokens": 0,
      "total_cost": 0.0
    }
  ]
}
```

## Componentes Principais

### core/data_manager.py

Singleton `get_data_manager()` que unifica hosts e settings. Gerencia:
- Criptografia com master password opcional (PBKDF2 600k iterações)
- Cache de sessão para evitar digitar senha toda vez
- Caminho customizável via pointer.json (sincronização Dropbox/OneDrive)
- Export/import com senha de proteção opcional
- Migração automática de arquivos legados

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RedesBrasil/rb-terminal](https://github.com/RedesBrasil/rb-terminal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
