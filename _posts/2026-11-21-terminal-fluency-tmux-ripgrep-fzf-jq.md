---
title: "Terminal Fluency for the Agent Era: tmux, ripgrep, fzf and jq From Zero"
date: 2026-11-21 22:30:00
author: Felipe Silvestre
categories: [tech]
tags: [terminal, tmux, ripgrep, fzf, jq, cli, productivity, developer-tools, ai-agents]
---

In my last post about [running parallel agents with git worktrees]({% post_url 2026-11-21-parallel-agents-git-worktrees-beyond-the-prompt %}), I listed "terminal fluency" as one of the skills that matters more now. A few people asked the obvious follow-up: *fine, but where do I start?* Several of them had never opened tmux, or had heard of `jq` but never used it.

So this post starts from zero. No assumptions. If you know what a terminal is and you can run `ls`, you are ready.

We will cover four small tools:

- **tmux**: keeps your terminal sessions alive and lets you split one window into many.
- **ripgrep (`rg`)**: searches your whole codebase in milliseconds.
- **fzf**: a "type a few letters and pick" menu for anything.
- **jq**: reads, filters and reshapes JSON.

## Why Bother, If an Agent Can Do It?

Fair question. Here is my honest answer.

When you work with coding agents, you spend a lot of time **around** the agent: starting several of them, checking what they changed, finding where something is used, reading a giant JSON response or a log file, jumping back to a session you started two hours ago. An agent run costs time and money. A two-second `rg` costs neither.

There is also a trust angle. When an agent tells you "I updated all the callers", you want a fast, independent way to check. These four tools are that. They are also tiny, free, stable (most are older than the current AI wave) and they will still be useful in ten years.

---

## Before We Start: Setup

**macOS:**

```bash
brew install tmux ripgrep fzf jq
```

**Ubuntu, Debian and WSL:**

```bash
sudo apt update
sudo apt install tmux ripgrep fzf jq
```

**Windows:** tmux does not run natively on Windows. The simplest path is **WSL** (Windows Subsystem for Linux): install an Ubuntu distribution, open it, and use the Ubuntu commands above. Everything in this post works there. ripgrep, fzf and jq also have native Windows builds if you prefer to stay in PowerShell, but tmux needs WSL.

To check everything is installed:

```bash
tmux -V && rg --version && fzf --version && jq --version
```

I tested the examples in this post with tmux 3.4 (on Ubuntu in WSL), ripgrep 15.1, fzf 0.72 and jq 1.8.1. Older versions mostly behave the same, and I will point out the one place where version matters.

---

# Part 1: tmux

## What Problem Does It Solve?

A normal terminal window has three annoying limits:

1. **Close the window and everything running in it dies.** A long build, a dev server, an agent mid-task: gone.
2. **If you work over SSH and your connection drops, same story.** Your process is killed with the connection.
3. **One window is one shell.** To run an agent, a dev server and a test watcher, you juggle three windows or tabs, and you lose track of which is which.

**tmux** (short for *terminal multiplexer*) fixes all three. It runs as a background service that owns your terminal sessions. Your terminal window becomes just a *viewer* attached to it. Close the viewer and the sessions keep running. Open a new terminal later and re-attach, and everything is exactly where you left it.

A good mental model is a **TV and a streaming box**. The tmux session is the box, running continuously. Your terminal window is the TV: turning it off does not stop the box.

## The Three Words You Need

tmux has three nested ideas, and once you know them, the rest is detail:

| Concept | What it is | Analogy |
|---|---|---|
| **Session** | A whole workspace that keeps running in the background. | A desk set up for one project. |
| **Window** | A full-screen "tab" inside a session. | The tabs in a browser. |
| **Pane** | A split inside a window; each pane is its own shell. | Two documents side by side. |

So one session can contain several windows, and each window can be divided into several panes.

## Your First Five Minutes

Start a new session and give it a name:

