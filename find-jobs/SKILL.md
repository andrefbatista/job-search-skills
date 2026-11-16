---
name: find-jobs
description: "Run a job discovery sweep for [YOUR TARGET ROLE FAMILY]: find roles, read each full JD, confirm the posting is live on the employer's own board, and deliver a maximum of 5 survivors to the existing Notion job tracker. Also handles a single role the user names or pastes."
---

# Job discovery sweep

Find hands-on `[YOUR TARGET SENIORITY]` `[YOUR TARGET ROLE FAMILY]` roles, plus lower-tier roles that fit on everything but seniority, read each full JD, confirm the posting is live on the employer's own board, and deliver a maximum of 5 survivors to the existing Notion tracker.

Two things make this skill worth running: the full job description is read before anything reaches the tracker, and every link in the tracker has been traced back to the employer. Never write a role that fails either test into the tracker without the user's explicit say-so (see Single-role mode).

Killed roles are not thrown away. Every killed role goes on the killed list in the chat output (Step 8), with its reason and the best link found, so the user can second-guess the kill. Kill rules are judgment calls and the killed list is how the user checks them.

> Template note: everything in `[SQUARE BRACKETS]` is a placeholder. Fill them in once, then delete this note. The kill rules and band test are written for a design search. Swap the examples for your own field, but keep the structure.

## Which mode

- **Sweep mode:** the user asks to find jobs, run the sweep, or invokes `/find-jobs` with no specific role. Run Steps 1 to 8.
- **Single-role mode:** the user names one or more specific roles, pastes a link or a JD, or asks to add the role they are looking at (for example from the Chrome side panel) to the tracker. Run the section below. This applies on every surface: chat, Cowork, Claude Code and Claude in Chrome.

## Single-role mode

Use exactly the same rules as the sweep. Only discovery and the run-level bookkeeping are skipped.

1. **Get the employer text.** If the page the user is on is the employer's own page or ATS, read the full JD there. If it is LinkedIn, a job board or an aggregator, trace it to the employer (Step 2 ladder) and read the JD there. In the browser, the current page counts as read only when it is employer-side.
2. **Check the link** against the link rules, including the board index check and the LinkedIn-only exception. Never put an aggregator URL in `Link`.
3. **Dedup** against the tracker on Company plus Role (Step 6). If it is already there, say so with its status and stop.
4. **Run the kill rules and the band test** (Step 3). Read the `Killer` column on existing rows first.
5. **If a kill rule fires,** do not write the row yet. Tell the user the rule and the specific fact in one line and ask whether to add it anyway. If they say yes, write it with `Status` `Not applied` and the kill reason as the first line of `Notes`, prefixed `kill rule:`.
6. **Otherwise write the row** with the Step 6 field values: soft flags in `Notes`, the score in `Match`, one line in `Why`, and `Killer` left empty.
7. **Several roles at once:** handle each separately, one `notion-create-pages` call per role, spaced out. The 5-row cap does not apply in single-role mode, because the user chose each role.
8. **Output:** one line per role: `Company, Role, Match, one reason`, plus `(lower tier)` where it applies, the link source if not the employer's own careers page, and one line for any flag that could be a dealbreaker. Nothing else.

If the employer text cannot be read, say so and ask the user to paste the JD. Do not write the row from a header or an aggregator summary.

## Hard limits

- In sweep mode, write at most 5 new rows per run. If more than 5 survive, keep the 5 highest Match scores and move the rest to the killed list with the reason `over cap`.
- The cap applies to tracker rows only. The `needs paste` list and the killed list do not count toward it, and the killed list has no cap.
- Roles posted in the last 14 days only, judged by the employer's own date (see Evidence rule). In single-role mode an older date is a soft flag, since the user picked the role.
- Never invent a posting, a link, a salary, or a posting date.
- Never record an aggregator URL in the tracker's `Link` field. See the link rules below.
- Never write to the `Killer` column. It is the user's own record of their reasons.

## Evidence rule: only employer facts can kill

