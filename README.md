# Boostcamp MCP Server

A Model Context Protocol (MCP) server for integrating with the [Boostcamp](https://www.boostcamp.app/) fitness platform. It gives Claude Desktop and Claude Code seamless access to your training history, workout programs, custom exercises, and analytics.

Built on the [`boostcamp-api`](https://github.com/dcaslin/boostcamp-api) Python library — a wrapper around Boostcamp's private API.

> **Attribution:** This project began as a fork of [`Alex-Keyes/boostcamp-mcp`](https://github.com/Alex-Keyes/boostcamp-mcp) and builds on [`Alex-Keyes/boostcamp-api`](https://github.com/Alex-Keyes/boostcamp-api) by [Alex Keyes](https://github.com/Alex-Keyes). It is now maintained as a standalone project. See [Credits](#-credits).

## 🚀 Quick Start

### 1. Installation

1. **Clone this repository**:
   ```bash
   git clone https://github.com/dcaslin/boostcamp-mcp.git
   cd boostcamp-mcp
   ```

2. **Install dependencies** with [`uv`](https://docs.astral.sh/uv/):
   ```bash
   uv sync
   ```

### 2. One-Time Authentication

Authentication runs through a standalone script so your credentials are never handled by the MCP client. From the project directory:

```bash
uv run login
```

Follow the prompts:
- Enter your Boostcamp email and password.
- The script authenticates against the Boostcamp API and saves your session token as `BOOSTCAMP_AUTH_TOKEN` in a local `.env` file (ignored by git).

Your email and password are only used to obtain the token and are never stored.

### 3. Register the Server with Claude

**Claude Code (CLI):**
```bash
claude mcp add boostcamp -- uv run --directory /path/to/your/boostcamp-mcp boostcamp-mcp
```

**Claude Desktop:** add this to your config file —
- **macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "boostcamp": {
      "command": "uv",
      "args": [
        "run",
        "--directory",
        "/path/to/your/boostcamp-mcp",
        "boostcamp-mcp"
      ]
    }
  }
}
```

**Important**: Replace `/path/to/your/boostcamp-mcp` with the actual path to your clone. The server reads `.env` from that directory, so it must match where you ran `uv run login`. Restart Claude Desktop after editing the config.

### 4. Start Using

Once connected, ask Claude to use the tools directly, e.g.:
- "Show my Boostcamp profile" → `get_my_profile`
- "What programs am I enrolled in?" → `list_enrolled_programs`
- "Review my recent workouts" → `get_training_history`
- "What's my dashboard streak and totals?" → `get_home_summary`

## ✨ Features

### 📊 Fitness Analytics
- **Home Summary**: total workouts, total weight moved, and current week streak.
- **Volume Charts**: training volume over time.
- **Muscle Distribution**: which muscle groups you've been targeting.

### 🏋️ Workout Management
- **Program Details**: full workout plans, including sets, reps, and coach notes.
- **Enrolled Programs**: track your progress in active training plans.
- **Custom Exercises**: access exercises you've manually created.

### 📚 Content & Discovery
- **Program Catalog**: search and list programs available on the platform.
- **Blog Access**: read the latest articles and training guides from the Boostcamp blog.

## 🛠️ Available Tools

| Tool | Description | Parameters |
|------|-------------|------------|
| `get_my_profile` | Get user profile and settings | None |
| `list_enrolled_programs` | List your active programs | None |
| `get_training_history` | Get detailed workout history | `timezone_offset` |
| `get_payment_history` | View your subscription/orders | None |
| `list_custom_exercises` | List your custom exercises | None |
| `list_all_programs` | Search the program catalog | `page`, `page_size`, `keyword` |
| `get_program_details` | Get the full plan for a program ID | `program_id` |
| `list_blogs` | List recent blog posts | `page`, `page_size` |
| `get_home_summary` | Dashboard stats (streak/totals) | `timezone_offset` |
| `get_home_programs` | Active/recent program summary | `timezone_offset` |
| `get_home_chart` | Training volume chart data | `timezone_offset` |
| `get_home_muscle` | Muscle group distribution | `timezone_offset` |

`timezone_offset` is in minutes from UTC and defaults to `-300`.

## 🔧 Troubleshooting

### Authentication Issues
If a tool returns an "Authentication Error" or your token has expired:
1. Re-run the login command from the project directory: `uv run login`
2. Restart your MCP client (Claude Desktop or Claude Code).

### Security Notes
- The token lives in `.env` as `BOOSTCAMP_AUTH_TOKEN`; `.env` is excluded by `.gitignore`.
- Never commit your `.env`.

## 📄 License

Released under the [MIT License](LICENSE).

## 🙏 Credits

- Original MCP server: [`Alex-Keyes/boostcamp-mcp`](https://github.com/Alex-Keyes/boostcamp-mcp) by [Alex Keyes](https://github.com/Alex-Keyes).
- Underlying API library: [`Alex-Keyes/boostcamp-api`](https://github.com/Alex-Keyes/boostcamp-api) by [Alex Keyes](https://github.com/Alex-Keyes), maintained here as [`dcaslin/boostcamp-api`](https://github.com/dcaslin/boostcamp-api).
