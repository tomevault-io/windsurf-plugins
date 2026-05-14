---
trigger: always_on
description: Guidelines and best practices for building applications with [Beefree SDK](https://docs.beefree.io/beefree-sdk), including installation, authentication, configuration, customization, and template management
---

# Beefree SDK Guidelines
Guidelines and best practices for building applications with [Beefree SDK](https://docs.beefree.io/beefree-sdk), including installation, authentication, configuration, customization, and template management.

## Installation Guidelines

### Package Installation
- Install the Beefree SDK package using npm or yarn:
  ```bash
  npm install @beefree.io/sdk
  # or
  yarn add @beefree.io/sdk
  ```

### Dependencies
- Beefree SDK requires the following core dependencies:
  ```json
  {
    "dependencies": {
      "@beefree.io/sdk": "^9.0.2-fix-optional-url-config.0",
      "axios": "^1.10.0",
      "express": "^5.1.0",
      "cors": "^2.8.5",
      "dotenv": "^17.2.0"
    }
  }
  ```

### Environment Setup
- Create a `.env` file in your project root with your Beefree credentials:
  ```env
  BEE_CLIENT_ID=your_client_id_here
  BEE_CLIENT_SECRET=your_client_secret_here
  ```

## Authentication Guidelines

### Proxy Server Setup
- ALWAYS use a proxy server for authentication to protect your credentials
- Create a proxy server file (e.g., `proxy-server.js`) to handle authentication:
  ```javascript
  import express from 'express';
  import cors from 'cors';
  import axios from 'axios';
  import dotenv from 'dotenv';

  dotenv.config();

  const app = express();
  const PORT = 3001;

  app.use(cors());
  app.use(express.json());

  const BEE_CLIENT_ID = process.env.BEE_CLIENT_ID;
  const BEE_CLIENT_SECRET = process.env.BEE_CLIENT_SECRET;

  // V2 Auth Endpoint
  app.post('/proxy/bee-auth', async (req, res) => {
    try {
      const { uid } = req.body;
      
      const response = await axios.post(
        'https://auth.getbee.io/loginV2',
        {
          client_id: BEE_CLIENT_ID,
          client_secret: BEE_CLIENT_SECRET,
          uid: uid || 'demo-user'
        },
        { headers: { 'Content-Type': 'application/json' } }
      );
      
      res.json(response.data);
    } catch (error) {
      console.error('Auth error:', error.message);
      res.status(500).json({ error: 'Failed to authenticate' });
    }
  });

  app.listen(PORT, () => {
    console.log(`Proxy server running on http://localhost:${PORT}`);
  });
  ```

### Authentication Process
- Use the V2 authentication endpoint: `https://auth.getbee.io/loginV2`
- Pass the ENTIRE API response to the Beefree SDK, not just the token
- Example authentication call:
  ```typescript
  const token = await fetch('http://localhost:3001/proxy/bee-auth', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ uid: 'demo-user' })
  }).then(res => res.json());
  ```

## Container Setup Guidelines

### HTML Container
- Create a dedicated container element for the Beefree SDK:
  ```html
  <div id="beefree-sdk-container"></div>
  ```

### CSS Styling
- Style the container to ensure proper display:
  ```css
  #beefree-sdk-container {
    position: absolute;
    top: 0px;
    bottom: 0px;
    left: 0px;
    right: 0px;
    height: 600px;
    width: 90%;
    margin: 20px auto;
    border: 1px solid #ddd;
    border-radius: 8px;
  }
  ```

### React Container
- For React applications, the following code snippet shows an example using refs to manage the container:
  ```typescript
  const containerRef = useRef<HTMLDivElement>(null);

  return (
    <div
      id="beefree-react-demo"
      ref={containerRef}
      style={{
        height: '600px',
        width: '90%',
        margin: '20px auto',
        border: '1px solid #ddd',
        borderRadius: '8px'
      }}
    />
  );
  ```

## Configuration Guidelines

### Required Configuration Parameters
- ALWAYS include the `container` parameter in your configuration:
  ```typescript
  const beeConfig = {
    container: 'beefree-sdk-container', // Required
    language: 'en-US'
  };
  ```

### Optional Configuration Parameters
- Customize your SDK with optional parameters:
  ```typescript
  const beeConfig = {
    container: 'beefree-sdk-container', // Required
    language: 'en-US',
    specialLinks: [
      {
        type: "unsubscribe",
        label: "Unsubscribe",
        link: "http://[unsubscribe]/",
      },
      {
        type: "subscribe",
        label: "Subscribe",
        link: "http://[subscribe]/",
      },
    ],
    mergeTags: [
      {
        name: "First Name",
        value: "[first_name]",
      },
      {
        name: "Last Name",
        value: "[last_name]",
      },
      {
        name: "Email",
        value: "[email]",
      },
    ]
  };
  ```

### Callback Functions
- Implement essential callback functions for proper functionality:
  ```typescript
  const beeConfig = {
    container: 'beefree-sdk-container',
    onSave: function (jsonFile, htmlFile) {
      console.log("Template saved:", jsonFile);
      // Implement custom save logic here
    },
    onAutoSave: function (jsonFile) {
      console.log("Auto-saving template...");
      localStorage.setItem("email.autosave", jsonFile);
    },
    onSend: function (htmlFile) {
      console.log("Email ready to send:", htmlFile);
      // Implement custom send logic here
    },
    onError: function (errorMessage) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [XD3an/awesome-ai-coding-all-in-one](https://github.com/XD3an/awesome-ai-coding-all-in-one) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