A kill must rest on a fact read on the employer's own page or ATS: its posting date, its location, its salary, its stated requirements. Facts that come only from job boards, aggregators, AI summaries on boards, or LinkedIn alerts never kill a role on their own. They become soft flags instead, and the role is judged on the employer's text.

This rule exists because three roles were wrongly killed in one run: one on a job board's posting date, one on a board's "entry level" label when the employer's own page said "Experienced" and "quality over years", and one on an assumed location. Boards re-list, re-date and re-summarise postings constantly and get seniority wrong.

If the employer's page cannot be read at all, the role is `needs paste`, not killed.

## Link rules, read before writing anything

These rules govern the tracker's `Link` field. The killed list uses the best link found, whatever its source, and labels it.

A successful fetch is not proof that a posting is live. WebFetch caches each URL for 15 minutes, and CDNs routinely ignore unknown query strings, so cache-busting parameters prove nothing either. Aggregators keep serving postings for weeks after the employer pulls them, and they rotate their own listing IDs, which breaks saved links.

So:

- **`Link` must be employer-side.** The company's own careers page, or its ATS: personio.de, greenhouse.io, lever.co, ashbyhq.com, factorialhr.de, teamtailor, workday, smartrecruiters, join.com, softgarden, traffit, recruitee, oraclecloud.
- **Never record** a link from any job board, aggregator or LinkedIn alert email in `Link`, however good it was for discovery. The one exception is the LinkedIn-only case below. Regional boards and aggregators never qualify.
- **LinkedIn-only exception.** When the employer posts the role only on LinkedIn and its only apply route is LinkedIn Easy Apply (no careers page or ATS listing exists after a real search), record the clean `https://www.linkedin.com/jobs/view/<id>` URL in `Link` and add the soft flag `LinkedIn-only`. If the employer has any careers page or ATS listing, that wins and the exception does not apply.
- **Confirm the role appears on the employer's board index**, not only at a deep link. A deep link that works while the index does not list the role means the posting was pulled. This check has caught dead postings that still rendered, five of them in a single run. Never skip it. For LinkedIn-only roles, the LinkedIn company jobs list is the index.
- **Trust the employer's posting date over a job board's.** Boards re-list old postings with a fresh date. One company showed a recent date on a board and a date months older on its own page.
- **If no employer-side link can be found, the role cannot go in the tracker** (unless the LinkedIn-only exception applies, or the user says to add it anyway in single-role mode). In sweep mode it goes on the killed list with the reason `no employer-side link` and the best link found. Two in-band roles failed for this reason in one run.
- **Recruiter proxies** such as Jobgether post on their own ATS. That counts as employer-side for the link, but their postings rotate fast, so confirm the role on the proxy's board index in the same run.
- **Companies with several ATS boards** (for example a separate local-language board and English board) must have every board checked before a role counts as pulled.

## First run only: tracker setup

Data source: `collection://[YOUR TRACKER DATA SOURCE ID]` (Job Search Tracker)

Fetch the data source first. If these are missing, add them with `notion-update-data-source`:

- `Match` (number, 0 to 100)
- `Why` (text)
- `Killer` (text)
- A `Passed` option on the existing `Status` select

Existing fields to use as they are: `Company` (title), `Role` (text), `Link` (url), `Status` (select), `Notes` (text).

Skip this whole section on re-runs.

## Inputs

Load before searching or scoring:

- Base CV: the Google Doc "[YOUR BASE CV DOC NAME]" in Drive folder `[YOUR DRIVE FOLDER ID]`. If it cannot be read, stop and say so rather than scoring from memory. Reading the file metadata returns a full content snippet, which is cheaper than opening the document.
- The constraints below. Do not ask the user to restate them.

## Constraints

Fill this section in once. It is the part of the skill that is most personal.

**Target titles** (search terms, not a filter): `[YOUR TARGET TITLES, 10 TO 25 VARIATIONS, INCLUDING LOCAL-LANGUAGE TITLES]`. Experience level: `[YOUR YEARS]`+ years.

