# Job search skills for Claude

3 skills that run a job search end to end: find roles, tailor the CV, and keep your own voice.

## What is inside

- **find-jobs:** sweeps boards, alerts and company career pages, reads every full JD, kills bad fits with a reason, scores the rest 0 to 100, writes max 5 per run to Notion.
- **tailor-application:** takes each role marked `Ready to apply`, edits a copy of your HTML CV, writes a cover letter only when needed, suggests a salary number, updates the tracker.
- **voice-and-tone-check:** runs every draft against your written voice before you see it.

## Setup, 6 steps

1. **Create the tracker** in Notion with these fields: `Company` (title), `Role` (text), `Link` (url), `Status` (select: Not applied, Ready to apply, Personalize, Passed), `Match` (number), `Why` (text), `Killer` (text), `Notes` (text).
2. **Build your base files** as plain HTML: `BASE CV.html` and `BASE Cover Letter.html`. Skills and tools are `<span class="chip">` elements, the cover letter placeholders carry `class="fill"`. The tailor skill expects those class names. Add a hidden comma after every chip except the last (see ATS notes below).
3. **Write a voice profile** as a Notion page: 2 or 3 of your own texts plus a short list of do and don't.
4. **Fill every placeholder** in the 3 `SKILL.md` files (table below).
5. **Connect** Notion, Gmail (for LinkedIn alerts), Google Drive (for the base CV) and a folder on your computer.
6. **Test with one role** in single-role mode before you run a full sweep.

## Placeholders

| Placeholder | What goes there |
|---|---|
| `[YOUR NAME]`, `[YOUR FULL NAME]` | Name as used in file names and the letter title |
| `[YOUR PHONE]` | Phone number in the CV masthead |
| `[YOUR HOME CITY, COUNTRY]` | Location line in the CV masthead |
| `[TARGET CITY]` | City you would relocate to, or delete the relocation rule |
| `[YOUR PORTFOLIO URL]` | Portfolio or website link used in CV and letter |
| `[YOUR APPLICATIONS FOLDER]` | Folder on your computer that holds the base files and one folder per role |
| `[YOUR TRACKER DATA SOURCE ID]` | Notion data source ID of the tracker |
| `[YOUR VOICE PROFILE PAGE NAME]`, `[YOUR VOICE PROFILE PAGE URL]` | Notion voice profile page |
| `[YOUR BASE CV DOC NAME]`, `[YOUR DRIVE FOLDER ID]` | Base CV in Google Drive |
| `[YOUR TARGET ROLE FAMILY]`, `[YOUR TARGET SENIORITY]`, `[YOUR TARGET TITLES]`, `[YOUR YEARS]`, `[YOUR BAND]` | What you are searching for |
| `[YOUR SALARY FLOOR]`, `[YOUR CURRENCY FORMAT]` | Pay rules |
| `[YOUR EXCLUDED DOMAINS]`, `[YOUR LANGUAGE RULE]`, `[YOUR EMPLOYMENT TYPE RULE]` | Personal no-gos |
| `[CORE SKILL AREA 1]`, `[CORE SKILL AREA 2 AND 3]` | The 2 scoring categories tied to your field |
| Watchlist and job board sections | Your own companies and boards |
| `[YOUR SIGNATURE ASIDE EXAMPLES]` | Habits in your writing worth protecting |

## ATS notes

- **Line breaks:** a PDF stores text line by line, so copying from it breaks sentences at every wrap. The tailor skill also writes a plain `.txt` with whole paragraphs, for application form boxes.
- **Chip separators:** chips sitting on one line come out as "Lottie Files Figma Weave" in pasted text. A transparent comma after each chip fixes that without changing the layout.
- **Honest limit:** I tested this by exporting a PDF with Chromium and reading it with `pdftotext`. I did not test it against any specific ATS, and hidden punctuation could in theory be flagged by a tool that hates hidden text.

```css
.chip{position:relative}
.chip .sep{position:absolute;right:4pt;top:0;line-height:var(--chip-h);white-space:nowrap;color:transparent}
```

```html
<span class="chip">Figma<span class="sep">,</span></span>
<span class="chip">Gemini</span> <!-- last chip: no separator -->
```

## Notes

- The kill rules and the band test were written for a design search. The structure works for any field. Swap the examples.
- Salary logic uses your own tracker rows as the anchor. It gets better the more roles you log.
- Nothing is sent or submitted for you. Every apply step stays manual.

## License

MIT. See the LICENSE file. Use it, change it, share it.
