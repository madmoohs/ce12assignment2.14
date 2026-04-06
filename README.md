# ce12assignment2.14

1 - No, SNS does not guarantee exactly-once delivery to subscribers. Instead, SNS follows an at-least-once delivery model, meaning a message is delivered one or more times, but duplicates can occur. This design prioritizes high availability and durability, ensuring that messages are not lost even in the presence of transient failures such as network issues or endpoint unavailability. As a result, subscribers (e.g., HTTP endpoints, Lambda functions, or SQS queues) must be designed to handle potential duplicate messages, typically by implementing idempotency mechanisms.


2 - A Dead-Letter Queue (DLQ) is a mechanism used to capture and isolate messages that fail to be processed successfully after multiple attempts. Its primary purpose is to prevent problematic messages from continuously retrying and potentially disrupting the system, while also allowing developers to inspect and debug those failures later.

In a typical flow, when a message is consumed (for example, by a Lambda function or an application) and processing fails, the system will retry delivery several times based on a defined retry policy. However, if the message keeps failing due to issues like malformed data, application bugs, or downstream service errors, it is eventually moved to the DLQ after exceeding a configured threshold. This ensures that the main queue or event pipeline continues processing other valid messages without being blocked or slowed down by repeatedly failing ones.

The DLQ serves several important purposes. First, it acts as a safety net, ensuring that no messages are permanently lost even if they cannot be processed immediately. Second, it provides a debugging and auditing tool, where developers can examine failed messages, identify root causes, and fix issues in the system. Third, it helps maintain system stability and performance by removing “poison messages” that would otherwise cause repeated failures and retries. Finally, DLQs enable reprocessing workflows, where once the issue is resolved, messages can be replayed or manually handled.



3 - First, you configure your DLQ to emit metrics to Amazon CloudWatch, which automatically tracks metrics like ApproximateNumberOfMessagesVisible—this indicates how many messages are currently sitting in the DLQ. Then, you create a CloudWatch Alarm that watches this metric and triggers when the value exceeds a threshold (for example, ≥ 1 message), meaning at least one failed message has landed in the DLQ.

Next, you create an SNS topic using Amazon SNS, and subscribe your email address to this topic. After that, you link the CloudWatch Alarm to the SNS topic as its notification action. This means whenever the alarm state is triggered (i.e., messages appear in the DLQ), SNS will automatically send an email notification to all subscribed recipients.
