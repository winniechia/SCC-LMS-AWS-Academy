# Module 12 — Caching Content Cheat Sheet

## Core split

```text
CloudFront = content caching/delivery near viewers
ElastiCache = application/database data caching
```

## Fast recognition

| Exam clue | Think |
| --- | --- |
| Multi-tiered/regional content caching | CloudFront |
| Edge location + low latency | CloudFront |
| On-demand video storage/origin | Amazon S3 |
| On-demand video delivery | CloudFront |
| Frequently accessed application/database data | ElastiCache |
| Cache first -> miss -> DB -> cache result | Lazy loading / cache-aside |
| Update cache when application writes data | Write-through |
| Data must be updated in real time (course framing) | Write-through |
| Scale actual database reads | RDS Read Replica |

## CloudFront memory

> **S3 stores. CloudFront delivers.**
>
> **S3 負責存；CloudFront 負責送。**

```text
Viewer -> Edge Cache -> Origin
```

**Cache hit** = edge/cache already has the requested content.

**Cache miss** = retrieve the content upstream/from the origin.

## Edge-cache examples from the reviewed question

Good candidates in the course question:

- web objects, such as hyperlinks;
- video files, such as a product demo.

The personalized/user-specific examples in that question were not selected.

## CloudFront vs ALB

```text
CloudFront
= Can content be closer to the viewer?

ALB
= Which healthy backend gets the request?
```

## CloudFront security distinction

```text
CloudFront = edge delivery/network
Shield     = DDoS protection
WAF        = web-request filtering
```

## ElastiCache strategies

### Lazy loading

```text
Read cache
  |
miss
  v
Database
  |
  v
populate cache
```

### Write-through

```text
Application write
  +--> Database
  +--> Cache
```

## ElastiCache vs RDS Read Replica

```text
ElastiCache
= fast in-memory cache
= frequently reused data

RDS Read Replica
= actual database replica
= scale database reads
```

## 3rd-grade memory / 三年級記憶法

> **CloudFront = 附近取貨站**
>
> **Origin = 總店／大倉庫**
>
> **Cache hit = 架上有，直接拿**
>
> **Cache miss = 架上沒有，回總店拿**
>
> **ElastiCache = 櫃台旁快速架子**
>
> **Lazy loading = 客人問了、架上沒有，才去倉庫拿**
>
> **Write-through = 正式資料一改，cache 也跟著更新**

## One-line SAA memory

> **CloudFront = near the viewer. ElastiCache = near the application/database.**
>
> **CloudFront = 靠近使用者；ElastiCache = 靠近 application/database。**
