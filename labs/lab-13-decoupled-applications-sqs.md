# Lab 13 — Building Decoupled Applications by Using Amazon SQS

[Study index](../README.md) | [Printable cheat sheet](../cheat-sheets/lab-13-decoupled-applications-sqs-cheat-sheet.md) | [Personal rebuild plan](../personal-labs/lab-13-decoupled-applications-sqs-companion-plan.md)

## Purpose and completed lab record

AWS Academy Guided Lab: **Building Decoupled Applications by Using Amazon SQS**.

The lab demonstrated how to move from direct application-to-application processing toward an event-driven, decoupled workflow using Amazon S3, Amazon SNS, Amazon SQS, DynamoDB, and separate web/application servers.

**Class lab: completed.**

Temporary account IDs, ARNs, bucket names, queue URLs, public IPs, endpoints, credentials, and other classroom identifiers are intentionally omitted.

## 1. Final Phase 2 architecture

```text
User uploads PNG
      |
      v
Amazon S3
      |
      | S3 object-created event
      v
Amazon SNS topic
   /          \
  v            v
SQS queue     Email subscriber
  |
  | polling
  v
App Server
  |
  | get original -> process/resize/tint -> save adjusted image
  v
Amazon S3
  |
  v
Web application displays processed image
```

The central lesson is that the upload path and image-processing path no longer need to execute as one tightly coupled synchronous chain.

## 2. What AWS Academy prepared for me

**[AWS Academy prepared this]**

The lab environment supplied starter application code and classroom infrastructure/scaffolding needed for the exercise. The application already contained the web/app-server logic, status workflow, and configuration placeholders used during the guided lab.

This means completing the guided lab is **not the same as designing the application and infrastructure from zero**.

## 3. What I built/configured

**[I built/configured this]**

- Installed the Node.js dependencies for the Phase 2 web server and application server.
- Configured the Phase 2 application to use the lab DynamoDB table and S3 bucket.
- Configured an S3 bucket policy required by the guided exercise.
- Created/configured the SNS topic used by the Phase 2 event flow.
- Configured S3 Event Notifications so object creation publishes to SNS.
- Subscribed the `ImageApp` SQS queue to the SNS topic.
- Configured the application server with the SQS queue URL.
- Started the Phase 2 web server and application server.
- Uploaded a PNG through the Image Tinter application.
- Started SQS polling and verified that the worker processed the queued image.
- Verified the processed/tinted 300x300 image appeared in the web application.

## 4. Decoupling mental model

Phase 1 mental model:

```text
Web Server ---> App Server
```

If downstream processing is unavailable or slow, the upstream path can be affected.

Phase 2 mental model:

```text
Producer ---> SNS ---> SQS ---> Consumer
```

The queue gives the producer and consumer more independence.

> **Decoupling = the sender can leave work safely without requiring the worker to process it immediately.**
>
> **解耦 = 前面的人可以把工作安全留下，不必等後面的人立刻做完。**

## 5. S3 Event Notification

The Phase 2 S3 bucket was configured so an object-created event could publish a notification to the SNS topic.

```text
New object in S3
      |
      v
Event Notification
      |
      v
SNS
```

This converts a storage event into an event-driven application signal.

## 6. SNS and SQS together

SNS and SQS solve different problems.

| Service | Lab role | Memory |
| --- | --- | --- |
| Amazon SNS | Receive the S3 event and distribute the notification | 廣播站 |
| Amazon SQS | Hold work until the app server is ready to process it | 待辦工作箱 |

The lab also used an email subscription, demonstrating the fan-out idea:

```text
                +--> SQS -> App Server
S3 -> SNS ------|
                +--> Email
```

> **SNS spreads the message; SQS holds the work.**
>
> **SNS 把訊息傳出去；SQS 把工作留著排隊。**

## 7. Polling and the consumer

The application server acted as the SQS consumer.

The web interface's **Poll SQS** action started the processing mechanism. The consumer could then retrieve queued work and process the corresponding image.

Successful status flow:

```text
1. Sent to S3
2. App server received image URL
3. App server got original image buffer
4. Processed image
5. Saved adjusted to S3
6. Complete
```

The final processed image provided end-to-end evidence that the event/message path worked.

## 8. Troubleshooting lesson 1 — wrong SNS topic type

