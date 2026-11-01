# Phase 2 — Reflection: where does parallelism stop helping?

Copy this file to `submission/phase2_reflection.md`, fill in every answer slot, and commit it:

```bash
cp phase-2-tiny-gpt/reflection_template.md submission/phase2_reflection.md
```

**How to fill it in.** Each answer sits between a pair of `<!--answer:...-->` markers. Replace
the `TODO` line with your answer — **leave the markers themselves untouched**, they are how the
grader finds your answers. Anything you write outside the markers is ignored, so add extra
prose, tables or your threading plot freely.

Numbers in section 1 must **match `submission/phase2_report.json`** — copy them across, do not
retype from memory; they are checked against it.

---

## 1. Your measurements

| | Value |
|---|---|
| Naive attention, ms (`attention.naive_ms`) | <!--answer:attn_naive_ms-->393.482<!--/answer--> |
| Vectorized attention, ms (`attention.vectorized_ms`) | <!--answer:attn_vectorized_ms-->1.126<!--/answer--> |
| CPU training, ms for 100 steps (`devices.cpu_ms`) | <!--answer:cpu_ms-->409601.3<!--/answer--> |
| GPU training, ms for 100 steps (`devices.gpu_ms`) | <!--answer:gpu_ms-->12836.5<!--/answer--> |
| Final training loss (`training.final_loss`) | <!--answer:final_loss-->1.4353<!--/answer--> |

**The thread sweep.** Give your four timings from `threads` — 1, 2, 4 and 8 threads — and say
which count was fastest. (verbatim: the numbers, in order)

<!--answer:thread_sweep-->
1 thread: 2355.948 ms
2 threads: 2188.014 ms
4 threads: 2353.202 ms
8 threads: 2267.554 ms

The fastest configuration was 2 threads (2188.014 ms).
<!--/answer-->

---

## 2. Intra-node parallelism, three ways

**Why vectorizing wins.** Both attention implementations do the same arithmetic and produce the
same result (the unit test checks that). Explain where the naive version's time actually goes,
and what the vectorized one does differently — in terms of the machine, not the Python.
(~70 words)

<!--answer:vectorization_why-->
The naive approach spends a lot of time repeatedly running small operations, reading data, processing it, and storing the results. On the other hand, the vectorized approach uses SIMD instructions to process multiple values at once. It also uses optimized matrix operations to handle data more efficiently. So even though both approaches do the same calculations, vectorization is much faster because it makes better use of the CPU's hardware.
<!--/answer-->

**Where the thread curve flattens, and why.** Using your sweep: more threads stopped helping —
or started hurting — at some point. Say where, and give the reason. Your Colab runtime's vCPU
count is part of the answer; so is what the threads have to coordinate on. (~80 words)

<!--answer:threading_plateau-->
My Colab runtime had 2 vCPUs. Using just 1 thread didn't fully use the available CPU resources, while 2 threads gave the best result. With 4 or 8 threads, performance got worse because there were more threads than available vCPUs. This can cause context switching (switching between threads), synchronization overhead (threads waiting for each other), and memory contention (threads competing for shared memory resources). So, adding more threads doesn't always mean better performance.
<!--/answer-->

**What made the GPU faster.** The model, the data and the step count were identical; only the
device changed. Say what about a transformer training step suits a GPU, and name one workload
that would *not* see this speed-up. (~70 words)

<!--answer:gpu_why-->
Transformer attention involves a lot of matrix multiplications. GPUs use SIMT, where many threads run the same instructions on different data in parallel. For example, each thread can calculate a different element of the result matrix, which makes GPUs great for large matrix operations.

On the other hand, sequential tasks with strong data dependencies need to wait for previous calculations to finish. This limits parallelism, so GPUs may not provide much speed-up for those tasks.
<!--/answer-->

---

## 3. The question

**Where does parallelism stop helping, and why?** Pull the three experiments together. Your
answer should distinguish the *kinds* of limit you met — work that cannot be divided, hardware
you do not have, coordination that costs more than it saves, data that has to move — and say
which one bound each experiment. One concrete number from section 1 per claim. (~150 words)

<!--answer:parallelism_limit-->
In the vectorization experiment, naive attention took 393.482 ms, while vectorized attention took only 1.126 ms, giving a 349.45x speed-up. However, strong data dependencies can limit parallelism because each calculation may need to wait for the previous result.

In the CPU threading experiment, 2 threads were the fastest at 2188.014 ms. Since my Colab runtime had only 2 vCPUs, adding more threads didn't provide more hardware resources. Instead, overhead from context switching, synchronization, and memory contention could outweigh the benefits.

In the GPU experiment, training took 409601.3 ms on the CPU but only 12836.5 ms on the GPU, making it about 31.9x faster. However, tasks with strong data dependencies cannot fully use GPU parallelism. Also, moving data between CPU and GPU memory takes time, which can outweigh the speed-up for small workloads.
<!--/answer-->

**What you would measure next.** One experiment this phase did not run that would sharpen the
answer, and what result would change your mind. (~50 words)

<!--answer:next_experiment-->
Next, I would like to test how data dependencies affect vectorization and GPU performance. I would compare tasks with low data dependencies and tasks that heavily depend on previous results, measuring their execution times using both CPU vectorization and GPU parallelism.

My hypothesis is that both would lose most of their speed-up with strong data dependencies. If they still achieve significant speed-ups, my hypothesis would be challenged.
<!--/answer-->
