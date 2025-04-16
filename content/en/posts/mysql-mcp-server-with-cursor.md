+++
title = 'Using MySQL MCP Server with Cursor'
date = 2025-04-15T11:56:28+08:00
draft = false
+++

## Introduction

Are you using Cursor?
I recently started using it and found it so convenient that I subscribed to the Pro membership.

Recently, I learned about something called MCP servers, which connect AI chat tools with external tools.
I tried setting up a MySQL MCP server and was able to access MySQL databases directly from Cursor's chat.
This allows Cursor to directly access the database and execute queries for you.
It can retrieve data from the database when there's an error or modify table definitions.
It's extremely convenient.

In this article, I'll introduce how to set up a MySQL MCP server and how to access it from Cursor, partly as a record for myself.

## What is an MCP Server?

So what exactly is an MCP server? Based on my understanding, it's a system for connecting "AI generation tools" with "external tools."

Similar mechanisms have existed before, like ChatGPT's Function Calling and plugins.
However, these mechanisms weren't universally usable for the many other generative AIs (Claude, DeepSeek, etc.) that now exist.
Therefore, when trying to connect with other generative AIs, you had to rewrite the code each time.

With the emergence of MCP servers, these standards have been unified.
In other words, once you've written code for connecting generative AI with external tools according to the standard, you can reuse it indefinitely, whether with Claude, DeepSeek, or others.

If I elaborate too much on this part, I'll stray from the main topic of this article, so if you want to know more, please refer to articles like the one below:
<https://zenn.dev/nanami_bitwise/articles/966efd5438e75b>

## Using a MySQL MCP Server

For this article, I'll be using the MCP server that came up first in my online search:

<https://github.com/designcomputer/mysql_mcp_server>

Please download it using your preferred method: GitHub Desktop, command line, Visual Studio, etc.

### Prerequisites: Execution Environment

My execution environment is as follows:

- Windows 11
- Cursor 0.48.9
- PowerShell 7.5.0
- For Method 1
  - Python 3.13
  - pip 25.0.1
  - uv 0.6.13
- For Method 2
  - npm 10.9.2

To use the MCP server, you'll need Python, pip, and uv, or npm.
Please ensure you can run these in your local environment.

### Method 1: Download and Run the MCP Server Application Locally

First, run the following command locally.
Since you'll be using the path where this command was executed later, please save it in a location that's easy for you to manage.

```pws
pip install mysql-mcp-server
```

Next, from Cursor's settings, select MCP → Add new global MCP server, and configure it as follows:

```json
{
  "mcpServers": {
    "mysql": {
      "command": "full path to uv",
      "args": [
        "--directory",
        "full path to src\\mysql_mcp_server",
        "run",
        "mysql_mcp_server"
      ],
      "env": {
        "MYSQL_HOST": "hostname",
        "MYSQL_PORT": "3306 (adjust according to your host settings)",
        "MYSQL_USER": "username",
        "MYSQL_PASSWORD": "password",
        "MYSQL_DATABASE": "database name"
      }
    },
  }
}
```

Here are a few points to note:

#### Specify the Full Path to uv

In Windows environments, if you only write "uv", you'll get an error saying it can't find the command.
Therefore, you need to specify the full path to the uv command (uv.exe).
It should look something like `C:\\Users\\username\\.local\\bin\\uv`.

#### About Path Notation

In Windows environments, you should mostly use \ (backslash), but it seems that current Windows shells handle the conversion well.
Therefore, / (forward slash) is also recognized without issues.
I've written with double backslashes (for JSON escaping) just to be safe, but you can use whichever you prefer.
I'd like to verify this area further in the future.

#### MySQL Hostname and Database Name

I misunderstood this part and spent quite a bit of time on it.
The database URL I was trying to connect to looked like this in my program:

jdbc:mysql://hoge.com:3306/fuga

This string means connecting to the fuga table on hoge.com using jdbc for mysql.
So in this case, the hostname is hoge.com and the database name is fuga.
In the Cursor settings file, you need to specify hoge.com for MYSQL_HOST and fuga for MYSQL_DATABASE.

### Method 2: Launch through Smithery

This method should eliminate the need to download data locally, but it didn't work well in my environment.
If you want to try it, check it out here:

<https://smithery.ai/server/mysql-mcp-server>

## Verification

First, check that Cursor's MCP settings section looks like the following:
If there's a green circle, it's generally fine.

![Screenshot of Cursor's MCP settings](/images/mysql-mcp-server-with-cursor/mysql-mcp-setting.png)

Next, in Cursor's chat, try asking something like:

`Please retrieve data from the XX table`

If it returns data as shown below, you've succeeded:

![Screenshot of retrieving MySQL data with Cursor](/images/mysql-mcp-server-with-cursor/mysql-mcp-screen-shot.png)

## If It Doesn't Work

Press `Ctrl + Shift + U` to open "Output", and select "Cursor MCP" from the dropdown in the top right.
MCP-related logs are output there, which should provide hints for resolving errors.

Also, if it still doesn't work, try pressing the update button in Cursor's MCP settings or restart Cursor.
Since Cursor is an editor still in active development, this behavior might be somewhat unstable.

## Conclusion

While I found Cursor to be extremely convenient, I wondered if it could also automatically handle database operations.
Using this MCP server, AI can now perform database operations for you.
When using it in actual business applications, you'll need to consider issues like security and hallucinations, but I think it's a wonderful MCP server with great potential.

