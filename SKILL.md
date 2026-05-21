---
name: databrokersbegone
description: Find data brokers selling your personal information and generate CCPA/OCPA opt-out requests. Use when the user wants to remove their personal data from data brokers, people-search sites, or exercise privacy rights.
when_to_use: "remove my data from data brokers, CCPA opt-out, delete my personal information, people search removal, opt out of data selling, privacy cleanup, data broker removal"
disable-model-invocation: true
argument-hint: "[optional: person's name]"
allowed-tools:
  - Agent
  - WebSearch
  - WebFetch
  - AskUserQuestion
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - TaskCreate
  - TaskUpdate
  - TaskList
  - mcp__claude_ai_Gmail__authenticate
  - mcp__claude_ai_Gmail__complete_authentication
  - mcp__claude_ai_Gmail__*
---

<command-name>databrokersbegone</command-name>

# DataBrokersBegone — Automated Privacy Opt-Out Workflow

You are running the DataBrokersBegone skill. This is a multi-phase interactive workflow that helps a person find data brokers selling their personal information and generates CCPA/OCPA opt-out requests.

**IMPORTANT:** This workflow is interactive. You MUST pause and ask the user questions at key decision points. Do NOT rush through all phases without user input.

---

## Phase 1: Collect Personal Information

Gather the user's information through a series of AskUserQuestion calls. If a name was passed as `$ARGUMENTS`, greet them by name.

**CRITICAL WORKFLOW RULE:** You MUST collect information in sequential steps. After each AskUserQuestion, STOP and WAIT for the user's response before asking the next question. Do NOT call multiple AskUserQuestion tools in parallel. Do NOT list questions as plain text output — every question MUST be an AskUserQuestion call so the user gets an actual input prompt to type their answer.

### Step 1 — Identity basics
Use AskUserQuestion to ask:
> "Let's build your profile so I can find which data brokers have your information. To start, I need your **identity basics**:
>
> 1. Full legal name
> 2. Other names you go by or have been listed under (aliases, maiden names, nicknames, initials)
> 3. Date of birth (MM/DD/YYYY)"

WAIT for response. Record the answers.

### Step 2 — Addresses
Use AskUserQuestion to ask:
> "Now I need your **addresses** — data brokers keep historical records, so include everywhere you've lived:
>
> 1. Current address (street, city, state, ZIP)
> 2. All previous addresses (especially across state lines)"

WAIT for response. Record the answers.

### Step 3 — Contact info
Use AskUserQuestion to ask:
> "Next, your **contact information** — include old/unused ones too, brokers keep everything:
>
> 1. Email address(es) — all of them, current and old
> 2. Phone number(s) — current and old"

WAIT for response. Record the answers.

### Step 4 — Social media & online presence
Use AskUserQuestion to ask:
> "Now your **online presence** — these are all OSINT vectors that data brokers use:
>
> 1. Social media handles (Instagram, Twitter/X, Facebook, LinkedIn, TikTok, Reddit, YouTube, etc.)
> 2. Usernames you commonly use across sites (gaming tags, forum handles, etc.)
> 3. Any domains registered in your name (WHOIS records are a major source)
> 4. Any data breaches you know you've been part of (check haveibeenpwned.com if unsure)
>
> List whatever applies — skip what doesn't."

WAIT for response. Record the answers.

### Step 5 — Professional & asset info
Use AskUserQuestion to ask:
> "Almost done. Your **professional and asset information** — B2B brokers like ZoomInfo sell work profiles, and property/vehicle data brokers exist too:
>
> 1. Current and previous employers + job titles
> 2. Professional licenses or certifications (nursing, real estate, etc. — these are public record)
> 3. Education history (schools attended)
> 4. Vehicles owned (make, model, year)
> 5. Properties you own or have owned
>
> List whatever applies — skip what doesn't."

WAIT for response. Record the answers.

### Step 6 — Family disambiguation
Use AskUserQuestion to ask:
> "Last question: do any **family members share your name** or a very similar name? This is critical — data brokers constantly mix up family members (e.g., father/son with the same first and last name).
>
> For each person, tell me their full name, relationship to you, and where they live so I can exclude their results from yours."

WAIT for response. Record the answers.

