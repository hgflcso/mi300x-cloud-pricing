# MI300X cloud pricing: what a GPU-hour costs, who actually has stock, and will your CUDA code run? (192 GB vs H100, H200 and B200)

Oracle charges **$6.00 per GPU-hour** for an MI300X with no commitment attached. Runpod lists the same card at **$2.39 per hour**, billed by the second. Vultr's lowest published AMD rate is **$2.290 per GPU-hour**, but that number belongs to the MI355X and only holds on a 48-month reservation. Across the eight providers tracked by cloud-gpus.com on 24 October 2026, on-demand MI300X pricing ran **$1.79 to $6.00** per GPU-hour.

People search out this card for one reason: capacity. A 70B-parameter model needs roughly 140 GB just for FP16 weights. That fits on a single 192 GB MI300X with room to spare and does not fit on a single 80 GB H100. Fewer GPUs means fewer cross-card hops, which shows up in both latency and the invoice.

Below is the rental map as it actually stands, including the providers that do not sell MI300X at all, the arithmetic behind the discounted contract rates, and the question that decides most projects: whether your stack runs on ROCm.

> **Quick pick:** Vultr carries AMD Instinct MI300X, MI325X and MI355X, with self-service cluster deployment on the MI300X and MI325X side and ROCm shipped through its GPU-enabled images. Its published AMD rate starts at $2.290/GPU-hour on a reserved term, and the MI300X on-demand rate is worth confirming at checkout before you plan a budget around it.
> 👉 [See Vultr's MI300X and MI325X cluster options](https://www.vultr.com/products/cloud-gpu/amd-mi325x-mi300x/?ref=9738262-9J)

## What Vultr lists for AMD Instinct, and what each tier buys

Vultr's AMD lineup covers all three current Instinct generations, and they share the same 8-GPU system design, so moving up a tier is mostly a memory and bandwidth decision rather than a platform migration.

| GPU | Memory | Published rate | Terms and stock notes | Link |
| --- | --- | --- | --- | --- |
| AMD Instinct MI300X | 192 GB HBM3, 5.3 TB/s | No on-demand figure published on the pages verified here | Self-service cluster deployment, no special request required per Vultr's product page | [Check MI300X availability](https://www.vultr.com/products/cloud-gpu/amd-mi325x-mi300x/?ref=9738262-9J) |
| AMD Instinct MI325X | 256 GB HBM3E, 6.0 TB/s | Not verified | Same self-service cluster path, ROCm images included | [Check MI325X availability](https://www.vultr.com/products/cloud-gpu/amd-mi325x-mi300x/?ref=9738262-9J) |
| AMD Instinct MI355X | 288 GB HBM3E, 8.0 TB/s | $2.290/GPU-hour | 48-month reservation, billed at 730 hours per month; site wording is "reserve now" | [Check MI355X reservation terms](https://www.vultr.com/products/cloud-gpu/amd-mi325x-mi300x/?ref=9738262-9J) |

Two details matter more than the table. First, Vultr's cluster product ships with a Slurm or Kubernetes scheduler and an observability suite, so multi-node work does not require you to build the orchestration layer yourself. Second, the reservation language is not uniform across the lineup: the MI355X is marketed as something you reserve, while the MI325X and MI300X cluster page states that deployment needs no reservations or special requests. Stock and lead time still vary by region, and that is a question for the checkout page rather than the brochure.

A note on the MI300X line price: one third-party rate tracker lists Vultr's MI300X at $3.99/GPU-hour, a figure I could not confirm against Vultr's own pages. Treat any MI300X number you see for this provider as a starting point to verify, not a quote. 👉 [Check the live MI300X rate on Vultr](https://www.vultr.com/products/cloud-gpu/amd-mi325x-mi300x/?ref=9738262-9J)

Vultr also lists NVIDIA accelerators, including H100, B200, A100, L40S and GH200, through the same console and API. I am not quoting their rates here because I could not verify them against a current official price page, and a wrong GPU-hour number is worse than no number.

## 192 GB is the entire argument: how the cards stack up on memory

| GPU | Memory per GPU | Bandwidth | What it changes |
| --- | --- | --- | --- |
| NVIDIA H100 SXM | 80 GB | 3.35 TB/s | Two cards for a 70B FP16 model |
| NVIDIA H200 | 141 GB | 4.8 TB/s | Still short of 192 GB for the largest single-card fits |
| NVIDIA B200 | ~175 GB | Not compared here | Figure derived from a 1.4 TB eight-GPU node |
| AMD MI300X | 192 GB HBM3 | 5.3 TB/s | One card for a 70B FP16 model |
| AMD MI325X | 256 GB HBM3E | 6.0 TB/s | Headroom for longer context and KV cache |
| AMD MI355X | 288 GB HBM3E | 8.0 TB/s | Largest single-card pool in this group |

The B200 number is a per-GPU average taken from NVIDIA's stated 1.4 TB for an eight-GPU node, not a datasheet figure for a standalone card. The MI300X figure is 2.4 times the H100's, and that ratio, not raw FLOPS, is what most buyers are actually paying for: it decides whether a model runs on one accelerator or has to be sharded across several.

Oracle's own validation gives a sense of what that looks like in production. On its MI300X bare-metal shape, time to first token stayed within 65 milliseconds, average latency was 1.5 seconds at batch size 1, and throughput scaled to a maximum of 3,643 tokens across 256 concurrent requests.

The card itself is an OAM module drawing up to 750W with passive cooling, sitting eight to a UBB 2.0 board. It is not something you install in a workstation, which is the practical reason renting beats buying for anyone testing the platform.

## Who sells MI300X, and who quietly doesn't

| Provider | Offering | Status | Published rate |
| --- | --- | --- | --- |
| Oracle Cloud | BM.GPU.MI300X.8, 8× MI300X, 1.5 TB HBM3 total, 2× 56-core Sapphire Rapids, 2 TB DDR5 | Generally available since 26 Sep 2024; Supercluster scales to 16,384 MI300X | $6.00/GPU-hour, no stated commitment |
| Microsoft Azure | Standard_ND96isr_MI300X_v5, 8× MI300X, 96 vCPU, 1,850 GiB RAM, 400 Gbps InfiniBand per GPU (3.2 Tb/s total) | Available | Not listed on the page checked |
| IBM Cloud | 8× MI300X, 208 vCPU, 1,792 GB RAM, 8× 3.2 TB storage | Select availability as of October 2025, not general availability | Not listed |
| Amazon Web Services | None | No AMD Instinct instance type in the EC2 catalogue | — |
| Google Cloud | None found | Not in the Compute Engine catalogue | — |
| Vultr | MI300X, MI325X, MI355X | Listed, cluster deployment self-service | From $2.290/GPU-hour (MI355X, 48-month term) |
| Runpod | MI300X on Secure Cloud | Listed, per-second billing, no minimum | $2.39/hour |
| TensorWave | MI300X through MI455X | Listed | $2.95/GPU-hour for MI355X, term not stated |

The two absences are the part most comparisons skip. If your procurement runs through AWS or Google Cloud, you cannot rent an MI300X there today, full stop. IBM's entry exists but sits at select availability, so capacity is not something to design a schedule around. That leaves Oracle as the hyperscaler option, Azure as the second, and a group of neoclouds handling everyone else.

## The real cost of a cheap contract rate

The headline gap between $2.290 and $6.00 looks enormous. Run the arithmetic and it shrinks to something you can actually judge.

Vultr's $2.290/GPU-hour, billed at 730 hours a month, is **$1,671.70 per GPU per month**. Over the full 48-month term that is **$80,241.60 per GPU**, or roughly **$641,900 for an eight-GPU node**. Note the term applies to the MI355X, not the MI300X.

Oracle at $6.00/GPU-hour with no commitment is **$4,380 per GPU per month**. Four years at that rate would be $210,240 per GPU, which is why the reservation looks so attractive on paper.

The comparison is not apples to apples in two ways. Oracle's number covers an MI300X, one generation below the MI355X in the Vultr rate, so part of the gap is silicon and part is commitment. And the commitment is the real cost: $80,241.60 per GPU is owed whether the card runs or sits idle, and AMD's roadmap does not pause for your four-year term. TensorWave already advertises coverage from MI300X up to MI455X, which tells you how quickly this stack is moving.

So the rule is conditional, not general. If you are certain the workload stays on MI355X-class hardware for four years, the reserved rate cuts your effective hourly cost by roughly 62% against Oracle's on-demand price. If there is any chance you migrate generations or your job mix changes, the zero-commitment hourly rate is cheaper than a contract you have to escape.

## Will your code run? The ROCm boundary

Framework-level code usually moves. Hand-written CUDA kernels usually don't.

AMD's compatibility matrix for Instinct lists official install paths for **PyTorch, JAX, vLLM and SGLang**, and supported operating systems across Ubuntu, RHEL, Debian, Oracle Linux, Rocky Linux and SLES. Instinct support is Linux only; there is no Windows or WSL2 entry in that table. TensorFlow and Triton do not appear in the Instinct compatibility list, which matters if either is load-bearing in your pipeline.

For most teams the practical split looks like this: training and inference built on PyTorch plus vLLM tends to port with little more than a container swap, because AMD ships its own builds rather than relying on community forks. Custom CUDA kernels, and libraries with no ROCm equivalent, need a rewrite and engineering time you should budget before signing anything.

One version caveat. ROCm release numbering has moved quickly, and published compatibility tables are not always in step with the images providers actually deploy. Rather than trusting a version number in a blog post, ask what ROCm version ships inside the image and check it against the framework build you have pinned. On Vultr, ROCm arrives through GPU-enabled images, so that is a question you can answer from the console before you commit to a cluster.

The cheapest way to settle it is to rent one card for an afternoon and run your own workload. Per-second and per-hour billing exist precisely so that this test costs tens of dollars instead of a procurement cycle. 👉 [Rent a single MI300X on Vultr and test your stack](https://www.vultr.com/products/cloud-gpu/amd-mi325x-mi300x/?ref=9738262-9J)

## Why you won't find a trustworthy MI300X vs H100 speed number

AMD does not publish independent benchmarks for the MI300X, and every performance figure in circulation traces back to a vendor. Vultr's own datasheet quotes 2,614.9 TFLOPS FP8 and 5,229.8 TFLOPS with structured sparsity. AMD's marketing tables mix FP16, FP8, FP6 and FP4 results, sometimes sparse and sometimes dense, without labelling them consistently enough to compare against NVIDIA's published numbers on equal footing. One published analysis declined to make an AMD-versus-NVIDIA compute comparison for exactly that reason, and compared memory only.

Crusoe's "up to 13.7× AI/ML performance" claim is measured against the MI250X, two AMD generations back, not against an H100. It is a legitimate number and a useless one for cross-vendor shopping.

What you can compare safely is capacity, bandwidth, power, form factor and price. Those are the numbers this article uses, and they are the ones that decide whether your model fits on one card.

## Which provider fits which job

**Inference on a model that won't fit in 80 GB.** One MI300X replaces two H100s. Oracle's $6.00 hourly rate with zero commitment is the low-risk way to prove it, and the Supercluster option goes to 16,384 GPUs if the pilot becomes a product.

**Multi-node work with a scheduler you don't want to build.** Vultr's cluster product bundles Slurm or Kubernetes plus monitoring, and the MI300X/MI325X side deploys self-service. 👉 [Browse all Vultr GPU options](https://bit.ly/VuLtr)

**Short experiments and CI-style testing.** Runpod's per-second billing and $2.39 hourly rate on Secure Cloud suit throwaway runs where you care about the bill for 40 minutes of work.

**A fixed four-year training budget on current silicon.** The reserved MI355X rate is the cheapest published number in this group, provided you can genuinely commit for the full term.

**A stack built on hand-written CUDA kernels, TensorFlow or Triton-heavy tooling.** Stay on NVIDIA. The porting cost will exceed whatever you save per GPU-hour, and no amount of memory capacity fixes a missing library.

## Five questions to settle before you sign

1. What is the on-demand rate with no contract attached?
2. What is the shortest term that qualifies for the discounted rate, and what does early termination cost?
3. Can you get hardware now, or is the listing a reservation with a lead time?
4. Which ROCm version ships in the image, and does it match your pinned framework build?
5. Will the provider let you benchmark your own workload before you commit?

Ask the fifth one first. A provider that says yes is telling you something the price list cannot.

If you want to run that test on AMD hardware without a sales call, start with a single card and a real workload rather than a synthetic benchmark. 👉 [Start with Vultr's AMD Instinct GPU page](https://www.vultr.com/products/cloud-gpu/amd-mi325x-mi300x/?ref=9738262-9J)
