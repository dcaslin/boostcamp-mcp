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
| `get_training_history` | Workout history, compact by default. Returns JSON with pagination metadata and a `has_more` flag so large histories can be walked in chunks. | `start_date`, `end_date`, `detail`, `page`, `page_size`, `timezone_offset` |
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

### Working with training history

`get_training_history` is built to keep responses small enough for any MCP
client. By default it returns a **summary** of your **50 most recent** workouts
(date, program, exercises, and total volume) plus pagination metadata:

```json
{
  "workouts": [ { "date": "2026-06-10", "program_name": "...",
                  "exercises": ["Squat (Barbell)", "..."],
                  "total_volume": 11570, "volume_unit": "lbs" } ],
  "pagination": {"page": 1, "page_size": 50, "total": 231,
                 "returned": 50, "has_more": true},
  "filters": {"start_date": null, "end_date": null, "detail": "summary"},
  "hint": "231 workouts match. Showing 50 (page 1, summary). Use page=2 ..."
}
```

- **Filter by date:** `start_date` / `end_date` as `YYYY-MM-DD` (inclusive),
  e.g. *"my workouts since 2026-04-07"*.
- **Page through history:** bump `page` while `pagination.has_more` is `true`
  (`page_size` caps at 100 for summary, 25 for full).
- **Get every set and rep:** `detail="full"` adds per-exercise records with each
  set's weight, reps, target, and RPE. Use a small `page_size` here.

Supersets are flattened: each exercise inside a superset is listed individually
(with its sets counted toward `total_volume`), and in `full` detail those
exercises carry a `superset` id so the grouping is still visible.

## 🔧 Troubleshooting

### Authentication Issues
If a tool returns an "Authentication Error" or your token has expired:
1. Re-run the login command from the project directory: `uv run login`
2. Restart your MCP client (Claude Desktop or Claude Code).

### Session Management
- Sessions are stored in `.boostcamp/session.pickle` as `{token, refresh_token}`.
  The Firebase **refresh token** (not your password) is persisted, so an expired
  ID token is renewed automatically on the next request — no re-login needed for
  routine expiry.
- A `BOOSTCAMP_AUTH_TOKEN` is also saved to `.env` and used as a fallback for
  logins that predate refresh-token support (token-only, no auto-renewal).
- **Upgrading:** if you logged in before refresh-token support was added, run
  `uv run login` once to store the refresh token and enable auto-renewal.
- You only need to re-run `uv run login` if the refresh token itself becomes
  invalid (e.g. after a password change).
- **Security Note**: Never commit your `.env` or `.boostcamp/` folder. They are included in `.gitignore` by default.

## 📄 License

Released under the [MIT License](LICENSE).

## 🙏 Credits

- Original MCP server: [`Alex-Keyes/boostcamp-mcp`](https://github.com/Alex-Keyes/boostcamp-mcp) by [Alex Keyes](https://github.com/Alex-Keyes).
- Underlying API library: [`Alex-Keyes/boostcamp-api`](https://github.com/Alex-Keyes/boostcamp-api) by [Alex Keyes](https://github.com/Alex-Keyes), maintained here as [`dcaslin/boostcamp-api`](https://github.com/dcaslin/boostcamp-api).
