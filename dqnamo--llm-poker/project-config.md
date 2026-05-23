---
trigger: always_on
description: Manages failed workflow runs with options to resume, restart, or retry failure callbacks.
---


# Upstash Workflow

Upstash Workflow is a serverless orchestration framework that enables durable, reliable, and performant long-running functions without infrastructure management. Built on QStash, it transforms serverless functions into resilient workflows by breaking complex logic into individual steps, each with automatic retries, state persistence, and independent execution timeouts. This step-based architecture eliminates traditional serverless limitations like function timeouts and transient failures.

The framework provides delivery guarantees with at-least-once execution semantics, automatic failure recovery through a Dead Letter Queue, and real-time observability. Workflows can sleep for days or months, wait for external events, make HTTP calls lasting up to 2 hours without consuming function execution time, run steps in parallel, and handle scheduled recurring tasks. Available for Next.js, Cloudflare Workers, FastAPI, and other platforms in both TypeScript and Python.

## Core Workflow Methods

### Define a workflow endpoint with serve

Creates a workflow endpoint that orchestrates multi-step serverless functions with automatic state management and retry logic.

```typescript
// api/workflow/route.ts
import { serve } from "@upstash/workflow/nextjs";

interface UserData {
  userId: string;
  email: string;
  name: string;
}

export const { POST } = serve<UserData>(
  async (context) => {
    const { userId, email, name } = context.requestPayload;

    const user = await context.run("register-user", async () => {
      const newUser = await db.users.create({ userId, email, name });
      console.log(`Registered user: ${userId}`);
      return newUser;
    });

    await context.sleep("wait-for-3-days", 60 * 60 * 24 * 3);

    const message = await context.call("generate-welcome-message", {
      url: "https://api.openai.com/v1/chat/completions",
      method: "POST",
      headers: { authorization: `Bearer ${process.env.OPENAI_API_KEY}` },
      body: {
        model: "gpt-4o",
        messages: [
          {
            role: "system",
            content: "Generate personalized welcome messages.",
          },
          { role: "user", content: `Welcome message for ${name}` },
        ],
      },
      retries: 3,
    });

    await context.run("send-welcome-email", async () => {
      await sendEmail(email, message.body.choices[0].message.content);
    });

    return { success: true, userId: user.id };
  },
  {
    failureFunction: async ({ context, failStatus, failResponse }) => {
      console.error(`Workflow ${context.workflowRunId} failed:`, failResponse);
      await notifyAdmin(
        `User onboarding failed for ${context.requestPayload.email}`
      );
      return `Failed with status ${failStatus}`;
    },
  }
);
```

```python
# main.py
from fastapi import FastAPI
from upstash_workflow.fastapi import Serve
from upstash_workflow import AsyncWorkflowContext
from typing import TypedDict

app = FastAPI()
serve = Serve(app)

class UserData(TypedDict):
    user_id: str
    email: str
    name: str

async def failure_handler(context, fail_status, fail_response, fail_headers):
    print(f"Workflow {context.workflow_run_id} failed: {fail_response}")
    await notify_admin(f"Onboarding failed for {context.request_payload['email']}")

@serve.post("/api/onboarding", failure_function=failure_handler)
async def onboarding_workflow(context: AsyncWorkflowContext[UserData]) -> dict:
    data = context.request_payload

    async def _register_user():
        user = await db.users.create(data)
        print(f"Registered user: {data['user_id']}")
        return user

    user = await context.run("register-user", _register_user)

    await context.sleep("wait-for-3-days", 60 * 60 * 24 * 3)

    message = await context.call(
        "generate-welcome-message",
        url="https://api.openai.com/v1/chat/completions",
        method="POST",
        headers={"authorization": f"Bearer {os.environ['OPENAI_API_KEY']}"},
        body={
            "model": "gpt-4o",
            "messages": [
                {"role": "system", "content": "Generate personalized welcome messages."},
                {"role": "user", "content": f"Welcome message for {data['name']}"}
            ]
        },
        retries=3
    )

    async def _send_email():
        await send_email(data["email"], message.body["choices"][0]["message"]["content"])

    await context.run("send-welcome-email", _send_email)

    return {"success": True, "user_id": user["id"]}
```

### Execute workflow steps with context.run

Executes a function as an independent workflow step with automatic retries and state persistence.

```typescript
import { serve } from "@upstash/workflow/nextjs";

export const { POST } = serve<{ orderId: string }>(async (context) => {
  const { orderId } = context.requestPayload;

  // Serial execution
  const inventory = await context.run("check-inventory", async () => {
    const stock = await db.inventory.findOne({ orderId });
    if (!stock.available) throw new Error("Out of stock");
    return stock;
  });

  const payment = await context.run("process-payment", async () => {
    const result = await stripe.charges.create({
      amount: inventory.price,
      currency: "usd",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dqnamo/llm-poker](https://github.com/dqnamo/llm-poker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