The bare titles in that list are deliberately wide. They exist to widen discovery, not to decide fit: a lot of genuinely senior work is posted under a plain title, and a lot of junior work is posted under a senior one. The band test in Step 3 sorts each role into the main band or the lower tier, so let these titles into the funnel and let the body of the JD decide. Expect more volume per run as a result.

Titles that sit close to a kill rule need a careful read. Ask what the role is centred on, not what its title says.

**Two different title ladders exist, and the band covers both.**

- The individual contributor ladder: Senior, Staff, Principal. Going up means wider scope and more influence over the standard of the work, still hands-on, usually no direct reports.
- The management ladder: Senior, Lead, Director, Head of. Going up means more people and less making.

The main band is `[YOUR BAND, for example Senior, Staff, Principal and Lead]`. Anything above it, `[for example Director and up]`, is too high and gets killed, however well the brief reads, because it sits further from the hands-on work than the user wants to be.

**Lower tier.** The user is open to roles below the band: around 2 to 3 years asked, junior-adjacent titles in the local market, or no seniority stated, where the body fails the band test but the role is otherwise a fit. These are not killed. They carry the soft flag `lower tier` and score lower on scope (Step 5), so main-band roles naturally outrank them for the 5 slots. Roles explicitly titled or pitched as Junior, Trainee, Graduate, intern or working student stay killed.

**Making over managing.** `[DELETE OR EDIT: the original author prefers being a senior maker over managing, with leadership arriving as a consequence of seniority.]` A Staff or Principal role with no reports is a better fit than a Lead role that is mostly people management, even though both sit in the band. Weight this in the score rather than killing anything over it.

**Geography.** Replace the tiers with yours. The original structure:

- **Home city:** any arrangement (onsite, hybrid, remote) is never flagged and scores full marks.
- **Target relocation city** (`[TARGET CITY]`): any arrangement. The user will relocate. Never flagged.
- **Remote from the home country** and **remote from the home region** (for example EU-remote): always fine, scores the same as the target city.
- **Onsite or office-first elsewhere in the home country:** not a kill. Soft flag `office-first outside [HOME CITY] and [TARGET CITY]` naming the city and any remote allowance.
- **Onsite or hybrid elsewhere in the home region:** not a kill. The user will apply and push for remote. It gets the soft flag `abroad onsite, push for remote` naming the city, and scores lower on location.
- **Outside the home region:** a location kill unless the role is genuinely remote from the home region.

`[YOUR EMPLOYMENT TYPE RULE, for example: Only full-time employed positions.]`

## Step 1: discover

### LinkedIn, via the user's job alert emails

LinkedIn blocks direct fetching, and a scheduled run has no browser, so read LinkedIn through Gmail instead. This works in every run, attended or not.

The user runs several saved searches on a daily schedule, so the volume is high and the repetition is heavy. The same role appears across several searches and again the next day. Handle it like this:

1. **Search `from:jobalerts-noreply@linkedin.com newer_than:4d`**, not 14d. On a twice-weekly cadence that covers the gap with overlap, and daily alerts mean nothing is missed. A 14-day window would pull roughly 80 emails of mostly duplicates.
2. **Fetch threads with `messageFormat: PLAIN_TEXT`.** The full-content format carries the HTML body and each email runs to about 100KB.
3. **Build one deduplicated list of company plus role across every email before opening anything.** Keep each role's LinkedIn job ID from the `jobs/view/<id>` URL, so the killed list can carry a clean `linkedin.com/jobs/view/<id>` link.
4. **Then trace every lead to its employer page and read the JD there** (Step 2). Do not kill on the alert line, except for the header-only cases allowed in Step 2.

The subject line often carries the saved-search term and the posting date, for example `"visual designer": Acme Co - Creative Designer (m/f/d) posted on 10/24/26`. Alerts arrive in several languages, and some will point at countries that fail the geography rule.

**Never record a LinkedIn URL in the tracker**, except under the LinkedIn-only exception in the link rules. The alert carries only title, company and city, never the job description. LinkedIn-sourced leads are pulled at least as often as board-sourced ones.

### Sibling postings: sweep the whole employer once you find one role

Companies often post the same JD several times: at different seniority levels, in different locations, or as a repost. One company ran at least two postings with the same description at once, and another listed a plain version of a role next to a pulled senior version.

