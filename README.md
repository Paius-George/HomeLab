# 🖥️ HomeLab

*[Versiunea în limba română →](README.ro.md)*

It all started with an **Asus VivoBook** that had been gathering dust for a while. Even though it had a capable processor, it was considered "outdated" for a modern day-to-day workflow. The inspiration came from exploring the _HomeLab_ communities, where I realized that "old" hardware is, in fact, an untapped resource that can host an entire ecosystem of services and automations.

I decided to give it a second chance and a critical mission: turning an idle laptop sitting around the house into a **Proxmox Virtualization Server**.


## 🛠️ The Hardware Challenge

The biggest challenge of this project was optimizing resources. In an era of servers with hundreds of GB of RAM, this project demonstrates maximum efficiency on a modest but well-balanced configuration:

### 🛠️ Hardware

I didn't need a whole rack of servers. Everything runs on an Asus laptop with the following specs, squeezed to the last drop:

| Component   | Specification                                 | Role in the ecosystem                                                                                                                                              |
| :---------- | :-------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **CPU**     | **Intel Core i7-10510U** (4 Cores, 8 Threads) | Handles the computation for the LLM and n8n, as well as video transcoding in Jellyfin.                                                                            |
| **RAM**     | **16 GB DDR3**                                | Split strategically between the AI VM (which "swallows" the most) and the LXC containers.                                                                         |
| **Storage** | **SSD** (Boot + VM/Containers Storage)        | The SSD handles the fast loading of the AI models (which are massive) and ensures n8n processes automations instantly, without waiting on reads/writes.           |

---

## 🎮 The Services

Each service has its own well-defined role. I chose to combine **Virtual Machines (VMs)** for maximum isolation and **Containers (LXC)** for speed and low resource usage.

<img alt="image" src="https://github.com/user-attachments/assets/2ddab235-2501-4eff-b2b4-96a3b7095017" />


---
### 🧠 1. Local LLM (VM 117)

This is my favorite part. It's a virtual machine dedicated to a **local LLM**, run through `llama-cpp`.

<img alt="image" src="https://github.com/user-attachments/assets/2a039612-b86d-4cbf-8b6c-15bc39353159" />

- **Why a VM?** Because the LLM needs "reserved" resources and better isolation at the kernel level.
- **The setup:** I allocated **10 GB of RAM** so it can load decent language models.
- **Performance:** I use GGUF versions of the models to squeeze the most speed out of the i7 CPU, without needing a dedicated graphics card.

<img alt="image" src="https://github.com/user-attachments/assets/4e918833-c6ed-4e18-83b5-ec4e6b29d6d3" />

I use a **Qwen3 model with 4 billion parameters**, specially trained for "Reasoning" (complex logical thinking), which was distilled to deliver performance similar to much larger models. It's in **GGUF format with Q4_K_M quantization**, so the model has a reduced size of just **2.32 GB**, which lets it run decently on the i7 CPU while leaving plenty of free RAM for the other services on the server.

![llm](https://github.com/user-attachments/assets/5e444ce4-d9c1-406a-bbf3-035a79a4a0cb)

### ⚡ Response Speed

Running an AI model on a laptop CPU (with no dedicated graphics card) might sound slow, but optimizing through **GGUF** and **Q4_K_M** quantization works wonders. On my setup, generation speeds are surprisingly usable:

- **7-8 tokens/second (light tasks):** This speed is comparable to the pace at which an average person reads. Text appears on screen naturally, as if someone were typing quickly in real time. For simple questions, summaries, or formatting data, you don't feel any annoying delay.

- **3-4 tokens/second (complex/reasoning tasks):** My model, **Qwen3-4B-Thinking**, is a "reasoning" type. Even if the speed drops by half, the quality of the answer is much higher. For solving logic problems or debugging code, I'd rather wait an extra 2 minutes for a correct answer than get a fast, wrong one.
---

### 🗺️ 2. n8n workflow (CT 100)

I installed it in an LXC container because it's incredibly efficient: it boots instantly and takes up very few resources.


<img alt="image" src="https://github.com/user-attachments/assets/68ce9697-a7c8-4f70-baa3-37b9b5ec6457" />

### ⚙️ Container Specs (LXC)

- **Resources:** 2 vCPU | 2 GB RAM — it may seem like little, but it's more than plenty (it would probably do just as well with 1 vCPU and 1 GB RAM).
- **Network:** Integrated into the Proxmox bridge to communicate quickly with the rest of the local services.

### ❓ Why n8n and not Python scripts?

- **Visual:** I can see exactly where the data flows and where something gets stuck (if it does).
- **Integration:** It has hundreds of ready-made nodes for Telegram, Discord, Google Sheets, or HTTP requests to my own AI VM.
- **Speed:** I modify a workflow instantly, without restarting any service.

---

### 🎬 3. Jellyfin (CT 111)

This is my open-source alternative to Netflix. I have full control over my media library, zero ads, and a super clean interface that pulls in all the movie details on its own (posters, descriptions, trailers).


<img alt="image" src="https://github.com/user-attachments/assets/37d38051-ea58-4899-a8c9-c794618d4b85" />

#### ⚙️ Container Specs (LXC)

- **Resources:** 2 vCPU | 2 GB RAM — although it seems like little, the container is extremely efficient for streaming.

#### ❓ Why Jellyfin and how I tested it

- **The performance test:** I haven't added a movie/series library yet. To check that everything is configured correctly, I uploaded 1-2 personal test recordings.

- **The result:** Streaming ran flawlessly, without a bit of lag or buffering, confirming that the resource allocation and disk access are optimal.
---
### 🌐 4. Tailscale

Tailscale creates a mesh VPN network between all my devices, without me touching the router or opening any port. Each device gets a fixed IP address from the private Tailscale network, and the communication between them is end-to-end encrypted.
#### Why it's the most efficient solution:

- **Zero router configuration:** I didn't touch the router's network settings. Tailscale gets through any firewall and creates a secure tunnel between my devices.

- **Remote Control:** If I need the local LLM or any other service on the server, I can access it directly from my phone.

<img alt="image" src="https://github.com/user-attachments/assets/beb87fff-0ddf-4357-a63e-91bb9d8dba9b" />

---
