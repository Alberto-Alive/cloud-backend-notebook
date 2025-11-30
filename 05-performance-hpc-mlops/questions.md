
---

## 1. General Performance Mindset

### Q1. When someone says “the system is slow”, what are the first things you want to clarify?

```text
I’d clarify:

Where it feels slow (which endpoints / features).

How it’s slow: high latency, errors, timeouts?

When it happens: all the time or only under peak load?

Scale: how many users / requests?

Any recent changes (deployments, config changes).
Then I’d look at metrics/logs to confirm objectively where the bottleneck is.
```

---

### Q2. How do you generally approach performance optimisation in an application?

```text
Measure: use metrics and profiling to find the real bottleneck.

Hypothesise & fix: optimise the slowest part first (queries, code, caching).

Verify: re-measure to see if latency/throughput improved.

Repeat until performance meets the target or further gains aren’t worth the complexity.
```

---

### Q3. What’s the difference between latency and throughput?

```text
Latency: how long a single request takes end to end.

Throughput: how many requests per second the system can handle.
```

---

### Q4. What are some common causes of poor performance in backend systems?

```text
Common causes:

Slow or unindexed database queries.

N+1 query patterns.

Lack of caching.

Network latency / too many remote calls.

Inefficient algorithms and unnecessary work.

Poor use of concurrency (blocking on I/O, oversized payloads).
```

---

## 2. Profiling & Identifying Bottlenecks

### Q5. What tools or techniques can you use to find performance bottlenecks in code?

```text
Profilers: cProfile, py-spy, line-profiler to see where CPU time goes.

Tracing: logs or distributed traces to see slow steps.

DB tools: slow query logs, EXPLAIN.

Load testing (k6, Locust) to reproduce issues under load.
```

---

### Q6. How would you profile a Python backend to see where it spends most of its time?

```text
I’d run the backend under a profiler like cProfile or py-spy, exercise a typical workload, and then inspect which functions and queries consume the most time. I’d also check database slow-query logs.
```

---

### Q7. Once you’ve found a hot function that’s slow, what kinds of optimisations might you consider?

```text
Check if there’s a more efficient algorithm or data structure.

Avoid repeated work (cache results, precompute).

Reduce I/O in the hot path (fewer DB calls).

Vectorise operations or move heavy loops to C/NumPy if numeric.

Only consider async if the function is I/O-bound, not CPU-bound.
```

---

## 3. Concurrency, Parallelism & Async

### Q8. What is the difference between concurrency and parallelism?

```text
Concurrency: dealing with multiple tasks at once by interleaving them (they appear to run at the same time, even on one core).

Parallelism: tasks literally run at the same time on multiple cores/CPUs.
```

---

### Q9. In a Python web backend, when would you consider using async I/O instead of threads?

```text
I’d consider async I/O when the workload is mostly waiting on I/O: HTTP calls, DB, file operations, etc. Async lets a single thread handle many connections efficiently.

Threads are usually fine for blocking I/O too, but in Python they have more overhead and are limited by the GIL for CPU-bound work. For heavy CPU-bound tasks I’d use processes, not async.
```

---

### Q10. Why can I/O-bound workloads benefit from concurrency even on a single CPU core?

```text
Because while one request is waiting on I/O (DB, network), the CPU is idle and can work on another request. Even on a single core, you can interleave tasks while they’re waiting, which improves throughput.
```

---

### Q11. Give an example of a CPU-bound task and an I/O-bound task in typical backend work.

```text
CPU-bound: heavy JSON serialization of huge payloads, image processing, large in-memory computations.

I/O-bound: calling an external API, reading/writing to DB, reading files from disk or object storage.
```

---

## 4. Performance Patterns (Batching, Caching, Vectorisation)

### Q12. What is batching and why can it improve performance?

```text
Half-right; key idea is fewer, larger operations:

Batching means combining many small operations into one bigger operation.

e.g. one SQL query for 100 rows instead of 100 separate queries.

e.g. sending 128 inputs to a model at once.
It improves performance by reducing overhead per operation and often letting the underlying system optimise better.
```

