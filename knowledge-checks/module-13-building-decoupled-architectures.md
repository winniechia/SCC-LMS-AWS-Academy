# Module 13 — Building Decoupled Architectures Study Notes

## Status

**Student Guide concepts documented — Knowledge Check review pending**

These notes capture the Module 13 concepts reviewed from the supplied AWS Academy Student Guide recording. The Knowledge Check has not yet been studied question-by-question, so this file is intentionally **not** marked complete.

---

## Module 13 concept map

```text
Building Decoupled Architectures
        |
        +--> Loose coupling / decoupling
        |
        +--> Amazon SQS
        |      +--> message queue
        |      +--> producer / consumer
        |      +--> Standard vs FIFO
        |      +--> long polling
        |      +--> buffer workload
        |      +--> Auto Scaling workers
        |
        +--> Amazon SNS
               +--> topic
               +--> publisher / subscriber
               +--> fan-out
```

---

## 1. Tight coupling vs loose coupling

### Tight coupling

With tightly coupled components, one component depends directly on another component being available and ready to respond.

```text
Component A ---> Component B
                  |
             unavailable?
                  |
                  v
          A can be affected too
```

### Loose coupling / decoupling

A queue can sit between components so the producer does not need the consumer to process the work immediately.

```text
Producer ---> Queue ---> Consumer
```

### 3rd-grade analogy / 三年級比喻

Imagine a restaurant.

Without a queue, the waiter must find a chef and wait for the chef to accept each order immediately.

With a queue, the waiter puts the order into an **order box** and continues serving customers. The chef takes orders from the box when ready.

> **Decoupling = producer and consumer can work more independently.**
>
> **解耦 = 前面的人把工作留下來就可以繼續；後面的人按照自己的速度處理。**

---

## 2. Amazon SQS

Amazon SQS provides a message queue that can be used to decouple application components.

```text
Producer
   |
   | message
   v
+-------------+
| SQS Queue   |
| [] [] [] [] |
+-------------+
      |
      v
  Consumer
```

### Mental model

- **Producer** = creates/sends work.
- **Queue** = temporarily holds messages/work.
- **Consumer** = retrieves and processes work.

### 3rd-grade analogy

> **SQS = 餐廳的待辦訂單盒。**
>
> **Producer = 放訂單的人。**
>
> **Consumer = 拿訂單來處理的人。**

### SAA-C03 recognition

Think about SQS when a scenario emphasizes:

- decoupling components;
- buffering work;
- asynchronous processing;
- producers and consumers operating at different rates;
- absorbing a temporary workload spike.

---

## 3. Message processing and visibility

A consumer can receive a message and process it. Receiving a message should not be mentally treated as identical to successfully completing the work.

The reviewed learning model is:

```text
Receive message
      |
      v
temporarily unavailable to other consumers
      |
      +--> processing succeeds --> delete message
      |
      +--> processing does not finish
             |
             v
        message can become visible again
```

### Visibility Timeout

Visibility Timeout gives a consumer time to process a received message without another consumer immediately receiving that same message.

### 3rd-grade analogy

A chef takes an order card from the order box. While that chef is working, the card is temporarily hidden from the other chefs.

If the work finishes successfully, the order can be removed. If processing does not finish before the visibility period expires, the order can become available for processing again.

### Memory

> **Receive is not the same as successful completion.**
>
> **收到 message 不等於工作已經成功完成。**

---

## 4. Standard Queue vs FIFO Queue

### Standard Queue

The study model emphasizes high-throughput, scalable message processing where strict ordering is not the central requirement.

### FIFO Queue

FIFO means **First-In, First-Out** and is used when message ordering matters.

```text
A -> B -> C

FIFO:
A -> B -> C
```

### 3rd-grade analogy

**Standard Queue** = a busy restaurant order box where the main goal is processing lots of work.

**FIFO Queue** = people standing in a numbered line: 1, then 2, then 3.

### SAA-C03 memory

```text
high throughput / strict order not required
 -> Standard Queue

order matters / sequence matters
 -> FIFO Queue
```

> **Standard = 大量快速處理。**
>
> **FIFO = 排好隊 1 -> 2 -> 3。**

---

## 5. Long polling

Consumers poll SQS to receive messages.

The Student Guide material includes **long polling**, which allows a receive request to wait for a message rather than immediately returning an empty response when no message is available.

### 3rd-grade analogy

Short-polling mental model:

> 每幾秒跑去問：「我的飯好了嗎？」

Long-polling mental model:

> 「餐好了再叫我，我可以等一下。」

### Memory

> **Long polling helps reduce empty responses.**
>
> **Long Polling 可以減少沒有拿到 message 的空回應。**

---

## 6. SQS with Auto Scaling

A queue can buffer incoming work when producers create work faster than consumers can process it.

