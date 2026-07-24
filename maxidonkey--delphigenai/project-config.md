---
trigger: always_on
description: - [Client initialization](#client-initialization)
---

# Google Gemini compatibility

- [Client initialization](#client-initialization)
- [Text generation](#text-generation)
- [Thinking](#thinking)
- [Function calling](#function-calling)
- [Vision](#vision)
- [Generate an image](#generate-an-image)
- [Audio understanding](#audio-understanding)
- [Embeddings](#embeddings)
- [List models](#list-models)
- [Retrieve a model](#retrieve-a-model)
- [Notebook Colab](#notebook-colab)
___

## Client initialization

To initialize a client compatible with Gemini models, first [obtain an API key](https://aistudio.google.com/api-keys?hl=fr) from Google. Then create the `IGenAI` client instance as shown below:

```pascal
//uses GenAI, GenAI.Types;

  Client := TGenAIFactory.CreateGeminiInstance(GeminiKey);
```

>[!NOTE]
> A single application may instantiate multiple clients simultaneously:
> - one client to access OpenAI Cloud APIs
> - another client for Gemini Cloud APIs
> - and a third client for locally hosted models via LM Studio.

<br>

___

## Text generation

- [Non-streamed](#non-streamed)
- [Streamed](#streamed)

<br>

When targeting **Gemini models**, text generation is supported **exclusively** via the `/v1/chat/completions` endpoint.

The legacy `/v1/completions` endpoint and the agentic `/v1/responses` API are **not supported** for Gemini models.

To use this endpoint, please refer to the internal documentation available at the following link:
- [Chat/completions](ChatCompletion.md#text-generation) 

### Non-streamed

```pascal
//uses GenAI, GenAI.Types, GenAI.Tutorial.VCL;

  TutorialHub.JSONRequestClear;

  //Asynchronous promise example
  Display(TutorialHub, 'This may take a few seconds.');
  var Promise := Client.Chat.AsyncAwaitCreate(
    procedure (Params: TChatParams)
    begin
      Params
        .Model('gemini-2.5-flash')
        .Messages([
          FromSystem('You are a comedian looking for jokes for your new show.'),
          FromUser('What is the difference between a mathematician and a physicist?')]);
      TutorialHub.JSONRequest := Params.ToFormat(); //to display JSON Request
    end);

  promise
    .&Then<string>(
      function (Value: TChat): string
      begin
        for var Item in Value.Choices do
          Result := Result + Item.Message.Content;
        Display(TutorialHub, Value);
        ShowMessage(Result);
      end)
    .&Catch(
      procedure (E: Exception)
      begin
        Display(TutorialHub, E.Message);
      end);

  //Synchronous example
//  var Value := Client.Chat.Create(
//    procedure (Params: TChatParams)
//    begin
//      Params
//        .Model('gemini-2.5-flash')
//        .Messages([
//          FromSystem('You are a comedian looking for jokes for your new show.'),
//          FromUser('What is the difference between a mathematician and a physicist?')]);
//      TutorialHub.JSONRequest := Params.ToFormat(); //to display JSON Request
//    end);
//  try
//    Display(TutorialHub, Value);
//  finally
//    Value.Free;
//  end;
```

Please note that Gemini APIs do not support the `store` property, which is specific to OpenAI models.
If this property is included in a request, the API will return a **400 Bad Request error** indicating incompatible parameters.

<br>

### Streamed

Please note that Server-Sent Events (SSE) chunk generation differs significantly between Gemini and OpenAI models.
- OpenAI models emit one chunk per generated token.
- Gemini models emit a text segment per chunk, rather than token-level chunks.

As a result, fewer chunks are received when using a Gemini model compared to an OpenAI model.

Gemini models support including token usage information within SSE chunks, in the same way as OpenAI models.
This can be enabled using the following option: `StreamOptions(TIncludeUsage.New);`

However, Gemini models do not support obfuscation.
Attempting to enable this feature using: `StreamOptions(TIncludeObfuscation.New(False));` will result in a 400 Bad Request error.

```pascal
//uses GenAI, GenAI.Types, GenAI.Tutorial.VCL;

  TutorialHub.JSONRequestClear;

  //Asynchronous promise example
  var Promise := Client.Chat.AsyncAwaitCreateStream(
    procedure(Params: TChatParams)
    begin
      Params
        .Model('gemini-2.5-flash')
        .Messages([
          FromSystem('You are a comedian looking for jokes for your new show.'),
          FromUser('What is the difference between a mathematician and a physicist?')])
        .StreamOptions(TIncludeUsage.New)
        .Stream;
      TutorialHub.JSONRequest := Params.ToFormat();
    end,
    function : TPromiseChatStream
    begin
      Result.Sender := TutorialHub;
      Result.OnStart := Start;

      Result.OnProgress :=
        procedure (Sender: TObject; Chunk: TChat)
        begin
          DisplayStream(Sender, Chunk);
        end;

      Result.OnDoCancel := DoCancellation;

      Result.OnCancellation :=
        function (Sender: TObject): string
        begin
          Cancellation(Sender);
        end
    end);

  Promise
    .&Then<string>(
      function (Value: string): string
      begin
        Result := Value;
        ShowMessage(Result);
      end)
    .&Catch(
      procedure (E: Exception)
      begin
        Display(TutorialHub, E.Message);
      end);

  //Synchronous example
//  Client.Chat.CreateStream(

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MaxiDonkey/DelphiGenAI](https://github.com/MaxiDonkey/DelphiGenAI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