---

### Q13. When would you choose to cache results vs recompute them each time?

```text
I’d cache when:

the result is expensive to compute or fetch,

it’s requested frequently,

and can tolerate some staleness or can be invalidated properly.
```

---

### Q14. What does “vectorisation” mean in the context of numeric computing (e.g. NumPy)?

```text
Vectorisation means expressing operations on whole arrays instead of writing Python loops, so the heavy work runs in fast C code, often using SIMD instructions internally.
```

---

### Q15. Why is looping in pure Python often slower than using vectorised operations?

```text
Because each Python loop iteration has a lot of interpreter overhead (bytecode, dynamic types). Vectorised operations move the loop into optimised C code that runs tight loops over raw memory, which is much faster.
```

---

## 5. HPC & Scaling Compute (Conceptual)

### Q16. What kinds of problems are typically suited for HPC (High-Performance Computing) or clusters?

```text
heavy compute ones, like scientific computational ones
```

---

### Q17. What is the basic idea behind data parallelism vs task parallelism?

```text
Data parallelism: same computation on different chunks of data (e.g. split dataset across workers).

Task parallelism: different tasks/operations run in parallel (e.g. pipeline stages or independent jobs).
```

---

### Q18. Why is network communication often a bottleneck in distributed/HPC workloads?

```text
because data exchanges explodes as the computation continues and the bandwidth struggles to keep pace 
```

---

### Q19. How would you roughly explain the difference between scaling up on a single powerful machine vs scaling out across many machines?

```text
give the app more access to resources while the other is add new machines to the cluster
```

---

## 6. GPU vs CPU (Conceptual)

### Q20. Why are GPUs often used for ML workloads?

```text
they can run matrix multiplications because they are done in parallel
```

---

### Q21. What kinds of operations run well on GPUs, and which are less suited?

```text
GPUs: great for massively parallel, numeric operations (matrix multiplies, convolutions).

Less suited: branchy, irregular control flow, small tasks with lots of overhead, heavy CPU-only system tasks.
```

---

### Q22. What is the downside of always “just using GPUs” for everything?

```text
Expensive hardware.

Higher operational complexity (drivers, CUDA, scheduling).

For small workloads, startup and transfer overhead can dominate.

Some workloads just don’t map well to GPU, so you pay more without benefit.
```

---

## 7. ML Systems Basics

*(Even if you’re not an ML engineer, it’s useful to speak high-level.)*

### Q23. In an ML project, what is the difference between training and inference?

```text
training updates the weights but inference doesn't
```

---

### Q24. Where might ML models live in a production system, and how are they typically called?

```text
be embedded in the main backend service,

or be served by a separate model server (e.g. FastAPI/Flask service, TensorFlow Serving, TorchServe).
The backend calls them via:

function calls (if embedded),

or HTTP/gRPC requests (if separate service).
```

---

### Q25. What are some common patterns for serving ML models (e.g. in a backend API)?

```text
Online prediction API:

Model is loaded in memory in a service; API receives requests, runs inference, returns results.

Batch prediction:

Periodic jobs run the model over large datasets and store outputs for later use.

Streaming:

Model consumes events from a stream (Kafka) and produces predictions in real time.
```

---

## 8. MLOps: Pipelines, Versioning, Deployment

### Q26. What does “MLOps” mean to you?

```text
automating data pipelines, training, evaluation, deployment, monitoring,

managing versions of data, code, and models,

keeping models reliable in production.
```

---

### Q27. What are typical stages in a basic ML pipeline?

```text
Data collection and labelling.

Data cleaning and feature engineering.

Training models on historical data.

Evaluation and model selection.

Packaging and deploying the best model.

Monitoring performance and retraining when needed.
```

---

### Q28. Why is versioning important in ML (for data, code, and models)?

