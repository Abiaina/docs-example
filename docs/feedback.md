---
title: Selam's Feedback
---

A place for user or contributor feedback and suggestions.

# Building Your First Cadence Workflow: From Local Docker to Cadence Web

> A beginner-friendly, hands-on guide for running your own Cadence workflow locally, complete with working code, gotchas, and tips to avoid common frustrations.

---

## TL;DR

This article shows you how to:

- Run a working Java Cadence workflow that simulates a food order
- Use Docker + Cadence Web to inspect your workflow
- Handle signals, timeouts, and child workflows
- Learn how Cadence works — in a real way

With some code examples, gotchas, and things I wish I'd known.

---

## Demo First

Here's what we built: a food delivery workflow.

1. You run the workflow.
2. The workflow simulates a restaurant accepting an order (via signal).
3. It waits 3 seconds.
4. It spawns a child workflow for delivery.
5. That workflow waits 4 seconds and then logs the delivery.

You can see this entire process in **Cadence Web**, running via Docker.

![](https://your-gif-or-screenshot-url.gif)

---

## What You'll Learn

- What is Cadence?
- Key concepts: Workflows, Activities, Workers, Signals, Child Workflows
- How to run Cadence with Docker
- How to get logs, debug timeouts, and see results in Cadence Web
- Where you might get stuck — and how to get unstuck

---

## Architecture Snapshot

```plaintext
+--------------+       +--------------+       +-----------------+
|  Client CLI  |-----> |   Cadence    | <---- |    Worker App   |
|   (API)      |       |  (via Docker)|       | (Java Workflow) |
+--------------+       +--------------+       +-----------------+
                                 |
                                 v
                         Cadence Web UI
```

- You start a workflow using an API or CLI
- Cadence queues it up
- A worker (your app) picks it up, runs your workflow logic
- You can inspect it all in Cadence Web

---

## Code Overview

### `WorkerStarter.java`

This is where the magic begins — we connect to Cadence and start polling for tasks.

```java
WorkflowClientOptions options = WorkflowClientOptions.newBuilder()...build();
```

Sets up options for the Cadence client, like timeouts and metrics.

```java
WorkflowClient workflowClient = WorkflowClient.newInstance(service, domain);
```

Your connection to Cadence. Lets you start workflows, signal them, and fetch results.

```java
WorkerFactory factory = WorkerFactory.newInstance(workflowClient);
Worker worker = factory.newWorker(TASK_QUEUE);
```

Spins up a background job (worker) that polls for tasks — think of this as your "order fulfillment team."

```java
worker.registerWorkflowImplementationTypes(OrderWorkflowImpl.class);
worker.registerActivitiesImplementations(new ActivitiesImpl());
factory.start();
```

**This connects the dots**: Tells Cadence, "When a task hits this queue, use this workflow and these activities to do the work."

---

### `OrderWorkflowImpl.java`

This is your main workflow — the order lifecycle.

```java
workflow.await(() -> accepted);
```

Pause the workflow and wait for a signal like "restaurant accepted the order."

```java
Workflow.sleep(Duration.ofSeconds(3));
```

**Simulating time**: Models cooking/preparation time.

```java
WorkflowClient.start(DeliverOrderWorkflow::deliver, orderId);
```

**Calling the child**: Fires off the delivery workflow, like telling a driver to pick up the food.

---

### `DeliverOrderWorkflowImpl.java`

Handles the delivery part of the order.

```java
Workflow.sleep(Duration.ofSeconds(4));
activities.orderDelivered(orderId);
```

**What's happening**: Simulates travel time, then confirms delivery.

---

### `ActivitiesImpl.java`

The actual side-effects like logging.

```java
public void orderReceived(Order order) { ... }
public void orderDelivered(String orderId) { ... }
```

**Plain and simple**: Could call an API, push a message, or log.

---

## How to Run It

1. Clone the repo
2. Run `docker-compose up`
3. Start the worker with `WorkerStarter`
4. Start a workflow using the `ClientStarter` (or API)
5. Signal acceptance using API or `ClientStarter`
6. Watch it in Cadence Web at `localhost:8088`

---

## Gotchas

- **Workflow timeout**: Cadence uses timeouts everywhere. Set generous timeouts when learning.

- **Logging**: If logs don't show up, check where your activity is running. Log from the activity itself.
- **Signal delay**: Cadence doesn't "check for signals" actively — signals are picked up when a workflow yields.
- **No worker, no progress**: You need your worker to be actively polling or nothing will happen.
- **Multiple docs, scattered info**: It took time to unify Java docs + Docker instructions + examples.
- **Docker networks**: Make sure ports and services are reachable on your machine.

---

## FAQs

**Q: How do I know my workflow ran?**

A: Check the logs in your IDE and go to Cadence Web. You should see the workflow and its status.

**Q: Why isn't my signal working?**

A: You must signal the correct workflow ID **and** your workflow needs to be in a state where it's awaiting that signal.

**Q: Why does nothing happen after I start a workflow?**

A: Is your worker running? It has to be polling the task queue to pick up the job.

**Q: Can I run multiple workflows?**

A: Yes! Just start more with different IDs.

**Q: What's the difference between a workflow and an activity?**

A: Workflows = state + logic. Activities = external calls like logging, APIs, etc. Activities run outside the workflow sandbox.

---

## Troubleshooting FAQ

**Q: My worker is running but not picking up workflows. What should I check?**

- Ensure the worker is connected to the correct Cadence service (host: 127.0.0.1, port: 7933).
- Make sure the workflow is started with the correct domain (`samples-domain`) and task list (`HandleEatsOrderTaskList`).
- Check the worker logs for any exceptions or validation errors.

**Q: I see `Error: Signal workflow failed. Error details: workflow execution already completed`. What does this mean?**

- The workflow either finished too quickly or was never started. Double-check your workflow start parameters and task list.
- Review the worker logs for any exceptions or validation errors.

**Q: There is nothing in the logs. How do I fix this?**

- Confirm the worker is running and connected to the correct Cadence service.
- Ensure logging is set to DEBUG in `logback.xml`.
- Make sure you are starting the workflow with the correct domain and task list.

**Q: I get build errors about missing methods like `setMaxConcurrentActivityTaskPollSize`.**

- These methods may not exist in your version of the Cadence Java SDK. Remove or comment out those lines and use only the supported concurrency settings.

---

## My Experience Learning Cadence

### Feedback

### 1. Technical Requirements Analysis

- Broken down workflow structure, checked activity status
- Highlighted gaps in versioning, parameter handling, etc.

### 2. Technical Improvement Suggestions

- Utility functions for UUID
- Better error handling and timeouts
- Add more structured testing

### 3. Process Improvements

- Improve local dev setup docs
- Use fewer scattered tutorials; more complete examples
- Clear path to start: `docker-compose → worker → client`

### 4. Future Enhancements

- Add retry logic and exponential backoff
- Add metrics and monitoring to workers
- Explore multi-language SDKs (Go, TypeScript)

### Reflections

- I struggled early on understanding what a "workflow" actually _is_ in practice.
- Needed better visual examples — like diagrams showing queues and signals.
- The split across multiple docs made it hard to get started.
- Versioning in Docker and getting Cadence Web running was more painful than expected.
- The intuition of "create task → queue → worker picks it up" was easy to grasp conceptually but tricky in code.

Cadence can be overwhelming at first, but once you wire up a simple workflow and see it run in Cadence Web, things click.

I hope this walkthrough helps demystify the process and gives you a solid base for building your own resilient systems.

Want the code? Ping me. Want a visual explainer? I'll make one.

# Feedback and Improvements

## Latest Changes

1. **Timeout and Retry Configuration**

   - Standardized timeout and retry settings across all workflows and activities
   - Main workflow: 10min execution, 1min task, 2 retries
   - Child workflow: 10min execution, 1min task, 2 retries
   - Activities: 2min schedule-to-close, 30s start-to-close, 10s schedule-to-start, 2 retries

2. **Build Process**

   - Added `--no-cache` flag to all build commands to ensure latest changes are applied
   - Updated README with new build instructions
   - Improved build reliability and consistency

3. **Child Workflow Handling**

   - Improved child workflow result handling using `Async.function()`
   - Added proper Promise handling for child workflow results
   - Enhanced error handling and logging

4. **Message Handling**
   - Removed duplicate "front door" messages
   - Standardized message format across workflows
   - Improved message clarity and consistency

## Previous Improvements

1. **Order Processing**

   - Added validation for order items
   - Improved error handling for invalid orders
   - Enhanced logging for order processing steps

2. **Workflow Structure**

   - Separated delivery into child workflow
   - Added proper workflow ID handling
   - Improved workflow state management

3. **Activity Implementation**

   - Enhanced activity error handling
   - Added detailed logging
   - Improved activity timeout handling

4. **Signal Handling**
   - Added proper signal timeout handling
   - Improved signal validation
   - Enhanced signal error handling

## Known Issues

1. **Build Process**

   - Docker Compose version warning (obsolete version attribute)
   - Need to use `--no-cache` for reliable builds

2. **Workflow Execution**

   - Child workflow deserialization issues with void return type
   - Need to use String return type for child workflows

3. **Logging**
   - Some duplicate log messages
   - Need to standardize log levels

## Future Improvements

1. **Build Process**

   - Update Docker Compose configuration
   - Add build caching strategy
   - Improve build performance

2. **Workflow Execution**

   - Add workflow versioning
   - Implement workflow migration
   - Add workflow metrics

3. **Logging**

   - Implement structured logging
   - Add log aggregation
   - Improve log filtering

4. **Testing**
   - Add unit tests
   - Add integration tests
   - Add performance tests

## Best Practices

1. **Build Process**

   - Always use `--no-cache` for builds
   - Rebuild both worker and processor when making changes
   - Check build logs for errors

2. **Workflow Development**

   - Use proper timeout and retry configurations
   - Handle child workflow results properly
   - Implement proper error handling

3. **Activity Development**

   - Use appropriate timeout settings
   - Implement proper retry logic
   - Add detailed logging

4. **Signal Handling**
   - Use proper signal timeouts
   - Validate signal data
   - Handle signal errors properly
