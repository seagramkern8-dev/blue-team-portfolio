---
tags: [linux, cheatsheet, reference, soc]
updated: 2026-11-06
---

# Linux Cheat Sheet (Blue Team / SOC)

Commands I use for the Roppers Linux course and for basic defensive work on Ubuntu. Placeholders look like `<file>`, `<user>`, `<host>`.

> **Don't memorize, look it up.** `man <cmd>` · `tldr <cmd>` · `<cmd> --help` · explainshell.com

---

## Getting help
| Command | What it does |
|---|---|
| `man <cmd>` | Full manual. `/word` searches, `n` jumps to the next match, `q` quits |
| `man 2 <name>` | Manual from a specific section (2 = system calls, 1 = commands, 5 = config files) |
| `man -k <word>` | Search all manuals by keyword |
| `tldr <cmd>` | Short, example-first help (`sudo apt install tldr`, then `tldr --update`) |
| `<cmd> --help` | Quick options list |
| `which <cmd>` | Where a program lives (`which ssh` → `/usr/bin/ssh`) |
| `type <cmd>` | Program, alias, or shell built-in? |
| `whereis <cmd>` | Program + manual locations |

## Moving around
| Command | What it does |
|---|---|
| `pwd` | Where am I? |
| `ls -la` | List everything, including hidden (`.`) files, with details |
| `cd <dir>` / `cd` / `cd ~` | Go to a folder / go home |
| `cd ..` / `cd -` | Up one folder / back to the previous folder |
| `tree -L 2` | Folder tree, 2 levels deep (`sudo apt install tree`) |

## Files and folders
| Command | What it does |
|---|---|
| `touch <file>` | Create an empty file (or update its timestamp) |
| `mkdir -p a/b/c` | Make folders, including parents |
| `cp <src> <dst>` / `cp -r <dir> <dst>` | Copy a file / copy a folder |
| `mv <src> <dst>` | Move or rename |
| `rm -i <file>` | Delete, asking first. **No Trash in Linux** |
| `rmdir <dir>` | Remove an empty folder |
| `"file name.txt"` | Quote names with spaces or `( )`, or press **Tab** to auto-complete |

## Reading files
| Command | What it does |
|---|---|
| `cat <file>` | Print the whole file |
| `less <file>` | Scroll a big file (`/word` search, `q` quit) |
| `head -n 20 <file>` / `tail -n 20 <file>` | First / last 20 lines |
| `tail -f <log>` | Watch a log live (Ctrl+C to stop) |
| `wc -l <file>` | Count lines |
| `file <file>` | What kind of file is it *really*? (don't trust the extension) |
| `strings <file>` | Readable text inside a binary (`sudo apt install binutils`) |

## Editing with nano
`nano <file>` → **Ctrl+O** save (then Return) · **Ctrl+X** exit · **Ctrl+W** search · **Ctrl+K** cut line · **Ctrl+U** paste

## Searching
| Command | What it does |
|---|---|
| `find ~ -name "*.txt"` | Find by name in your home folder |
| `sudo find / -name "<name>" 2>/dev/null` | Search the whole system, hiding permission errors |
| `find <dir> -mtime -1 -type f` | Files changed in the last 24 hours |
| `find <dir> -size +100M` | Files over 100 MB |
| `grep -i "word" <file>` | Find lines containing a word (case-insensitive) |
| `grep -rn "word" <dir>` | Search every file in a folder, with line numbers |
| `grep -v "word"` | Lines that do **not** match |
| `grep -c "word" <file>` | Count matching lines |
| `grep -E "this|that"` | Match either pattern (regex) |

## Pipes and redirection
| Symbol | What it does |
|---|---|
| `a \| b` | Send output of `a` into `b` |
| `> file` | Write output to a file (**overwrites**) |
| `>> file` | Add output to the end of a file |
| `2>/dev/null` | Throw away error messages |
| `2>&1` | Send errors to the same place as normal output |
| `\| tee file` | Show output **and** save it |
| `\| sort \| uniq -c \| sort -nr` | Count repeats, most common first (**the SOC classic**) |
| `\| awk '{print $1}'` | Print just column 1 |
| `\| cut -d: -f1` | Split on `:` and print field 1 |

## Permissions
`ls -l` shows `-rwxr-x---` → type · **owner** (rwx) · **group** (r-x) · **others** (---)

| Command | What it does |
|---|---|
| `chmod 600 <file>` | Owner read/write only (private keys, secrets) |
| `chmod 644 <file>` | Owner write, everyone read (normal files) |
| `chmod 700 <dir>` / `755` | Private folder / normal program or folder |
| `chmod u+x <script>` | Make a script executable |
| `sudo chown <user>:<group> <file>` | Change owner |
| `umask` | Default permissions for new files |
| `sudo <cmd>` / `sudo !!` | Run as admin / re-run the last command as admin |

## Users and groups
| Command | What it does |
|---|---|
| `whoami` / `id` | Who am I, and which groups am I in? |
| `who` / `w` | Who's logged in right now (and what they're doing) |
| `last -i \| head` | Recent logins with IP addresses |
| `sudo adduser <user>` / `sudo deluser <user>` | Add / remove a user |
| `sudo usermod -aG <group> <user>` | Add a user to a group |
| `passwd` | Change your password |
| `/etc/passwd` · `/etc/group` · `/etc/shadow` | Accounts · groups · password hashes (root only) |

