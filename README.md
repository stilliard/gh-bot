# d2

## gh-bot

`bin/gh-bot` posts comments and creates/updates issues as a GitHub App bot
instead of your own `gh auth login` identity. Useful so automated activity
(Claude Code, Codex, etc.) is visibly distinguishable from actions you take
yourself.

The App's installation ID is looked up per-repo at runtime (via the App's
own JWT), so the same App ID/private key work across any account or org the
App is installed on — no per-org config needed.

### Required App permissions

Your GitHub App needs the following repository permissions (GitHub App
settings → Permissions & events):

| Permission              | Access         | Needed for                                     |
|-------------------------|----------------|------------------------------------------------|
| Metadata                | Read-only      | Required baseline for all GitHub Apps          |
| Issues                  | Read and write | `comment`, `issue-create`, `issue-update`      |
| Pull requests           | Read and write | `comment`, `reply` (PR review comment threads) |

After changing permissions on an existing App, you must accept the updated
permissions on each installation (GitHub will prompt you, or re-visit
`https://github.com/settings/installations`).

### Setup

1. Create a GitHub App and install it on whichever accounts/orgs you want it
   to act in, and on the specific repos within each.
2. Generate a private key for the App (GitHub App settings → "Generate a
   private key") and save the `.pem` file somewhere local (not in the repo).
3. Copy `.env.example` to `.env` and fill in:
   - `GH_BOT_APP_ID` — the App's ID
   - `GH_BOT_PRIVATE_KEY_PATH` — path to the `.pem` file

### Making it available on your PATH

`gh-bot` resolves its own real location (following symlinks) to find `.env`
next to it, so it's safe to symlink onto your `PATH` instead of always
calling it via `bin/gh-bot`:

```sh
ln -s "$(pwd)/bin/gh-bot" ~/.local/bin/gh-bot   # user-level, no sudo
# or
sudo ln -s "$(pwd)/bin/gh-bot" /usr/local/bin/gh-bot   # system-wide
```

(Avoid `/usr/bin` — that's reserved for distro-managed packages; use
`/usr/local/bin` for locally-installed system-wide tools.) Once linked, just
run `gh-bot comment ...` from anywhere.

### Usage

```sh
# General issue/PR comment
gh-bot comment --repo owner/repo --number 123 "Looks good to me."

# Pipe a longer body in via stdin
gh-bot comment --repo owner/repo --number 123 <<'EOF'
Multi-line
comment body.
EOF

# Reply to an existing PR review (line) comment thread
gh-bot reply --repo owner/repo --pr 123 --comment-id 987654321 \
  "Fixed in a1b2c3d, thanks!"

# Create an issue
gh-bot issue-create --repo owner/repo --title "Bug: crash on save" \
  --label bug --assignee octocat "Steps to reproduce..."

# Update an issue (title/state/labels/assignees and/or body)
gh-bot issue-update --repo owner/repo --number 42 --state closed
gh-bot issue-update --repo owner/repo --number 42 --title "New title" --label bug --label p1
```

Run `gh-bot --help` or `gh-bot <command> --help` (e.g. `gh-bot issue-update
--help`) for full option details on any command.

Each invocation mints a fresh short-lived installation token (no caching),
so no bot credentials are ever persisted outside your `.env`/`.pem`.