```bash
tmux new -s work
```

Your screen clears and you will see a green bar at the bottom. That is the tmux status line. You are now *inside* tmux, and it looks like a normal terminal. Type `echo hello` to prove it still works.

### The One Thing to Understand: the Prefix Key

tmux needs a way to tell "this keystroke is for the shell" from "this keystroke is for tmux". It does that with a **prefix key**. By default it is **`Ctrl+b`**.

Every tmux command is: press `Ctrl+b`, **release both keys**, then press one more key. This trips up almost everyone at the start. It is *not* `Ctrl+b+d` held down together. It is two separate moments.

I will write it as `Ctrl+b` then `d`.

### Detach and Re-Attach (the Magic Trick)

Press `Ctrl+b` then **`d`**. You are dropped back to your normal terminal, with a line like `[detached (from session work)]`. Your session is still running in the background. Prove it:

```bash
tmux ls
# work: 1 windows (created Sat Nov 21 23:17:02 2026)
```

Now re-attach:

```bash
tmux attach -t work
```

Everything is exactly as you left it. You can now close your terminal window completely, reopen it tomorrow, run `tmux attach -t work`, and carry on. If you only remember one thing from this whole section, remember this: **detach and attach is the point of tmux.**

A handy shortcut: `tmux new -A -s work` means "attach to the session called `work` if it exists, otherwise create it". It is the only command you really need day to day.

### Splitting the Screen

Inside a session, try these:

| Keys | What happens |
|---|---|
| `Ctrl+b` then `%` | Split into **left and right** panes |
| `Ctrl+b` then `"` | Split into **top and bottom** panes |
| `Ctrl+b` then an **arrow key** | Move to the pane in that direction |
| `Ctrl+b` then `o` | Cycle to the next pane |
| `Ctrl+b` then `z` | **Zoom** the current pane full screen; press again to restore |
| `Ctrl+b` then `x` | Close the current pane (it asks you to confirm with `y`) |

The `z` key is underrated. When you have four panes open and want to read one properly, zoom it, read it, zoom back.

### Windows (Tabs)

| Keys | What happens |
|---|---|
| `Ctrl+b` then `c` | Create a new window |
| `Ctrl+b` then `n` / `p` | Next / previous window |
| `Ctrl+b` then `0`...`9` | Jump to window by number |
| `Ctrl+b` then `,` | Rename the current window |
| `Ctrl+b` then `w` | Pick from a list of all windows |

The names appear in the status bar at the bottom, so renaming windows to `agent`, `server`, `tests` makes it obvious where everything is.

### Sessions

| Keys / command | What happens |
|---|---|
| `Ctrl+b` then `d` | Detach (leave the session running) |
| `Ctrl+b` then `s` | Pick from a list of sessions |
| `Ctrl+b` then `$` | Rename the current session |
| `tmux ls` | List sessions (from outside tmux) |
| `tmux attach -t name` | Re-attach to a session |
| `tmux kill-session -t name` | Destroy a session for good |

### "Help, I Cannot Scroll!"

Everyone hits this. Your mouse wheel often does not scroll tmux's output the way a normal terminal does, because by default **tmux's mouse support is off** (I confirmed `mouse off` is the default). You have two options.

**Option 1: keyboard scrolling.** Press `Ctrl+b` then `[` to enter *copy mode*. Now the arrow keys and Page Up / Page Down scroll through history. Press `q` to leave.

**Option 2: turn the mouse on.** Create a file called `~/.tmux.conf` with this inside:

```text
# make the mouse work for scrolling and clicking panes
set -g mouse on

# keep more history
set -g history-limit 50000

# number windows and panes from 1, which matches the keyboard row
set -g base-index 1
setw -g pane-base-index 1
```

Then reload it without restarting. Inside tmux, press `Ctrl+b` then `:` (that opens a command prompt) and type:

```text
source-file ~/.tmux.conf
```

