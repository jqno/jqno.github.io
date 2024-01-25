---
title: "Running a local LLM with Ollama"
tags:
  - ai
excerpt: In which I describe how easy it is to set up a local LLM on my laptop.
---
It's January 2024 as I write this; I fully expect this post to be out of date by tomorrow, or even sooner. But I think this is exciting!

Running a Large Language Model, or LLM, or AI assistant locally always seemed like something that only the really dedicated hobbyists could do. It seemed to require lots of manual build steps and complicated tinkering to get something working. This is no longer true.

## Why?

Why would you want to run an LLM locally? I've written about [AI](/tags/#ai), and specifically ChatGPT, before. ChatGPT and most other popular AI tools have a big disadvantage: everything you put in can be used to train their underlying models. That's a big problem for privacy and compliance. Most of the companies I work for don't allow their data to be sent to these services, and therefore, I can't always use them at work.

If only I could run an LLM on my laptop: then the data would never leave my machine, and it would be OK to use. That would be amazing!

## How?

Using a tool called [Ollama](https://ollama.ai/), which could be described as a kind of Docker for LLMs, it's surprisingly easy to set up:

{% highlight shell %}
brew install ollama
ollama serve
{% endhighlight %}

Now you can open another terminal, and do this:

{% highlight shell %}
ollama run llama2
{% endhighlight %}

This gets you a ChatGPT-like prompt in your terminal. That's it!

Note that the first time you do this, it will pull the `llama2` model. This is a few gigs large, so the download might take a while. Of course that's no different from the average Docker image. (Note that there are many models to choose from; see [the list on Ollama's website](https://ollama.ai/library).)

Instead of running the prompt on your terminal, you can also integrate it with your IDE. There's [Gen.nvim](https://github.com/David-Kunz/gen.nvim) for Neovim, and Continue for [Jetbrains](https://plugins.jetbrains.com/plugin/22707-continue) and [VS Code](https://marketplace.visualstudio.com/items?itemName=Continue.continue). There's also an official [Docker image](https://hub.docker.com/r/ollama/ollama).

## Performance

If you're doing all this on an Apple Silicon MacBook, you're golden. These machines seem to be built for this kind of stuff.

However, if you have a different setup, things might be slow. For instance, I'm running Linux on a Dell XPS with two GPUs. By default, Ollama will use my built-in GPU, which is slower. I can't make it use the dedicated GPU directly. Instead, I have to install the [Nvidia Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html) and run Ollama inside a Docker container with the `nvidia` runtime, like this:

{% highlight shell %}
docker run -d --runtime=nvidia --gpus=all \
    -v ollama:/root/.ollama -p 11434:11434 \
    --name ollama ollama/ollama

docker exec -it ollama ollama run llama2
{% endhighlight %}

By doing this, Ollama will use the dedicated GPU. Unfortunately, it's a hassle and the speed improvement is not significant. I might have to switch to Apple…

In the meantime, it helps to use a smaller model. I haven't done a lot of research into this, but [DeepSeek-Coder 1.3B](https://ollama.ai/library/deepseek-coder:1.3b) seems to work well.

## Conclusion

I'm really impressed by this. It's impressive that it's possible to run a local LLM in the first place. It's impressive that it can be done on hardware that regular people can buy (even if it's still really really expensive hardware). And it's impressive how easy it is to get working.
