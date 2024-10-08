+++
title = "System Design: Back-of-the-envelope calculations"
date = "2024-02-12T01:28:09+01:00"
author = "Jiwei, Yuan"
authorTwitter = "JiweiYuan" #do not include @
cover = ""
tags = ["system design"]
keywords = ["", ""]
description = ""
showFullContent = false
readingTime = false
hideComments = false
color = "" #color from the theme settings
draft = false
+++



In physics, dimensional analysis is a go-to heuristic for checking equations. In computer science, back-of-the-envelope calculations serve a similar purpose for rough estimations. Jeff Dean emphasised the importance of mastering this skill for every programmer in his talk [Software Engineering Advice from Building Large-Scale Distributed Systems](https://static.googleusercontent.com/media/research.google.com/en//people/jeff/stanford-295-talk.pdf). Why do we need back-of-the-envelope calculations? The calculation allow us to quickly assess the physical limitations of a running system. If there has a sigficant gap between our expectations(theoretical limitation) and the actual cost, there might be an opportunity to optimize our systems. 

## Performance

{{< youtube IxkSlnrRFqc >}}

#### [Latency Comparison Numbers (~2012)](https://gist.github.com/jboner/2841832)

| Operation                           | Time   | Note                        |
| ----------------------------------- | ------ | --------------------------- |
| L1 cache reference                  | 0.5 ns |                             |
| Branch mispredict                   | 5 ns   |                             |
| L2 cache reference                  | 7 ns   | 14x L1 cache                |
| Mutex lock/unlock                   | 25 ns  |                             |
| Main memory reference               | 100 ns | 20x L2 cache, 200x L1 cache |
| Compress 1K bytes with Zippy        | 3 µs   |                             |
| Send 1K bytes over 1 Gbps network   | 10 µs  |                             |
| Read 4K randomly from SSD*          | 150 µs | ~1GB/sec SSD                |
| Read 1 MB sequentially from memory  | 250 µs |                             |
| Round trip within same datacenter   | 500 µs |                             |
| Read 1 MB sequentially from SSD*    | 1 ms   | ~1GB/sec SSD, 4x memory     |
| Disk seek                           | 10 ms  | 20x datacenter roundtrip    |
| Read 1 MB sequentially from disk    | 20 ms  | 80x memory, 20x SSD         |
| Send packet CA -> Netherlands -> CA | 150 ms | 300x datacenter roundtrip   |


##### PostgreSQL

| No  | Spec                 | Config  | CPU | Freq    | RO     | RW    | Cost ($/month) |
|-----|----------------------|---------|-----|---------|--------|-------|----------------|
| 2   | AWS z1d.2xlarge       | Normal  | 8   | 4GHz    | 162315 | 24808 | 300            |
| 4   | AWS c5d.metal         | Normal  | 96  | 3.6GHz  | 625849 | 71624 | 2500           |
| 5   | AWS c5d.metal         | Extreme | 96  | 3.6GHz  | 1998580| 137127| 5000           |
- RO: read only query(one row from 100M rows)
- RW: read write query(one query, one insert, three updates)

## Cost Estimation
As an architecture, what we are concerned with should not only focus on performance but also consider cost. Below are the computing costs and storage costs for the cloud-hosted service.

#### Base Costs:

- **CPU**: $10/Core/month
- **Memory**: $2-3/GB/month
- **SSD**: $0.10/GB/month
- **S3**: $0.02/GB/month
- **HDD**: $0.01 - $0.05/GB/month
- **Network** (egress, between zones or regions): $0.05 - $0.10/GB/month

Please note: For hardware, Reserved pricing is typically about half the cost of On-Demand Pricing.

##### Compute

| Provider      | (1 vCPU, 2 GB RAM) | Compute (8vCPUs, 32 GB RAM) |
| ------------- | ------------------ | --------------------------- |
| Render.com    | $ 25               | $ 450                       |
| Amazon EC2    | $ 15               | $ 185                       |


##### PostgreSQL Storage

|               | Storage(GB per Month) |
| ------------- | --------------------- |
| Reder.com     | $ 0.3                 |
| Amazon Aurora | $ 0.23                |

##### S3 Storage

|               | Storage Cost (per GB-month) | A Operation Cost (per million requests) | B Operation Cost (per million requests) | Egress Fee (per GB) |
| ------------- | --------------------------- | --------------------------------------- | --------------------------------------- | ------------------- |
| Amazon S3     | $0.023                      | $5.00                                   | $0.40                                   | $0.09               |
| Cloudflare R2 | $0.015                      | $4.50                                   | $0.36                                   | No egress fee       |

For more vendor pricing, please refer to [S3-Compatible Cloud Storage Costs](https://transactional.blog/blog/2023-cloud-storage-costs).

##### The Power of 2

| Power | Exact Value          | Approx Value | Bytes |
|-------|----------------------|--------------|-------|
| 5     | 32                   |              |       |
| 6     | 64                   |              |       |
| 7     | 128                  |              |       |
| 8     | 256                  |              |       |
| 10    | 1,024                | 1 thousand   | 1 KB  |
| 16    | 65,536               |              | 64 KB |
| 20    | 1,048,576            | 1 million    | 1 MB  |
| 30    | 1,073,741,824        | 1 billion    | 1 GB  |
| 32    | 4,294,967,296        |              | 4 GB  |
| 40    | 1,099,511,627,776    | 1 trillion   | 1 TB  |


#### Reference
- https://github.com/sirupsen/napkin-math
- https://www.youtube.com/watch?v=IxkSlnrRFqc
- https://deploy.equinix.com/product/servers/c3-small/
