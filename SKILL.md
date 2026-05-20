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
---

<command-name>databrokersbegone</command-name>

# DataBrokersBegone — Automated Privacy Opt-Out Workflow

You are running the DataBrokersBegone skill. This is a multi-phase interactive workflow that helps a person find data brokers selling their personal information and generates CCPA/OCPA opt-out requests.

**IMPORTANT:** This workflow is interactive. You MUST pause and ask the user questions at key decision points. Do NOT rush through all phases without user input.

---

## Phase 1: Collect Personal Information

Use AskUserQuestion to gather the user's information. If a name was passed as `$ARGUMENTS`, greet them by name and start collecting details.

Collect the following in a conversational way (you can ask multiple related fields at once, but don't dump all questions in one wall of text):

**Required:**
- Full legal name
- Other names / aliases / maiden names / nicknames they may be listed under
- Current address (street, city, state, ZIP)

**Important:**
- Previous addresses (especially across state lines — brokers keep historical records)
- Email address(es)
- Phone number(s) (current and old)

**Optional but helpful:**
- Family members with similar names (to avoid false matches — this is critical for common names or family naming patterns like father/son with same first name)
- Approximate age or date of birth (helps disambiguate)
- Employer (to find B2B data brokers like ZoomInfo)

Store all collected information as a structured profile. Create a working file at `./databrokersbegone_profile.json` to persist the profile during the session.

Example profile structure:
```json
{
  "full_name": "Jane Marie Smith",
  "aliases": ["Jane M Smith", "J Smith", "Jane Doe"],
  "current_address": "123 Main St, Portland, OR 97201",
  "previous_addresses": ["456 Oak Ave, Seattle, WA 98101"],
  "emails": ["jane@example.com"],
  "phones": ["5035551234"],
  "old_phones": ["2065559876"],
  "employer": "Acme Corp",
  "age_or_dob": "35",
  "family_exclusions": [
    {"name": "John Smith", "relationship": "father", "notes": "Same last name, lives in Portland too"}
  ],
  "state_of_residence": "OR"
}
```

After collecting info, show the user a summary and ask them to confirm it's correct before proceeding.

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

### 3. Summary Report
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
