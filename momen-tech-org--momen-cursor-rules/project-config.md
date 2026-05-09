---
trigger: always_on
description: How to interact with Momen.app's AI Agent
---


# Momen.app's AI agents

## Overview
Momen.app has an integrated AI agent builder, which supports multi-modal (text, video, image) inputs and outputs, prompt templating, context fetching (via database and third-party APIs), tool use (actionflows, third-party APIs and other AI agents) and structured output (JSON according corresponding JSONSchema).  
AI Agents' results are delivered differently by the GraphQL service depending on the configuration of its output, namely, whether it is streaming and whether it is structured. A structured output can not be streamed but plain text can be either streamed or not. A structured output must be accompanied by a JSONSchema that describes the JSON's type.  
In order to invoke an AI agent, the id and the input arguments must be obtained from the project schema. An AI agent built in Momen.app's agent builder can only be invoked via the GraphQL API asynchronously. 


## Invocation process for streaming output
An example AI Agent configuration whose output is a streaming plain text will be used to illustrate this process. Its configuration is: 
```json
{
    "id": "mgzzu8jp",
    "summary": "An example summary of what the agent does",
    "inputs": {
      "mgzzufo2": {
        "type": "VIDEO",
        "displayName": "the_video",
      },
      "mh4cjjcf": {
        "type": "TEXT",
        "displayName": "text",
      },
      "mh4cjkyv": {
        "type": "BIGINT",
        "displayName": "some_int",
      },
      "mh4cjoof": {
        "type": "array",
        "itemType": "IMAGE",
        "displayName": "images",
      }
    },
    "output": "Unstructured Text"
}
```
1.  A mutation is sent to start the AI agent, supplying the arguments as inputArgs and the id as zAIConfigId. The response value only contains the id of the corresponding conversation. The keys of inputArgs should be the same keys in the inputs object from the schema. Input parameters of Image / video or other binary assets types, or arrays of such types are handled slightly differently. Their key names wihtin the inputArgs object have `_id` suffix. e.g. the following configuration  
```json
{ 
  "inputs": {
    "mgzzufo2": {
      "type": "VIDEO",
      "displayName": "the_video",
    }
  }
}
```
Corresponds to:
```json
{
  "inputArgs": {
    "mgzzufo2_id": 1030000000000002,
  }
}
```  

    Mutation request:  
    Query:
    ```gql
    mutation ZAICreateConversation($inputArgs: Map_String_ObjectScalar!, $zaiConfigId: String!) {
     fz_zai_create_conversation(inputArgs: $inputArgs, zaiConfigId: $zaiConfigId)
    }
    ```
    Variables:
    ```json
    {
      "inputArgs": {
        "mgzzufo2_id": 1030000000000002,
        "mh4cjjcf": "Just some text",
        "mh4cjkyv": 23,
        "mh4cjoof_id": [
          1020000000000097,
          1020000000000111,
          1020000000000120
        ]
      },
      "zaiConfigId": "mgzzu8jp"
    }
    ```
    Mutation response:
    ```json
    {
      "data": {
        "fz_zai_create_conversation": 1480
      }
    }
    ```
2.  Using the obtained conversation id to subscribe to the result of the previous invocation of the AI Agent. Multiple messages may be received. The messages' status may transition from IN_PROGRESS to STREAMING to eventually COMPLETED. The last message always gives you COMPLETED status and its data field will contain the consolidated output from all the previous STEAMING messages' data field. 
For models that have reasoning content output, it works similarly as the actual output. i.e. Partial reasoning content will be emitted first in multiple messages in the reasoningContent field, and then when everything is ready, the entirety of reasoningContent will be emitted again the COMPLETED message. 
    Subscription request:  
    Query: 
    ```gql
    subscription ZaiListenConversationResult($conversationId: Long!) {
      fz_zai_listen_conversation_result(conversationId: $conversationId) {
        conversationId
        status
        reasoningContent
        images {
          id
          __typename
        }
        data
        __typename
      }
    }
    ```
    Variables: 
    ```json
    {
      "conversationId": 1480
    }
    ```
    Subscription response messages:
    ```json
    {
      "data": {
        "fz_zai_listen_conversation_result": {
          "__typename": "ConversationResult",
          "conversationId": 1480,
          "data": null,
          "images": null,
          "reasoningContent": null,
          "status": "IN_PROGRESS"
        }
      }
    }
    ```
    ```json
    {
      "data":{
        "fz_zai_listen_conversation_result": {
          "__typename": "ConversationResult",
          "conversationId": 1480,
          "data": "This collection features three images and a short video. Two photos show the famous Chinese comedian and actor, Zhao Benshan. A third",
          "images": null,
          "reasoningContent": null,
          "status": "STREAMING"
        }
      }
    }
    ```
    ```json
    {
      "data":{
        "fz_zai_listen_conversation_result": {
          "__typename": "ConversationResult",
          "conversationId": 1480,
          "data": " image is an anime illustration of a young woman in a \"SHOHOKU\" basketball jersey, resembling the character Haruko Akagi from the series *Slam Dunk*.",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [momen-tech-org/momen-cursor-rules](https://github.com/momen-tech-org/momen-cursor-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