During the lab, an SNS topic was initially created as FIFO, producing a name ending in `.fifo`. The guided lab expected the topic named `uploadnotification`, and the S3 event-notification configuration did not succeed with the mistaken topic setup.

The fix was to create/use the intended **Standard** topic named `uploadnotification` and then configure the S3 event notification against that topic.

### Lesson

Do not infer that two resources are interchangeable merely because both are SNS topics. Queue/topic type and service integration requirements matter.

## 9. Troubleshooting lesson 2 — placeholder SQS URL

After the event flow was configured, clicking **Poll SQS** initially appeared to do nothing. The app-server terminal revealed the actual evidence:

```text
ERR_INVALID_URL
input: 'https://[Phase2 SQS URL]'
```

The application server configuration still contained the lab placeholder rather than the real queue URL.

The fix:

1. copy the `ImageApp` queue URL;
2. replace the placeholder in `phase_2/app_server_2/libs/config.js`;
3. save the file;
4. restart the Node.js app server so it reloads the configuration;
5. poll SQS again.

After restart, the queued image was consumed and processing completed.

### SAA troubleshooting habit

> **Read the error before changing AWS resources.**

The queue was not necessarily broken. The consumer simply had an invalid endpoint configuration.

## 10. Configuration reload lesson

Saving a Node.js configuration file does not automatically mean an already-running process has reloaded it.

```text
edit config
   |
   v
save file
   |
   v
restart process
   |
   v
new configuration loaded
```

This was why the old placeholder error remained visible until `app_server_2` was restarted.

## 11. 🧒 Explain It Like I'm in 3rd Grade / 三年級也能懂

Imagine a restaurant:

- **S3 = 收件櫃台** — receives the picture.
- **SNS = 廣播員** — announces that new work arrived.
- **SQS = 待辦工作箱** — safely holds the work.
- **App Server = 廚師** — takes a job when ready and processes it.
- **DynamoDB = 工作狀態記錄簿** — keeps application metadata/status.
- **Poll SQS = 廚師去工作箱看看有沒有新訂單。**

Without the work box, the front counter may depend directly on the chef being ready.

With SQS:

> 「訂單先放進箱子。廚師準備好時再拿。」

That is the heart of decoupling.

## 12. SAA-C03 takeaways

- SQS is a strong answer when a design needs to **decouple** producers and consumers.
- SQS can **buffer** asynchronous work when producers and consumers operate at different rates.
- SNS provides **publish/subscribe** and **fan-out**.
- SNS + SQS can distribute one event into independently buffered processing paths.
- S3 Event Notifications can initiate downstream event-driven workflows.
- A consumer must have the correct queue endpoint/configuration before polling can work.
- Troubleshoot from application logs/errors rather than randomly changing infrastructure.
- A configuration change may require an application restart.
- A successful end-to-end test should verify the business result, not merely that resources exist.

## 13. Connection to previous modules

### Module 10

```text
Auto Scaling = How many workers?
SQS          = Where does unfinished work wait?
```

A queue can absorb work while processing capacity changes.

### Module 12

Module 12 focused on reducing repeated work and latency with caching. Module 13 addresses a different architecture problem: allowing components to communicate asynchronously and independently.

```text
Caching   -> avoid/reduce repeated expensive work
Decoupling -> separate producer timing from consumer timing
```

## 14. Lab Completion Checkpoint

- [x] Class Lab Complete
- [x] Phase 2 dependencies installed
- [x] Phase 2 S3/DynamoDB application configuration verified
- [x] SNS topic configured
- [x] S3 Event Notification configured
- [x] SQS queue subscribed to SNS
- [x] App server SQS URL configured
- [x] Phase 2 web/app servers running
- [x] PNG upload tested
- [x] SQS polling tested
- [x] Processed/tinted image verified
- [x] SNS topic-type troubleshooting captured
- [x] Invalid SQS URL troubleshooting captured
- [x] SAA-C03 takeaways captured
- [x] 3rd-grade analogy captured
- [x] Sensitive temporary AWS identifiers omitted

## 15. Personal AWS Rebuild Decision

**Yes — Very High Learning Value.**

The Academy environment supplied substantial application code and lab scaffolding. A personal rebuild should therefore start from an empty personal AWS environment and independently create the event-driven infrastructure and a minimal producer/consumer application.

See [Lab 13 personal rebuild plan](../personal-labs/lab-13-decoupled-applications-sqs-companion-plan.md).
