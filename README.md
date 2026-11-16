# AWS Lightsail Alternatives: Cheaper VPS, More Regions, or Stay on AWS? (Real Costs, Instance Comparison and Migration Notes)

Lightsail's entry price is $5/month for a Linux instance with 512 MB of memory and a public IPv4 address, or $3.50/month if IPv6-only is fine. For a single WordPress site or a small API that number still holds up, and moving away buys you very little. The search for alternatives usually starts somewhere else: you added a managed database at $15/month and a load balancer at $18/month, you need a region Lightsail doesn't have, or your backend needs SQS, RDS read replicas or Fargate autoscaling, none of which Lightsail sells.

This breaks the decision into three paths (stay on AWS, move to a bundled VPS, move to a PaaS), prices out what a realistic Lightsail stack actually costs, lists Vultr's instance families with current entry pricing, and covers what migration really involves.

> **Quick pick for most readers:** if you want Lightsail's flat monthly model with more regions and managed databases, Vultr is the closest match. Its Cloud Compute plans bundle CPU, memory, storage and transfer into one price, it covers North America, Europe, Asia, Australia and South America, and its managed database lineup includes PostgreSQL, MySQL, Valkey and Kafka. 👉 [Compare Vultr's Cloud Compute instance families](https://www.vultr.com/products/cloud-compute?ref=9738262-9J)
>
> If your backend already leans on SQS, full RDS features or a shared VPC, skip to the ECS Express Mode and EC2 section first.

## Vultr's Cloud Compute lineup at a glance

Vultr publishes well over a hundred individual configurations across these families, all billed hourly against a monthly rate, so you only pay for the hours an instance exists. The table lists the families and the price anchors that are publicly listed; every size with its exact hourly and monthly rate sits on the pricing page.

| Instance family | Built for | Billing | Entry or listed price | Plans |
| --- | --- | --- | --- | --- |
| Cloud Compute, Regular Performance | Small web apps, APIs, dev environments | Hourly, monthly rate per plan | From $0.0034/hr (about $2.48/mo) | [View plans](https://www.vultr.com/products/cloud-compute?ref=9738262-9J) |
| Cloud Compute, High Frequency | Latency-sensitive apps on faster clock speeds | Hourly, monthly rate per plan | Varies by size | [View plans](https://www.vultr.com/products/cloud-compute?ref=9738262-9J) |
| Cloud Compute, High Performance | Steady production workloads needing more throughput | Hourly, monthly rate per plan | Varies by size | [View plans](https://www.vultr.com/products/cloud-compute?ref=9738262-9J) |
| Optimized Cloud Compute, General Purpose | Balanced CPU and memory for app servers | Hourly, monthly rate per plan | Varies by size | [View plans](https://www.vultr.com/products/cloud-compute?ref=9738262-9J) |
| Optimized Cloud Compute, CPU Optimized | Build servers, batch jobs, CPU-heavy services | Hourly, monthly rate per plan | Varies by size | [View plans](https://www.vultr.com/products/cloud-compute?ref=9738262-9J) |
| Optimized Cloud Compute, Memory Optimized | In-memory databases, caches, analytics | Hourly, monthly rate per plan | Varies by size | [View plans](https://www.vultr.com/products/cloud-compute?ref=9738262-9J) |
| Optimized Cloud Compute, Storage Optimized | High disk throughput workloads | Hourly, monthly rate per plan | Varies by size | [View plans](https://www.vultr.com/products/cloud-compute?ref=9738262-9J) |
| VX1 Cloud Compute | Dedicated-resource instances for price-sensitive production | Hourly, monthly rate per plan | Largest listed size around $2,102/mo | [View plans](https://www.vultr.com/products/cloud-compute?ref=9738262-9J) |

Both Intel and AMD hosts are available, and Vultr supports Terraform and Kubernetes-based orchestration if you plan to automate scale-out rather than click through a console.

## What Lightsail's $5 entry price leaves out

The $5 plan is a real server: 512 MB of memory, 2 vCPUs, 20 GB of SSD and 1 TB of transfer. A middle tier with 2 GB of memory and 2 cores runs $12/month with 60 GB of disk and 3 TB of transfer. Those prices are honest. The problem is what happens on either side of them.

Instances and container services come as fixed bundles, and resizing means moving to a bigger plan rather than adding a component. Databases are a separate set of bundles starting at $15/month. Load balancers are $18/month. Put a container service ($7), a high-availability database ($15) and a load balancer ($18) together and you're at **$40/month before you've paid for a single compute instance**. That gap between the advertised entry price and the real monthly bill is the most common reason people start comparing alternatives.

Regional coverage is the second limit: Lightsail runs 13 data centers, split across Asia, Europe, North America and Oceania. Nothing in South America, and if your users cluster somewhere outside those 13 locations, a CDN becomes mandatory rather than optional.

The third limit is structural. SQS, Fargate autoscaling, and the full RDS feature set live outside Lightsail, so any backend that grows into queues, autoscaling or read replicas has to leave the bundle anyway.

If the fixed-bundle model is the part that bothers you, Vultr keeps the same bundled pricing idea but lets you pick from a much wider range of families and sizes, billed hourly. 👉 [Check Vultr's current Cloud Compute pricing](https://www.vultr.com/products/cloud-compute?ref=9738262-9J)

## The three kinds of alternatives

### Stay on AWS: ECS Express Mode and EC2

**ECS Express Mode** is where AWS now points new container users, largely because App Runner is closed to new customers. You hand it a container image and it creates a Fargate service with a URL, a load balancer with TLS, autoscaling and monitoring. Express Mode itself adds no charge, but you pay for the Fargate tasks, the Application Load Balancer, CloudWatch and data transfer separately.

It's the closest functional match to Lightsail's container service: instead of choosing a fixed node size and count, you specify CPU and memory and a scaling range, and tasks are added based on CPU utilization. Several Express services sharing one network configuration also share an ALB, which brings the cost down when you run more than one service.

The trade-offs are real. You lose the single bill and the bundled transfer allowance, and Express Mode only supports stateless HTTP services. Databases, buckets, cron jobs and queues all have to be wired up separately, usually with Terraform or CloudFormation and IAM roles.

**EC2** keeps the server mental model you already know: Ubuntu or Amazon Linux, SSH, systemd, nginx, a process manager. Existing Lightsail snapshots can be exported as EC2 AMIs and EBS snapshots, so you don't rebuild the box. What you gain is RDS with read replicas and Multi-AZ, S3, SQS, every service in the same VPC, and Savings Plans for steady workloads. What you lose is the flat monthly price: compute, EBS, data transfer and every public IPv4 address are billed separately, and the outbound traffic that a Lightsail bundle absorbed becomes its own line item.

### Bundled VPS: Vultr, DigitalOcean, Hetzner

**Vultr** keeps the bundled model, adds managed PostgreSQL, MySQL, Valkey and Kafka, and covers more continents than Lightsail. It's the strongest match if "same billing shape, more places to run it, real managed databases" describes what you want.

**DigitalOcean Droplets** are the plan Lightsail's model was copied from, and shared-CPU sizes sit close to Lightsail on price. As of November 2026, shared and dedicated CPU sizes go up to 48 vCPUs, managed databases cover PostgreSQL, MySQL, Valkey, MongoDB, Kafka and OpenSearch with standby nodes, read-only nodes and point-in-time recovery, Spaces adds S3-compatible object storage, and Volumes can grow without a power-down. Droplet autoscale pools scale a group of Droplets on resource usage, and load balancers, cloud firewalls and VPCs all live in the same console. A Droplet can be running in about 55 seconds. On G2 it holds roughly 4.6/5 from 749+ reviewers, and reviewers consistently describe it as easier to admin and set up than Lightsail.

**Hetzner Cloud** is the value option: servers, Volumes block storage, private networks, stateful firewalls, load balancers, snapshots and S3-compatible object storage, all manageable through a REST API, the `hcloud` CLI and a Terraform provider. Regions cover the EU, the US and Singapore. The pitch is cheap servers with large traffic allowances.

### PaaS and container platforms: Render, Fly.io, Cloud Run

**Render** deploys web services, workers and cron jobs from Git, with managed Postgres. A crontab and a background worker on a Lightsail box become separate Render services that are easier to see and restart, and scaling means raising the instance count. The catch is the filesystem: it's ephemeral by default, and a persistent disk can only attach to one instance, so a service with a disk can't scale horizontally or deploy without downtime. Apps that write uploads to local disk usually need to move to object storage first.

**Fly.io** runs Machines in regions you pick, with Fly Managed Postgres handling backups, failover, connection pooling, pgvector and PostGIS inside your organization's private network. As of November 2026 the docs list a 1 TB storage limit, no in-place major version upgrades and no metric alerts, and imports go through `pg_dump` and `pg_restore`. Volumes are tied to a single Machine, so a volume per Machine means replication you handle yourself.

**Google Cloud Run** scales containers per request, including down to zero, with Cloud SQL as the database. **Encore** lets you declare containers in code and runs them on ECS Fargate or Cloud Run with RDS or Cloud SQL Postgres, inside your own AWS or GCP account.

## Where Vultr fits, and where it doesn't

Vultr's Cloud Compute instances use the same bundled billing as Lightsail: CPU, memory, storage and transfer in one monthly price, billed hourly. What changes is the selection and the reach. Coverage spans North America, Europe, Asia, Australia and South America, and managed PostgreSQL, MySQL, Valkey and Kafka are available as first-class products rather than an add-on bundle.

The honest picture from review data: Vultr sits around **4.3/5 on G2**, and reviewers compared to Lightsail describe it as easier to manage and set up with better support. That's a review aggregate, not a benchmark. DigitalOcean scores higher on the same site, so if community ratings are your tiebreaker, DigitalOcean wins that comparison. Vultr's case rests on the regions-plus-managed-databases combination, not on being rated first overall.

For new accounts, Vultr currently lists a $300 free credit under code **FLY300VULTR**. The terms on its coupon page are specific: new customers only, credit applies to select products only, and it can't be combined with other offers. There's also a deposit-match offer (VULTRMATCH) that doubles your first deposit up to $100, which suits you better if you're funding a long-lived account rather than evaluating. 👉 [Check which Vultr credit offer applies to you](https://www.vultr.com/coupons?ref=9738262-9J)

## How the options compare

| Option | Billing model | Regions | Managed databases | Main trade-off |
| --- | --- | --- | --- | --- |
| Vultr | Bundled VM, hourly billing | North America, Europe, Asia, Australia, South America | PostgreSQL, MySQL, Valkey, Kafka | Smaller service ecosystem than AWS; you still patch the OS |
| DigitalOcean | Bundled VM, hourly billing | Multiple regions, up to 48 vCPU sizes | PostgreSQL, MySQL, Valkey, MongoDB, Kafka, OpenSearch | Same as any VPS: OS patching and deploys are yours |
| Hetzner Cloud | Bundled VM, hourly billing | EU, US, Singapore | Self-managed | Narrowest geographic spread of the three |
| ECS Express Mode | Per-resource AWS billing | AWS regions | Bring your own RDS | Stateless HTTP only; Fargate, ALB, CloudWatch and transfer billed separately |
| Render | Per-service PaaS | Render's regions | Managed Postgres | Persistent disks can't attach to more than one instance |

Which one fits:

- Running one WordPress site or a small API and want the lowest monthly number: compare same-size bundled plans at Vultr, DigitalOcean and Hetzner before anything else.
- Backend depends on SQS, full RDS features or a shared VPC: ECS Express Mode or EC2, because leaving AWS only makes sense if nothing else in your stack is AWS-native.
- Want Git push deploys and no servers to patch: Render or Cloud Run.
- Need a persistent disk and horizontal scaling at the same time: fix the storage design first, since Render's disk model blocks both.

## Migration notes before you commit

Moving a Lightsail instance to EC2 is the easy case: snapshots export directly as AMIs and EBS snapshots, so the server comes across without a rebuild.

Moving to any VPS provider, Vultr included, means rebuilding. You create the instance, install the stack with your existing scripts, dump and restore the database with `pg_dump` and `pg_restore` (or use an online migration tool), and cut DNS over. You keep patching the OS and running deployments yourself afterward. Budget an afternoon for a simple site and considerably more if there's a database with ongoing writes.

One thing to settle before the move: whether anything else in your stack talks to AWS. If your app writes to S3, publishes to SNS or reads secrets from SSM, a cheaper VPS becomes an expensive rewrite.

## FAQ

**Is the $5 Lightsail plan enough for production?**
For a low-traffic WordPress site or an internal tool, yes. At 512 MB of memory, anything running a JVM, a Node build step or a database on the same box will swap. The $12 plan with 2 GB and 2 cores is the realistic starting point for a small production app.

**What's the catch with the $3.50 IPv6-only plan?**
You get no public IPv4 address. Clients, CI runners and mail servers that only speak IPv4 can't reach the instance directly, so you need a proxy, a CDN in front, or an IPv4 address added. It's a good fit for a backend behind a load balancer, and a bad fit for a mail server.

**Why is ECS Express Mode more expensive than the Lightsail container service in practice?**
Express Mode doesn't charge for itself. You pay for Fargate task hours, the Application Load Balancer, CloudWatch and data transfer as separate line items. Sharing one ALB across several Express services is the main lever for keeping that down.

**Can I move a Lightsail snapshot to Vultr?**
Lightsail snapshots export as EC2 AMIs and EBS snapshots, which is an AWS format. On any other VPS provider the migration is a rebuild plus a database dump and restore, not an image import.

**How much is the Vultr $300 credit actually worth?**
It's promotional credit for new customers, restricted to select products and not combinable with other offers, so read the terms on the coupon page before you plan around it. It's useful for running a parallel environment for a month while you compare, less so as permanent free hosting.

## The decision, shortened

- One small site, price is the only variable: compare entry bundled plans at Vultr, DigitalOcean and Hetzner at the same size.
- Need more regions and managed PostgreSQL, MySQL, Valkey or Kafka without leaving the flat-monthly model: Vultr.
- Need SQS, RDS read replicas or a VPC: ECS Express Mode for one stateless container, EC2 if you want the server model with the full AWS catalog.
- Need Git deploys and zero server admin: Render or Cloud Run, and check the disk model before you commit.

Before you order anywhere, confirm three things on the provider's own pages: the billing period and whether the displayed price is the renewal price, what the overage rate is once you exceed the included transfer, and whether the trial or credit has an expiry date. Those three answers move the real monthly cost more than the headline number does.

👉 [Start with Vultr's Cloud Compute lineup and current pricing](https://www.vultr.com/products/cloud-compute?ref=9738262-9J)
