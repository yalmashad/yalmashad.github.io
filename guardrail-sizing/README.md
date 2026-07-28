# F5 AI Guardrail GPU Resource Sizing & Capacity Calculator

An interactive, production-grade capacity planning tool for **F5 AI Guardrails**. This calculator helps security architects and IT teams accurately size the GPU infrastructure (NVIDIA H100, A100, L40, A10, L4) needed to inspect GenAI prompts and LLM responses in production.

---

## 💡 Quick Start Concepts (If You Are New to AI & GPUs)

### 1. What is an AI Guardrail?
An **AI Guardrail** acts as a security firewall for Generative AI applications. Before a user's prompt reaches your Large Language Model (LLM) — or before the LLM's response returns to the user — the Guardrail scans the text for **Prompt Injections**, **Jailbreaks**, **PII leaks**, **System Prompt leaks**, and **Toxicity**.

### 2. What is a "Token"?
In AI, text is measured in **Tokens** rather than words or bytes:
> 💡 **Rule of thumb:** `1 Token ≈ 0.75 Words` (or 4 characters of English text).
> * **Short Chat Message (e.g. "What is my account balance?"):** ~30 tokens
> * **Standard Prompt:** ~256 tokens (~190 words)
> * **Long Document / RAG Context:** ~4,000+ tokens (~3,000 words)

### 3. Why Do Guardrails Need GPUs?
Guardrails run deep learning classification models to detect attacks in real-time. Because GPUs process thousands of mathematical matrix calculations concurrently, they perform guardrail screening **10× to 50× faster** than standard CPUs.

---

## ⚙️ Input Parameters Explained

| Parameter | Meaning | Why It Matters |
| :--- | :--- | :--- |
| **Deployment Lifecycle** | Choose **Out-of-Band (`/scans`)** or **Inline (`/prompts`)**. | **Out-of-Band** scans input prompts before the LLM. **Inline** inspects both input prompts AND returning LLM responses for total protection. |
| **Target Latency SLA (ms)** | The maximum total time allowed for guardrail inspection (e.g., 100 ms, 500 ms, 2,000 ms). | **Crucial for sizing!** Tighter SLAs (e.g., 100 ms) require **more GPUs** to prevent queueing delays, while relaxed SLAs allow higher GPU utilization. |
| **Target Throughput (RPS)** | Requests Per Second at peak traffic. | Determines total system capacity. `50 RPS = 4.32 Million requests/day`. |
| **Traffic Headroom Buffer (%)** | Extra reserved capacity (e.g., 25%). | Protects system against sudden traffic spikes and prevents queue build-up. |
| **Average Prompt Size (Tokens)** | Length of incoming user query. | Longer prompts take more GPU compute time to scan. |
| **Average Response Size (Tokens)** | Length of LLM text returned (Inline mode only). | Scanned on return in Inline mode. |
| **Active Guardrail Scanners** | Selected security modules (Prompt Injection, Jailbreak, PII, Toxicity, etc.). | F5 runs active scanners **in parallel**. Enabling more scanners adds a small token processing overhead. |

---

## ⏱️ The Latency SLA vs. GPU Sizing Formula

Why does demanding lower latency increase the number of required GPUs?

When requests arrive at peak volume, running GPUs at **100% capacity** causes incoming requests to wait in line (queueing delay), breaching strict SLAs. 

Using **Queueing Theory ($M/G/1$ Queue Model)**, the calculator computes the **Maximum Allowable GPU Utilization ($\rho_{\text{max}}$)**:

$$\rho_{\text{max}} = 1 - \frac{\text{Single Scan Execution Time}}{\text{Target SLA}}$$

* **Strict Low Latency SLA (e.g., 100 ms):** GPU load is capped at ~**12% – 25%** per instance so requests execute immediately without waiting. **Requires MORE GPUs**.
* **Relaxed SLA (e.g., 4,000 ms):** GPU load can run up to **95%** capacity. **Requires FEWER GPUs**.

---

## 📊 Understanding Calculated Output Metrics

| Output Metric | Description |
| :--- | :--- |
| **Recommended GPUs** | The exact number of GPU cards required (e.g. `12 × NVIDIA H100 80GB`). |
| **Scanning Latency (ms)** | Expected execution time for the guardrail inspection alone. |
| **Target Utilization Cap ($\rho_{\text{max}}$)** | Maximum safe load percentage per GPU to satisfy your SLA. |
| **Cluster VRAM / CPU / RAM** | Minimum host server hardware specifications required to drive the GPU cluster. |

---

## 📁 Practical Sizing Examples

### Example 1: Customer Service Chatbot (Balanced)
* **Goal:** Screen customer queries with reasonable response times.
* **Settings:** `50 RPS` · `500 ms SLA` · `256 Token Prompt` · `3 Scanners`
* **Result:** **12 × NVIDIA H100 (80GB)** GPUs (or 18 × A100 / 30 × L40).

### Example 2: Ultra-Fast Security Gateway (Strict SLA)
* **Goal:** High-volume API firewall with imperceptible overhead.
* **Settings:** `500 RPS` · `100 ms SLA` · `128 Token Prompt` · `5 Scanners`
* **Result:** **68 × NVIDIA H100 (80GB)** GPUs (Low SLA forces low GPU queue depth).

---

## 🚀 Running the Tool Locally

No build tools or server installation required! The calculator is a standalone single-page Web application.

1. Clone or download this repository.
2. Open [`index.html`](file:///Users/y.elmashad/Documents/Antigravity/guardrail-sizing/index.html) in any web browser (Chrome, Edge, Safari, Firefox).
3. Use the sliders and presets to plan your production deployment.
4. Click **📄 Export Deployment Specification Report** to copy a markdown technical spec for your infrastructure procurement team.

---

## 📄 License & Attribution
Powered by the F5 AI Security Benchmark Model. © 2026 F5, Inc. All Rights Reserved.