Now the mouse wheel scrolls and you can click on a pane to focus it. That small file is enough for a good first setup. Resist the urge to copy a 300-line config from the internet until you know what you actually miss.

### Forgot a Key?

`Ctrl+b` then `?` shows the full list of key bindings. Press `q` to close it.

## Using tmux With Coding Agents

This is where tmux earns its place in an agent workflow:

- **A long agent run survives you closing the laptop lid or losing SSH.** Start the agent inside tmux, detach, come back later.
- **One pane per agent.** If you are running agents in separate git worktrees (as in the last post), you get a dashboard: one agent per pane, all visible at once, and `Ctrl+b` then `z` zooms whichever one needs attention.
- **A reproducible layout** is a few commands. For two worktrees:

```bash
tmux new-session -d -s agents -c ~/code/myapp-fix-login
tmux split-window -h -c ~/code/myapp-search
tmux attach -t agents
```

`-c` sets the starting directory, `-d` creates the session without attaching to it yet, and the final command attaches. Put those lines in a script and your whole agent setup is one command.

**Practice exercise (5 minutes):** create a session, split it into three panes, run `ls` in one, `git status` in another and `date` in the third, detach, close your terminal, reopen it, and re-attach. If you can do that, you know enough tmux.

---

# Part 2: ripgrep

## What Is It?

**ripgrep**, whose command is `rg`, searches for text across every file in a folder, recursively, extremely quickly. If you have used `grep -r`, think of it as the modern replacement, with sensible defaults:

- It searches the **current folder and everything below it** without you asking.
- It **respects your `.gitignore`**, so it skips `node_modules`, build output and anything else your project ignores. That alone removes most of the noise.
- It skips hidden files and binary files by default.
- It is fast enough that you stop thinking about whether a search is "too big".

## The Basics

I made a tiny project to try these. It has a Python file with a `get_user` function, a JavaScript file, a test, and a `node_modules` folder that is in `.gitignore`.

Search for a word:

```bash
rg get_user
# tests/test_users.py:def test_get_user(): pass
# src/api/users.py:def get_user(user_id):
```

It found both places and skipped the copy inside `node_modules`, because that folder is gitignored. In a real terminal, the output is grouped by file and the matches are highlighted.

Show line numbers, which you will almost always want:

```bash
rg -n TODO
# src/api/users.py:2:    # TODO: add caching
# src/ui/button.js:2:  // TODO: handle disabled state
```

### The Flags Worth Learning

| Flag | Meaning | Example |
|---|---|---|
| `-n` | Show line numbers | `rg -n TODO` |
| `-i` | Ignore upper/lower case | `rg -i button` |
| `-w` | Match whole words only | `rg -w user` |
| `-F` | Treat the pattern as plain text, not a regular expression | `rg -F "a.b(c)"` |
| `-l` | Only list the **files** that match | `rg -l get_user` |
| `-c` | Count matches per file | `rg -c TODO` |
| `-C 2` | Show 2 lines of context around each match | `rg -C 2 delete_user` |
| `-t py` | Only search one file type | `rg -t py get_user` |
| `-g` | Filter by file glob; `!` excludes | `rg -g '!tests' get_user` |
| `--no-ignore` | Also search ignored files | `rg --no-ignore -l get_user` |
| `--files` | List files instead of searching them | `rg --files -g '*.py'` |

A few of those in action:

```bash
# every Python file that mentions get_user
rg -t py -l get_user
# tests/test_users.py
# src/api/users.py

# the same search, but skip the tests folder
rg -g '!tests' get_user
# src/api/users.py:def get_user(user_id):

# two lines of context after each match
rg -C1 delete_user
# src/api/users.py-
# src/api/users.py:def delete_user(user_id):
# src/api/users.py-    raise NotImplementedError

# search even the ignored folders (here that finds the node_modules copy too)
rg --no-ignore -l get_user
```

