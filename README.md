# linux-learning-journal

Welcome to my Linux Learning Journal! This repository is a daily log of my journey mastering Linux. Instead of just taking static notes, I treat this as a public learning log where I document concepts, commands, and practical scripts day by day.

## 📂 Repository Structure

All notes are organized chronologically by year and day:

```text
linux-learning-journal/
└── 2026/
    └── Days/
        ├── linux-learning-log-2026-10-18.md
        ├── linux-learning-log-2026-10-19.md
        ├── ...
        ├── ETL-Script-explanation.md
        └── linux-learning-log-2026-11-5.md

## 🗺️ Learning Progress & Milestones

Based on my daily logs, here are the major milestones I've completed so far:

- **Day 8: Service & Package Management** *(Logged Oct 18 - Oct 25)*
  - Managing system services (`systemctl`)
  - Package management (`apt`, `dnf`)
- **Day 10: SSH & Remote Server Management** *(Logged Nov 3)*
  - Secure Shell (`ssh`), file transfers (`scp`), and remote administration.
- **Day 12: Shell/Bash Scripting** *(Logged Nov 5)*
  - Core scripting concepts: variables, loops, conditionals, and functions.
  - **Special File:** [`ETL-Script-explanation.md`](linux-learning-journal/2026/Days/ETL-Script-explanation.md) — A deep dive and line-by-line explanation of a real-world ETL script that extracts data from `/etc/passwd` and loads it into a MySQL Docker container.

## 📝 What's Inside the Daily Logs?

Each `linux-learning-log-YYYY-MM-DD.md` file contains:
- **Concepts Learned:** The theory behind the day's topic.
- **Command References:** Cheat sheets for commands used that day.
- **Scripts & Snippets:** Practical Bash scripts written during the lesson.
- **Challenges:** Any errors I ran into and how I solved them.

## 🛠️ How to Browse

You can browse the files directly on GitHub, or clone the repo to use `grep` to search for specific commands or topics locally:

```bash
git clone https://github.com/<your-username>/linux-learning-journal.git
cd linux-learning-journal/2026/Days
grep -rn "systemctl" .
```

## 🎯 Goal

The goal of this repository is to build a comprehensive, searchable knowledge base for Linux administration and Bash scripting, while proving consistency through daily commits.

