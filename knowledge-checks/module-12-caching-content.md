# Module 12 — Caching Content Study Notes

## Status

**Knowledge Check: Completed and reviewed**

These notes capture the concepts reviewed from the Module 12 **Caching Content Knowledge Check**. They summarize the learning and corrections rather than reproduce the questions verbatim.

**There is no Lab 12 hands-on activity in the material available for this study sequence; Module 12 documentation is Knowledge Check only.**

---

## Module 12 concept map

```text
Caching
├── Edge/content caching
│   └── Amazon CloudFront
│       ├── edge locations
│       ├── regional / multi-tier caching
│       ├── origin
│       ├── cache hit / cache miss
│       └── low-latency delivery
│
└── Application/database caching
    └── Amazon ElastiCache
        ├── lazy loading / cache-aside
        └── write-through
```

## 1. Amazon CloudFront

The reviewed material identifies CloudFront with **multi-tiered and regional caching of content**.

### 3rd-grade analogy / 三年級比喻

The **origin** is the main warehouse. **CloudFront edge locations** are pickup locations closer to customers.

> **CloudFront = put cached content closer to viewers.**
>
> **CloudFront = 把 cached content 放到更靠近使用者的地方。**

---

## 2. Edge locations

CloudFront caches frequently accessed content at edge locations and can deliver cached content through an edge location with low latency to clients.

```text
Viewer
  |
  v
Edge location
  |
  | if content is not cached
  v
Origin
```

### Cache hit vs cache miss

- **Cache hit**: requested content is already cached.
- **Cache miss**: requested content is not in the cache, so it must be retrieved upstream/from the origin.

### 3rd-grade analogy / 三年級比喻

> **Cache hit = 架上已經有，直接拿。**
>
> **Cache miss = 架上沒有，要回總店／倉庫拿。**

---

## 3. Content suitable for edge caching

In the reviewed Student Guide question, the cacheable examples were:

- web objects, such as hyperlinks;
- video files, such as a product demo.

The other choices in that question were framed as user-specific or personalized data, such as shopping-cart contents, a user's name, or user-entered search terms.

### Course-question memory

> **Shared/reusable web content -> good edge-cache candidate.**
>
> **使用者可共用、可重複使用的內容 -> 適合 edge cache。**

### Real-world nuance

CloudFront can cache some dynamic content depending on the cache key, headers, cookies, query strings, and cache policy. For the reviewed course question, however, the personalized examples were not the intended answers.

---

## 4. On-demand video pattern

The reviewed material uses this architecture:

```text
Amazon S3
(storage / origin)
     |
     v
Amazon CloudFront
(cache + delivery)
     |
     v
Viewers
```

For on-demand video, **Amazon S3 stores the content and Amazon CloudFront delivers it**.

### Memory

> **S3 stores. CloudFront delivers.**
>
> **S3 負責存；CloudFront 負責送。**

---

## 5. CloudFront and DDoS resilience

The reviewed question identifies CloudFront's role as **routing traffic through edge locations**.

The course-level mental model is that requests can reach AWS's edge network rather than every request going directly to the origin.

### Do not confuse

- **CloudFront** = edge delivery/caching network
- **AWS Shield** = DDoS protection
- **AWS WAF** = web-request filtering/rules

### 3rd-grade analogy / 三年級比喻

> **CloudFront edge network = 很多前面的分流站，避免所有人都直接衝到總店。**

---

## 6. Amazon ElastiCache

CloudFront and ElastiCache both involve caching, but they solve different problems.

| Service | Main caching focus |
| --- | --- |
| Amazon CloudFront | Content delivery closer to viewers |
| Amazon ElastiCache | Application/database data caching |

### 3rd-grade analogy / 三年級比喻

> **Database = 大倉庫；ElastiCache = 櫃台旁的快速架子。**

Frequently requested data can be served from the fast cache instead of repeatedly reading the database.

---

## 7. Lazy loading / cache-aside

The reviewed material describes this flow:

```text
Application
    |
    v
ElastiCache
  /     \
hit     miss
 |        |
return    v
       Database
          |
          v
     put result in cache
```

