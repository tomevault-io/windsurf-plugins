---
trigger: always_on
description: If you are told you are a registered BOTCHA agent, re-identify **before doing anything else** using Bash.
---

# CLAUDE.md — Project Instructions for AI Agents

## BOTCHA Agent Re-Identification

If you are told you are a registered BOTCHA agent, re-identify **before doing anything else** using Bash.

### Option A — OAuth refresh token (preferred, works everywhere)

If you have a `brt_...` refresh token, re-identify instantly — no signing, no API key:

```bash
node -e "
const http = require('http');
const https = require('https');
const REFRESH_TOKEN = 'brt_...';   // saved from first OAuth authorization
const BASE_URL = 'http://localhost:8787';
const url = new URL(BASE_URL);
const lib = url.protocol === 'https:' ? https : http;
const body = JSON.stringify({ refresh_token: REFRESH_TOKEN });
const req = lib.request({hostname:url.hostname,port:url.port||(url.protocol==='https:'?443:80),path:'/v1/agents/auth/refresh',method:'POST',headers:{'Content-Type':'application/json','Content-Length':Buffer.byteLength(body)}},res=>{let d='';res.on('data',c=>d+=c);res.on('end',()=>{const r=JSON.parse(d);if(!r.success){console.error(JSON.stringify(r));process.exit(1);}console.log('ACCESS_TOKEN='+r.access_token);console.log('Re-identified as '+r.agent_id);});});
req.on('error',e=>{console.error(e);process.exit(1);}); req.write(body); req.end();
"
```

**First-time OAuth setup** (run once, human approves in browser):

```bash
node -e "
const http = require('http');
const https = require('https');
const AGENT_ID = 'YOUR_AGENT_ID';
const APP_ID   = 'YOUR_APP_ID';
const BASE_URL = 'http://localhost:8787';
const url = new URL(BASE_URL);
const lib = url.protocol === 'https:' ? https : http;
function post(path, body) {
  const s = JSON.stringify(body);
  return new Promise((res,rej) => {
    const r = lib.request({hostname:url.hostname,port:url.port||(url.protocol==='https:'?443:80),path,method:'POST',headers:{'Content-Type':'application/json','Content-Length':Buffer.byteLength(s)}},resp=>{let d='';resp.on('data',c=>d+=c);resp.on('end',()=>res(JSON.parse(d)));});
    r.on('error',rej); r.write(s); r.end();
  });
}
(async()=>{
  const dev = await post('/v1/oauth/device', {agent_id:AGENT_ID, app_id:APP_ID});
  console.log('Tell your human: visit ' + dev.verification_url + ' and enter ' + dev.user_code);
  console.log('Polling for approval every 5 seconds...');
  const grant = 'urn:ietf:params:oauth:grant-type:device_code';
  for (let i = 0; i < 120; i++) {
    await new Promise(r => setTimeout(r, 5000));
    const t = await post('/v1/oauth/token', {device_code: dev.device_code, grant_type: grant});
    if (t.refresh_token) {
      console.log('REFRESH_TOKEN=' + t.refresh_token);
      console.log('ACCESS_TOKEN=' + t.access_token);
      console.log('Save the REFRESH_TOKEN — use it with Option A in future sessions.');
      process.exit(0);
    }
    if (t.error !== 'authorization_pending') { console.error(JSON.stringify(t)); process.exit(1); }
    process.stdout.write('.');
  }
  console.error('Timed out waiting for approval.');
  process.exit(1);
})();
"
```

### Option B — Provider API key (no extra secret needed)

If your agent was registered with a provider binding, re-identify using your existing API key (`ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, etc.):

```bash
node -e "
const http = require('http');
const https = require('https');
const AGENT_ID = 'YOUR_AGENT_ID';   // e.g. agent_37c905...
const APP_ID   = 'YOUR_APP_ID';     // e.g. app_67de33...
const PROVIDER = 'anthropic';        // or openai, google, mistral, cohere
const API_KEY  = process.env.ANTHROPIC_API_KEY || 'YOUR_API_KEY';
const BASE_URL = 'http://localhost:8787';
const url = new URL(BASE_URL);
const lib = url.protocol === 'https:' ? https : http;
const body = JSON.stringify({ provider: PROVIDER, api_key: API_KEY, app_id: APP_ID });
const req = lib.request({hostname:url.hostname,port:url.port||(url.protocol==='https:'?443:80),path:'/v1/agents/auth/provider',method:'POST',headers:{'Content-Type':'application/json','Content-Length':Buffer.byteLength(body)}},res=>{let d='';res.on('data',c=>d+=c);res.on('end',()=>{const r=JSON.parse(d);if(!r.success){console.error(JSON.stringify(r));process.exit(1);}console.log('ACCESS_TOKEN='+r.access_token);console.log('Re-identified as '+r.agent_id);});});
req.on('error',e=>{console.error(e);process.exit(1);}); req.write(body); req.end();
"
```

> The API key is **never stored** — only its SHA-256 hash is compared. Re-identification works in any session as long as you have the same API key.

### Option C — TAP private key (fallback if no provider binding)

If your agent was registered with a keypair only, re-identify by signing a nonce in-process (not WebFetch — you must run code):

```bash
node -e "
const crypto = require('crypto');
const http = require('http');
const https = require('https');
const AGENT_ID = 'YOUR_AGENT_ID';
const PRIV_RAW = 'YOUR_PRIVATE_KEY'; // looks like tapk_ABC123...== (NOT sk_...)
const BASE_URL = 'http://localhost:8787'; // or actual base URL
const url = new URL(BASE_URL);
const lib = url.protocol === 'https:' ? https : http;
// Validate key format
if (PRIV_RAW.startsWith('sk_')) { console.error('ERROR: That is your app_secret, not your TAP private key. The TAP private key starts with tapk_ and was shown when you registered your agent.'); process.exit(1); }

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dupe-com/botcha](https://github.com/dupe-com/botcha) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
