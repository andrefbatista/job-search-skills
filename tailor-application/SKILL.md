---
name: "tailor-application"
description: "Tailor the user's CV copy and, when needed, a cover letter for every role marked Ready to apply on the Notion Job Search Tracker, one at a time, build each tailored CV and cover letter HTML file in its own role folder, suggest a salary number, and update each row."
---

# Tailor applications

Take a role, produce paste-ready copy changes, build the tailored CV file, write and build a cover letter only if it is actually needed, give the user a salary number, and update the tracker row.

This is the writing mode. It never discovers roles.

> Template note: everything in `[SQUARE BRACKETS]` is a placeholder. Fill them in once, then delete this note. See the README for the full list.

## Hard limits

- One role at a time. Never batch roles into a single pass of research, copy or file building.
- Work the queue to the end without asking permission between roles.
- Never modify `BASE CV.html` or `BASE Cover Letter.html`. Every role duplicates them into its own role folder first and edits the copies.
- Never fabricate a skill, a metric, a client, or a date that is not already in the base CV.
- **Salary never appears in the CV or the cover letter.** It is given to the user in chat, every time. See Step 6.
- Seniority framing is non-negotiable. If the role reads as a step down, say so in one line before writing anything.

## Inputs

1. **The role or roles.** If the user named a role, a pasted JD or a URL, that is the queue. If they named nothing, the queue comes from the tracker (Step 0). If a URL cannot be fetched, ask for the JD text pasted in. LinkedIn URLs never fetch.
2. **Base CV.** `BASE CV.html` in the connected folder `[YOUR APPLICATIONS FOLDER]` on the user's computer. Re-read it at the start of every run: it gets edited often, so cached knowledge of its contents from an earlier run is wrong.
3. **Base cover letter.** `BASE Cover Letter.html` in the same folder. Same fonts, accent color and `:root` tokens as the CV, so the two read as a set. Re-read it at the start of every run too.
4. **Voice profile.** Notion page "[YOUR VOICE PROFILE PAGE NAME]": [YOUR VOICE PROFILE PAGE URL]

Tracker data source: `collection://[YOUR TRACKER DATA SOURCE ID]`

If the device is not linked, produce the change list, cover letter and salary number as normal, say the files could not be built, and stop there.

## Step 0: build the queue

Query the tracker. Take every row where `Status` is `Ready to apply`. That tag is the user's signal that they have vetted the role and want it written up, so they never have to say which one.

- **Sort by `Match`, highest first.** That is the running order.
- **Skip a row already tailored.** The marker is the sentence "CV file built and render-checked: <file name>" which lives in the row's PAGE BODY, not in `Notes`. `Notes` is a one-line summary and will not carry it, so fetch the page before deciding a row is untouched. Say so in one line and move on. Redo it only if the user asks, or if `BASE CV.html` has changed since that file was built.
- **If the queue is empty**, say so in one line, name the highest-Match rows sitting at `Not applied`, and stop. Do not tailor a row the user has not marked.

Then run Steps 1 to 8 for the first role, deliver its output, and start the next one in the same turn. Do not wait for the user between roles.

A queue of more than one is normal, not a reason to ask. Set up a task list when the queue has two or more roles, one task per role, so the user can see where the run has got to if they step away.

## Step 1: read the JD properly

Read the whole thing. Pull out:

- The exact job title, character for character.
- The 5 requirements that carry the most weight.
- The language the company uses for its own work, for example their framework or values vocabulary.
- Any stated salary range, and whether the form asks for expectations.
- Any named people on the hiring team, for the cover letter greeting.

## Step 2: research the company briefly

One or two searches, no more. Look for the company's own framing of how they work.

Use that vocabulary in the copy without naming the framework directly. Framework-informed, never framework-quoting. This is the house approach.

## Step 3: CV copy changes

Produce a change list, not a document. For each change give:

- **Section:** where it lives
- **Now:** the current line, verbatim from the base CV
- **New:** the replacement, ready to paste

Rules:

