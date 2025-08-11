# AWS - Simple Query Service (SQS)
## Logs of SQS
## 1. Where SQS Logs Come From

Amazon SQS (Simple Queue Service) is a system for sending and receiving messages between applications, like passing notes in a busy office. It doesn’t create logs on its own, but two AWS services help track what’s happening:

- **AWS CloudTrail**: Like a security camera, it records actions on SQS queues (e.g., creating a queue or sending a message).
- **Amazon CloudWatch Logs**: Like a worker’s notebook, it stores details about what your app does with messages (e.g., processing an order).

Think of CloudTrail as tracking “who did what to the queue or messages” and CloudWatch Logs as tracking “what happened when we processed those messages.”

---

## 2. Management Events: Actions on the Queue Itself

**What They Are**: Management events log changes to the queue’s setup, like creating, deleting, or tweaking its settings. These are automatically recorded in CloudTrail.

**Purpose**: To check who’s making changes to queues for security or troubleshooting (e.g., “Who deleted our queue?”).

**Examples and Scenarios**:

### Example 1: CreateQueue (Making a New Queue)
- **Scenario**: Your e-commerce team needs a queue for customer orders. Sarah, a developer, creates a queue called `OrderQueue`.
- **What Happens**: Sarah uses the AWS Console to create `OrderQueue`.
- **What CloudTrail Logs**:
  ```json
  {
    "Time": "2025-08-11 10:00 AM",
    "Action": "CreateQueue",
    "Queue": "OrderQueue",
    "User": "Sarah",
    "Region": "us-east-1"
  }
  ```
- **Why It Helps**: If someone asks, “Who made this queue?” you can see Sarah created it. If the queue has wrong settings (e.g., no encryption), you can check its setup details.

### Example 2: DeleteQueue (Removing a Queue)
- **Scenario**: During cleanup, Mike accidentally deletes `OrderQueue`.
- **What Happens**: Mike clicks “Delete” in the AWS Console.
- **What CloudTrail Logs**:
  ```json
  {
    "Time": "2025-08-11 10:15 AM",
    "Action": "DeleteQueue",
    "Queue": "OrderQueue",
    "User": "Mike",
    "Region": "us-east-1"
  }
  ```
- **Why It Helps**: If orders stop processing, you can see Mike deleted the queue and ask him why or restore it.

### Example 3: SetQueueAttributes (Changing Queue Settings)
- **Scenario**: Sarah changes `OrderQueue` to keep messages for 7 days instead of 4.
- **What Happens**: Sarah updates the setting in the AWS Console.
- **What CloudTrail Logs**:
  ```json
  {
    "Time": "2025-08-11 10:20 AM",
    "Action": "SetQueueAttributes",
    "Queue": "OrderQueue",
    "User": "Sarah",
    "Change": "MessageRetentionPeriod = 7 days"
  }
  ```
- **Why It Helps**: If messages disappear too soon, you can check if someone changed the retention period.

### Example 4: PurgeQueue (Clearing All Messages)
- **Scenario**: During testing, Sarah clears all test messages from `OrderQueue`.
- **What Happens**: Sarah selects “Purge Queue” in the AWS Console.
- **What CloudTrail Logs**:
  ```json
  {
    "Time": "2025-08-11 10:25 AM",
    "Action": "PurgeQueue",
    "Queue": "OrderQueue",
    "User": "Sarah",
    "Region": "us-east-1"
  }
  ```
- **Why It Helps**: If important messages are gone, you can see if a purge caused it and ensure it was intentional.

**Key Point**: Management events are like a log of who touched the queue’s structure or settings. They’re always on in CloudTrail, so you can easily track changes.

---

## 3. Data Events: Actions on Messages

**What They Are**: Data events log what happens to messages in the queue, like sending or receiving them. These are **not** automatically logged—you must turn them on in CloudTrail.

**Purpose**: To track who sent or received messages, useful for auditing (e.g., “Did that order get sent?”) or debugging (e.g., “Why wasn’t this message processed?”).

**Examples and Scenarios**:

### Example 1: SendMessage (Sending a Message)
- **Scenario**: Your e-commerce app sends an order confirmation to `OrderQueue` when Jane buys a book.
- **What Happens**: The app automatically sends a message with order details.
- **What CloudTrail Logs**:
  ```json
  {
    "Time": "2025-08-11 10:30 AM",
    "Action": "SendMessage",
    "Queue": "OrderQueue",
    "MessageID": "msg123",
    "User": "AppRole",
    "Region": "us-east-1"
  }
  ```
