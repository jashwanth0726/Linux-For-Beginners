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

> **What are Man Pages?**
> Man pages (short for *manual pages*) are the built-in documentation system in Linux. Every command comes with a man page that explains what it does, what options it accepts, and how to use it — all right from your terminal, no internet needed. Think of it as the official instruction manual bundled with every tool on your system.
>
> **How to use:** Run `man <command>` to open the manual for any command. For example, `man ls` opens the manual for the `ls` command. Use the arrow keys to scroll, and press `q` to quit.

- Use `/` inside a man page to search for any string.
- Press `n` to jump to the **next** match, `N` for the **previous** match.
- Use `?` to search **backwards**.

---

## Globbing

> **What is Globbing?**
> Globbing is the shell's way of matching multiple file names using special wildcard characters. Instead of typing out every file name one by one, you write a pattern and the shell expands it into all the matching file names before running your command. It happens automatically — you never have to do anything special to trigger it.
>
> For example, `ls *.txt` doesn't literally look for a file called `*.txt`. The shell sees `*`, expands it to every file in the directory, and then `ls` only receives the actual matched names.

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

> **What is Piping & Redirection?**
> Every Linux command has three standard channels: input (where it reads from), output (where it writes results), and error (where it writes error messages). By default, input comes from your keyboard and output/errors go to your screen. Piping and redirection let you change this — you can save output to a file, feed a file as input to a command, or chain commands together so the output of one becomes the input of the next.
>
> The pipe symbol `|` connects two commands directly. Redirection symbols like `>` and `<` connect commands to files. This is one of the most powerful ideas in Linux — small, simple commands become incredibly powerful when you combine them.

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

> **What is Data Manipulation?**
> Linux gives you a powerful set of small tools designed to slice, sort, filter, and transform text. Since almost everything in Linux is text — logs, configs, command output — knowing how to manipulate text means you can answer almost any question about your system. These tools are designed to be chained together with pipes, so you can build complex data processing pipelines out of simple building blocks.
>
> For example, you might pipe a log file through `grep` to find errors, then through `cut` to extract just the timestamps, then through `sort` and `uniq` to count how many unique error times there are — all in a single command.

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

> **What are Processes & Jobs?**
> A process is any program that is currently running on your system. Every time you run a command, Linux creates a new process for it and assigns it a unique ID called a PID (Process ID). Your system is always running hundreds of processes at once — from your terminal to background system services.
>
> A job is a process that was started from your current shell session. Linux lets you manage jobs directly — you can pause them, send them to run in the background so you get your terminal back, or bring them back to the foreground. This is especially useful when you're running something slow and want to keep working at the same time.

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

> **What are Users?**
> Linux is a multi-user operating system — it was designed from the ground up to have many people using the same system at the same time, each with their own files and permissions. Every action on the system is tied to a user account. This is what allows Linux to keep one user's files private from another, and to prevent regular users from accidentally (or intentionally) breaking system-wide settings.
>
> Not all users are human. Many services like web servers or databases run as their own dedicated user accounts with limited permissions — this way, even if a service is compromised, the attacker can't access the rest of the system. You can see all user accounts on the system in the `/etc/passwd` file.

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

> **What are Permissions?**
> Linux uses a permission system to control who can read, write, or execute any given file. Every file has an owner (a user) and a group, and permissions are set separately for three audiences: the owner, the group, and everyone else. This is what stops one user from reading another user's private files, and what prevents regular users from modifying system files.
>
> Permissions are shown as a 9-character string like `rwxr-xr--`. Each group of three characters represents read (`r`), write (`w`), and execute (`x`) access for the owner, group, and others respectively. A `-` means that permission is not granted.

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

> **What is Chaining Commands?**
> In Linux, you can run multiple commands in sequence on a single line by chaining them together. The real power comes from *conditional* chaining — you can tell Linux to only run the next command if the previous one succeeded, or only if it failed. This lets you write simple but robust one-liners that handle success and failure gracefully.
>
> Commands communicate success or failure through an *exit code*. A zero exit code means success; anything else means failure. The chaining operators (`&&`, `||`, `;`) use this exit code to decide whether to run the next command. Shell scripts are built almost entirely on this idea.

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

> **What is Terminal Multiplexing?**
> A terminal multiplexer lets you run multiple terminal sessions inside a single window. You can split your screen into panes, switch between separate windows, and — most importantly — *detach* from a session and leave it running in the background. If your SSH connection drops, your work keeps going and you can reattach to it later, right where you left off.
>
> Think of it like virtual desktops but for your terminal. Instead of opening ten separate terminal windows, a multiplexer lets you manage everything from one place. The two most popular options are `screen` (older, simpler) and `tmux` (modern, more features).

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

> **What is PATH?**
> When you type a command like `ls` or `python3`, your shell doesn't search your entire hard drive for it — that would be way too slow. Instead, it only looks in a specific list of directories defined in an environment variable called `PATH`. The shell checks each directory in the list, in order, until it finds an executable with that name.
>
> This is why you can run `python3` from anywhere without typing `/usr/bin/python3` every time. You can add your own directories to `PATH` so your custom scripts are just as easy to run as any built-in command.

```bash
PATH=/home/hacker/scripts    # Override PATH
echo $PATH                   # View current PATH
```

📖 More info: https://www.linfo.org/path_env_var.html

---

## Tricks & Shenanigans

> **What are these Tricks?**
> Linux has a handful of powerful but easy-to-overlook features that experienced users rely on every day. These aren't obscure hacks — they're built into the shell and designed to save you time. From startup scripts that automatically configure your environment to security gotchas that can catch you off guard, knowing these tricks separates a beginner from someone who's truly comfortable with the command line.
>
> The `.bashrc` file in particular is worth getting to know. It runs every time you open a new terminal, making it the perfect place to set up aliases, customize your prompt, and configure your environment exactly how you like it.

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

*These are personal notes compiled while learning Linux. Corrections and contributions welcome!*