- **Job titles from the JD are copied verbatim** into the masthead. Never paraphrase a title. Drop only the gender marker: `(m/w/d)`, `(f/m/x)`, `(all genders)`, and the German `:in` form all come off. Say in one line when you have done that.
- **Historical role titles stay exactly as they are.** Only the masthead and page title change to the target role.
- **Relocation note (optional rule, delete it if you are not relocating).** When the role is on-site or hybrid in `[TARGET CITY]`, add "open to relocate to [TARGET CITY]" right after `[YOUR HOME CITY, COUNTRY]` in the masthead contact line, so it reads `[YOUR HOME CITY, COUNTRY], open to relocate to [TARGET CITY] | [YOUR PHONE]`. List it as a change. Leave the line as it is for remote roles and for any other city.
- **Profile section covers soft skills only.** Do not repeat hard skills that already appear in Skills or Tools. Light first person is fine here. The profile opens with the target title, so when the verbatim masthead would read badly in a sentence, use the clean form of the title in the profile and keep the verbatim one in the masthead.
- **Bolded sub-labels inside experience bullets use a colon**, for example "Project Name: Owned and scaled..."
- **Ownership-led verbs.** Owned, scaled, built, led. Not supported, helped, assisted.
- Keep changes tight. 4 to 8 changes is a good run. If you are proposing 15, the role is a bad fit and that is the more useful thing to say. Fewer than 5 is its own signal: the role does not need what makes the CV strong, so name that.

End this step with one line naming which requirement from the JD is still not evidenced anywhere in the CV, if any.

If a requirement is something the user has told you they have but the base CV does not state, do not put it in the tailored copy. Name it as a gap and offer to draft the line for them to paste into the base themselves.

## Step 4: build the tailored CV file

Work on the device through its shell, in `[YOUR APPLICATIONS FOLDER]`.

1. **Create the role folder first.** Every role gets its own folder inside Applications, named `<Company> - <Position>`, for example `Acme - Senior Brand Designer`. Position is the JD's title with the gender marker dropped. Replace any `/` in the company or position with `-`, since a slash cannot sit in a folder name. Use `mkdir -p`, so a folder that already exists from an earlier run is reused, not duplicated.
2. **Duplicate, never edit in place.** Copy `BASE CV.html` into that folder as `<Company> - [YOUR NAME] - <Position>.html`. That naming format is fixed. Always copy from `BASE CV.html` in the Applications root, never from the file built for the previous role in the queue. Use `cp -n`, and if a file of that name is already in the folder, say so in one line and stop for that role, unless the user asked for a redo; then copy over it with plain `cp`.
   The cover letter, when there is one, is duplicated into this same folder in Step 5. The whole application, CV, letter and the PDFs the user exports later, lives in this one folder.
3. **Inspect the real markup before writing any edit.** Find each target line with grep and read its surrounding block. The file's structure is not what a plain-text CV would suggest:
   - Skills and Tools are `<span class="chip">` elements, one per line. Every chip except the last in a block carries a hidden separator, `<span class="sep">,</span>`, so pasted or parsed text reads "Figma, After Effects" and not "Figma After Effects". A Skills change means rewriting the chip block, and the rewritten block keeps a separator after every chip but the last.
   - The role appears twice: `<title>` in the head and `<p class="job-title">` in the masthead. Both change.
   - The location sits in `<p class="contact">` after the `<br>`, as `[YOUR HOME CITY, COUNTRY] | [YOUR PHONE]`. If you use the relocation rule, that `old` string becomes `[YOUR HOME CITY, COUNTRY], open to relocate to [TARGET CITY] | [YOUR PHONE]`.
   - An HTML comment block near the top explains how to edit the file and contains example markup. Ignore it; it does not render.
   - `grep -c` counts matching lines, not matches. For link counts use a real match count.
4. **Portfolio links are the thing most easily destroyed.** Experience bullets carry `<a href="[YOUR PORTFOLIO URL]/...">` and other portfolio links wrapped around specific phrases. Any rewritten bullet must carry its links through, re-wrapped around equivalent wording. Reshape the sentence around the link rather than dropping the linked phrase.
5. **Apply the edits with a Python script**, written outside the connected folder, run with `python3 -I` and the file path as an argument. The script must:
   - hold each edit as an exact old/new string pair taken from the file as it actually reads,
   - assert each `old` occurs exactly once, since labels like `Product Experience:` repeat across jobs,
   - count `<a href="http` before and after and require the two to match,
   - check every chip block: a `<span class="sep">,</span>` after each chip except the last,
   - abort without writing if any assertion fails, printing which edit failed.
   Never rebuild the file's contents from tool output, which can be truncated.
