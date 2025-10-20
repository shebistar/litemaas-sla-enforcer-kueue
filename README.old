# `LiteMaaS-SLA-Enforcer-Kueue`

**Theme:** **LiteMaaS (or MaaS) Capacity Manager: Guaranteeing AI/ML Service Level Agreements (SLAs) through Dynamic Resource Tiers and Preemption**.

## 💡 Overview and Business Value

This project provides an automated, repeatable demonstration of how to centralize, monetize, and control expensive AI compute infrastructure (like GPUs and specialized nodes) using the **Red Hat build of Kueue Operator** on **Red Hat OpenShift AI (RHOAI)**.

The MaaS concept centralizes AI infrastructure and delivers models via API endpoints, thereby simplifying consumption for end users and allowing centralized teams to optimize hardware utilization and enforce control. This demo shows the underlying technical enforcement mechanism required by the central **MaaS Provider**.

### Business Value: Guaranteeing the LiteMaaS Model

Organizations often struggle with fragmented AI deployments, leading to **idle GPUs** and soaring operational costs. This solution solves this critical resource management challenge by demonstrating:

| MaaS Challenge (Business Goal) | Kueue Solution (Demo Action) |
| :--- | :--- |
| **Monetization & SLAs (LiteMaaS):** Offering guaranteed vs. best-effort service levels. | **Pricing Tiers:** Define segregated `ClusterQueues` with `nominalQuota` to guarantee capacity for premium users (**Reserved Tier**). |
| **SLA Enforcement & Control:** Guaranteeing mission-critical workloads always run. | **Preemption:** Use a high-priority job (`prod-priority`) representing a critical MaaS deployment to automatically **evict** a running low-priority job (`dev-priority`) when resources are required. |
| **Efficient Utilization:** Allowing development teams to borrow idle resources safely. | **Cohort Sharing:** Enable teams (`team-a` and `team-b`) to draw from a `shared-cq` (Cohort) for generalized resources like CPU/Memory when available. |

## 🚀 DemoJam Alignment

This solution is designed to meet all DemoJam submission requirements:

1.  **Alignment to Red Hat TDPs:** Directly supports **AI/ML** (managing distributed RayClusters and LLM infrastructure) and **Hybrid Cloud Management** (policy and quota enforcement on OpenShift).
2.  **Utilization of Multiple Red Hat Products:** Leveraging **OpenShift**, **OpenShift AI (RHOAI)**, the **Red Hat build of Kueue Operator**, and **Ray/RayCluster** workloads. Visualization is provided by **Kueue Visualization (KueueViz)**.
3.  **Real World Business Value:** Solves the problem of **monetizing and guaranteeing Service Level Agreements (SLAs)** for expensive AI compute. This capability is highly reusable **across geos and customers** in critical sectors like financial services or scientific research.
4.  **Exciting and Eye Catching:** The demonstration hinges on the **live preemption scenario**, where a running, lower-priority workload is instantly evicted to satisfy a higher-priority SLA request, a powerful visual proof of policy enforcement.
5.  **Easy to Reproduce:** The entire setup is defined by YAML and is designed for automated deployment and execution on the **Red Hat Demo Platform (RHDP)**.

## 🛠️ Prerequisites and Setup

This demonstration requires the following components to be installed on your cluster:

1.  **Red Hat OpenShift** (Platform).
2.  **OpenShift AI Operator (RHOAI)**.
3.  **Red Hat build of Kueue Operator** (Installed globally with preemption policy set to `Classical`).
4.  An OpenShift cluster with at least one **GPU worker node** that has been appropriately tainted (`nvidia.com/gpu=Exists:NoSchedule`).

### Installation Steps (Referencing Lab Guides)

Follow the initial setup steps from the Kueue setup lab:
1.  **Install Kueue Operator:** Ensure the `openshift-kueue-operator` namespace and the required `Kueue` CR are deployed to enable preemption.
2.  **Install Kueue Visualization (Optional but recommended for DemoJam):** Deploy KueueViz for visual insight into queuing and resource allocation. Access requires port-forwarding as outlined in the installation guide.