So whenever any lead turns up a company, list every posting in your target role family on that employer's own board, not only the one in the alert. Assess each separately. Dedupe by job ID or URL, never by title, because siblings can share a title and differ in location or pay. When siblings differ, say which one is which (location, pay) in `Notes`.

### Employer ATS APIs and careers pages, the most reliable source

Many corporate careers pages are marketing pages that list no roles, and some ATS board pages need JavaScript. Calling the ATS directly gets around both. The endpoint list lives in Step 2, rung 1.

**WebFetch does not filter JSON.** It returns the raw response and ignores the prompt, 100,000 characters per page. A full board with descriptions can run to millions of characters (boards of 400K, 1M and 2M characters turned up in one run). One such call can eat the budget for a dozen JD reads. So prefer calls that return titles only, then fetch single roles. WebFetch on careers pages also tends to return the whole page regardless of the prompt, so budget for that.

**Core watchlist: read the careers index every run.** For every core watchlist company, open its careers index (or its ATS title list) and read every title in your target role family on it, not only what alerts and searches surfaced. One company's index showed a live role that the board sweep had misjudged.

When a company's careers page comes back with no roles, do not conclude it has nothing. Find its ATS and query that. One company's role was invisible on its own site and sitting on its Greenhouse board. Another's own careers page omitted two roles that a parent-company careers search listed.

### Web search, for leads with no known employer link

Web search is a discovery tool in its own right, not only a fallback. For any lead where the employer board is unknown, search the company name plus the exact role title in quotes, and prefer results on the employer's domain or a known ATS domain. For Ashby roles, `"<company>" "<exact title>" ashbyhq.com/<org>` in extended mode returns the job page with its `datePosted`.

### Job boards to sweep

`[LIST YOUR BEST 5 TO 10 BOARDS IN ORDER OF YIELD, with notes on what each is good for. Examples of useful notes: which board redirects straight to employer postings, which one has the densest local market coverage, which one is remote-only, which one is curated.]`

Technical notes worth keeping, whatever boards you use:

- **Query by discipline as well as by title.** Local-language titles vary a lot, so request the full list by date rather than filtering by city or title where the board allows it.
- **A join.com job URL is employer-side**, so a hit there can go straight to the tracker once read.
- **JavaScript-gated boards return nothing to WebFetch.** Do not list them as checked. Go to the employers' career pages and their ATS APIs instead.
- **Personio career pages (`<org>.jobs.personio.de`) block WebFetch via robots.txt.** Find the posting through web search instead; Personio job URLs (`/job/<id>`) still count as employer-side links.
- **The Greenhouse EU board API (`boards-api.eu.greenhouse.io`) is blocked by robots.txt.** Fetch the HTML board `job-boards.eu.greenhouse.io/<org>` instead; it lists titles and links.
- **Some custom careers sites render nothing without JavaScript.** Treat those roles as `needs paste`, not as kills.

Every one of these is a lead source, never a tracker destination and never a source of kill facts. Trace each hit back to the employer before judging it.

**Boards go stale within a day of a run.** On a re-run within a few days, the boards will mostly repeat what was already assessed. Spend that run on LinkedIn alerts, sibling sweeps and the watchlist's careers pages instead, which is where the fresh in-house roles are.

### Browser, manual runs only

When the user is present and Claude in Chrome is available, their real Chrome carries their LinkedIn session and can open what WebFetch cannot. Use it for a specific lead, not for bulk scraping. A scheduled run executes in the cloud with no browser and no access to the user's computer, so never make a run depend on it.

The container's own network cannot reach job-board hosts either, so shell fetches and Playwright are useless here; use WebFetch and web search.

## Watchlist

Check the core list every run, reading each careers index as described above. Sample the wider list, rotating which ones you check, rather than attempting all of them.

### Core, every run

`[10 TO 40 COMPANIES YOU WOULD GENUINELY TAKE, GROUPED BY CATEGORY, for example: agencies and studios, tech and AI companies, remote-friendly companies]`

### Wider, sampled

