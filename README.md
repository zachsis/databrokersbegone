# DataBrokersBegone

A [Claude Code](https://claude.ai/code) skill that helps you find data brokers selling your personal information and generates CCPA/OCPA opt-out requests to get your data deleted.

## What It Does

1. **Collects your information** interactively (name, addresses, phone, email, family members to exclude)
2. **Searches the web** using parallel agents to find which data brokers have your data
3. **Asks you to verify** which results are actually you (handles common name collisions, family members, etc.)
4. **Generates personalized opt-out requests** — pre-filled emails and a checklist of web forms to submit

Covers 50+ data brokers across people-search sites, commercial data brokers, B2B platforms, and telecom databases.

## Installation

### Option 1: Symlink (recommended — easiest to update)

Clone the repo anywhere you like, then symlink it into your Claude Code skills directory:

```bash
git clone https://github.com/zachsis/databrokersbegone.git ~/repos/databrokersbegone
ln -s ~/repos/databrokersbegone ~/.claude/skills/databrokersbegone
```

To update later, just pull:

```bash
cd ~/repos/databrokersbegone && git pull
```

The symlink means Claude Code picks up changes immediately — no copying needed.

### Option 2: Clone directly into skills

```bash
git clone https://github.com/zachsis/databrokersbegone.git ~/.claude/skills/databrokersbegone
```

To update:

```bash
cd ~/.claude/skills/databrokersbegone && git pull
```

### Option 3: Project-level (for a specific repo)

```bash
cd your-project
mkdir -p .claude/skills
git clone https://github.com/zachsis/databrokersbegone.git .claude/skills/databrokersbegone
```

## Usage

In Claude Code, run:

```
/databrokersbegone
```

Or with a name:

```
/databrokersbegone Jane Smith
```

The skill will walk you through the process interactively.

## What Gets Generated

After running, you'll have these files in your current directory:

| File | Purpose |
|------|---------|
| `databrokersbegone_profile.json` | Your verified personal profile (used for searches) |
| `databrokersbegone_checklist.md` | Phased action plan with all opt-out steps |
| `databrokersbegone_emails/*.txt` | Pre-filled opt-out emails ready to send |
| `databrokersbegone_report.md` | Summary report with follow-up dates |

## Requirements

- [Claude Code](https://claude.ai/code) CLI, desktop app, or IDE extension
- WebSearch permission enabled (Claude Code will prompt you)
- Internet access for data broker searches

## Legal Coverage

The generated opt-out requests cite:

- **CCPA** (California Consumer Privacy Act) — applies to all CA-registered data brokers regardless of where you live
- **OCPA** (Oregon Consumer Privacy Act) — for Oregon residents
- Additional state privacy laws are referenced in `brokers.md` for CO, CT, VA, TX, MT, UT

## Tips

- **Run quarterly.** Data brokers re-add your data from new public records. Set a reminder to re-run every 3 months.
- **Start with the California DROP portal** (privacy.ca.gov/drop/) — one request covers all CA-registered brokers.
- **Be persistent with difficult brokers** (MyLife, Radaris). File complaints with your state AG if they don't comply within 45 days.
- **Use all name variations** when searching — brokers may file you under different spellings.

## License

MIT