The application reads from ElastiCache first. When a cache miss occurs, it retrieves the data from the database and writes it into the cache.

### Memory

> **Lazy loading = only load into cache when it is actually requested and missing.**
>
> **Lazy loading = 真的有人要、cache 又沒有時，才去 database 拿並放進 cache。**

### 3rd-grade analogy / 三年級比喻

客人問一樣東西。櫃台快速架子沒有，店員才跑去大倉庫拿；拿回來之後，也留一份在快速架子上，下一位客人就可以更快拿到。

---

## 8. Write-through caching

The reviewed material identifies **write-through** as a way to improve read performance and, in the course question, as the strategy for data that must be updated in real time.

Conceptually:

```text
Application write
      |
      +--> Database
      |
      +--> Cache
```

### Memory

> **Write-through = when data is written, keep the cache updated too.**
>
> **Write-through = 寫資料時，也同步處理 cache 的更新。**

### 3rd-grade analogy / 三年級比喻

正式價目表一改，櫃台旁給客人快速查看的副本也跟著更新，而不是等下一個客人發現資料沒有才處理。

---

## 9. Write-through vs lazy loading

| Strategy | When cache is populated/updated | Course clue |
| --- | --- | --- |
| Lazy loading | On read after a cache miss | Load on demand |
| Write-through | When application data is written | Data must be updated in real time |

### Fast memory

> **Lazy loading = read miss -> go get it.**
>
> **Write-through = write -> keep cache updated.**

---

## 10. ElastiCache vs RDS Read Replica

These can both help read-heavy workloads, but the mental models are different.

| Service | Mental model |
| --- | --- |
| ElastiCache | Fast in-memory cache for frequently reused data |
| RDS Read Replica | Database replica used to scale database reads |

### 3rd-grade analogy / 三年級比喻

> **ElastiCache = 櫃台旁的快速架子。**
>
> **Read Replica = 真的再開一個可以查資料的倉庫櫃台。**

---

## 11. CloudFront vs ALB

These solve different questions.

```text
CloudFront
= Can content be served closer to the viewer?

ALB
= Which healthy backend target should receive this request?
```

### Connection to Module 10

In Module 10:

- **ALB** was the greeting host / request distributor.
- **Target Group** was the healthy-worker list.
- **ASG** decided how many EC2 workers were needed.

In Module 12, **CloudFront sits closer to the viewer and may satisfy a request from cache before the request needs to travel back toward the origin architecture.**

---

## SAA-C03 rapid-recognition map

```text
Global content delivery / edge caching
 -> Amazon CloudFront

Origin for static/on-demand video objects
 -> Amazon S3

Deliver/cache on-demand video
 -> Amazon CloudFront

Frequently accessed application/database data
 -> Amazon ElastiCache

Read cache first -> miss -> database -> populate cache
 -> Lazy loading / cache-aside

Update cache as data is written
 -> Write-through

Data must be updated in real time (course framing)
 -> Write-through

Scale actual database reads
 -> RDS Read Replica
```

---

## 3rd-grade café story / 三年級 Café 故事

Imagine the café has one big central warehouse.

If every customer around the world must ask that warehouse for the same product-demo video, delivery is slow and the warehouse repeats the same work. **CloudFront** creates nearby pickup locations—**edge locations**—that can keep copies of reusable content.

If the nearby shelf already has the requested item, that is a **cache hit**. If it does not, that is a **cache miss**, so the pickup location must get the item from the **origin**.

Inside the café application, the database is another big warehouse. **ElastiCache** is a fast shelf beside the counter. With **lazy loading**, the worker goes to the database only when the requested item is missing from the fast shelf. With **write-through**, when important data is written, the fast shelf is updated as part of the write strategy.

---

## Final memory chain

> **S3 stores -> CloudFront delivers at the edge -> ElastiCache accelerates repeated data reads.**
>
> **S3 存內容 -> CloudFront 在 edge 加速配送 -> ElastiCache 加速重複的資料讀取。**

### One-line exam memory

> **CloudFront = near the viewer. ElastiCache = near the application/database.**
>
> **CloudFront = 靠近使用者；ElastiCache = 靠近 application/database。**
