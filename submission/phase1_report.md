# Phase 1 — Comparison Report

Copy this file to `submission/phase1_report.md`, fill in every answer slot, and commit it:

```bash
cp phase-1-echo-bot/report_template.md submission/phase1_report.md
```

**How to fill it in.** Each answer sits between a pair of `<!--answer:...-->` markers. Replace
the `TODO` line with your answer — **leave the markers themselves untouched**, they are how the
grader finds your answers. Anything you write outside the markers is ignored, so add extra
prose, tables or images freely.

Numbers must **match `submission/phase1_report.json`** exactly — that file is produced by
`measure.py` and is what your figures are checked against. Copy them across; do not retype from
memory.

---

## 1. Setup facts

**Your image size**, in MB, from `gcloud artifacts docker images list` (number only, e.g. `142`):

<!--answer:image_size_mb-->
48.98
<!--/answer-->

**The custom VPC and subnet you created** (names only):

<!--answer:vpc_name-->
echo-net
<!--/answer-->

<!--answer:subnet_name-->
echo-subnet
<!--/answer-->

---

## 2. Measurements

Copy these from `submission/phase1_report.json`. Cold is `cold_ms`; warm median is
`stats.median`. Numbers only, in milliseconds (e.g. `812.4`).

| Platform | Cold (ms) | Warm median (ms) |
|---|---|---|
| IaaS VM | <!--answer:vm_cold_ms-->255.3<!--/answer--> | <!--answer:vm_warm_median_ms-->213.2<!--/answer--> |
| Cloud Run | <!--answer:cloudrun_cold_ms-->1865.8<!--/answer--> | <!--answer:cloudrun_warm_median_ms-->324.7<!--/answer--> |
| Cloud Function | <!--answer:function_cold_ms-->1569.9<!--/answer--> | <!--answer:function_warm_median_ms-->137.9<!--/answer--> |

Include your histogram or plot of the warm distribution below (an image, or the ASCII chart
`measure.py` prints). This is read by a human, not the grader:

<!--answer:latency_plot-->
```text
Warm response time distribution (ms); # = 1 request
vm (n=20)
   200- 299 ms | #################### (20)
cloudrun (n=20)
   100- 199 ms | ########## (10)
   500- 599 ms | # (1)
   600- 699 ms | ######### (9)
function (n=20)
   100- 199 ms | ############# (13)
   600- 699 ms | ####### (7)
```
<!--/answer-->

---

## 3. Per-platform analysis

**Scaling behaviour.** Imagine requests arriving one at a time, then ten at once, then a
hundred. For each of the three platforms, answer concretely:

- Does anything get created automatically to absorb the load — and if so, *what* exactly?
  (One platform here creates nothing at all.)
- What is the hard limit, and where does it come from — a flag you passed yourself, or the
  size of the machine you picked?

(~80 words. Your `--machine-type=e2-micro` and `--max-instances=3` are both concrete answers
to the second question.)

<!--answer:scaling_behaviour-->
The VM runs on one e2-micro machine. No additional VM is created automatically as requests increase. Its CPU and memory limit its capacity. Cloud Run creates container instances as needed and can scale to zero. We set a maximum of 3 instances. Cloud Function also creates instances as needed and can scale to zero. Its underlying service shows a maximum of 12 instances and handles one concurrent request per instance. Requests may wait or fail when capacity is exhausted.
<!--/answer-->

**Deployment effort.** What did you have to build, configure and maintain to get each of the
three running? (~100 words)

<!--answer:deployment_effort-->
For the VM, we created a VPC, subnet, firewall rules and a Debian instance. We granted registry access, connected with SSH, installed and authenticated Docker, then pulled and started the image. The firewall uses the echo-http tag to apply the rules to the VM. We must maintain the VM and its software. For Cloud Run, we provided the existing image and set the region, port, scaling limits and public access. Google manages the servers. For Cloud Function, we provided the source code, Python runtime, entry point and HTTP trigger. Google built the image and managed the execution environment. We also granted the build account access to the source bucket and image repository.
<!--/answer-->

---

## 4. Why do two of your three numbers look alike?

With Cloud Run and your Function both deployed, run `gcloud run services list` and paste the
**complete output** here, unedited:

<!--answer:run_services_list-->
```text
✔
SERVICE: echo
REGION: us-central1
URL: https://echo-468962333350.us-central1.run.app
LAST DEPLOYED BY: service-468962333350@gcf-admin-robot.iam.gserviceaccount.com
LAST DEPLOYED AT: 2026-10-29T21:13:12.090895Z

✔
SERVICE: echo-bot
REGION: us-central1
URL: https://echo-bot-468962333350.us-central1.run.app
LAST DEPLOYED BY: wonseok.choi@eurecom.fr
LAST DEPLOYED AT: 2026-10-29T20:58:51.012758Z
```
<!--/answer-->

Now explain what that output tells you about how your Function is actually being run, why its
cold start resembles Cloud Run's so closely, and what — if anything — genuinely differs between
the two deployment paths. (~120 words)

<!--answer:convergence_explanation-->
The service list shows both echo-bot and echo. The echo service is created for our second generation Cloud Function, so both paths run on Cloud Run. Both can scale to zero and need to start a container to serve a later request. This may explain why their first requests took 1865.8 ms and 1569.9 ms, while the continuously running VM took 255.3 ms. The VM was already running, so its first request is not a meaningful cold start. One measurement does not prove the cause of the difference. The deployment paths remain different. We supplied a Docker image to Cloud Run. We supplied Python source and an entry point to Cloud Function, and Google built its image.
<!--/answer-->