6. **Verify by looking at it.** Stage the file and render it to a full-page screenshot, then actually view the image. Check the masthead role, the chip rows, every rewritten bullet, and that links still render as links.
7. **Write the paste-ready text version.** A PDF stores text as separate lines, so copying from it breaks every sentence at the page's wraps, which shows when the text goes into an application form box. Build the `.txt` from the finished CV file with the script below, and save it in the role folder as `<Company> - [YOUR NAME] - <Position>.txt`. It writes whole paragraphs, puts each chip block on one comma-separated line, and skips the hidden separators. Save the script outside the connected folder and run `python3 -I cv_to_text.py "<cv.html>" "<out.txt>"`. Then compare the words in the text against the PDF or the HTML (nothing missing, nothing extra) before reporting it done.

   ```python
   """Turn the CV html into paste-ready plain text: whole paragraphs, chips on one line.

   Usage: python3 -I cv_to_text.py <cv.html> <out.txt>
   """
   import re
   import sys
   from html.parser import HTMLParser


   def tidy(s):
       return re.sub(r"\s+", " ", s).strip()


   class CV(HTMLParser):
       def __init__(self):
           super().__init__(convert_charrefs=True)
           self.lines, self.buf, self.stack = [], [], []
           self.skip = self.sep = 0
           self.chips = None
           self.chip = None

       def gap(self):
           if self.lines and self.lines[-1] != "":
               self.lines.append("")

       def flush(self, fmt=str):
           text = tidy("".join(self.buf))
           self.buf = []
           if text:
               self.lines.append(fmt(text))

       def handle_starttag(self, tag, attrs):
           cls = (dict(attrs).get("class") or "").split()
           self.stack.append((tag, cls))
           if tag in ("style", "script", "title"):
               self.skip += 1
           elif "sep" in cls:
               self.sep += 1
           elif "chip" in cls:
               self.chip = []
           elif tag == "br" or tag in ("p", "h1", "li"):
               self.flush()
           elif "bar" in cls:
               self.flush()
               self.gap()
           elif "chips" in cls:
               self.chips = []
           elif "role" in cls:
               self.flush()
               self.gap()

       def handle_endtag(self, tag):
           _, cls = self.stack.pop() if self.stack else (tag, [])
           if tag in ("style", "script", "title"):
               self.skip -= 1
           elif "sep" in cls:
               self.sep -= 1
           elif "chip" in cls:
               self.chips.append(tidy("".join(self.chip)))
               self.chip = None
           elif "chips" in cls:
               self.lines.append(", ".join(self.chips))
               self.chips = None
           elif "bar" in cls:
               self.flush(str.upper)
           elif "role" in cls or tag in ("p", "h1"):
               self.flush()
           elif tag == "li":
               self.flush(lambda t: "- " + t)

       def handle_data(self, data):
           if self.skip or self.sep:
               return
           (self.chip if self.chip is not None else self.buf).append(data)


   parser = CV()
   parser.feed(open(sys.argv[1], encoding="utf-8").read())
   parser.flush()
   text = "\n".join(parser.lines).strip() + "\n"
   text = re.sub(r"\n{3,}", "\n\n", text)
   assert "\u2014" not in text, "em dash in output"
   open(sys.argv[2], "w", encoding="utf-8").write(text)
   print(f"{len(text.split())} words, {text.count(chr(10))} lines")
   ```

Tell the user the folder and file name, and anything the base itself needs fixing, for example an empty `<a href=""></a>` anchor. Report a base CV problem once per run, not once per role.

## Step 5: cover letter, only if needed

Write one if the JD or the application form asks for one, or the tracker Match is 80 or above.

Otherwise skip it and say in one line that it was skipped and can be added on request.

### Writing it

