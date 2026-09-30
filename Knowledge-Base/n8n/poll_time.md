# n8n Polling Triggers — What Poll Time Actually Means

## What is Poll Time?

Some n8n trigger nodes use **polling** instead of receiving events instantly from an external service.

When a trigger is configured with a poll time such as **Every Minute**, n8n periodically checks the external service to see whether a relevant change has occurred.

For example:

```text
Poll Time: Every 1 minute

10:00 → Check Notion
10:01 → Check Notion
10:02 → Check Notion
10:03 → Check Notion
...
```

The important point is that **checking every minute does not mean the entire workflow executes every minute**.

## What happens when there is no change?

If n8n checks Notion and doesn't find a qualifying change:

```text
Every minute
     ↓
Check Notion
     ↓
Change detected?
   /       \
 No        Yes
 ↓          ↓
Stop      Start workflow
```

When there is no relevant change, the trigger produces no new event, so the downstream workflow nodes don't execute.

## What happens when a change is detected?

If the polling check detects a qualifying change, the trigger emits the corresponding data and the workflow execution starts:

```text
n8n polling
     ↓
Check Notion
     ↓
New/changed data detected
     ↓
Trigger fires
     ↓
Workflow execution starts
     ↓
Next nodes execute
```

For example, in a Notion → AI → Ghost blogging workflow:

```text
Notion Trigger
      │
      │ Every 1 minute: check
      ↓
New qualifying page?
      │
     YES
      ↓
Read page content
      ↓
Generate blog with AI
      ↓
Generate thumbnail
      ↓
Publish to Ghost
```

If nothing relevant changed in Notion, the AI generation and Ghost publishing steps don't run.

## Is polling the same as a cron job?

Polling and cron/scheduled execution are related, but they have different purposes.

### Schedule/Cron

A schedule trigger says:

> "Run the workflow at this time."

Example:

```text
Every 5 minutes
      ↓
Start workflow
```

It doesn't matter whether anything changed externally.

### Polling

A polling trigger says:

> "Check the external service at this interval and run the workflow only if something relevant is detected."

Example:

```text
Every 5 minutes
      ↓
Check API
      ↓
Change detected?
   /       \
 No        Yes
 ↓          ↓
Nothing    Run workflow
```

So the key distinction is:

**Schedule/Cron → time causes execution.**

**Polling → time causes a check; a detected change causes execution.**

## Why use polling?

Polling is useful when an external service doesn't provide a webhook/event mechanism for the particular event you need.

The trade-off is that polling introduces a delay.

For example, with a one-minute polling interval:

```text
Change happens at 10:00:20
        ↓
Next check at 10:01:00
        ↓
Workflow can start
```

The workflow therefore doesn't necessarily start at the exact moment the change occurs.

A shorter polling interval can reduce this delay, but it also means n8n checks the external service more frequently.

## Mental Model

A useful way to remember it:

> **Polling interval = how often n8n asks "Did anything happen?"**

It is **not**:

> "Run my entire workflow every minute."

For a polling trigger, think:

```text
Timer
  ↓
Check external service
  ↓
┌───────────────┐
│ Change?       │
└───────────────┘
   ↓        ↓
  No       Yes
   ↓        ↓
 Wait     Trigger
            ↓
      Execute workflow
```

This distinction is especially important when working with APIs, Notion, email services, databases, and other external systems.