- **Why It Helps**: If Jane says her order didn’t process, you can confirm the message was sent to the queue.

### Example 2: ReceiveMessage (Reading a Message)
- **Scenario**: A program (Lambda function) reads Jane’s order message from `OrderQueue` to process it.
- **What Happens**: The Lambda function automatically fetches the message.
- **What CloudTrail Logs**:
  ```json
  {
    "Time": "2025-08-11 10:32 AM",
    "Action": "ReceiveMessage",
    "Queue": "OrderQueue",
    "MessageID": "msg123",
    "User": "LambdaRole",
    "Region": "us-east-1"
  }
  ```
- **Why It Helps**: You can see the Lambda function picked up the message, so you know it reached the processing stage.

### Example 3: DeleteMessage (Removing a Message)
- **Scenario**: After processing Jane’s order, the Lambda function deletes the message to avoid reprocessing.
- **What Happens**: Lambda automatically deletes the message.
- **What CloudTrail Logs**:
  ```json
  {
    "Time": "2025-08-11 10:33 AM",
    "Action": "DeleteMessage",
    "Queue": "OrderQueue",
    "MessageID": "msg123",
    "User": "LambdaRole",
    "Region": "us-east-1"
  }
  ```
- **Why It Helps**: Confirms the message was processed and removed, ensuring no duplicates.

### Example 4: SendMessageBatch (Sending Multiple Messages)
- **Scenario**: Your app sends three order messages at once for efficiency.
- **What Happens**: The app sends a batch of messages for orders from Jane, Tom, and Lisa.
- **What CloudTrail Logs**:
  ```json
  {
    "Time": "2025-08-11 10:35 AM",
    "Action": "SendMessageBatch",
    "Queue": "OrderQueue",
    "MessageIDs": ["msg123", "msg124", "msg125"],
    "User": "AppRole",
    "Region": "us-east-1"
  }
  ```
- **Why It Helps**: Verifies all three orders were sent together, and you can check if any failed.

**Key Point**: Data events track the lifecycle of messages (sent, received, deleted). You need to enable them in CloudTrail, and they’re great for tracking message flows.

---

## 4. CloudWatch Logs: What Your App Does with Messages

**What They Are**: CloudWatch Logs store details about what your app does after receiving a message (e.g., processing an order). SQS doesn’t send logs here directly—your app (like a Lambda function) must write them.

**Purpose**: To debug issues (e.g., “Why did this order fail?”) or track business details (e.g., “Which orders were processed?”).

**Example Scenario**:
- **Scenario**: Your Lambda function processes orders from `OrderQueue`. It logs whether it succeeded or failed.
- **What Happens**: Lambda reads a message, tries to process it, and writes to CloudWatch Logs.
- **What CloudWatch Logs Show**:
  - Success: 
    ```
    2025-08-11 10:32 AM: Processed order 12345 successfully
    ```
  - Failure: 
    ```
    2025-08-11 10:33 AM: Failed to process message msg123: Bad order data
    ```
- **Why It Helps**: If Jane’s order fails, you can check CloudWatch to see the error (“Bad order data”) and fix it (e.g., correct the app’s input).

**Key Point**: CloudWatch Logs are like your app’s diary, showing what it did with each message. You control what gets logged.

---

## 5. Simple Setup Steps

To see these logs in action:

1. **CloudTrail for Management Events**:
   - Go to AWS Console > CloudTrail > Trails.
   - Create a trail (if none exists) and choose an S3 bucket to store logs.
   - Management events (like `CreateQueue`) are logged automatically.

2. **CloudTrail for Data Events**:
   - In your trail, turn on “Data events” for SQS.
   - Select `OrderQueue` and actions like `SendMessage`, `ReceiveMessage`.
   - Logs will appear in your S3 bucket or CloudTrail Event History.

3. **CloudWatch Logs for Your App**:
   - In your app (e.g., Lambda), add simple log statements (e.g., “Processed order X”).
   - Logs go to CloudWatch under `/aws/lambda/<function-name>`.

---

## 6. Real-World Use Case: Tracking a Missing Order

**Problem**: Jane says her order (ID 12345) didn’t process.

1. **Check CloudTrail for SendMessage**:
   - Log shows `msg123` was sent at 10:30 AM.
2. **Check CloudTrail for ReceiveMessage**:
   - Log shows Lambda received `msg123` at 10:32 AM.
3. **Check CloudWatch Logs**:
   - Log shows “Failed to process message msg123: Bad order data.”