### Step 7 — Confirm profile
After all steps are complete, compile everything into a JSON profile and save it to `./databrokersbegone_profile.json`. Then show the user a formatted summary of their complete profile and use AskUserQuestion to ask:
> "Here's your complete profile. Does everything look correct? Anything to add or change?"

WAIT for confirmation before moving to Phase 2.

### Profile JSON structure
```json
{
  "full_name": "Jane Marie Smith",
  "aliases": ["Jane M Smith", "J Smith", "Jane Doe"],
  "date_of_birth": "1989-03-15",
  "current_address": "123 Main St, Portland, OR 97201",
  "previous_addresses": ["456 Oak Ave, Seattle, WA 98101"],
  "emails": ["jane@example.com", "jsmith_old@yahoo.com"],
  "phones": ["5035551234"],
  "old_phones": ["2065559876"],
  "social_media": {
    "instagram": "@janesmith",
    "twitter": "@janemsmith",
    "linkedin": "linkedin.com/in/janemsmith",
    "facebook": "Jane Marie Smith",
    "tiktok": null,
    "reddit": "u/jmsmith89",
    "youtube": null
  },
  "usernames": ["jmsmith89", "janesmith_pdx"],
  "employers": [
    {"name": "Acme Corp", "title": "Software Engineer", "current": true},
    {"name": "Widgets Inc", "title": "Junior Dev", "current": false}
  ],
  "vehicles": ["2020 Toyota Camry"],
  "properties_owned": ["123 Main St, Portland, OR"],
  "professional_licenses": [],
  "education": ["University of Oregon, BS Computer Science 2011"],
  "domains_registered": ["janemsmith.com"],
  "known_breaches": ["LinkedIn 2012", "Adobe 2013"],
  "family_exclusions": [
    {"name": "John Smith", "relationship": "father", "notes": "Same last name, lives in Portland too"}
  ],
  "state_of_residence": "OR"
}
```

---

## Phase 2: Search for Data Broker Listings

Once the profile is confirmed, spawn **4 parallel search agents** to find the user's data across brokers.

### Agent 1: People-Search Sites
Search for the user's name variations on major people-search sites. Use web searches like:
- `"Full Name" site:spokeo.com`
- `"Full Name" city state`
- `site:whitepages.com "Full Name"`
- `site:truepeoplesearch.com "Full Name"`
- etc.

Target these sites: Whitepages, Spokeo, BeenVerified, Intelius, TruePeopleSearch, FastPeopleSearch, ThatsThem, PeopleFinder, USSearch, Radaris, MyLife, Nuwber, CyberBackgroundChecks, Addresses.com, InfoTracer, SearchPeopleFree, ZabaSearch, PublicRecordsNow, USPhoneBook, AnyWho, Peekyou, Cocofinder, Neighbor.Report

For each, report: whether a listing was found, what data appears exposed, and the likely profile URL.

### Agent 2: B2B & Professional Data Brokers
Search for the user on business/professional data brokers:
- `site:zoominfo.com "Full Name"`
- `"Full Name" site:rocketreach.co`
- `"Full Name" site:apollo.io`
- `"Full Name" employer`

### Agent 3: Commercial Data Brokers & Registries
Search for:
- California Data Broker Registry status at cppa.ca.gov
- California DROP portal availability at privacy.ca.gov/drop/
- Oregon Data Broker Registry at dfr.oregon.gov (if user is in OR)
- General search: `"Full Name" data broker people search`
- Phone number lookups: search each phone number

### Agent 4: Direct Identity Searches
Run direct web searches for the user's personal identifiers:
- `"Full Name"` (in quotes)
- `"email@address.com"`
- Each phone number
- `"Street Address" "Last Name"`
- Name + each city/state combination

