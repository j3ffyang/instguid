# AGENTS.md — instguid

Personal reference repo (name = "install-guide", kept short). Two kinds of content:
- Live per-device configs for Arch Linux + Hyprland — hyprland, kitty, vim.rc, waybar, etc. — archived as reference snapshots of what production actually runs.
- `shell*.txt` (3 files): bash shell-scripting tutorial notes plus sample shell scripts for production engineering, personal work carried since ~year 2000.

The big knowledge body lives in `README.md` and `instguid_2000-2025.md`; this file is the working constitution on top of the global rules. The repo is operated and maintained together with AI agents going forward.

## Working rules

- **Get approval before changes.** Present the plan and wait for the explicit go-ahead before editing files or running state-changing commands. Raise hands (ask) whenever anything is ambiguous — never guess at intent.
- **Proceed step-by-step, gradually and patiently.** Work section by section, verifying each step before the next; never batch-rewrite a whole file in one shot.
- **Get approval before touching a production file.** The `*.gpd`/live-config snapshots and the knowledge bodies (`README.md`, `instguid_2000-2025.md`) are the production truth — present plan + diff preview and wait for the go-ahead before editing them.
- **Cross-check ground truth when necessary.** When a line looks like an upstream rename/deprecation, a changed value, or an anomaly, verify against upstream and the installed system before deciding (see Ground truth below) — an error message or a diffed line is a clue, not a conclusion.
- **Commit only when asked.** Stage only intended files; never commit credentials or `.env`/key material (see global rules).
- **Never poison the working tree** with scratch artifacts; throwaway work goes in `/tmp/opencode/`.

## README.md formatting conventions

`README.md` is one big ```bash```-fenced text document. Conventions (verified 2026-09):

- **Single code fence.** The whole file lives inside one opening ```bash and one closing ``` — never add nested fences.
- **Indentation is TAB-based.** Commands and code blocks use a leading tab (tab + 4-space steps for nested/shell body lines, matching script function style). Blank lines and simple prose lines stay at col 0.
- **Left as-is, never "normalized":** verbatim terminal captures (`[root@... ~]#`, `mbp:~ jeff$`), ASCII-art/flowcharts, GUI-navigation prose (`panel > ...`, `system pref > ...`), and verbatim copies of config files (raidtab, XF86Config, OpenWrt `/etc/config/network`, fontconfig XML). These are content, not formatting noise.
- **Whitespace normalization is whitespace-only.** After any indentation pass, the content-integrity check is `git diff --ignore-all-space --ignore-blank-lines` — the only visible changes must be real, user-approved content edits. Anything else means a mistake (e.g. a Python slicing bug ate a character — recheck).

### Separator walls & the legend

The file uses repeated-character "walls" as searchable section markers. **All walls are exactly 60 characters, columns 0.** Word walls repeat the token and may truncate the last repeat mid-word (`cachyos` ends `...cach`) — that is intended; `grep '^mac'` still works.

Recipe (Python): `wall = (token * (60 // len(token) + 1))[:60]` — repeats the token to just past 60, truncates to exactly 60. For punctuation tokens (multiples of 60: `#`, `-`, `.`, `` ` ``) a plain `token * 60` is the same thing.

| Token  | Meaning |
|--------|---------|
| `=-=-...` (pair) | wraps a code block |
| `^^^^...` / `vvvv...` (pair) | wraps a command block with sample output in between |
| `arch...` | Arch Linux section |
| `FFFF...` | Fedora section |
| `UUUU...` | Ubuntu section |
| `python...` | Python section |
| `debian...`, `proxy...`, `mac...`, `cachyos...`, `hyprland...`, `nodejs...`, `vim...`, `git...`, `ssh...`, `openvpn...`, `markdown...`, `HSLT...` | sub-topic walls for that domain |
| `####...` / `....` | top-level section dividers (AIX, Linux command/network/management, ...) — legend item "=====" refers to these two |
| `<...` | a one-off wall before the Android section |

Keep new walls to the same 60-char width and reuse the existing tokens rather than inventing new ones.

## Config file conventions

- **Per-device suffix = that machine's live config.** Files like `hyprland.lua.gpd` are snapshots of the running `~/.config/...` on that specific device (e.g. `.gpd` = GPD Win 4, `.xps`, `.rog`). They are the source of truth for what production actually runs.
- **Date suffix = freshly downloaded upstream sample.** Files like `hyprland.lua.260917` are the latest example pulled from the project's GitHub repo. **Never edit a sample file** — it is re-downloaded as the baseline.
- **Sync workflow.** When a new release drops, keep the production file (`*.gpd`) in sync with the newest sample using the `sync-config-with-sample` skill (`.opencode/skills/`). Only copy reference info/URL comments from the sample into the production file — **never change actual configuration** in the production file, and verify the end state with `diff` so the only remaining differences are the user's real config.
- **Ground truth before deciding.** When a diffed line looks like it could be an upstream rename/deprecation (e.g. `dampening` → `damping`), verify against upstream PRs/wiki and the currently installed version (`hyprctl version`, `pacman -Q <pkg>`) before changing anything — keeping the old spelling is usually correct if the installed release predates the rename.

## Commit style

Short, descriptive, verb-first subject lines, no `type:` prefix (matches recent history):

- `sync hyprland.lua.gpd with current config`
- `add egpu priority verification by glxinfo`
- `remove nvtop and add amdgpu_top for amdgpu on gpd gamepad win4`

## Devices

`hyprland.lua`/`hyprland.conf` variants are keyed to the machine; verify which device a change targets before touching its file.