- **Default tone is professional.** Enthusiasm toned down. Full enthusiastic and personal tone only when the user has said this role is one they are genuinely excited about.
- Open warm and human, with context before the point.
- Keep the user's parenthetical asides, for example `[YOUR SIGNATURE ASIDE EXAMPLES]`. One per text, and they survive even in professional drafts.
- Vary transitions. Do not lean on one connector phrase.
- No abrupt jumps between sentences.
- Nothing cocky. Skill shows through the hands-on work and through empathy, not through claims.
- Correct non-native grammar slips rather than preserving them.
- Never em dashes.
- No salary figure, ever, even when the posting asks for expectations. The number goes in chat, not in the letter.
- 250 to 350 words.
- The house shape, which the template is built around: an opening paragraph, a paragraph on the one thing in the JD that matters most, a lead-in line plus 3 bullets mapping JD requirements to hands-on work, an optional logistics line (location, remote, time zones), and a closing line that links to [YOUR PORTFOLIO URL].

If the JD states a hard requirement the user clearly does not meet, such as fluency in a language, name it plainly in one short paragraph near the end rather than leaving it to be discovered. Offer the version without that paragraph.

If the user has applied to this company before, consider naming it in the opening line. It reads as deliberate, and their ATS keeps the old record anyway. Offer the version without it.

If the user dictated input by voice, clean it up: strip fillers and restarts, move it to their written voice.

Then run the `voice-and-tone-check` skill against the draft before showing it. Show the final draft only, not the check output, unless something failed.

### Building the letter file

Once the draft has passed the voice check, build it from the template. Same device, same shell.

1. **Duplicate, never edit in place.** Copy `BASE Cover Letter.html` from the Applications root into the role folder created in Step 4 (`<Company> - <Position>`), as `Cover Letter - <Company> - [YOUR NAME] - <Position>.html`. Naming is fixed, with the same Position as the CV file. Same `cp -n` rule as the CV: an existing letter of that name stops the role unless the user asked for a redo.
2. **Fill these fields.** Each one carries `class="fill"` (or `fill` alongside another class) in the template:
   - `<title>`: `Cover Letter, [YOUR FULL NAME], <Position>`
   - `<p class="job-title fill">`: the same verbatim title as the CV masthead
   - `<p class="recipient fill">`: `<Company> Team`, unless the JD names a specific team
   - Date `<p class="fill">`: today, in `[YOUR DATE FORMAT]`
   - `<div class="subject fill">`: `Application for <Position>`
   - Greeting `<p class="fill">`: the named people from the JD, for example `Hello Ana, Bram and Sudarshan.`, otherwise `Hello <Company> team.`
   - The body: replace every placeholder `<p class="fill">` and the placeholder `<ul class="fill">` with the draft as plain `<p>` paragraphs and one `<ul>` of `<li>` for the bullets. Drop the logistics paragraph when the draft has none.
   - Contact line: when the CV got the relocation note, add the same note to the letter's contact line so the two match. Otherwise leave it alone.
   - Leave the masthead name, the sign-off and the signature alone.
3. **Apply it with a Python script**, same rules as Step 4: written outside the connected folder, run with `python3 -I`, exact old/new pairs, each `old` asserted to occur exactly once, abort without writing on any failure. Write `&` as `&amp;` and apostrophes as `&rsquo;`. Then the script must also assert, before writing:
   - no `fill` class remains anywhere after `<main`,
   - no `[` placeholder bracket remains after `<main`,
   - no em dash anywhere in the file,
   - the portfolio link in the closing line is still there.
4. **Verify by looking at it.** Stage the file, print it to an A4 PDF with backgrounds on and zero margins, and view the image. It must be exactly 1 page, and the signature must be visible: the sheet is locked to one A4 page and hides overflow, so a missing signature means the letter is too long. Cut copy, never shrink the type.

## Step 6: salary number

Give a number for every role, **even when the posting does not ask for one**. The user needs it for the form, the recruiter call, or to decide whether to apply at all.

Give three things, never just a range:

- **The number to type into a form.** One figure, gross annual, full-time.
- **The range to say out loud** in a call, roughly the number minus 5k to plus 5k.
- **The walk-away floor**, below which the role is not worth the user's time.

**Build the number from the user's own tracker first.** Many rows carry a posted range the user recorded, in `Notes` or in the row's page body. Pull every row at the same level in the same city, take the midpoints, and use that cluster as the anchor. That dataset beats any public source, because it is current, local, and matches the exact profile.

**Use public aggregators only as a sanity check, and say so.** Glassdoor, StepStone, Indeed and similar sites can read low for senior roles, because they sweep in other cities, agency-mid and junior-titled roles on small samples. In the original author's market they read roughly 20 to 30 percent low. Calibrate that against your own tracker data. Never let an aggregator set the anchor, and warn the user if a recruiter quotes one.

