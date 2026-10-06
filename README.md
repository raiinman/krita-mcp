<!-- project-centered:start -->
<div align="center">

<a name="readme-top"></a>
<h1 align="center">Krita MCP Server</h1>

<!-- project-header:start -->
<p align="center"><img src="readme-banner.png" alt="krita-mcp — original decorative project artwork" width="100%"></p>
<!-- project-header:end -->

<!-- project-badges:start -->
<p align="center"><a href="https://github.com/raiinman/krita-mcp"><img src="https://img.shields.io/badge/bridge-creative_tools-BF559F?logo=krita&amp;logoColor=white" alt="bridge: creative tools"></a> <a href="https://github.com/raiinman/krita-mcp"><img src="https://img.shields.io/badge/access-public-BF559F?logo=github&amp;logoColor=white" alt="access: public"></a> <a href="#readme-index"><img src="https://img.shields.io/badge/docs-explore_the_index-BF559F?logo=readthedocs&amp;logoColor=white" alt="docs: explore the index"></a></p>
<!-- project-badges:end -->

<!-- project-live-badges:start -->
<p align="center"><a href="https://github.com/raiinman/krita-mcp/commits/master"><img src="https://img.shields.io/github/last-commit/raiinman/krita-mcp?color=BF559F&amp;logo=git&amp;logoColor=white" alt="GitHub last commit"></a> <a href="https://github.com/raiinman/krita-mcp/issues"><img src="https://img.shields.io/github/issues/raiinman/krita-mcp?color=BF559F&amp;logo=github&amp;logoColor=white" alt="GitHub open issues"></a> <a href="https://github.com/raiinman/krita-mcp/stargazers"><img src="https://badgen.net/github/stars/raiinman/krita-mcp?icon=github&amp;color=BF559F" alt="GitHub stars"></a></p>
<!-- project-live-badges:end -->

<!-- project-index:start -->
<a name="readme-index"></a>
<h3 align="center">✦ Explore this project</h3>
<table align="center"><tbody><tr><td align="center"><a href="#readme-overview"><strong>Overview</strong></a></td><td align="center"><a href="#readme-how-it-works"><strong>How It Works</strong></a></td></tr><tr><td align="center"><a href="#readme-setup"><strong>Setup</strong></a></td><td align="center"><a href="#readme-available-tools"><strong>Available Tools</strong></a></td></tr><tr><td align="center"><a href="#readme-the-export-timeout-fix"><strong>The Export Timeout Fix</strong></a></td><td align="center"><a href="#readme-configuration"><strong>Configuration</strong></a></td></tr><tr><td align="center"><a href="#readme-painting-approach"><strong>Painting Approach</strong></a></td><td align="center"><a href="#readme-license"><strong>License</strong></a></td></tr></tbody></table>
<h4 align="center">Project shortcuts</h4>
<table align="center"><tbody><tr><td align="center"><a href="server.py"><strong>server.py</strong></a></td><td align="center"><a href="requirements.txt"><strong>requirements.txt</strong></a></td></tr><tr><td align="center"><a href="krita-plugin"><strong>krita-plugin</strong></a></td><td align="center"><a href="LICENSE"><strong>License</strong></a></td></tr></tbody></table>
<!-- project-index:end -->

<a name="readme-overview"></a>
<h2 align="center">Overview</h2>

