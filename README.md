# ComfyUI on GCP NVIDIA L4 - Setup Notes

## Goal

Provision a temporary NVIDIA L4 GPU on Google Cloud Platform, install ComfyUI, download a model, generate images, then destroy the infrastructure when finished.

---

# 1. Create GCP GPU VM

Configuration used:

```text
Machine Type: g2-standard-4
GPU: NVIDIA L4 (1 GPU)
Disk: 100 GB
Image: Deep Learning VM (CUDA enabled)
```

Verify GPU:

```bash
nvidia-smi
```

Expected:

```text
GPU: NVIDIA L4
VRAM: ~23 GB
CUDA: 13.0
```

---

# 2. Connect via SSH

Use the browser SSH terminal from GCP:

```text
Compute Engine
→ VM Instances
→ SSH
```

---

# 3. Verify Python

```bash
python3 --version
```

Output:

```text
Python 3.10.12
```

---

# 4. Clone ComfyUI

```bash
git clone https://github.com/comfyanonymous/ComfyUI.git

cd ComfyUI
```

---

# 5. Install Python Venv Support

Ubuntu initially failed with:

```text
ensurepip is not available
```

Install venv package:

```bash
sudo apt update

sudo apt install -y python3-venv
```

---

# 6. Create Virtual Environment

```bash
python3 -m venv venv

source venv/bin/activate
```

Verify:

```bash
python --version
```

---

# 7. Install ComfyUI Dependencies

```bash
pip install --upgrade pip

pip install -r requirements.txt
```

---

# 8. Start ComfyUI

```bash
python main.py --listen 0.0.0.0 --port 8188
```

Expected:

```text
Starting server

To see the GUI go to:
http://0.0.0.0:8188
```

---

# 9. Open Firewall Port

GCP:

```text
VPC Network
→ Firewall
→ Create Rule
```

Configuration:

```text
Name: comfyui-8188

Direction:
Ingress

Source:
0.0.0.0/0

Protocol:
tcp:8188
```

---

# 10. Open ComfyUI

Open browser:

```text
http://<EXTERNAL_IP>:8188
```

Example:

```text
http://34.xxx.xxx.xxx:8188
```

---

# 11. Download Juggernaut XL

Install Hugging Face CLI:

```bash
pip install -U huggingface_hub
```

Download model:

```bash
cd ~/ComfyUI/models/checkpoints

hf download RunDiffusion/Juggernaut-XL-v9 \
  Juggernaut-XL_v9_RunDiffusionPhoto_v2.safetensors \
  --local-dir .
```

Verify:

```bash
ls -lh ~/ComfyUI/models/checkpoints
```

Expected:

```text
Juggernaut-XL_v9_RunDiffusionPhoto_v2.safetensors
```

---

# 12. Build Basic Workflow

Nodes:

```text
Load Checkpoint

CLIP Text Encode (Positive)

CLIP Text Encode (Negative)

Empty Latent Image

KSampler

VAE Decode

Save Image
```

---

# 13. Node Connections

```text
Load Checkpoint MODEL
    →
KSampler model
```

```text
Load Checkpoint CLIP
    →
Positive Prompt clip
```

```text
Load Checkpoint CLIP
    →
Negative Prompt clip
```

```text
Positive Prompt CONDITIONING
    →
KSampler positive
```

```text
Negative Prompt CONDITIONING
    →
KSampler negative
```

```text
Empty Latent Image LATENT
    →
KSampler latent_image
```

```text
KSampler LATENT
    →
VAE Decode samples
```

```text
Load Checkpoint VAE
    →
VAE Decode vae
```

```text
VAE Decode IMAGE
    →
Save Image images
```

---

# 14. KSampler Settings

```text
Steps: 25

CFG: 7

Sampler: Euler

Scheduler: Normal
```

---

# 15. Positive Prompt Example

```text
beautiful warrior queen,
cinematic lighting,
ultra detailed,
masterpiece
```

---

# 16. Negative Prompt Example

```text
blurry,
low quality,
deformed,
extra fingers
```

---

# 17. Generate Image

Click:

```text
Queue Prompt
```

Expected generation time:

```text
5–15 seconds
```

on NVIDIA L4.

---

# 18. Output Location

Generated images:

```bash
/home/aashlibi1111/ComfyUI/output
```

Check:

```bash
ls -lh ~/ComfyUI/output
```

---

# 19. Disk Space Commands

Check free disk:

```bash
df -h
```

Check model sizes:

```bash
du -sh ~/ComfyUI/models/*
```

Find files:

```bash
find ~ -iname "*filename*" 2>/dev/null
```

---

# 20. Chroma1-HD Attempt

Downloaded:

```bash
hf download lodestones/Chroma1-HD
```

Model stored under:

```bash
~/.cache/huggingface/hub/models--lodestones--Chroma1-HD
```

Size:

```text
~43 GB
```

Not fully wired into ComfyUI workflow.

---

# 21. Destroy Infrastructure

When finished:

```text
Compute Engine
→ VM Instances
→ Select VM
→ Delete
```

This removes:

```text
GPU
CPU
Disk
External IP
Charges
```

---

# Lessons Learned

* Model ≠ Workflow
* Checkpoint ≠ Diffusion Model Package
* Positive prompt = desired concepts
* Negative prompt = undesired concepts
* ComfyUI is optional for inference
* SSH-only workflows are often simpler and safer
* L4 GPU handled SDXL/Juggernaut comfortably
* Cloud GPU setup can be recreated in ~20 minutes

```
```