**Then move off the anchor by employer type:**

- **In-house tech, scaleups and listed companies:** above the anchor. Add more when the role spans several markets, reports into a Director, or carries a Manager or Lead title.
- **Agencies and studios:** on or just below the anchor. Brand prestige does not raise pay.
- **Startups and roles with no seniority marker in the title:** below the anchor.
- **A stated minimum of 3 years is a budget signal.** The role has been costed as mid-level, so asking at the top of the band prices the user out. Say that plainly.

**Always state:** the figures are gross annual salary, full-time, in `[YOUR CURRENCY FORMAT]`, and whether a bonus or OTE sits on top. If the posting states its own range and it falls below the user's floor, say so in one line before anything else.

When screen-out risk is real, give both the aggressive number and the safer one in a single line, and let the user choose.

This is the user's decision, not a recommendation to act on blind. Say where the data is thin.

## Step 7: update the tracker

Every row has two places to write. `Notes` is the column the user scans in the table view. The row's page body is the full record. Never put the full write-up in `Notes`.

**`Status`:** `Personalize` while the role is being worked, then back to `Ready to apply` once the copy is delivered. `Ready to apply` stays the resting state, because the row is genuinely ready for the user to apply.

**`Notes`: one short line, under about 250 characters.** Written to be read at a glance, in this order, dropping anything that does not apply:

- Location and contract type
- Start date, if stated
- The salary ask and the floor
- What has been built, for example "CV + cover letter done"
- How to apply, for example an email-only address
- The words "Detail in page."

For a row that has not been tailored yet, use the same shape but lead with the flags: language level, title drift, relocation, a 3-year minimum, doubtful recency.

**The page body carries everything else.** Write it with `notion-update-page`, command `insert_content` on a blank page, or `replace_content` when rewriting one this run already created. Use these headings, skipping any that would be empty:

- `## Role facts`: location, contract, language, start date, benefits, stated salary, posting date and where it was confirmed
- `## Company`: what they do, size, clients, how they describe their own way of working
- `## Flags`: language level, title drift, relocation, level signals, recency doubts
- `## How to apply`: the form, the email address, file limits, what they ask for
- `## Tailoring, <date>`: the angle taken, the role folder name, and the sentence "CV file built and render-checked: <file name>", plus whether a cover letter was written and why, and when it was, the sentence "Cover letter built and render-checked: <file name>"
- `## Salary`: the three figures and the reasoning behind them
- `## Gaps`: the requirements not evidenced anywhere in the base CV, and anything the user has to decide at the application form

The "CV file built and render-checked" sentence is what stops a later run re-tailoring the row, so it is not optional, and Step 0 looks for it in the page body.

If no row exists, create one. One page per call with `notion-create-pages`. Batching consistently fails. Space Notion calls out; the API rate limits at around 30 seconds.

## Step 8: output

For each role, return in this order and nothing more:

1. One line: role, company, and the angle in a few words.
2. The CV change list.
3. One line: the unevidenced requirement, or that there is none.
4. One line: the role folder, the CV file and its paste-ready `.txt` built, and whether the render checked out.
5. The cover letter draft plus one line naming the letter file and whether its render checked out, or one line saying the letter was skipped.
6. The salary number, range and floor, with one line of reasoning.
7. One line: the next action, normally opening the files and exporting them to PDF into the same role folder.

Deliver each role's output as it finishes rather than holding everything to the end, so the user can redirect early.

When the queue had more than one role, close the run with a short table of company, role folder, letter (yes or no), and salary number, plus one line on anything that needs a decision from the user.

No preamble. No recap of the JD. No explanation of the tailoring approach.

## Formatting

- Never use em dashes, in chat or in any draft.
- Short lines, bold labels, no paragraphs in the chat parts.
- 24-hour time.
- Numbers over adjectives.
- Salary figures in `[YOUR CURRENCY FORMAT]`.
- PDF exports: Chrome, Print, Save as PDF, Margins None, Background graphics on, A4 at 100%. Save them in the role folder. Name the letter PDF `Cover Letter - <Company> - [YOUR NAME] - <Position>.pdf`.
