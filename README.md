# 🐧 Linux For Beginners

A personal reference guide documenting core Linux concepts, commands, and tricks. Built while learning through hands-on practice.

---

## 📚 Table of Contents

1. [Man Pages](#man-pages)
2. [Globbing](#globbing)
3. [Piping & Redirection](#piping--redirection)
4. [Data Manipulation](#data-manipulation)
5. [Processes & Jobs](#processes--jobs)
6. [Users](#users)
7. [Permissions](#permissions)
8. [Chaining Commands](#chaining-commands)
9. [Terminal Multiplexing](#terminal-multiplexing)
10. [PATH](#path)
11. [Tricks & Shenanigans](#tricks--shenanigans)

---

## Man Pages

- Use `/` inside a man page to search for any string.
- Press `n` to jump to the **next** match, `N` for the **previous** match.
- Use `?` to search **backwards**.

---

## Globbing

Globbing is how the shell expands patterns to match file paths before running a command.

```bash
echo LOOK: /he*/j*
```

| Pattern | What it matches |
|---------|----------------|
| `*` | Everything — a wildcard for any number of characters |
| `?` | A single character wildcard |
| `[abc]` | Any **one** character from the set (e.g., `a`, `b`, or `c`) |
| `[!ab]` | Any character **except** those in the set |

**Examples:**
```bash
echo file_?         # matches file_1, file_a, etc.
cd ?????            # cd into any directory with exactly 5 chars
ls file_[pwn]       # matches file_p, file_w, or file_n
ls file_[!ab]       # excludes file_a and file_b
```

> 💡 **Tab completion tip:** Press `Tab` after typing a few letters to auto-complete file names or commands. If multiple matches exist, press `Tab` twice to list them all.

---

## Piping & Redirection

### Output Redirection

```bash
cmd > file        # Save stdout to a file (overwrites)
cmd >> file       # Append stdout to a file
cmd 2>/dev/null   # Discard stderr
cmd > out.log 2> err.log  # Redirect stdout and stderr separately
```

### File Descriptors (FDs)

| FD | Name | Description |
|----|------|-------------|
| 0 | stdin | Standard Input |
| 1 | stdout | Standard Output |
| 2 | stderr | Standard Error |

> `echo hi > file1` is implicitly `echo hi 1> file1` — the `1` is optional for stdout.

### Input Redirection

```bash
rev < message     # Feed a file as input to a command
```

### Piping stderr

Normally you can't pipe stderr directly. Use `>&` to redirect FD 2 into FD 1 first:

```bash
/challenge/run 2>&1 | grep flag
```

### Useful Pipe Commands

| Command | Purpose |
|---------|---------|
| `grep pattern` | Filter lines matching a pattern |
| `grep -v pattern` | Invert match — exclude lines with the pattern |
| `tee file1 file2` | Write output to screen **and** one or more files |
| `sed "s/old/new/g"` | Replace all occurrences of `old` with `new` |

**sed breakdown:**

```
s/oldword/newword/g
│  │        │      └─ global: replace ALL occurrences
│  │        └──────── replacement string
│  └───────────────── pattern to find
└──────────────────── substitute command
```

### Process Substitution

```bash
<(command)
```

Bash runs the command and gives you a path (like `/dev/fd/63`) that you can pass as an argument to other commands:

```bash
diff <(cat file1) <(cat file2)
echo <(echo pwn) <(echo college)   # prints /dev/fd/63 /dev/fd/64
```

### Storing Command Output

```bash
FLAG=$(cat /flag)
echo "$FLAG"
```

### Reading User Input

```bash
read -p "INPUT: " MY_VARIABLE
echo "You entered: $MY_VARIABLE"
```

---

## Data Manipulation

| Command | Purpose |
|---------|---------|
| `awk` | Complex text processing |
| `cut -d " " -f 2` | Extract specific columns (field 2, delimiter space) |
| `less` | Interactive file reader |
| `more` | Read a file page by page |
| `paste` | Combine files side by side |
| `sed "s/old/new/g"` | Complex text manipulation / find-replace |
| `sort` | Sort lines of text |
| `tail` | Show last N lines (inverse of `head`) |
| `uniq` | Filter duplicate lines in a stream |
| `tr A B` | Translate character A to character B |

**sort flags:**

```bash
sort -r   # Reverse order (Z → A)
sort -n   # Numeric sort
sort -u   # Unique lines only (removes duplicates)
sort -R   # Random order
```

**tr example:**

```bash
echo PWM | tr M N          # → PWN
echo PWM.COLLAGE | tr MA NE  # → PWN.COLLEGE
```

---

## Processes & Jobs

### Listing Processes

```bash
ps -ef        # Standard syntax: every process, full format
ps aux        # BSD syntax: all processes, user-readable
```

> **BSD style** comes from Berkeley Software Distribution Unix. For `ps`, both syntaxes are supported and parsed differently.

### Managing Processes

| Command | Action |
|---------|--------|
| `kill PID` | Terminate a process by its PID |
| `Ctrl + Z` | Suspend (pause) the current process |
| `fg` | Resume a suspended process in the **foreground** |
| `bg` | Resume a suspended process in the **background** |

---

## Users

- Linux has many users — some are real accounts, others are **service accounts** that support installed software.
- Passwords used to live in `/etc/passwd` (world-readable ⚠️) but are now stored as **hashes** in `/etc/shadow`.
- Hashes can be cracked with tools like **john-the-ripper**.

### Switching Users

```bash
su            # Switch to root (prompts for root's password)
su username   # Switch to a specific user
```

`su` is a **setuid binary**:
```bash
ls -l /usr/bin/su
# -rwsr-xr-x 1 root root ...
```

### sudo

```bash
sudo command  # Run a command as root
```

Unlike `su`, `sudo` checks `/etc/sudoers` to determine authorization — no root password needed.

---

## Permissions

### Reading Permission Strings

```
drwxr-xr-x 1 hacker hacker 0 Mar 22 Desktop
│└─┘└─┘└─┘
│  │  │  └── Others: r-x (read, execute)
│  │  └───── Group:  r-x
│  └──────── Owner:  rwx (read, write, execute)
└─────────── File type: d=directory, -=file, l=symlink, c=char device
```

### Changing Permissions

```bash
chmod u+r file       # Add read for owner
chmod g+wx file      # Add write+execute for group
chmod a-rwx file     # Remove all permissions for everyone
chmod a+x file       # Make executable for all
chmod u=rw,g=r file  # Set exact permissions (overwrite)
```

### Changing Ownership

```bash
chown user file      # Change file owner
chgrp group file     # Change file group
```

### Special Permission Bits

| Bit | Octal | Meaning |
|-----|-------|---------|
| SUID | 4 | Run as the **owner** regardless of who executes it |
| SGID | 2 | Run as the **group** |
| Sticky | 1 | Only the owner can delete/rename files in a directory |

```bash
chmod u+s program      # Set SUID
chmod 2755 script      # SGID example
chmod 4755 program     # SUID example
```

A program with SUID set shows an `s` in its permissions:
```
-rwsr-xr-x 1 root root ... /usr/bin/sudo
```

### Checking Exit Codes

```bash
echo $?   # 0 = success, non-zero = error
```

---

## Chaining Commands

| Operator | Behavior |
|----------|----------|
| `;` | Run both commands regardless of exit codes |
| `&&` | Run second command **only if** first succeeds (exit code 0) |
| `\|\|` | Run second command **only if** first fails (non-zero exit code) |

```bash
touch file; cat file        # Always runs both
cmd1 && cmd2                # cmd2 only if cmd1 succeeded
cmd1 || cmd2                # cmd2 only if cmd1 failed
```

### Shell Scripts (Shebangs)

```bash
#!/bin/bash           # Bash script
#!/usr/bin/python3    # Python script
#!/bin/sh             # POSIX shell script
```

When you run `./script.sh`, Linux reads the first line, extracts the interpreter path, and calls it with your script as the argument.

---

## Terminal Multiplexing

### screen

```bash
screen           # Start a new session
Ctrl-A d         # Detach (keep running in background)
screen -r        # Reattach to a session
```

All `screen` shortcuts begin with `Ctrl-A`.

### tmux

A modern alternative to `screen`. Uses `Ctrl-B` as its prefix.

```bash
tmux             # Start a new session
tmux ls          # List sessions
tmux attach      # Reattach to last session
```

| Shortcut | Action |
|----------|--------|
| `Ctrl-B c` | Create a new window |
| `Ctrl-B n` | Next window |
| `Ctrl-B p` | Previous window |
| `Ctrl-B 0–9` | Jump to window by number |
| `Ctrl-B w` | Visual window picker |

---

## PATH

The `PATH` variable tells your shell where to look for executables.

```bash
PATH=/home/hacker/scripts    # Override PATH
echo $PATH                   # View current PATH
```

📖 More info: https://www.linfo.org/path_env_var.html

---

## Tricks & Shenanigans

### .bashrc Backdoor / Customization

When your shell starts, it automatically runs `~/.bashrc`. You can use this to:
- Set environment variables
- Define aliases and functions
- Run startup scripts

```bash
# Example .bashrc additions
export PATH="$HOME/scripts:$PATH"
alias ll='ls -la'
```

> ⚠️ **Security note:** Anyone with write access to a directory can move or delete files in it — even files they don't own. Be careful with world-writable directories.

---

## Resources

- [Linux man pages online](https://linux.die.net/man/)
- [PATH variable explained](https://www.linfo.org/path_env_var.html)
- [pwn.college](https://pwn.college) — where most of these concepts were practiced

---

*These are personal notes compiled while learning Linux. Corrections and contributions welcome!*