```text
so you can benchmark, observe and deploy new features
You need to know exactly:

which data,

which code,

and which model version
produced which behaviour, so you can reproduce bugs, compare models, roll back, and audit decisions.
```

---

### Q29. How might you deploy a trained model so that a backend service can use it?

```text
Package the model and code into a Docker image, deploy as an API service.

Use a model server (TensorFlow Serving, TorchServe) and point it at your model file.

Or embed the model in an existing backend service if latency requirements are strict.
```

---

### Q30. What are some ways to roll out a new model version safely?

```text
Run A/B tests or canary:

send a small percentage of traffic to the new model, compare metrics.

Shadow mode:

run new model in parallel, don’t show results, just compare offline.

Keep ability to quickly roll back to previous model version.
```

---

## 9. Monitoring ML in Production

### Q31. What would you monitor for an ML-powered endpoint in production, beyond normal API metrics?

```text
Input data characteristics (distributions, missing values).

Model output distribution (are predictions drifting?).

Business KPIs affected by the model (conversion rate, click-through, etc.).

Feedback / labelled data if available (accuracy, precision/recall over time).

Resource usage (GPU/CPU utilization, latency per prediction).
```

---

### Q32. What is data drift / concept drift, in simple terms?

```text
Data drift: the input data distribution changes over time (e.g. user behaviour changes, new kinds of inputs).

Concept drift: the relationship between input and correct output changes (e.g. what “fraud” looks like evolves).
```

---

### Q33. What could you do if you suspect that a model’s performance in production is degrading over time?

```text
Check if input data has drifted; monitor distributions.

Re-evaluate the model on fresh labelled data.

Retrain with more recent data, maybe adjust features.

If needed, roll back to a previous, better-performing model while you investigate.
```

---

## 10. Performance & Cost in Cloud / ML Context

### Q34. How do performance and cost interact in cloud environments?

```text
more performance (bigger machines, more replicas, GPUs) → more cost.
But:

better optimised code (less waste) can improve performance and reduce cost.
Key is to balance meeting SLAs with avoiding overprovisioning.
```

---

### Q35. Why might you prefer to batch predictions instead of doing them one by one?

```text
Better throughput: hardware (especially GPUs) is more efficient on batches.

Less overhead per request: fewer network calls, fewer context switches.

Can reduce cost by making better use of each CPU/GPU invocation.
```

---

### Q36. How can autoscaling help for ML inference services?

```text
scales up instances when traffic increases, preventing high latency,

scales down when demand drops, saving cost.
```

---

## 11. Scenario Questions

### Q37. You have an API endpoint that runs a heavy computation and takes ~5 seconds to respond, causing timeouts for some users.

What are some strategies to improve the situation?

```text
Profile to see where time is spent.

Optimise algorithm / queries.

Cache results if repeated.

Offload to async job + polling/notification if user can wait.

If truly CPU-bound and must be synchronous, consider using more workers or separate compute service.
```

---

### Q38. A batch job (e.g. nightly data processing) is starting to overflow its time window because the dataset is growing.

How would you approach optimising or redesigning it?

```text
Optimise queries and processing (indexes, better algorithms).

Use batching/parallelism across multiple workers or machines.

Process incrementally (only new/changed data) instead of full recompute.

Scale up or out as needed after optimisations.
```

---

### Q39. A real-time recommendation endpoint calls a model server which sometimes takes too long and slows down your API.

How could you architect around this?

```text
Add caching for common recommendations.

Use async calls with timeouts and fallbacks (e.g. default recommendations).

Precompute recommendations in a batch job where possible.

Scale model server separately and maybe co-locate it to reduce network latency.
```

---

### Q40. You’re asked to “make this ML system faster” without losing quality. What questions would you ask first?

```text
Which part is slow: data loading, feature extraction, model inference, networking?

What are the SLOs: target latency and throughput?

What hardware we’re currently using (CPU vs GPU)?

How often is the model called and by whom?

Are there quality constraints we can’t violate?
```

---