4. **Fix**: Update the app to handle order data correctly.

**Outcome**: You trace the issue to bad data and fix it, using logs to follow the message’s journey.

---

## 7. Explaining this to someone

**Analogy**:
- **Management Events**: Like a log of who built or changed the office mailbox (`CreateQueue`, `SetQueueAttributes`).
- **Data Events**: Like a record of who dropped off or picked up mail (`SendMessage`, `ReceiveMessage`).
- **CloudWatch Logs**: Like notes from the worker who opened the mail and tried to process it.

**One-Liner**: “CloudTrail tracks queue changes and message actions, while CloudWatch Logs shows what our app does with messages, helping us audit and debug everything.”

---

<xaiArtifact artifact_id="624e5323-b7bd-427e-80dd-1335b58c4e68" artifact_version_id="1a226a79-6063-45fe-88e1-7cbe515a41c8" title="SQS_Logging_Examples.md" contentType="text/markdown">

# Simplified SQS Logging Examples

## Management Events (Queue Changes)

### 1. CreateQueue
- **Scenario**: Sarah creates `OrderQueue` for customer orders.
- **Log**:
  ```json
  {
    "Time": "2025-08-11 10:00 AM",
    "Action": "CreateQueue",
    "Queue": "OrderQueue",
    "User": "Sarah"
  }
  ```
- **Use**: Confirms who created the queue.

### 2. DeleteQueue
- **Scenario**: Mike deletes `OrderQueue` by mistake.
- **Log**:
  ```json
  {
    "Time": "2025-08-11 10:15 AM",
    "Action": "DeleteQueue",
    "Queue": "OrderQueue",
    "User": "Mike"
  }
  ```
- **Use**: Shows why the queue is gone.

### 3. SetQueueAttributes
- **Scenario**: Sarah sets `OrderQueue` to keep messages for 7 days.
- **Log**:
  ```json
  {
    "Time": "2025-08-11 10:20 AM",
    "Action": "SetQueueAttributes",
    "Queue": "OrderQueue",
    "User": "Sarah",
    "Change": "MessageRetentionPeriod = 7 days"
  }
  ```
- **Use**: Checks if settings caused issues.

### 4. PurgeQueue
- **Scenario**: Sarah clears test messages from `OrderQueue`.
- **Log**:
  ```json
  {
    "Time": "2025-08-11 10:25 AM",
    "Action": "PurgeQueue",
    "Queue": "OrderQueue",
    "User": "Sarah"
  }
  ```
- **Use**: Verifies if a purge deleted messages.

## Data Events (Message Actions)

### 1. SendMessage
- **Scenario**: App sends Jane’s order to `OrderQueue`.
- **Log**:
  ```json
  {
    "Time": "2025-08-11 10:30 AM",
    "Action": "SendMessage",
    "Queue": "OrderQueue",
    "MessageID": "msg123",
    "User": "AppRole"
  }
  ```
- **Use**: Confirms the order was sent.

### 2. ReceiveMessage
- **Scenario**: Lambda reads Jane’s order.
- **Log**:
  ```json
  {
    "Time": "2025-08-11 10:32 AM",
    "Action": "ReceiveMessage",
    "Queue": "OrderQueue",
    "MessageID": "msg123",
    "User": "LambdaRole"
  }
  ```
- **Use**: Shows the message was picked up.

### 3. DeleteMessage
- **Scenario**: Lambda deletes Jane’s order after processing.
- **Log**:
  ```json
  {
    "Time": "2025-08-11 10:33 AM",
    "Action": "DeleteMessage",
    "Queue": "OrderQueue",
    "MessageID": "msg123",
    "User": "LambdaRole"
  }
  ```
- **Use**: Proves the message was processed.

### 4. SendMessageBatch
- **Scenario**: App sends three orders at once.
- **Log**:
  ```json
  {
    "Time": "2025-08-11 10:35 AM",
    "Action": "SendMessageBatch",
    "Queue": "OrderQueue",
    "MessageIDs": ["msg123", "msg124", "msg125"],
    "User": "AppRole"
  }
  ```
- **Use**: Verifies multiple orders were sent.

## CloudWatch Logs (App Processing)

- **Scenario**: Lambda processes orders and logs results.
- **Logs**:
  - Success: `2025-08-11 10:32 AM: Processed order 12345 successfully`
  - Failure: `2025-08-11 10:33 AM: Failed to process message msg123: Bad order data`
- **Use**: Debugs why an order failed.

</xaiArtifact>

---

