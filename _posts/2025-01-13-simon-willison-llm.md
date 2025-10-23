---
layout: post
title: "Generate Plain-Language System Summaries using Simon Willison's llm Package"
date: 2025-01-13 12:00:00 +0000
categories: til
tags: [machine learning, llm, llms, ai]
excerpt:
  "Simon Willison's `llm` package can be used to easily interact with large
  language models like gpt-4o and o1-preview in the command line, which is especially
  powerful when you want the model to have immediate access to responses to bash commands,
  for example to generate system detail summaries. "
---

Simon Willison has created a great open source `Python` package `llm` for working with OpenAI's large language models (e.g. `gpt-4o-mini`, `gpt-4o`, `o1-mini` and `o1 preview`) on the command line. I wrote about creating my own version of this to experiment witn OpenAI's API, but Simon Willison's version is much more robust and mature, with significantly more features. The github repo for the project is [simonw/llm](https://github.com/simonw/llm?tab=readme-ov-file), where it is extensively documented.

<div style="text-align: center">
<p style="margin: 0; text-align: center">
Header of README.md from the llm package on github
</p>
    <img src= "{{ site.baseurl }}/assets/posts/llm-package-readme-header.png" style="width: 75%">
</div>

## Installation Instructions and Background

Intstall llm using `pip`:

```bash
pip install llm
```

You can then set your OpenAI API key:

```bash
% llm keys set openai
Enter key:
```

The API key/keys set in this way are stored in `keys.json`, the path for which can be found using the following command:

```bash
llm keys path
```

You can then test the package with a prompt to the default model (`gpt-4o-mini` at the time of writing):

```bash
% llm "Give me an itinerary of the walking route from Caterbury to Rome"
The walking route from Canterbury to Rome is known as the Via Francigena, a historic pilgrimage route that has been traveled since medieval times. Here’s a suggested itinerary, breaking the journey into manageable segments. Please note that the distances and duration may vary depending on specific preferences, physical condition, and daily walking pace.

### General Overview
- **Total Distance**: Approximately 1,900 km (1,180 miles)
- **Estimated Duration**: 90 to 120 days, depending on pace and rest days

### Suggested Itinerary

#### England
1. **Canterbury to Dover** (approximately 30 km)
   - Walk through the picturesque Kent countryside.
   - Overnight in Dover.

2. **Channel Crossing**
   - Take a ferry from Dover to Calais, France.

#### France
3. **Calais to Guînes** (approximately 20 km)
   - Begin your journey through the French countryside.
...
...
```

You can change the default model to `gpt-4o` (or any of the other available models):

```bash
llm models default gpt-4o
```

and print a full list of available models:

```bash
llm models
```

## Commands to Generate System Summaries

You can incorporate the outputs of bash commands directly into text promps by using the pattern `$(COMMAND)` within the text prompt. There are countless applications of this functionality, but I will list a few that I like here:

Generate a summary of basic system information:

```bash
% llm "Generate a summary of my basic system information: $(uname -a)"
Your system information indicates the following:

- **Hostname:** [REDACTED]-MacBook-Pro.local
- **Operating System:** Darwin (macOS)
- **Kernel Version:** 23.X.X [TRUNCATED]
- **Kernel Release Date:** November 15, 2023
- **Architecture:** ARM64
- **Processor Type:** T6030

This suggests that you are using a MacBook Pro with an ARM-based Apple silicon processor, running a recent version of macOS.
```

Generate a summary of the top 10 running processes:

```bash
% llm "Generate a summary of the top 10 running processes: $(ps aux | head -n 10)
Based on the list of processes you have provided, here is a summary of what each of them is doing:

1. **_windowserver (PID [XXXXX])**
   - **CPU Usage:** 5.5%
   - **Memory Usage:** 0.7%
   - **Function:** `WindowServer` is a core component of macOS responsible for the graphical interface and managing windows. Its relatively high CPU usage is normal as it processes the visual rendering for the system.

2. **mdworker_shared (PID [XXXXX], [XXXXX], [XXXXX], [XXXXX])**
   - **CPU Usage:** Around 2.6% for each process
   - **Memory Usage:** 0.1% for each process
   - **Function:** `mdworker_shared` processes are associated with Spotlight indexing. They help to index files and metadata for quick search capabilities. It's common to see multiple instances running, especially if new files have been added or changed, or if you are running a search.
...
...
```

Generate a summary of my `ifconfig`:

```bash
% llm "Generate a summary of my ifconfig: $(ifconfig)"
The output you provided is from the `ipconfig` or `ifconfig` command, which displays the network configuration of your system's network interfaces. Here's a breakdown of the key parts:

### Loopback Interface (`lo0`)
- **Flags**: `UP`, `LOOPBACK`, `RUNNING`, `MULTICAST`
- **IP Address (IPv4)**: `[REDACTED]` with netmask `[REDACTED]`
- **IP Address (IPv6)**: `::1` and `[REDACTED]` (link-local)
- This interface is used for the local machine to communicate with itself and is always present in any network configuration.
...
...
```

Generate a summary of my open network connections:

```bash
% llm "Generate a summary of my open network connections: $(netstat -an | grep ESTABLISHED)"
Your network has several active TCP connections, both IPv4 and IPv6. Here's a summary of these connections:

### Localhost Connections (IPv4)

- Your system is using `[REDACTED]` for local communication with established connections between ports [REDACTED] and other local ports such as [REDACTED], [REDACTED], [REDACTED], and [REDACTED].

### External IPv4 Connections

- **[REDACTED]** (likely your local network IP) has established connections to:
  - `[REDACTED]` on port [REDACTED] (SSL/TLS)
  - `[REDACTED]` on port [REDACTED] (possibly for Apple's Push Notification Service)
    ...
    ...
```

END!