`[LONGER LIST, GROUPED BY REGION OR CATEGORY. The original author's list had around 150 names across 8 groups.]`

Remember the geography rule: onsite elsewhere in the home region is a flag, not a kill; onsite outside the region is a kill unless remote from the region is offered.

## Step 2: read the full JD

Read the complete job description on the employer's own page or ATS for every candidate before any kill rule runs. Header text is not enough. The disqualifier, and often the redeeming detail, is usually in paragraph four.

Header-only kills are allowed only when the header itself is an employer-independent hard fact that no JD could overturn: a stated pay of a few thousand euros a month, a gig or task platform, a role outside your field. Mark them `header only` on the killed list. Seniority, location and date are never header-only kills.

Try the ladder below in order. Stop at the first rung that returns the full description text.

### Rung 1: the employer's ATS

- **Ashby: fetch the single job page.** `https://jobs.ashbyhq.com/<org>/<job_id>` returns the JD through WebFetch in its meta description, though the meta can be truncated; a web search snippet of the same page often carries more of the text and the `datePosted`. It does not show the posting date or, sometimes, the location, so:
  1. **Find job IDs** with a web search such as `site:jobs.ashbyhq.com/<org> <role keyword>` or `"<company>" "<exact title>" ashby`, or from aggregator listings that link to Ashby.
  2. **Read each JD** from its single job page and run the kill rules on that text.
  3. **Only for a role that survives,** confirm it is listed and get `publishedAt` and location from the board API `https://api.ashbyhq.com/posting-api/job-board/<org>?includeCompensation=true`. For a small board, one call is fine. For a large board, page with `offset` at most 3 times looking for the job ID; if it has not turned up, write the role with its Ashby job URL as `Link` and the soft flag `Ashby listing and date unconfirmed`.
  4. **Never sweep a large Ashby board API for discovery.** It returns every description at once.
  5. **Watch pay and benefits for location clues** when the page text has no location. US-style benefits (401(k), commuter benefits, in-office culture) and a higher USD band suggest the US version of a sibling posting; say so in `Notes`.
- **Greenhouse:** list first without descriptions, `https://boards-api.greenhouse.io/v1/boards/<org>/jobs`, which returns titles, locations, `updated_at` and job IDs in a compact list. Then fetch only the roles in your target family one at a time: `https://boards-api.greenhouse.io/v1/boards/<org>/jobs/<job_id>`, which returns that role's full description. Use `?content=true` on the list only for small boards. EU-hosted boards live at `job-boards.eu.greenhouse.io/<org>`; their API is robots-blocked, so read the HTML board.
- **Lever:** `https://api.lever.co/v0/postings/<org>?mode=json`. It sometimes errors. On error, go to rung 3.
- **Personio:** `https://<org>.jobs.personio.de/xml` returns all open roles with descriptions as XML. Untested, and the HTML career pages are blocked by robots.txt, so this may be too.
- **Recruitee:** `https://<org>.recruitee.com/api/offers/` returns JSON with descriptions. Untested.
- **SmartRecruiters:** `https://api.smartrecruiters.com/v1/companies/<org>/postings` lists roles, and each posting has its own detail URL with the full text. Untested.
- **Teamtailor:** `https://<org>.teamtailor.com/jobs.rss` returns an RSS feed with descriptions. Untested.
- **Large-company career portals** (for example a hotel group's or a bank's own search page): a keyword search lists roles with job links, and each job page usually carries the full JD as plain HTML. No posting date may be shown.
- **join.com:** job pages carry the full JD as plain HTML.
- **Custom careers pages** (many agencies): fetch the role page directly; agency pages usually carry the full JD and salary as plain HTML.

The first time an untested endpoint is used, note in the run output whether it returned full text. If it fails twice, stop using it.

### Rung 2: find the ATS when it is not known

Open the company careers page and look for the ATS domain in the apply links, for example greenhouse.io, ashbyhq.com, personio.de or recruitee.com. Then go back to rung 1 with that org name. When the careers page lists no roles, do not conclude there are none. The page is usually a marketing page in front of an ATS.