## 🎬 Demo Execution: SLA Enforcement

This execution sequence uses the configurations outlined in the "Advanced GPU Quota Management and Preemption" lab guide.

### Step 1: Configure MaaS Tiers and Quotas

Apply the configuration files (`setup.yaml`) that define the multi-team environment. This configuration establishes the service tiers, priority classes, and cohort sharing:

| Component | Configuration | Purpose | Source |
| :--- | :--- | :--- | :--- |
| **Namespaces** | `team-a`, `team-b` | Teams representing Reserved and On-Demand users. | |
| **Priorities** | `prod-priority` (1000), `dev-priority` (100) | Enforce MaaS SLA tiers. | |
| **Cohorts** | `team-ab` (Shared Cohort) | Allows `team-a` and `team-b` to borrow unused CPU/Memory capacity from the shared pool. | |
| **Reserved Tier CQ** | `team-a-cq` | Guarantees quota (`nominalQuota: "1"` for GPU). Set to reclaim within cohort (`reclaimWithinCohort: Any`). | |
| **On-Demand Tier CQ** | `team-b-cq` | Has no guaranteed GPU quota (`nominalQuota: "0"`). | |

```bash
# Apply namespaces, CQs, LQs, ResourceFlavors, and Priority Classes
oc apply -f setup.yaml
oc get cq # Verify shared-cq, team-a-cq, and team-b-cq are Active
```

### Step 2: Deploy Low-Priority (On-Demand) Workload

Simulate the On-Demand customer (`team-b`) deploying a non-critical workload (a `RayCluster` using `dev-priority`). This job requests CPU/Memory and consumes the shared cohort's capacity.

```bash
# Deploy Team B's workload (raycluster-dev, using dev-priority)
oc apply -f workload-b-low-priority.yaml
oc get workload -n team-b # ADMITTED should be True
oc get pods -n team-b -w # Wait for pods to start running
```

*MaaS Talking Point:* Team B is utilizing idle capacity, demonstrating efficient resource utilization.

### Step 3: Deploy High-Priority (Reserved) Workload

Simulate the Reserved MaaS customer (`team-a`) submitting a critical workload (`RayCluster` using `prod-priority`). Since the necessary shared CPU/Memory capacity is currently consumed by Team B, Kueue must enforce the SLA policy.

```bash
# Deploy Team A's workload (raycluster-prod, using prod-priority)
oc apply -f workload-a-high-priority.yaml
```

### Step 4: Observe SLA Enforcement and Preemption (The "Eye-Catching" Moment)

Monitor the workloads across all namespaces.

```bash
oc get workload -A -w
```

**Observation:**
1.  The `raycluster-dev` (Team B) workload will instantly switch its `ADMITTED` status from `True` to `False`.
2.  The `raycluster-prod` (Team A) workload will immediately switch its `ADMITTED` status to `True`.
3.  Check the pods: Team B's pods will enter the `Terminating` state as they are evicted.

**Verification of Policy Enforcement:** Describe the preempted workload to show the explicit audit trail.

```bash
oc describe workload -n team-b raycluster-dev
```

Look for the **Events** section confirming that the job was **Evicted** due to preemption by the higher-priority workload, proving that the LiteMaaS SLA policy was enforced automatically.

## 🗑️ Cleanup

To remove all resources created during this demonstration, execute the following commands:

```bash
# 1. Delete the namespaces and all contained resources
oc delete ns team-a team-b

# 2. Delete the cluster-scoped Kueue objects (CQs, RFS, etc.)
oc delete clusterqueue --all
oc delete resourceflavor --all
oc delete workloadpriorityclass --all
```

## 📖 Related Lab Guides

The foundational concepts and configurations for this demo were derived from the following lab guides:

*   Lab Guide: Advanced GPU Quota Management and Preemption with Kueue. (WIP)
*   Lab Guide: Implementing GPU Pricing Tiers with Kueue. (WIP)