Use `rg --type-list` to see which file types `-t` knows (it includes `py`, `js`, `ts`, `md` and many more).

### Two Gotchas

1. **The pattern is a regular expression by default.** If you search for something with special characters, like `foo(` or `a.b`, you will get surprising results or an error. Use `-F` to search for plain text.
2. **If you do not give a folder and the input is coming from a pipe, `rg` searches the piped input, not the current folder.** This bit me while writing this post: I ran `rg get_user` from a non-interactive script and it sat waiting for input that never came. In your normal terminal it is fine, but in scripts and automation, add a path: `rg get_user .`

## Using ripgrep With Coding Agents

- **Verify the agent's claims.** "I updated every caller of `get_user`" becomes `rg -n 'get_user\('` and thirty seconds of reading.
- **Size a change before you ask for it.** `rg -l oldFunctionName | wc -l` tells you how many files a rename touches. That is a good way to decide whether to split the work into several parallel tasks.
- **Find what to point the agent at.** Instead of "somewhere in the auth code", you can tell it exactly which files to read.
- **Find leftovers.** After a big agent change, `rg -n 'TODO|FIXME|console\.log|debugger'` catches the debris.

---

# Part 3: fzf

## What Is It?

**fzf** is a *fuzzy finder*. You give it a list of anything (files, git branches, command history, processes) and it gives you a live, filterable menu. You type a few letters, in any order, and the list narrows instantly. Press Enter and the chosen line is printed.

*Fuzzy* means your letters do not need to be next to each other. Typing `usr py` matches `src/api/users.py` and `tests/test_users.py`. Typing `btn` matches `src/ui/button.js`. I checked these with fzf's non-interactive mode:

```bash
printf 'src/api/users.py\nsrc/ui/button.js\ntests/test_users.py\n' | fzf --filter "usr py"
# src/api/users.py
# tests/test_users.py
```

(`--filter` just prints the matches instead of opening the interactive menu, which is handy for showing examples.)

## The Single Best Thing: Better Shell History

Install fzf's shell integration, and three keyboard shortcuts appear everywhere in your terminal. Add one line to your shell's config:

```bash
# in ~/.bashrc (bash)
eval "$(fzf --bash)"

# in ~/.zshrc (zsh)
source <(fzf --zsh)
```

(That form needs fzf 0.48 or newer. On older versions, the installer or the project README shows the equivalent script to source.) Restart your shell, then:

| Keys | What it does |
|---|---|
| `Ctrl+r` | Fuzzy-search your **command history**. Type a few letters of an old command, press Enter. |
| `Ctrl+t` | Fuzzy-pick a **file** and paste its path into the command you are typing. |
| `Alt+c` | Fuzzy-pick a **folder** and `cd` into it. |

`Ctrl+r` alone is worth installing fzf for. The classic "what was that long docker command I ran last week?" becomes four keystrokes.

## Using fzf in Your Own Commands

Anything that prints lines can be piped into `fzf`, and whatever you pick comes out the other side:

```bash
# open a file you pick in your editor
code "$(fzf)"

# switch git branch from a list
git switch "$(git branch --format='%(refname:short)' | fzf)"

# list files that mention TODO, pick one, and preview the matching lines as you move
rg -l TODO | fzf --preview 'rg -n --color=always TODO {}'
```

In the last example, `{}` is replaced by whatever line you are currently highlighting, and `--preview` shows a live panel on the right. It is a small thing that makes exploring a codebase pleasant.

Some combinations I use constantly: pick a **git worktree** to `cd` into (`git worktree list | fzf`), pick a **tmux session** to attach to (`tmux ls | fzf`), pick a **process** to inspect. Whenever you catch yourself copying a long name out of a list, fzf is probably the answer.

---

# Part 4: jq

## What Is It?

A huge amount of what developers read every day is **JSON**: API responses, configuration files, logs, package manifests, and the structured output of many tools, including the ones that run agents. Raw JSON is often one enormous line, which is unreadable. **jq** is a command-line tool to format, filter and reshape it.

Think of it as `grep` and `awk` for JSON, understanding the structure instead of treating it as text.

## A Worked Example

Imagine an API returned this, saved in `resp.json`:

```json
{
  "status": "ok",
  "users": [
    { "id": 1, "name": "Ana",   "email": "ana@example.com",   "active": true,  "roles": ["admin", "dev"] },
    { "id": 2, "name": "Bruno", "email": "bruno@example.com", "active": false, "roles": ["dev"] },
    { "id": 3, "name": "Carla", "email": "carla@example.com", "active": true,  "roles": [] }
  ]
}
```

(Pretend it arrived as one unbroken line. jq does not care.) Every jq command is: `jq 'FILTER' file`, where the filter is a small expression describing what you want. **Always wrap the filter in single quotes** so the shell does not eat the special characters.

**Pretty-print it.** The simplest filter is `.`, meaning "the whole thing":

```bash
jq . resp.json
```

That alone turns an unreadable blob into indented, colored output. You can pipe straight from a request too: `curl -s https://api.example.com/users | jq .`

**Pull out a field.** `.users` is the list. `.users[]` means "each item in the list". So:

```bash
jq '.users[].name' resp.json
# "Ana"
# "Bruno"
# "Carla"
```

**Remove the quotes** with `-r` (raw output), which you want whenever the result is going to another command:

```bash
jq -r '.users[].name' resp.json
# Ana
# Bruno
# Carla
```

**Filter with `select`.** Only the emails of active users. The `|` inside a filter works like a shell pipe, passing each item to the next step:

```bash
jq -r '.users[] | select(.active) | .email' resp.json
# ana@example.com
# carla@example.com
```

**Reshape.** Build new, smaller objects with only the fields you want. `-c` prints each on one line:

```bash
jq -c '.users[] | {id, name}' resp.json
# {"id":1,"name":"Ana"}
# {"id":2,"name":"Bruno"}
# {"id":3,"name":"Carla"}
```

**Count things:**

```bash
jq '.users | length' resp.json
# 3
```

**Look for a value inside an array**, like "who has the admin role?":

```bash
jq '[.users[] | select(.roles | index("admin")) | .name]' resp.json
# [
#   "Ana"
# ]
```

**Turn JSON into a table** that spreadsheets and Unix tools understand, with `@tsv`:

```bash
jq -r '.users[] | [.id, .name, .active] | @tsv' resp.json
# 1	Ana	true
# 2	Bruno	false
# 3	Carla	true
```

**Not sure what is in a file?** Start with `jq 'keys' file.json` to see the top-level fields, then dig down one level at a time.

## Logs: JSON Lines

Many modern tools write logs as **one JSON object per line** (called JSON Lines). jq handles that natively and it is perfect for finding the needle:

```bash
cat app.log
# {"level":"info","msg":"start"}
# {"level":"error","msg":"db down","svc":"api"}
# {"level":"error","msg":"timeout","svc":"web"}

jq -r 'select(.level == "error") | "\(.svc): \(.msg)"' app.log
# api: db down
# web: timeout
```

That last filter picked out only the errors and formatted each as `service: message`. The `\( ... )` part is jq's way of inserting a value into a string.

## Using jq With Coding Agents

- **Inspect structured output.** When a tool or agent produces JSON, `jq .` makes it readable, and `jq 'keys'` shows what is in there.
- **Read logs without scrolling.** `jq -r 'select(.level=="error")'` turns a 50,000-line log into the five lines that matter, which you can then paste into the agent's context.
- **Check config and manifests.** What version does `package.json` say? `jq -r .version package.json`.
- **Trim before you paste.** Giving an agent a 3,000-line API response wastes its context. A jq filter that keeps the 20 relevant lines gets you a better answer for less.

---

# Putting It All Together

None of these tools is impressive alone. The leverage comes from combining them. A morning example:

```bash
# 1. Open (or re-open) my workspace; the agents kept running overnight
tmux new -A -s agents

# 2. Which files still contain TODOs? Pick one and preview the matches
rg -l TODO | fzf --preview 'rg -n --color=always TODO {}'

# 3. What errors happened overnight, and in which service?
jq -r 'select(.level == "error") | "\(.svc): \(.msg)"' app.log | sort | uniq -c
```

Three commands, no agent runs, a couple of minutes, and you now know the state of your project.

## A Cheat Sheet to Screenshot

| Tool | The one thing to remember |
|---|---|
| **tmux** | `tmux new -A -s work` to start or resume; `Ctrl+b` then `d` to detach. |
| **tmux panes** | `Ctrl+b` then `%` or `"` to split; arrows to move; `z` to zoom. |
| **rg** | `rg -n pattern` to find; `-l` for files only; `-t py` and `-g` to narrow. |
| **fzf** | `Ctrl+r` for history; pipe any list into `fzf` to pick from it. |
| **jq** | `jq .` to pretty-print; `-r` for raw text; `select()` to filter. |

## A One-Week Plan

You do not need to learn all of this at once. Here is a gentle order:

- **Day 1:** install everything. Run `rg` instead of your editor's search, three times.
- **Day 2:** add the fzf line to your shell config and use `Ctrl+r` all day.
- **Day 3:** pipe one API response or log file through `jq .` and then through `select`.
- **Day 4:** do your normal work inside `tmux new -A -s work`. Detach and re-attach once, on purpose.
- **Day 5:** split panes, add the three-line config file, and set up a two-pane layout for your two projects.

By the end of the week, the muscle memory is mostly there.

---

# The Honest Caveats

- **tmux is not native on Windows.** You need WSL, which is a perfectly good setup, but it is one more thing to install. Alternatives like Zellij exist and are friendlier in some ways, but I have not tested one for this post, so I am only recommending what I used.
- **The keys I listed are tmux's defaults**, and I checked them against tmux 3.4. People often remap the prefix key (to `Ctrl+a`, for example) in their config, so if you borrow a colleague's setup, the keys may differ.
- **Tools evolve.** jq's newer versions and fzf's newer versions behave the same for everything in this post, but the exact install line for fzf's shell integration has changed over the years. If `fzf --bash` fails, check the fzf README for your version.
- **This is a starter kit, not a mastery guide.** Each tool goes deeper: tmux has plugins and scripting, rg has powerful regular expressions and replacement tricks, jq is a small programming language. Learn them as you hit real needs.
- **Some of it is taste.** If you prefer your editor's search over `rg`, or a different multiplexer, that is fine. The principle that matters is getting comfortable with *some* fast way to search, split, pick and filter without leaving the keyboard.

---

# Final Thoughts

It is tempting to think the terminal skills of the previous decade are obsolete, because you can now just ask an agent to find things. I think the opposite is true. Agents live in the terminal. The more you delegate, the more valuable it is to have fast, trustworthy tools of your own to check, explore and steer, without spending a model run on a question that `rg` answers in 50 milliseconds.

And the cost of learning is small. You can get genuinely useful with all four in a week of casual practice, and they will repay you every day after.

Open a terminal and run `tmux new -A -s work`. Everything else gets easier from there.

---

*Further reading: the [tmux wiki](https://github.com/tmux/tmux/wiki), the [ripgrep user guide](https://github.com/BurntSushi/ripgrep/blob/master/GUIDE.md), the [fzf README](https://github.com/junegunn/fzf), and the [jq manual](https://jqlang.org/manual/). If you want to go from here to parallel agents, my post on [git worktrees]({% post_url 2026-11-21-parallel-agents-git-worktrees-beyond-the-prompt %}) shows how these tools fit around them.*