### Rung 3: search for the posting

Run a web search for the company name plus the exact role title in quotes. Prefer a result on the employer's domain or a known ATS domain. Open it and read the full text. Do this for every lead that came from a LinkedIn alert email, because those carry only title, company and city. If nothing employer-side exists after this search and the role is Easy Apply only, the LinkedIn-only exception applies.

### Rung 4: Chrome, manual runs only

If the user is present and Claude in Chrome is available, open the specific lead in their browser, which carries their LinkedIn session. Use it for single leads, never for bulk scraping. Scheduled runs have no browser, so this rung never applies there.

### Rung 5: no employer text

Do not guess, do not kill, and do not drop the role silently. Keep it as a lead labelled `needs paste` and list it by name at the end of the run, one line per role, with the best link found. The user can paste the description and the role then goes through Step 3.

`needs paste` roles are never written to the tracker. They only appear in the chat output.

## Step 3: kill rules

A role that meets any of these, on employer facts (see Evidence rule), is killed: it stays out of the tracker and goes on the killed list. For every killed role, keep the company, the role title, a one-line reason naming the rule and the specific fact that triggered it (a date, a city, a salary, a quoted requirement), the best link found, and whether the kill came from the full JD or from the header only.

- **Location:** onsite or hybrid outside the home region with no remote-from-region option. Onsite or hybrid inside the region is never a kill (see Geography).
- **Adjacent discipline** (example rule, replace with yours): the role's centre of gravity is a neighbouring discipline rather than your target. In the original design search this meant product, UX or UI design instead of brand and visual design. Signals: the title names the neighbouring discipline; the must-haves say so; the portfolio they ask for runs from problem definition to final interface. This is a kill even when seniority and tooling look like a strong match.
  - **The distinction that matters:** a role in your target discipline that includes some execution in the neighbouring one is fine, and is in fact the target. A role centred on the neighbouring discipline is not. Ask what the role is centred on, not whether the neighbouring keyword appears in it. A target-discipline role with heavy neighbouring work alongside it passes with a soft flag.
- **Adjacent function** (example rule, replace with yours): the role is centred on running channels or coordination rather than making the work, for example managing an intranet, internal channels, or editorial and comms coordination with the craft as a side task. This is a kill even when the craft keywords appear in the JD.
- **Above the band:** the titles above your band (see Constraints). Titles inside the band are never killed on title alone.
- **Junior:** the employer's title or JD pitches the role as Junior, Trainee, Graduate or entry level. Roles that merely fail the band test are not killed; they go to the lower tier (see Constraints and the band test).
- **Not the right employment type:** freelance, contract, B2B or self-employment basis, agency pool, part-time only, or working-student, if you only want full-time employment. A fixed-term employed contract, such as a parental-leave cover, is not a kill but gets a soft flag.
- **Excluded domains:** `[YOUR PERSONAL NO-GO INDUSTRIES]`. Never argue the user out of one, and add to this list whenever they name more.
- **Company size or type:** `[YOUR RULE, for example: very small studios or pre-seed startups]`.
- **Dead posting:** the role is absent from every board index the employer runs, or the employer's own page or API shows it posted more than 14 days ago. A date that only a job board or LinkedIn shows is a soft flag, not a kill.
- **No employer-side link:** see link rules, including the LinkedIn-only exception.
- **Client-domain gate:** the employer's JD requires proven experience in a client industry or channel the user has never worked in. A required portfolio item you could plausibly build is not a kill; it is a soft flag.
- **Already declined:** a company or role the user has said they will not apply to, or one they have marked Passed in the tracker.

### The band test

Titles drift wildly between companies. The same scope is called one thing at one company and something else at the next, with or without a seniority word. A title filter throws away good roles and lets through weak ones, so read the body instead and ask one question: **does this person decide what good looks like, or do they receive that decision from someone else?**

A role sits in the main band when at least two of these signals appear in the JD:

- **Standard-setting:** sets the standard, defines or evolves the direction, holds the line on quality, helps set the approach.
- **Systems over execution:** has built and evolved systems rather than only working inside them.
- **End-to-end ownership:** owns quality from brief or idea to final output, leads launches, campaigns or announcements.
- **Craft authority:** obsesses over craft, is expected to raise the bar, holds deep expertise in a primary discipline.
- **Autonomy:** proposes work proactively rather than waiting for a complete brief.

A role falls to the **lower tier** (kept, flagged, scored lower) when the body reads like this:

- Supports or assists a more senior person, or works to briefs handed down.
- Applies established guidelines with no mandate to question or change them.
- Carries fewer than two of the signals above.

It is killed only when it is explicitly Junior, Trainee, Graduate or entry level (see kill rules).

Four worked examples, all of which a title or board filter would have thrown away:

- **Company A, "Media Designer".** No seniority word in the title at all. The body asks the person to own the quality of every deliverable from brief to final output and to develop deep expertise in a primary discipline. Two signals, so it is main band.
- **Company B, "Brand Designer, 4+ years".** The body asks for someone who has built and evolved systems rather than executing within them. That is standard-setting language, so the years figure is a footnote, not a verdict.
- **Company C, "Digital Brand Designer".** A job board labelled it entry level. The company's own page says "Experienced", values quality over years, asks for ideas-to-execution ownership, and says some days you set direction. Three signals, so it is main band; the modest base salary is a salary flag, not a kill.
- **Two lower-tier examples**, both kept: a local-market "Graphic Designer" role asking 2+ years for templates and brand guides, and a graphic designer role with no level stated, making social and performance assets.

**Years figures are weak evidence.** Many senior postings set a low floor to widen their funnel. A stated minimum of 4 or 5 years alongside ownership language tells you nothing useful and is not even worth flagging. A floor of 3 or below is worth a flag, and the body decides main band or lower tier.

`[YOUR LANGUAGE RULE, for example: Language requirements other than English are NOT a kill. The user will take the risk on roles that ask for a local language.]`

## Step 4: soft flags

These do not kill. Record them in `Notes`, one short line each.

- `lower tier`: the role failed the band test but is kept. Name what the body asks for (years, briefs handed down, guideline application).
- A language requirement, with the level the JD states.
- Stated minimum experience of 3 years or below. A floor of 4 or 5 alongside ownership language is normal funnel-widening and is not flagged.
- Title carries no seniority word, but the body passed the band test. Name the signals that carried it, so the user can judge for themselves.
- The band looks wide open, for example a salary range whose floor is priced for someone with little or no experience. The role may still be the user's, but they would be competing across several levels.
- The role is mostly people management, with little hands-on making. Not a kill, but it may run against what the user wants.
- Salary stated below `[YOUR SALARY FLOOR]` per year, or no range stated.
- Title drift inside the band, for example Art Director or Expert where the target is Senior.
- Heavily executional work, for example banner and asset production, rather than systems ownership.
- Generalist work, or anything else with thin ownership that still sits on the right side of your discipline line.
- `portfolio item required`: the JD makes a specific portfolio item mandatory (for example paid-social ad concepts), or the work is performance-marketing heavy.
- `adjacent work heavy`: a target-discipline role with substantial neighbouring-discipline work (for example UI and accessibility).
- `public speaking or travel`: the role requires public speaking, on-camera presence, or travel above 20%.
- Heavily compliance-oriented work, or a security clearance requirement.
- `office-first outside [HOME CITY] and [TARGET CITY]`: onsite or office-first elsewhere in the home country, naming the city and any remote allowance.
- `abroad onsite, push for remote`: onsite or hybrid elsewhere in the home region, naming the city.
- `LinkedIn-only`: the role exists only on LinkedIn with Easy Apply (see link rules).
- Fixed-term rather than permanent, or employment type not stated.
- Posting date not shown on the employer's page, so recency is unconfirmed. If a job board or LinkedIn showed a date outside the 14 day window, say which source and which date.
- A job board or aggregator disagrees with the employer page on seniority, location or salary; name the disagreement.
- Ashby listing and date unconfirmed (see Step 2, rung 1).
- Sibling postings exist; say which one this row is (location, pay).
- Employer hidden behind a recruiter or aggregator proxy.