```text
Producer
   |
   v
SQS Queue
[][][][][][][][][]
   |
   v
EC2 workers
```

If the backlog grows, processing capacity can scale.

```text
Large queue backlog
       |
       v
Need more processing capacity
       |
       v
Auto Scaling
       |
       v
More workers
```

### Connection to Module 10

Module 10:

> **ASG = 餐廳經理，決定需要多少工作人員。**

Module 13 adds:

> **SQS = 訂單箱，workers 還沒處理的工作可以先排隊。**

Together:

> **SQS buffers the work; Auto Scaling adjusts worker capacity.**
>
> **SQS 先接住工作；Auto Scaling 再調整處理工作的 capacity。**

---

## 7. Amazon SNS

Amazon SNS uses a publish/subscribe model.

```text
                 +--> Subscriber A
Publisher --> Topic --> Subscriber B
                 +--> Subscriber C
```

A publisher sends a message to a topic, and the message can be delivered to subscribers.

### 3rd-grade analogy

> **SNS = 學校廣播站。**

The principal makes one announcement through the broadcast system, and multiple listeners can receive it.

### Memory

> **SNS = publish / subscribe / fan-out.**
>
> **SNS = 發布訊息，再送給訂閱者。**

---

## 8. SQS vs SNS

| Service | Mental model | Primary idea |
| --- | --- | --- |
| Amazon SQS | Order box / 訂單箱 | Queue, buffer, asynchronous processing |
| Amazon SNS | Broadcast station / 廣播站 | Publish/subscribe, distribute messages |

### Fast memory

> **SQS holds the work. SNS spreads the message.**
>
> **SQS 把工作留著排隊；SNS 把訊息傳出去。**

---

## 9. SNS + SQS fan-out pattern

SNS and SQS can be combined conceptually:

```text
                         +--> SQS Queue A --> Consumer A
Publisher --> SNS Topic -|
                         +--> SQS Queue B --> Consumer B
```

SNS distributes the event while each SQS queue gives an independent consumer path its own buffer.

### 3rd-grade analogy

A school makes one broadcast, but different departments each have their own work box.

> **SNS broadcasts; each SQS queue lets its consumer process independently.**
>
> **SNS 負責廣播；每個 SQS Queue 讓自己的 Consumer 按自己的速度處理。**

---

## SAA-C03 rapid-recognition map

```text
Decouple application components
 -> Amazon SQS

Buffer asynchronous work
 -> Amazon SQS

Producer faster than consumer
 -> Amazon SQS

Strict message ordering required
 -> SQS FIFO Queue

High-throughput queue; strict ordering not central
 -> SQS Standard Queue

Reduce empty queue receive responses
 -> Long polling

Temporarily hide a received message while it is processed
 -> Visibility Timeout

Queue backlog grows and workers need more capacity
 -> SQS + Auto Scaling workers

One publication to multiple subscribers
 -> Amazon SNS

Publish / subscribe / fan-out
 -> Amazon SNS

Broadcast event + independent buffered processing paths
 -> SNS + SQS queues
```

---

## 3rd-grade café story / 三年級 Café 故事

The café receives orders faster than the kitchen can sometimes process them.

Instead of forcing the waiter to wait beside a chef, the waiter puts each order into an **SQS order box**. This decouples the waiter from the kitchen.

A chef takes an order when ready. While that chef works, the order is temporarily hidden from the other chefs. If the workload becomes very large, the café manager—our **Auto Scaling** analogy—can add more workers.

If the café needs to announce one event to several different teams, **SNS** works like the building's broadcast system. SNS can publish the announcement, while separate SQS queues can give different teams their own independent work boxes.

---

## Connection to previous modules

### Module 10 + Module 13

```text
Module 10:
Auto Scaling -> How many workers do we need?

Module 13:
SQS -> Where does work wait until workers can process it?
```

### Final mental picture

```text
Producer
   |
   v
SQS Queue --------> buffers work
   |
   v
Auto Scaling workers

Publisher
   |
   v
SNS Topic --------> distributes message
   |
   +--> subscriber(s)
```

---

## Final memory chain

> **Decouple -> SQS -> Queue -> Consumer -> Scale workers**
>
> **Broadcast -> SNS -> Topic -> Subscribers**

中文：

> **工作先排隊 -> SQS。**
>
> **訊息要廣播 -> SNS。**

---

## Knowledge Check checkpoint

**Not completed yet.**

When study resumes, review the Module 13 Knowledge Check question-by-question and add:

1. correct answer;
2. why it is correct;
3. why the alternatives are incorrect;
4. 3rd-grade analogy;
5. SAA-C03 trigger/cue;
6. any weak spots or corrections discovered during the review.

Do not mark Module 13 complete until that review is finished.
