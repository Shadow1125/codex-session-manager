# Codex Session Manager

A Codex skill for listing, searching, and safely deleting local Codex conversation history.

This repository is both:

- a portable Codex skill, defined by `SKILL.md`
- a small command-line utility, implemented in `scripts/delete_codex_session.py`

The script reads local Codex history files, builds a safer session list, and removes selected sessions with dry runs, confirmations, backups, and atomic JSONL rewrites.

## Features

- Lists sessions from multiple local Codex data sources
- Searches by session preview, title, id, and optional parsed session-file text
- Deletes by current list index or stable session id
- Shows a dry-run plan before changing files
- Moves matching session files into timestamped backups instead of unlinking them
- Backs up `history.jsonl` and `session_index.jsonl` before rewriting
- Writes JSONL files atomically
- Uses a lock file to avoid concurrent destructive operations

## Data Sources

By default, the script reads from `~/.codex`, or from `$CODEX_HOME` when that environment variable is set.

It combines session information from:

- `history.jsonl`
- `session_index.jsonl`
- `sessions/**/*.json`
- `sessions/**/*.jsonl`

## Requirements

- Python 3.10 or newer
- macOS, Linux, or another environment with a local Codex data directory
- No third-party Python dependencies

## Installation As A Codex Skill

Clone this repository into your Codex skills directory:

```bash
mkdir -p ~/.codex/skills
git clone https://github.com/Shadow1125/codex-session-manager.git ~/.codex/skills/codex-session-manager
```

After installation, ask Codex to use `codex-session-manager` when you want to list, search, inspect, or safely delete local Codex sessions.

## CLI Usage

Run commands from the repository directory:

```bash
cd ~/.codex/skills/codex-session-manager
```

List sessions:

```bash
python3 scripts/delete_codex_session.py --list
```

Search by preview, title, or id:

```bash
python3 scripts/delete_codex_session.py --list --search "keyword"
```

Search parsed session-file text too:

```bash
python3 scripts/delete_codex_session.py --list --search "keyword" --full-text
```

Preview deletion by current list index:

```bash
python3 scripts/delete_codex_session.py --delete 3 --dry-run
```

Preview deletion by a range of current list indexes:

```bash
python3 scripts/delete_codex_session.py --delete 2,5-7 --dry-run
```

Delete by stable session id:

```bash
python3 scripts/delete_codex_session.py --delete-id "00000000-0000-0000-0000-000000000000" --yes
```

Use a non-default Codex data directory:

```bash
python3 scripts/delete_codex_session.py --codex-dir /path/to/.codex --list
```

## Safety Model

Deletion is intentionally conservative.

Before destructive changes, the script can show:

- selected sessions
- matching history rows
- matching index rows
- matching session files
- backup directory location

When deletion runs, it creates a backup under:

```text
~/.codex/history_backups/session-delete-<timestamp>/
```

The backup includes:

- copies of affected history/index files
- moved session files
- `manifest.json` with deleted session ids and moved file paths

Use `--dry-run` whenever you are unsure:

```bash
python3 scripts/delete_codex_session.py --delete 3 --dry-run
```

Indexes are ephemeral. Always run the relevant list or search command immediately before deleting by index. If you searched with `--search` or `--full-text`, repeat the same filters for the delete command, or use `--delete-id`.

## Recovery

If you need to recover a deleted session, inspect the latest backup directory:

```bash
ls -lt ~/.codex/history_backups/
```

Each deletion backup contains a `manifest.json` file that records which session ids were deleted and where session files were moved.

## Development

Validate the script syntax:

```bash
python3 -m py_compile scripts/delete_codex_session.py
```

Check the CLI:

```bash
python3 scripts/delete_codex_session.py --help
```

Validate the skill structure with the Codex skill creator validator when available:

```bash
python3 /path/to/skill-creator/scripts/quick_validate.py .
```

## Project Layout

```text
codex-session-manager/
├── SKILL.md
├── agents/
│   └── openai.yaml
├── scripts/
│   └── delete_codex_session.py
├── LICENSE
└── README.md
```

## License

MIT
