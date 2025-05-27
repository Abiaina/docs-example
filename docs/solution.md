---
title: Solution
---

# Cadence Food Delivery Workflow Solution Walkthrough

## Architecture Overview

The solution implements a food delivery system using Cadence workflows, with a parent-child workflow pattern.

Here's how it works:

```plaintext
+------------------------+     +------------------------+
|   Main Workflow        |     |   Child Workflow       |
|   (HandleEatsOrder)    |     |   (DeliverOrder)       |
+------------------------+     +------------------------+
| - Order Reception      |     | - Delivery Simulation  |
| - Restaurant Decision  |     | - Delivery Notification|
| - Food Preparation     |     | - Delivery Confirmation|
+------------------------+     +------------------------+
           |                              ^
           |                              |
           +------------------------------+
```

## Workflow Components

### 1. Main Workflow (`HandleEatsOrderWorkflow`)

The main workflow handles the entire order lifecycle:

```java
public class HandleEatsOrderWorkflowImpl implements HandleEatsOrderWorkflow {
    // Activity stub for order processing
    private final EatsActivities activities = Workflow.newActivityStub(
        EatsActivities.class,
        new ActivityOptions.Builder()
            .setScheduleToCloseTimeout(Duration.ofMinutes(1))
            .setStartToCloseTimeout(Duration.ofSeconds(30))
            .build()
    );

    @Override
    public String handleOrder(String userId, Order order, String restaurantId) {
        // 1. Order Reception
        activities.processOrder("Your order received!");

        // 2. Wait for Restaurant Decision
        Workflow.await(Duration.ofSeconds(60), () -> signalPromise.isCompleted());

        // 3. If Accepted, Prepare Food
        if (restaurantDecision) {
            activities.prepareFood("Preparing order...");
            Workflow.sleep(Duration.ofSeconds(3));

            // 4. Start Delivery Child Workflow
            DeliverOrderWorkflow deliveryWorkflow = Workflow.newChildWorkflowStub(
                DeliverOrderWorkflow.class,
                childOptions
            );
            Promise<String> deliveryPromise = Async.function(
                deliveryWorkflow::deliverOrder,
                order.getId()
            );
            return deliveryPromise.get();
        }
        return "Order rejected";
    }
}
```

### 2. Child Workflow (`DeliverOrderWorkflow`)

The child workflow handles the delivery process:

```java
public class DeliverOrderWorkflowImpl implements DeliverOrderWorkflow {
    private final EatsActivities activities = Workflow.newActivityStub(
        EatsActivities.class,
        new ActivityOptions.Builder()
            .setScheduleToCloseTimeout(Duration.ofMinutes(2))
            .setStartToCloseTimeout(Duration.ofSeconds(30))
            .build()
    );

    @Override
    public String deliverOrder(String orderId) {
        // 1. Simulate Delivery
        Workflow.sleep(Duration.ofSeconds(4));

        // 2. Notify Delivery
        activities.notifyOrderDelivered(orderId);
        activities.printDeliveryConfirmation(orderId);

        return "Order " + orderId + " delivered successfully!";
    }
}
```

## Why Two Workflows?

The solution uses two separate workflows for several reasons:

1. **Task List Separation**:

   ```java
   private static final String MAIN_TASK_LIST = "HandleEatsOrderTaskList";
   private static final String DELIVERY_TASK_LIST = "DeliverOrderTaskList";
   ```

   - Main workflow runs on `HandleEatsOrderTaskList`
   - Delivery workflow runs on `DeliverOrderTaskList`

2. **Worker Registration**:

   ```java
   // Main worker for order processing
   Worker mainWorker = factory.newWorker(MAIN_TASK_LIST);
   mainWorker.registerWorkflowImplementationTypes(HandleEatsOrderWorkflowImpl.class);

   // Delivery worker for delivery processing
   Worker deliveryWorker = factory.newWorker(DELIVERY_TASK_LIST);
   deliveryWorker.registerWorkflowImplementationTypes(DeliverOrderWorkflowImpl.class);
   ```

3. **Workflow IDs**:
   ```java
   ChildWorkflowOptions childOptions = new ChildWorkflowOptions.Builder()
       .setTaskList("DeliverOrderTaskList")
       .setWorkflowId("deliver-order-" + order.getId())
       .build();
   ```

## Benefits of This Architecture

1. **Scalability**

   - Delivery can be handled by separate workers
   - Can scale delivery workers independently

2. **Fault Isolation**

   - If delivery fails, main workflow can handle the error
   - Each workflow has its own retry policies

3. **Parallel Processing**

   - Multiple deliveries can happen simultaneously
   - Main workflow can continue processing other orders

4. **Resource Management**
   - Different timeouts for each part
   - Separate retry policies
   - Independent scaling

## Workflow States

1. **Main Workflow States**:

   - Order Received
   - Waiting for Restaurant
   - Preparing Order
   - Delivery Started
   - Completed/Rejected

2. **Child Workflow States**:
   - Delivery Started
   - Delivery in Progress
   - Delivery Completed

## Timeout and Retry Configuration

1. **Main Workflow**:

   ```java
   @WorkflowMethod(
       executionStartToCloseTimeoutSeconds = 600, // 10 minutes
       taskStartToCloseTimeoutSeconds = 60 // 1 minute
   )
   @MethodRetry(
       initialIntervalSeconds = 1,
       maximumIntervalSeconds = 3,
       maximumAttempts = 2
   )
   ```

2. **Child Workflow**:

   ```java
   @WorkflowMethod(
       executionStartToCloseTimeoutSeconds = 600,
       taskStartToCloseTimeoutSeconds = 60
   )
   @MethodRetry(
       initialIntervalSeconds = 1,
       maximumIntervalSeconds = 3,
       maximumAttempts = 2
   )
   ```

3. **Activities**:
   ```java
   new ActivityOptions.Builder()
       .setScheduleToCloseTimeout(Duration.ofMinutes(2))
       .setStartToCloseTimeout(Duration.ofSeconds(30))
       .setScheduleToStartTimeout(Duration.ofSeconds(10))
       .build()
   ```

## Error Handling

1. **Main Workflow**:

   - Validates order inputs
   - Handles restaurant decision timeout
   - Manages child workflow failures

2. **Child Workflow**:
   - Handles delivery failures
   - Manages activity timeouts
   - Returns delivery status

## Best Practices Implemented

1. **Workflow Design**:

   - Clear separation of concerns
   - Proper error handling
   - Appropriate timeouts

2. **Activity Implementation**:

   - Idempotent operations
   - Proper logging
   - Error handling

3. **Signal Handling**:
   - Timeout management
   - Signal validation
   - State management

## Monitoring and Debugging

In the Cadence UI (http://localhost:8088), you'll see:

1. Main workflow execution
2. Child workflow execution
3. Activity executions
4. Signal events
5. Error events