Let AI paint in [Krita](https://krita.org/) via the [Model Context Protocol](https://modelcontextprotocol.io/).

This bridge allows Claude (or any MCP client) to create canvases, paint strokes, draw shapes, export images, and more — all inside a running Krita instance.

<p align="center"><a href="#readme-index">↑ Back to index</a></p>

<a name="readme-how-it-works"></a>
## How It Works


Two components:

<table align="center"><tbody><tr><td align="center">1</td><td align="center"><strong>Krita Plugin</strong> (<code>krita-plugin/</code>) — A Python plugin that runs inside Krita, exposing an HTTP server on <code>localhost:5678</code>. It receives paint commands and executes them on Krita's main thread via a command queue.</td></tr></tbody></table>


<table align="center"><tbody><tr><td align="center">2</td><td align="center"><strong>MCP Server</strong> (<code>server.py</code>) — A <a href="https://github.com/jlowin/fastmcp">FastMCP</a> server that exposes painting tools to any MCP client. It translates MCP tool calls into HTTP requests to the Krita plugin.</td></tr></tbody></table>


<table align="center"><tbody><tr><td align="left"><pre><code>MCP Client (Claude, etc.)  ←→  MCP Server (server.py)  ←→  Krita Plugin (HTTP on :5678)  ←→  Krita</code></pre></td></tr></tbody></table>


<p align="center"><a href="#readme-index">↑ Back to index</a></p>

<a name="readme-setup"></a>
## Setup


<a name="readme-1-install-the-krita-plugin"></a>
### 1. Install the Krita Plugin


Copy the plugin files to your Krita plugins directory:

<table align="center"><thead><tr><th align="center">OS</th><th align="center">Path</th></tr></thead><tbody><tr><td align="center">Windows</td><td align="center"><code>%APPDATA%\krita\pykrita\</code></td></tr><tr><td align="center">Linux</td><td align="center"><code>~/.local/share/krita/pykrita/</code></td></tr><tr><td align="center">macOS</td><td align="center"><code>~/Library/Application Support/krita/pykrita/</code></td></tr></tbody></table>


Copy both:
<table align="center"><tbody><tr><td align="center"><code>krita-plugin/kritamcp/</code> (the folder with <code>__init__.py</code>)</td></tr><tr><td align="center"><code>krita-plugin/kritamcp.desktop</code></td></tr></tbody></table>


Then in Krita: **Settings → Configure Krita → Python Plugin Manager → Enable "Krita MCP Bridge"** and restart Krita.

<a name="readme-2-install-the-mcp-server"></a>
### 2. Install the MCP Server


<table align="center"><tbody><tr><td align="left"><pre><code>pip install fastmcp httpx</code></pre></td></tr></tbody></table>


Or with a virtual environment:
<table align="center"><tbody><tr><td align="left"><pre><code>python -m venv .venv
source .venv/bin/activate  # or .venv\Scripts\activate on Windows
pip install -r requirements.txt</code></pre></td></tr></tbody></table>


<a name="readme-3-configure-your-mcp-client"></a>
### 3. Configure Your MCP Client


Add to your MCP client config (e.g., Claude Desktop's `claude_desktop_config.json`):

<table align="center"><tbody><tr><td align="left"><pre><code>{
  &quot;mcpServers&quot;: {
    &quot;krita&quot;: {
      &quot;command&quot;: &quot;python&quot;,
      &quot;args&quot;: [&quot;/path/to/server.py&quot;]
    }
  }
}</code></pre></td></tr></tbody></table>


If using a virtual environment:
<table align="center"><tbody><tr><td align="left"><pre><code>{
  &quot;mcpServers&quot;: {
    &quot;krita&quot;: {
      &quot;command&quot;: &quot;/path/to/.venv/Scripts/python&quot;,
      &quot;args&quot;: [&quot;/path/to/server.py&quot;]
    }
  }
}</code></pre></td></tr></tbody></table>


<p align="center"><a href="#readme-index">↑ Back to index</a></p>

<a name="readme-available-tools"></a>
## Available Tools


<table align="center"><thead><tr><th align="center">Tool</th><th align="center">Description</th></tr></thead><tbody><tr><td align="center"><code>krita_health</code></td><td align="center">Check if Krita is running with the plugin active</td></tr><tr><td align="center"><code>krita_new_canvas</code></td><td align="center">Create a new canvas (width, height, background color)</td></tr><tr><td align="center"><code>krita_set_color</code></td><td align="center">Set foreground paint color (hex)</td></tr><tr><td align="center"><code>krita_set_brush</code></td><td align="center">Set brush preset, size, and opacity</td></tr><tr><td align="center"><code>krita_stroke</code></td><td align="center">Paint a stroke through a list of [x, y] points</td></tr><tr><td align="center"><code>krita_fill</code></td><td align="center">Fill a circular area at a point</td></tr><tr><td align="center"><code>krita_draw_shape</code></td><td align="center">Draw rectangle, ellipse, or line</td></tr><tr><td align="center"><code>krita_get_canvas</code></td><td align="center">Export canvas to PNG (for AI to see progress)</td></tr><tr><td align="center"><code>krita_save</code></td><td align="center">Save canvas to a specific file path</td></tr><tr><td align="center"><code>krita_undo</code> / <code>krita_redo</code></td><td align="center">Undo/redo actions</td></tr><tr><td align="center"><code>krita_clear</code></td><td align="center">Clear canvas to a solid color</td></tr><tr><td align="center"><code>krita_get_color_at</code></td><td align="center">Eyedropper — sample color at a pixel</td></tr><tr><td align="center"><code>krita_list_brushes</code></td><td align="center">List available brush presets</td></tr><tr><td align="center"><code>krita_open_file</code></td><td align="center">Open an existing .kra, .png, .jpg, etc.</td></tr></tbody></table>


<p align="center"><a href="#readme-index">↑ Back to index</a></p>

<a name="readme-the-export-timeout-fix"></a>
## The Export Timeout Fix


**This is the main reason this repo exists.**

By default, HTTP requests and command queue operations time out after ~30 seconds. Canvas export (`get_canvas`) and file save (`save`) operations can easily exceed this on larger canvases, causing silent failures or timeout errors.

The fix is applied in two places:

**MCP Server (`server.py`)** — Extended timeout for export/save commands:
<table align="center"><tbody><tr><td align="left"><pre><code># In krita_get_canvas and krita_save:
result = send_command(&quot;get_canvas&quot;, {&quot;filename&quot;: filename}, timeout=120.0)
result = send_command(&quot;save&quot;, {&quot;path&quot;: path}, timeout=120.0)</code></pre></td></tr></tbody></table>


**Krita Plugin (`__init__.py`)** — Matching timeout in the command queue:
<table align="center"><tbody><tr><td align="left"><pre><code>def get_result(self, command_id, timeout=120):
    &quot;&quot;&quot;Wait for result with timeout.&quot;&quot;&quot;
    for _ in range(int(timeout * 10)):  # Check every 100ms
        ...</code></pre></td></tr></tbody></table>


**Both sides must match.** If only the MCP server timeout is increased, the plugin's command queue will still time out at 30s. If only the plugin timeout is increased, the HTTP request from the MCP server will time out first.

<p align="center"><a href="#readme-index">↑ Back to index</a></p>

<a name="readme-configuration"></a>
## Configuration


<table align="center"><thead><tr><th align="center">Setting</th><th align="center">Default</th><th align="center">How to Change</th></tr></thead><tbody><tr><td align="center">Plugin HTTP port</td><td align="center"><code>5678</code></td><td align="center">Edit <code>SERVER_PORT</code> in plugin <code>__init__.py</code></td></tr><tr><td align="center">MCP server URL</td><td align="center"><code>http://localhost:5678</code></td><td align="center">Set <code>KRITA_URL</code> env var</td></tr><tr><td align="center">Canvas output dir</td><td align="center"><code>~/krita-mcp-output</code></td><td align="center">Edit <code>CANVAS_OUTPUT_DIR</code> in plugin <code>__init__.py</code></td></tr></tbody></table>


<p align="center"><a href="#readme-index">↑ Back to index</a></p>

<a name="readme-painting-approach"></a>
## Painting Approach


The plugin paints using **direct pixel manipulation** (not Krita's native brush engine for strokes). This means:

<table align="center"><tbody><tr><td align="center">Strokes use a custom soft-circle renderer with configurable hardness</td></tr><tr><td align="center">Alpha blending is done manually in BGRA pixel format</td></tr><tr><td align="center">Shapes (rectangle, ellipse, line) are rasterized directly</td></tr><tr><td align="center">This approach is reliable and doesn't depend on Krita's internal brush state</td></tr></tbody></table>


The `set_brush` tool does set Krita's brush preset (for potential future use), but `stroke` currently uses its own pixel-level rendering.

<p align="center"><a href="#readme-index">↑ Back to index</a></p>

<a name="readme-license"></a>
## License


MIT

<p align="center"><a href="#readme-index">↑ Back to index</a> · <a href="#readme-top">Back to top ↑</a></p>

</div>
<!-- project-centered:end -->