## Processes
| Command | What it does |
|---|---|
| `ps aux` | Every running process |
| `ps aux --sort=-%cpu \| head` | Top CPU users |
| `pgrep -a <name>` | Find a process by name |
| `top` / `htop` | Live process view (`q` to quit) |
| `kill <pid>` / `kill -9 <pid>` | Ask a process to stop / force-stop (last resort) |
| **Ctrl+C** / **Ctrl+Z** | Stop the current command / pause it (`fg` resumes) |

## Services and logs
| Command | What it does |
|---|---|
| `systemctl status <svc>` | Is a service running? Recent log lines |
| `systemctl is-active <svc>` | Just `active` / `inactive` |
| `sudo systemctl start\|stop\|restart <svc>` | Control a service |
| `sudo systemctl enable\|disable <svc>` | Start at boot or not |
| `systemctl list-timers` | Scheduled jobs (check for persistence) |
| `journalctl -u ssh --since "1 hour ago"` | One service's logs |
| `journalctl -f` | Watch the system log live |
| `/var/log/auth.log` | Logins, `sudo`, SSH, new users |
| `/var/log/syslog` | General system messages |

## Networking
| Command | What it does |
|---|---|
| `ip a` / `hostname -I` | My IP addresses |
| `ip r` | Routes (default gateway) |
| `sudo ss -tulpn` | What's listening, on which port, run by which program |
| `ping -c 4 <host>` | Can I reach it? |
| `curl -I <url>` | Just the HTTP headers |
| `dig <domain>` / `host <domain>` | DNS lookup (`sudo apt install dnsutils`) |
| `resolvectl status` | Which DNS servers I'm using |
| `cat /etc/hosts` | Local name overrides (attackers sometimes edit this) |

## Packages (Ubuntu / Debian)
| Command | What it does |
|---|---|
| `sudo apt update` | Refresh the package lists |
| `sudo apt upgrade` | Install updates |
| `apt show <pkg>` | **What is this package?** Check before installing |
| `apt search <word>` | Find packages |
| `sudo apt install <pkg>` / `sudo apt install ./file.deb` | Install from repo / from a `.deb` |
| `sudo apt remove <pkg>` / `sudo apt autoremove` | Uninstall / clean up leftovers |
| `apt list --installed` / `dpkg -l` | What's installed |
| `dpkg -L <pkg>` | Every file a package installed |
| `dpkg -I <file.deb>` | Inspect a `.deb` before installing |

## Disk and system
| Command | What it does |
|---|---|
| `df -h` | Free disk space |
| `du -sh *` | Size of each item in this folder |
| `free -h` | Memory and swap |
| `lsblk` | Disks and partitions |
| `uname -a` / `cat /etc/os-release` | Kernel / OS version |
| `uptime` | How long it's been running, and the load |

