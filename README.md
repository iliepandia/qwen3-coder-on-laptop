# Free Code Generation with Local LLM - QWen3-coder, Ollama, OpenCode on an ASUS Strix Laptop

## Summary 

Get close to Opus quality code, review and exploration for **free** running a local LLM on your ASUS Laptop. 

## Pre-Requisites 

These instructions are for a very specific machine and setup. I expect these to work on anything like these or better.

Hardware: ASUS ROG Strix SCAR 16, NVidia GeForce RTX 4090 *Laptop GPU* **16 GB VRAM**, and **64GB RAM**

Software: WSL2 running on Windows 11

You will need at least 20GB of free space on your drive.

## Install Ollama

This is the local LLM service manager.

Download Ollama and install it from the official download page:

[Download Ollama](https://ollama.com/download)

And follow the instructions on screen. Everything happens in WSL2.

## Install OpenCode

OpenCode is a free open source alternative to ClaudeCode. 

Download it and install it from here:

[Download OpenCode](https://opencode.ai)

## Download Qwen3-coder LLM

Next you will need to download the model we will be using which is `qwen3-coder`

Run this command to download it. It is a **18GB file**, so be patient.

`ollama pull qwen3-coder`

## Close Any apps that are using the GPU

Run `nvidia-smi` to get information about the free VRAM. 

I had to have at least 13GB free, to be able to load qwen3. 

I had to close Photoshop, Slack, Resolve, Postman and a few more apps that were 
using VRAM. And I got from 3GB free to 13GB free VRAM.

If you skip this step you won't be able to launch the model.

## Create a second model with *larger context*

On my machine, when I tried to use the downloaded model it would default to a 4k context
window which is too small for a coding agent. And the side effect is that you will just 
get unusable responses that don't make any sense. 

The solution that worked for me was to create a fork of the model where I explicitly set 
the context size.

The main idea was to start the original model, explicitly set the context window, then 
save as a new model.

First start the original model with this command:

`ollama run qwen3-coder`

Once you have the chat prompt, give these commands:

```text
>>> /set parameter num_ctx 65536
>>> /save qwen3-coder-64k
>>> /bye
```

Make note of the name `qwen3-coder-64k` as we will be using that later.

Next let's confirm all is well so far.

Run

`ollama ls`

And you should have an output like this, with *both* models available.

```text
NAME                      ID              SIZE     MODIFIED
qwen3-coder:latest        06c1097efce0    18 GB    7 minutes ago
qwen3-coder-64k:latest    826791f78556    18 GB    2 hours ago
```

Next run:

`ollama show qwen3-coder-64k`

And look for this output. For capabilities you should still have "tools" and "completion". And
for `Parameters.num_ctx` you should see `65536`

```text
 Model
    architecture        qwen3moe
    parameters          30.5B
    context length      262144
    embedding length    2048
    quantization        Q4_K_M

  Capabilities
    completion
    tools

  Parameters
    top_p             0.8
    num_ctx           65536
    repeat_penalty    1.05
    stop              "<|im_start|>"
    stop              "<|im_end|>"
    stop              "<|endoftext|>"
    temperature       0.7
    top_k             20

  License
    Apache License
    Version 2.0, January 2004
    ...
```

## Restart Everything

To avoid hours of frustration I recommend you restart everything. This will make sure that your 
latest model with the larger context will be picked up.

Stop Ollama:

`sudo systemctl stop ollama`

Exit OpenCode and stop any background services

`opencode service stop`

Start Ollama

`sudo systemctl start ollama`

Use Ollama to start OpenCode with the new model

`ollama launch opencode --model qwen3-coder-64k`

Hopefully at this point you will see OpenCode using this model `qwen3-coder-64k`

Enter a query in the box, something like "what is this project about".

Once you start to get some output, in a different console tab run this command:

`ollama ps`

And look for this output, to confirm the right model is used, with the correct context size
and you will also be able to see how much of the model runs on GPU and how much on CPU.

If you see both models here, or the wrong context size trace back your steps. A 4k context will not be enough for anything.

```text
NAME                      ID              SIZE     PROCESSOR          CONTEXT    UNTIL
qwen3-coder-64k:latest    826791f78556    25 GB    42%/58% CPU/GPU    65536      4 minutes from now`
```

## Test it on a local project

Navigate to a project you want to test and run OpenCode:

`ollama launch opencode --model qwen3-coder-64k`

At this point you should be able to ask questions, generate code, review old code and
use tools successfully. 

On my setup I was able to get 12.5 tok/s which felt pretty snappy.

Quality wise, I have not noticed yet any major issues. Generated code looks good, asking questions about the codebase yield relevant responses. 

I don't feel as confident as when using Claude Code, but next time I hit an API usage limit I will get back to this for sure.

## Troubleshooting

### Getting horrible quality

Make sure you are using the larger context model. The default 4k is not enough for anything meaningful. And you will get strange errors, with invalid tools, and just answers that are very vaguely connected with what you are asking.

### CUDA Out of memory

Close all the apps that use the GPU. Seriously, free as much VRAM as you can. 

### Wrong Model Showing up

Both Ollama and OpenCode start background services. Until those are restarted old context,
sessions and models can hang around with the wrong configurations.

Restarting the services and killing any stray Ollama processes or OpenCode sessions is what 
fixed the problem and everything was finally aligned. 

In the end you should be seeing the model `qwen3-coder-64k` in all the relevant places.
