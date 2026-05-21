# DataBrokersBegone

A [Claude Code](https://claude.ai/code) skill that helps you find data brokers selling your personal information and generates CCPA/OCPA opt-out requests to get your data deleted.

## What It Does

1. **Collects your information** interactively through a step-by-step questionnaire (name, DOB, addresses, phone, email, social media, employers, and more)
2. **Searches the web** using parallel agents to find which data brokers have your data
3. **Asks you to verify** which results are actually you (handles common name collisions, family members, etc.)
4. **Generates personalized opt-out requests** — pre-filled emails and a checklist of web forms to submit
5. **Sends opt-out emails directly** from your Gmail account (optional, requires Gmail MCP)

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

## Requirements

### Required

- [Claude Code](https://claude.ai/code) CLI, desktop app, or IDE extension
- **WebSearch** permission enabled (Claude Code will prompt you on first use)
- Internet access for data broker searches

### Optional: Gmail MCP (for sending opt-out emails directly)

If you want Claude to send the opt-out emails directly from your Gmail account instead of just generating text files, you need the **Gmail MCP server** connected.

**Setup:**

1. In Claude Code, run `/mcp` and select **"claude.ai Gmail"**
2. Complete the OAuth flow in your browser to authorize access to your Gmail account
3. When running the skill, Claude will ask whether you want to send emails directly or just generate files

**What it does:** Sends the pre-filled CCPA/OCPA opt-out emails to each data broker's privacy address directly from your Gmail. You'll be asked to confirm before any email is sent.

**Without Gmail MCP:** The skill still works — it generates `.txt` files in `./databrokersbegone_emails/` that you can copy-paste into your email client manually.

## Usage

In Claude Code, run:

```
/databrokersbegone
```

Or with a name:

```
/databrokersbegone Jane Smith
```

The skill walks you through a step-by-step questionnaire:

1. Identity basics (name, aliases, DOB)
2. Addresses (current + previous)
3. Contact info (emails, phones — current and old)
4. Online presence (social media, usernames, domains, known breaches)
5. Professional & assets (employers, licenses, education, vehicles, property)
6. Family disambiguation (relatives with similar names to exclude)
7. Profile confirmation before searching

## What Gets Generated

After running, you'll have these files in your current directory:

| File | Purpose |
|------|---------|
| `databrokersbegone_profile.json` | Your verified personal profile (used for searches) |
| `databrokersbegone_checklist.md` | Phased action plan with all opt-out steps |
| `databrokersbegone_emails/*.txt` | Pre-filled opt-out emails ready to send |
| `databrokersbegone_report.md` | Summary report with follow-up dates |

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