## Hashing and verifying
| Command | What it does |
|---|---|
| `sha256sum <file>` | File fingerprint. Compare to the publisher's or search it on VirusTotal |
| `echo "<hash>  <file>" \| sha256sum -c` | Check a file against a known hash (two spaces) |
| `md5sum <file>` | Older hash. Fine for matching duplicates, not for security |
| `base64 -d <<< "<text>"` | Decode Base64 (common in obfuscated commands) |

## Archives
| Command | What it does |
|---|---|
| `tar -czf out.tar.gz <dir>` | Create a compressed archive |
| `tar -tzf out.tar.gz` | List what's inside **before** extracting |
| `tar -xzf out.tar.gz` | Extract |
| `unzip -l file.zip` / `unzip file.zip` | List / extract a zip |

## SSH and file transfer
| Command | What it does |
|---|---|
| `ssh <user>@<host>` | Log in to another machine (`exit` to leave) |
| `ssh-copy-id <user>@<host>` | Install your public key on a server |
| `scp <file> <user>@<host>:~` | Copy a file **to** a server |
| `scp <user>@<host>:~/<file> .` | Copy a file **from** a server |
| `ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub` | A server's fingerprint (compare on first connect) |
| `~/.ssh/known_hosts` | Servers you've trusted |

## Shell shortcuts
| Keys | What it does |
|---|---|
| **Tab** | Auto-complete commands and file names |
| **↑ / ↓** | Previous commands |
| **Ctrl+R** | Search command history |
| **Ctrl+C** | Cancel. Also escapes `quote>`, `dquote>`, `...>` |
| **Ctrl+D** | Exit a shell or program (logs out of SSH) |
| **Ctrl+L** / `clear` | Clear the screen |
| **Ctrl+A** / **Ctrl+E** | Jump to the start / end of the line |
| `history` / `!!` | Show history / repeat the last command |

---

## SOC quick checks
| Question | Command |
|---|---|
| Who's been trying to guess passwords? | `sudo grep "Failed password" /var/log/auth.log \| awk '{print $(NF-3)}' \| sort \| uniq -c \| sort -nr` |
| Successful logins | `sudo grep "Accepted" /var/log/auth.log` |
| Who used `sudo`, and for what? | `sudo grep "COMMAND" /var/log/auth.log` |
| New accounts created | `sudo grep -E "useradd\|adduser" /var/log/auth.log` |
| Recent logins with IPs | `last -i \| head` |
| What's listening on the network? | `sudo ss -tulpn` |
| What's using the CPU? | `ps aux --sort=-%cpu \| head` |
| Config files changed in the last day | `sudo find /etc -mtime -1 -type f` |
| Persistence: scheduled jobs | `crontab -l` · `ls /etc/cron.*` · `systemctl list-timers` |
| Programs that run as root (SUID) | `sudo find / -perm -4000 -type f 2>/dev/null` |
| Suspicious file triage | `file <f>` → `sha256sum <f>` → search the hash on VirusTotal → `strings <f> \| less` |

---

## Mac (zsh) vs Linux (bash)
| Task | Mac | Linux |
|---|---|---|
| Home folder | `/Users/<user>` | `/home/<user>` |
| Hash a file | `shasum -a 256 <file>` | `sha256sum <file>` |
| My IP | `ipconfig getifaddr en0` | `ip a` / `hostname -I` |
| Listening ports | `lsof -i -P \| grep LISTEN` | `sudo ss -tulpn` |
| Services | `launchctl` | `systemctl` |
| Install software | App Store / Homebrew (`brew`) | `apt` |
| Edit in place with sed | `sed -i '' 's/a/b/' f` | `sed -i 's/a/b/' f` |
| Default shell | zsh | bash |

## ⚠️ Danger zone: stop and double-check
- `rm -rf <path>`: permanent, no undo. One stray space can wipe a home folder.
- `dd`, `mkfs`: overwrite whole disks.
- `chmod -R 777`: gives everyone full access. Almost never the right fix.
- `curl <url> | bash`: runs internet code unseen. Read it first.
- Pasting text you haven't read into a terminal. Hidden characters can run commands.
