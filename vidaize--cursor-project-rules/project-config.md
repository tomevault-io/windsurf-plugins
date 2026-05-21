---
trigger: always_on
description: Learn how to use Google Generative AI Provider.
---


# Google Generative AI Provider

The [Google Generative AI](mdc:https:/ai.google/discover/generativeai) provider contains language and embedding model support for
the [Google Generative AI](mdc:https:/ai.google.dev/api/rest) APIs.

## Setup

The Google provider is available in the `@ai-sdk/google` module. You can install it with

<Tabs items={['pnpm', 'npm', 'yarn']}>
  <Tab>
    <Snippet text="pnpm add @ai-sdk/google" dark />
  </Tab>
  <Tab>
    <Snippet text="npm install @ai-sdk/google" dark />
  </Tab>
  <Tab>
    <Snippet text="yarn add @ai-sdk/google" dark />
  </Tab>
</Tabs>

## Provider Instance

You can import the default provider instance `google` from `@ai-sdk/google`:

```ts
import { google } from '@ai-sdk/google';
```

If you need a customized setup, you can import `createGoogleGenerativeAI` from `@ai-sdk/google` and create a provider instance with your settings:

```ts
import { createGoogleGenerativeAI } from '@ai-sdk/google';

const google = createGoogleGenerativeAI({
  // custom settings
});
```

You can use the following optional settings to customize the Google Generative AI provider instance:

- **baseURL** _string_

  Use a different URL prefix for API calls, e.g. to use proxy servers.
  The default prefix is `https://generativelanguage.googleapis.com/v1beta`.

- **apiKey** _string_

  API key that is being sent using the `x-goog-api-key` header.
  It defaults to the `GOOGLE_GENERATIVE_AI_API_KEY` environment variable.

- **headers** _Record&lt;string,string&gt;_

  Custom headers to include in the requests.

- **fetch** _(input: RequestInfo, init?: RequestInit) => Promise&lt;Response&gt;_

  Custom [fetch](mdc:https:/developer.mozilla.org/en-US/docs/Web/API/fetch) implementation.
  Defaults to the global `fetch` function.
  You can use it as a middleware to intercept requests,
  or to provide a custom fetch implementation for e.g. testing.

## Language Models

You can create models that call the [Google Generative AI API](mdc:https:/ai.google.dev/api/rest) using the provider instance.
The first argument is the model id, e.g. `gemini-1.5-pro-latest`.
The models support tool calls and some have multi-modal capabilities.

```ts
const model = google('gemini-1.5-pro-latest');
```

<Note>
  You can use fine-tuned models by prefixing the model id with `tunedModels/`,
  e.g. `tunedModels/my-model`.
</Note>

Google Generative AI models support also some model specific settings that are not part of the [standard call settings](mdc:docs/ai-sdk-core/settings).
You can pass them as an options argument:

```ts
const model = google('gemini-1.5-pro-latest', {
  safetySettings: [
    { category: 'HARM_CATEGORY_UNSPECIFIED', threshold: 'BLOCK_LOW_AND_ABOVE' },
  ],
});
```

The following optional settings are available for Google Generative AI models:

- **cachedContent** _string_

  Optional. The name of the cached content used as context to serve the prediction.
  Format: cachedContents/\{cachedContent\}

- **structuredOutputs** _boolean_

  Optional. Enable structured output. Default is true.

  This is useful when the JSON Schema contains elements that are
  not supported by the OpenAPI schema version that
  Google Generative AI uses. You can use this to disable
  structured outputs if you need to.

  See [Troubleshooting: Schema Limitations](mdc:#schema-limitations) for more details.

- **safetySettings** _Array\<\{ category: string; threshold: string \}\>_

  Optional. Safety settings for the model.

  - **category** _string_

    The category of the safety setting. Can be one of the following:

    - `HARM_CATEGORY_HATE_SPEECH`
    - `HARM_CATEGORY_DANGEROUS_CONTENT`
    - `HARM_CATEGORY_HARASSMENT`
    - `HARM_CATEGORY_SEXUALLY_EXPLICIT`

  - **threshold** _string_

    The threshold of the safety setting. Can be one of the following:

    - `HARM_BLOCK_THRESHOLD_UNSPECIFIED`
    - `BLOCK_LOW_AND_ABOVE`
    - `BLOCK_MEDIUM_AND_ABOVE`
    - `BLOCK_ONLY_HIGH`
    - `BLOCK_NONE`

You can use Google Generative AI language models to generate text with the `generateText` function:

```ts
import { google } from '@ai-sdk/google';
import { generateText } from 'ai';

const { text } = await generateText({
  model: google('gemini-1.5-pro-latest'),
  prompt: 'Write a vegetarian lasagna recipe for 4 people.',
});
```

Google Generative AI language models can also be used in the `streamText`, `generateObject`, `streamObject`, and `streamUI` functions
(see [AI SDK Core](mdc:docs/ai-sdk-core) and [AI SDK RSC](mdc:docs/ai-sdk-rsc)).

### File Inputs

The Google Generative AI provider supports file inputs, e.g. PDF files.

```ts
import { google } from '@ai-sdk/google';
import { generateText } from 'ai';

const result = await generateText({
  model: google('gemini-1.5-flash'),
  messages: [
    {
      role: 'user',
      content: [
        {
          type: 'text',
          text: 'What is an embedding model according to this document?',
        },
        {
          type: 'file',
          data: fs.readFileSync('./data/ai.pdf'),
          mimeType: 'application/pdf',
        },
      ],
    },
  ],
});
```

<Note>
  The AI SDK will automatically download URLs if you pass them as data, except
  for `https://generativelanguage.googleapis.com/v1beta/files/`. You can use the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VidAIze/cursor-project-rules](https://github.com/VidAIze/cursor-project-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