Report ALL sites that appear in results (excluding the user's own social media).

**IMPORTANT for all agents:** Include the user's `family_exclusions` context so agents can flag potential false matches. Tell agents to note when a result might be for a family member rather than the user.

---

## Phase 3: Interactive Result Verification

This phase is CRITICAL. Do NOT skip it.

After all agents return, compile the raw results into a deduplicated list. Then present findings to the user **grouped by confidence level:**

### Confirmed (your data is definitely here)
Results where name + address or name + phone or name + email matched directly.

### Likely Yours (high confidence but needs verification)
Results matching your name in your cities/states but without a second identifier match.

### Possibly Not You (may be a family member or different person)
Results that match the name but could belong to someone in the `family_exclusions` list, or are in unfamiliar locations.

### Probably Not You
Results with only a last name match or clearly different person.

For each result, show:
- Broker name
- What data is exposed (name, address, phone, etc.)
- Profile URL if available
- Why you think it's them (or might not be)

Then ask the user:
> "I found listings on [N] sites. I've grouped them by confidence. Please review and tell me:
> 1. Which 'Likely Yours' results are actually you?
> 2. Which 'Possibly Not You' results are actually you?
> 3. Any results I marked as confirmed that are NOT you?
> 4. Are there any brokers you want to skip (maybe you want to keep a listing somewhere)?"

Update the profile with the user's verified broker list.

---

## Phase 4: Generate Opt-Out Requests

Once the user confirms which brokers to target, generate:

### 1. Execution Checklist
Create a phased action plan saved to `./databrokersbegone_checklist.md`:

**Phase A — Bulk Actions (do first):**
- California DROP portal (privacy.ca.gov/drop/) if available
- OptOutPrescreen.com (credit bureau marketing)
- DoNotCall.gov (telemarketing)
- DMAchoice.org (direct mail)

**Phase B — Web Form Opt-Outs:**
For each confirmed people-search broker, list the opt-out URL and brief instructions.

**Phase C — Email Opt-Outs:**
For brokers requiring email (MyLife, commercial data brokers, etc.), generate pre-filled emails.

**Phase D — Follow-Up (45 days later):**
Re-check all sites, file complaints for non-compliant brokers.

### 2. Pre-Filled Email Templates
For each broker that accepts email opt-outs, generate a personalized email saved to `./databrokersbegone_emails/[broker_name].txt`.

Each email MUST include:
- Legal citations (CCPA § 1798.105 and § 1798.120)
- If user is in Oregon, also cite OCPA (ORS 646A.570 et seq.)
- All name variations
- All addresses (current + previous)
- All phone numbers and emails
- Family member disclaimer if relevant (e.g., "My father has the same name — please only delete MY records")
- Request for written confirmation of deletion
- Reference to 45-day compliance deadline

Use the email template from `${CLAUDE_SKILL_DIR}/templates/email_template.txt` as the base.

### 3. Send Emails via Gmail (Optional)

After generating the email files, check if Gmail MCP tools are available (look for `mcp__claude_ai_Gmail__` tools). If they are:

1. Use AskUserQuestion to ask: "I've generated opt-out emails for [N] brokers. Would you like me to send them directly from your Gmail account, or would you prefer to send them manually from the generated files?"
2. WAIT for response.
3. If the user wants to send directly:
   - If Gmail is not yet authenticated, call `mcp__claude_ai_Gmail__authenticate` and guide them through the OAuth flow
   - Before sending, show the user a summary table of all emails: recipient, subject, broker name
   - Use AskUserQuestion to ask: "Here are the [N] emails I'm about to send. Confirm to send all, or tell me which ones to skip."
   - WAIT for confirmation.
   - Send each confirmed email using the Gmail MCP send tool
   - Report success/failure for each email sent
4. If Gmail MCP is not available, inform the user the emails have been saved to `./databrokersbegone_emails/` and they can copy-paste them manually.

### 4. Summary Report
Save a final report to `./databrokersbegone_report.md` with:
- Total brokers identified
- Confirmed vs skipped
- Emails generated
- Web forms to visit manually
- Follow-up dates (45 days from today)

---

## Broker Database Reference

See `${CLAUDE_SKILL_DIR}/brokers.md` for the complete broker database with opt-out URLs, methods, and categories.

---

## Important Notes

- **Privacy:** The user's personal information is sensitive. Do NOT log it anywhere except the local working files. Do NOT send it to any service other than the opt-out requests.
- **Accuracy:** Always verify opt-out URLs are current by fetching them before including in templates. URLs change frequently.
- **Disambiguation:** The interactive verification phase exists because data brokers mix up family members constantly. Never assume a result is the user without checking.
- **Legal scope:** CCPA applies to California-registered data brokers regardless of where the consumer lives. OCPA applies to Oregon residents. Many brokers honor requests from any US resident as a matter of policy. Include both statutes when applicable.
- **Re-addition:** Warn the user that many brokers will re-add their data within 3-6 months from new public records. Recommend running this skill quarterly.
