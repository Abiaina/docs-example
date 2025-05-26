---
title: Cadence Uber Eats Example Readme
---

This would be your project overview, similar to your GitHub README.

# Cadence Food Delivery Workflow System

This project implements a food delivery workflow system using Uber's Cadence workflow engine. It demonstrates how to handle food orders, preparation, and delivery using Cadence's workflow and activity patterns.

## Workflow Steps

The workflow follows these specific steps:

1. Order Reception:

   - Activity prints "Your order received!" message with order details
   - System waits for restaurant decision

2. Restaurant Decision:

   - Workflow waits for a signal from the restaurant to accept/reject the order
   - If rejected, workflow ends
   - If accepted, proceeds to preparation

3. Food Preparation:

   - Simulates food preparation with a 3-second sleep
   - Prints preparation status

4. Delivery Process (Child Workflow):
   - Separate child workflow handles delivery
   - Takes order ID as input
   - Simulates delivery time with 4-second sleep
   - Prints "Order <id> delivered!" message
   - Confirms delivery with "Your order is in front of your door!" message

## Prerequisites

- Docker and Docker Compose installed
- [Cadence cli](https://cadenceworkflow.io/docs/cli)

## Setup and Running

### 1. Start Cadence Service

- In a separate terminal, clone Cadence repo to get a local version of Cadence service running on your machine.
- Start Cadence via docker-compose.

```bash
git clone git@github.com:cadence-workflow/cadence.git
cd cadence
docker-compose -f docker/docker-compose.yml up
```

- Register your domain using Cadence cli. Run `cadence --domain samples-domain domain register`. This register is used in the sample code, you will need to update this in the codebase if you use a different domain name.

The Web UI will be available at: http://localhost:8088

### 2. Start your local Worker

1. In a separate terminal, start the worker using this code base. Always use `--no-cache` to ensure latest changes are applied:

```bash
docker-compose build --no-cache worker && docker-compose up -d worker
```

### 3. Process Sample Orders

You can generate sample orders(sample_orders.csv) using out sample data and containerized helper script, or manually by starting an order.

Generating an order while the Cadence service and workers are running will:

1. Read orders from the CSV (or pull from manual input)
1. Start a workflow for each order
1. Simulate restaurant acceptance
1. Monitor order progress

#### Sample Orders with Helper Script

1. Open a separate terminal at same root directory as this code base.

1. Run docker-compose to start helper script and see the orders logged in the terminal. Always use `--no-cache` to ensure latest changes are applied:

```bash
# Using the included sample data
docker-compose build --no-cache processor && docker-compose run processor sample_orders.csv

# Using your own CSV file (must be in the project directory)
docker-compose build --no-cache processor && docker-compose run processor your_orders.csv
```

#### Manual Order Processing

If you prefer to process orders manually:

#### Start an Order

```bash
docker run --rm \
    --network=host \
    ubercadence/cli:master \
    --address localhost:7933 \
    --do samples-domain \
    workflow start \
    --tasklist HandleEatsOrderTaskList \
    --workflow_type HandleEatsOrderWorkflow::handleOrder \
    --execution_timeout 600 \
    --input '"order123"'
```

You will see the WORKFLOW_ID print out to the terminal. Copy this to input in the next command.

#### Send Restaurant Decision

Send restaurant acceptance signal (replace WORKFLOW_ID with yours):

```bash
docker run --rm \
    --network=host \
    ubercadence/cli:master \
    --address localhost:7933 \
    --do samples-domain \
    workflow signal \
    --workflow_id WORKFLOW_ID \
    --name signalRestaurantDecision \
    --input true
```

## Workflow States

The order goes through the following states:

1. Order Received - Initial state when order is submitted
2. Waiting for Restaurant - Waiting for accept/reject signal
3. Preparing Order - If accepted, sleeps for 3 seconds
4. Delivering Order - Child workflow starts, sleeps for 4 seconds
5. Delivered - Final state if successful
6. Rejected - Final state if restaurant rejects

## Timeout and Retry Configuration

The system uses the following timeout and retry configurations:

### Main Workflow

- Execution timeout: 10 minutes
- Task timeout: 1 minute
- Retry: initial=1s, max=3s, attempts=2

### Child Workflow (Delivery)

- Execution timeout: 10 minutes
- Task timeout: 1 minute
- Retry: initial=1s, max=3s, attempts=2

### Activities

- Schedule-to-close: 2 minutes
- Start-to-close: 30 seconds
- Schedule-to-start: 10 seconds
- Retry: initial=1s, max=3s, attempts=2

## Error Handling

The system handles various error scenarios:

- Invalid order items (null or empty)
- Missing or invalid IDs (auto-generates if not provided)
- Workflow timeouts (configured for 10 minutes)
- Activity timeouts (configured for 2 minutes)
- Delivery failures
- Signal timeouts

## Development

### Project Structure

```
.
├── Dockerfile           # Main worker Dockerfile
├── Dockerfile.processor # Order processor Dockerfile
├── docker-compose.yml  # Docker Compose configuration
├── process_orders.sh   # Order processing script
├── sample_orders.csv   # Sample order data
└── src/               # Java source files
```

### Adding Custom Orders

To process your own orders:

1. Create a CSV file following the format in `sample_orders.csv`
2. Place it in the project directory
3. Run the processor with your file name:
   ```bash
   docker-compose build --no-cache processor && docker-compose run processor your_orders.csv
   ```

## Customization

You can customize:

- Order items (any food/drink items)
- User ID (your own ID or auto-generated)
- Restaurant ID (your own ID or auto-generated)
- Order ID (your own ID or auto-generated)

## FAQs

**Q: Why is my worker running but not picking up workflows?**
A: Ensure the worker is connected to the correct Cadence service (host: 127.0.0.1, port: 7933), and that the workflow is started with the correct domain (`samples-domain`) and task list (`HandleEatsOrderTaskList`). Check the worker logs for any exceptions or validation errors.

**Q: Why do I see `Error: Signal workflow failed. Error details: workflow execution already completed`?**
A: This means the workflow either finished too quickly or was never started. Make sure the workflow is started with the correct parameters and task list. Check the worker logs for any exceptions or validation errors.

**Q: Why is there nothing in the logs?**
A: Double-check that the worker is running and connected to the correct Cadence service. Make sure logging is set to DEBUG in `logback.xml`. Also, ensure you are starting the workflow with the correct domain and task list.

**Q: What should I do if I encounter build errors about missing methods like `setMaxConcurrentActivityTaskPollSize`?**
A: These methods may not exist in your version of the Cadence Java SDK. Remove or comment out those lines and use only the supported concurrency settings.
