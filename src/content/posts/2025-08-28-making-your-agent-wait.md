---
title: Making Your Agent Wait
description: ""
date: 2025-08-28T19:22:55.414Z
preview: "Turns out waiting and repeating are two things most commercial agents (gemini, copilot etc) are not inclined to do. In this post we're trying to solve this problem."
draft: false
tags:
    - agent
    - AI
    - ai-agents
    - artificialIntelligence
    - fastmcp
    - gemini
    - mcp
    - python
    - tutorial
    - copilot
categories:
    - ai
    - tutorial
    - mcp
keywords:
    - ai
    - ai agents
    - copilot
    - gemini
    - mcp
    - agents
---

It may sound counterintuitive to make an agent wait. After all, we want our agents to be as fast and responsive as possible. However, there are scenarios where a delay is necessary, such as waiting for a long-running process to complete or for data to be returned from a slow API. It has been observed that many AI agents are not designed to handle long delays.

Try the following prompt with your agent:

`Wait for 10 seconds and then say hi to me`

Depending on the model you're using, it might or might not work. But the next query we try would definitely give you trouble:

`for the next 2 mins say hi to me in 10 second intervals`

Did your agent tell you it is unable to comply? Turns out waiting and repeating are two things most commercial agents (gemini, copilot etc) are not inclined to do. They need external tools to achieve both.

In this post we're trying to solve this problem, partially.

## Building a Waiting Tool

While there are many pre-built waiting tools available, building your own ensures that you have full control over its functionality and that it doesn't perform any unintended actions.

Writing this tool is so simple, it doesn't even have steps, here's the code:

```python
import json
import time
from fastmcp import FastMCP


mcp = FastMCP(name="WaitServer", version="0.0.1")

@mcp.tool(name="wait", description="Wait for a specified number of seconds.")
def wait(seconds: int):
    print(f"Waiting for {seconds} seconds...")
    time.sleep(seconds)
    print("Done waiting.")
    return json.dumps({"status": "success", "content": f"Waited for {seconds} seconds."})

if __name__ == "__main__":
    try:
        mcp.run(transport="streamable-http")
    except Exception as e:
        print("Server is stopped.")
```

That's it. Your own working mcp server is ready which can be used for your in-editor agent.

## Integrating With Agent

Just like adding any other mcp server, this can be added very easily with the following json:

```json
"WaitServer": {
            "type": "stdio",
            "command": "npx",
            "args": [
                "mcp-remote", 
                "http://127.0.0.1:8000/mcp/"
            ]
        }
```

Just restart your editor and try the query again:

`for the next 2 mins say hi to me in 10 second intervals`

This time, it should work better than the last time.

But, you would also observe some problems. Although the agent doesn't complain immediately, the process abruptly stops in between. That's because every model has an artificial limit to its think time - https://www.reddit.com/r/Bard/comments/1jyd7v8/fun_fact_gemini_models_cant_think_for_longer_than/

Similar limits have been imposed on models from other providers as well, that makes our task a bit harder. Let's tackle that in the next post.

## Conclusion

In this post, we learned:

* There are limitations to what existing in-editor agents can do.
* How to create a custom MCP Server
* Add an unsupported utility to existing agent making it more useful.