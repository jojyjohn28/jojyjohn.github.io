---
layout: post
title: "Using Syncthing, tmux, and Git to Sync My Bioinformatics Workflow Across Devices"
date: 2025-12-19
description: "How I synchronize files, terminal sessions, and version-controlled projects across my Linux desktop and laptop."
comments: true
giscus_comments: true
featured: true
permalink: /blog/syncthing-tmux-git-workflow/
---

#### 🖥️ Using Syncthing, tmux, and Git to Sync My Workflow Across Devices

Managing files and analysis sessions across multiple Linux machines can quickly become chaotic—especially when juggling bioinformatics scripts, results, manuscripts, figures, and a personal website at the same time.

Over the years, I tried:

● manual copying

● cloud storage

● external drives

● SSH-based syncing

Eventually, I realized that no single tool solves everything. Instead, I needed a small set of tools, each doing one job extremely well.

**My current setup is simple, reliable, and fully Linux-native:**

● Syncthing → real-time file synchronization

● tmux → persistent and shareable terminal sessions

● Git + GitHub → version control and publishing

Together, they let me move seamlessly between desktop, laptop, and HPC systems without losing context.

#### 🔷 1. Syncthing — real-time file synchronization

What Syncthing does

Syncthing keeps selected folders identical across machines, automatically and continuously.

● No USB drives

● No cloud storage

● No manual copying

● No “which version is correct?” confusion

**📦 Installing Syncthing**

On Ubuntu / Debian-based systems:

```bash
sudo apt install syncthing
```

Verify:

```bash
syncthing --version
```

#### 🔁 Starting Syncthing automatically on boot

I want Syncthing to start every time I log in, without manual intervention.

Enable and start the user service:

```bash
systemctl --user enable syncthing.service
systemctl --user start syncthing.service
```

Check status:

```bash
systemctl --user status syncthing.service
```

Now Syncthing:

● Starts automatically on login

● Runs quietly in the background

● Syncs even when the browser UI is closed

#### 🌐 Accessing the Syncthing interface

Open a browser and go to:

```txt
http://localhost:8384
```

This is where I:

● Add devices

● Select folders to sync

● Monitor sync status

#### 📂 My synced folder structure

I use one top-level folder across all machines:

```bash
~/Jojy_Research_Sync/
```

**Inside it:** I have many subfolder some examples are below

● scripts/ – Bash, SLURM, Snakemake, R, Python

● Projects/ – intermediate outputs, figures

● notes/ – daily research notes

● website_assets/ – content used by my blog

**These are files I edit frequently—sometimes switching machines multiple times in a day.**

#### 🔷 2. tmux — persistent and shared terminal sessions

While Syncthing synchronizes files, **it does not preserve live terminal state.**
For that, I use tmux.
**📦 Installing tmux**

```bash
sudo apt install tmux
```

Verify:

```bash
tmux -V
```

**🧠 Creating a persistent session**

To start a named tmux session:

```bash
tmux new -s shared
```

#### ✅ From your laptop: open the shared tmux session (desktop-hosted)

tmux sessions live on the machine where they were created (your desktop or an HPC node).
So on your laptop you don’t “open” it locally — you SSH into that same machine and then attach to the session.

```bash
ssh <user>@<DESKTOP_IP>
tmux attach -t shared
```

If the session doesn’t exist yet (first time), create it:

```bash
ssh <user>@<DESKTOP_IP>
tmux new -s shared
```

**🔍 Find your local IP address (most common case)**

On the machine hosting the tmux session (desktop or server), run:

```bash
ip a #Run on yur terminal
```

#### 👥 Sharing the same session across devices

If I SSH into the same machine from another device, I can attach to the same session.

```bash
tmux attach -t shared
```

This lets me:

● Resume work from a different device

● Monitor long-running commands

● Continue exactly where I left off

⚠️ tmux sessions are host-specific (not shared across different machines).

**🧩 tmux commands I use daily**
| Command | Action |
| ----------------------------- | -------------- |
| `tmux new -s shared` | Create session |
| `tmux ls` | List sessions |
| `tmux attach -t shared` | Reattach |
| `tmux kill-session -t shared` | End session |

#### 🔷 3. Git — version control and publishing

**Syncthing is not a replacement for Git.**

I use Git + GitHub for:

● My website repository

● Analysis pipelines

● Code meant to be shared or published

**📦 Installing Git**

```bash
sudo apt install git
```

**🔄 My daily Git workflow\***

When starting work on any machine:

```bash
git pull
```

This ensures:

● I’m working on the latest committed version

● No conflicts with work done on another device

**When something is finalized:**

```bash
git add .
git commit -m "Update analysis / figures / blog post"
git push
```

#### 🔗 How Syncthing, tmux, and Git work together

| Tool      | Role                            |
| --------- | ------------------------------- |
| Syncthing | Sync active working files       |
| tmux      | Preserve live terminal sessions |
| Git       | Track history and publish work  |

Typical workflow:

● Syncthing syncs drafts and scripts across devices

● tmux keeps my analysis session alive

● Git records finalized work and publishes it

Each tool does one job well, without overlap.

#### ✅ Why this is important for me

This setup:

● Reduces cognitive overhead

● Prevents accidental overwrites

● Supports HPC + local workflows

● Keeps everything Linux-native and reproducible

**Most importantly, it lets me focus on science, not file management.**

![deskto-laptop](/assets/img/syn.png)
