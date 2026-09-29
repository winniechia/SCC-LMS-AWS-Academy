# Module 13 — Building Decoupled Architectures Cheat Sheet

## Status

**Student Guide review documented — Knowledge Check pending**

## Core split

```text
SQS = queue / buffer / decouple work
SNS = publish / subscribe / distribute messages
```

## Fast recognition

| Exam clue | Think |
| --- | --- |
| Decouple components | Amazon SQS |
| Buffer asynchronous work | Amazon SQS |
| Producer faster than consumer | Amazon SQS |
| Strict message ordering | SQS FIFO |
| High-throughput queue; strict order not central | SQS Standard |
| Reduce empty receive responses | Long polling |
| Hide received message while processing | Visibility Timeout |
| Backlog grows; add processing capacity | SQS + Auto Scaling |
| One publication to multiple subscribers | Amazon SNS |
| Publish/subscribe or fan-out | Amazon SNS |
| Fan-out plus independent buffering | SNS + SQS |

## SQS message flow

```text
Producer
   |
   v
SQS Queue
   |
   v
Consumer
```

> **SQS = 訂單箱。Producer 放工作；Consumer 有空再拿。**

## Visibility Timeout

```text
receive message
     |
temporarily hidden
     |
     +--> success -> delete
     |
     +--> not completed -> can become visible again
```

> **Receive != successful completion.**
>
> **收到 != 已經成功做完。**

## Standard vs FIFO

```text
Standard
= scale / throughput
= strict ordering is not the central guarantee

FIFO
= First-In, First-Out
= ordering matters
```

### 3rd-grade memory

> **Standard = 大量快速處理**
>
> **FIFO = 排隊 1 -> 2 -> 3**

## Long polling

> **Long polling helps reduce empty responses.**
>
> **Long Polling = 餐好了再叫我，不用一直跑去問。**

## SQS + Auto Scaling

```text
Queue backlog grows
       |
       v
Need more workers
       |
       v
Auto Scaling
```

> **SQS = 訂單箱；ASG = 看工作量安排人手的經理。**

## SNS

```text
                 +--> Subscriber A
Publisher --> Topic --> Subscriber B
                 +--> Subscriber C
```

> **SNS = 廣播站。**

## SQS vs SNS

```text
SQS holds the work.  📥
SNS spreads the message. 📢
```

> **SQS 把工作留著排隊；SNS 把訊息傳出去。**

## SNS + SQS

```text
                    +--> SQS A --> Consumer A
Publisher -> SNS ---|
                    +--> SQS B --> Consumer B
```

> **SNS fan-out + SQS independent buffering.**

## Module 10 connection

```text
ASG = How many workers?
SQS = Where does unfinished work wait?
```

## One-line SAA memory

> **Need a buffer -> SQS. Need a broadcast -> SNS.**
>
> **要排隊緩衝 -> SQS；要廣播給多個接收者 -> SNS。**

## Next checkpoint

Module 13 Knowledge Check still needs question-by-question review before this module is marked complete.