## Step 5: score

Give each survivor a `Match` from 0 to 100. This is an indicative judgment against the CV, not a metric. Weight roughly, and rename the categories for your field:

- `[CORE SKILL AREA 1, the thing your CV is strongest at]`: 35
- Scope match: 20. Judge this on the band test signals, not the title. A role with no seniority word that owns the standard scores higher here than a Senior role that executes to a brief. Lower-tier roles score 0 to 8 here.
- `[CORE SKILL AREA 2 AND 3, for example craft areas]`: 20
- Hands-on balance: 10. Full marks for a role centred on making, with leadership as a consequence of seniority. Score down as people management takes over the job.
- Location and arrangement fit: 10. Home city, target city, remote from the home country or region: 10. Elsewhere in the home country: 6. Onsite elsewhere in the home region: 3.
- Company type fit against the watchlist categories: 5

Do not inflate. A 70 that is honest is more useful than a 90 that is not. Never pad the list to hit a requested count: report the shortfall and name the constraint that caused it.

## Step 6: dedup and write

Query the tracker and drop any candidate matching an existing row on Company plus Role, regardless of status. A reposted role the user was already rejected for counts as a match. A different role at the same company does not, and a sibling posting with the same title but a different location counts as different only when the location changes whether the user can take it. Tracker matches go on the killed list under `Already in tracker`, with the existing row's status.

**Read the `Killer` column on every row before scoring, not only Passed rows.** The user writes their own reasons and concerns there, including on rows they add themselves, and they are the best record of what the user will and will not take. When a pattern repeats in those notes, raise it in the run output as a possible rule change rather than silently applying it.

Write each survivor as its own page. Use one page per call with `notion-create-pages`. Batching consistently fails, and the Notion API rate-limits at around 30 seconds, so space the calls.

Field values:

- `Company`: company name
- `Role`: job title copied verbatim from the JD
- `Link`: the employer-side apply URL, per the link rules above
- `Status`: `Not applied`
- `Match`: the score
- `Why`: one line, the single strongest reason this role survived
- `Killer`: always empty. Concerns and flags go in `Notes`, never here.
- `Notes`: soft flags, one short line each, plus how to apply when it is unusual (for example an application form that is offline and applications go by email)

## Step 7: expire the backlog

After writing, find rows with Status `Not applied` created more than 14 days ago and flip them to `Passed`. Report the count in one line. Do not list them and do not ask for confirmation. Sweep mode only.

## Step 8: output

In chat, return these blocks in this order:

1. **New roles:** one line per new role: `Company, Role, Match, one reason`, with `(lower tier)` appended where it applies. Maximum 5 lines. If there are none, say so in one line and name the constraint that caused it.
2. **Counts:** one line with read, killed (split into full JD and header only), dropped for no employer-side link, expired, and `needs paste`. If a source group (alerts, boards, watchlist) was not swept this run, say so in one line.
3. **Needs paste:** one line per `needs paste` role: `Company, Role, best link`. Maximum 5 lines; summarise the rest by company name in one line.
4. **Killed:** every killed role from this run, no cap, grouped under a short heading per reason (for example Too old, Location outside region, Junior, Adjacent discipline, Adjacent function, Not employment, Posting pulled, No employer-side link, Already in tracker, Over cap). Each line: `Company, Role: reason with the specific fact. [link]`. Append `(header only)` when the JD was not read. Label the link source when it is not employer-side. Maximum 5 lines per heading; split a group into a continued heading when it runs longer.
5. **Worth a second look:** up to 5 killed roles where the kill was thin or rested on a single fact that could be wrong. One line each, naming what would flip it.
6. **Next:** one line naming a single next action and one specific role.

If a new role carries a flag that could be a dealbreaker rather than a footnote, add one short line saying so.

No recap of the search process. Everything else lives in the tracker.

## Formatting

- Never use em dashes.
- Short lines, bold labels, no paragraphs.
- 24-hour time.
- Numbers over adjectives.
- Links in the killed list as short markdown links, for example `[link](url)` or `[board](url)`, never pasted raw.
