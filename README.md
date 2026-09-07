# Nakshi — hosting the demo

`index.html` is the whole site. One file, no build step, no dependencies to install.
Put it anywhere that serves static files and it works.

Pick whichever route below you prefer. Both are free and give you a URL with no
mention of how the page was made.

---

## Option A — GitHub Pages (recommended)

Best for a competition: the URL carries your name, and judges can see the source.

1. Create a new **public** repository on GitHub called `nakshi`.
2. Upload `index.html` to the root of it (drag and drop works — *Add file → Upload files*).
3. Go to **Settings → Pages**.
4. Under *Build and deployment*, set **Source** to `Deploy from a branch`,
   branch `main`, folder `/ (root)`. Save.
5. Wait about a minute, then reload the Settings → Pages screen. Your URL appears there:

   ```
   https://<your-github-username>.github.io/nakshi/
   ```

That is the link for the form.

**Optional but worth it:** add a short `README.md` to the repo explaining the
reconciliation engine. A judge who clicks through to the source and finds a
documented repo forms a much better impression than one who finds a bare file.

---

## Option B — Netlify Drop (fastest, about two minutes)

1. Go to <https://app.netlify.com/drop>.
2. Drag `index.html` onto the page. It deploys immediately and gives you a URL like
   `https://cheerful-tesla-4f2a91.netlify.app`.
3. **Create a free account when prompted.** Without one the site is temporary and
   will disappear — with one it is permanent and free.
4. In *Site configuration → Change site name*, rename it to something like `nakshi-demo`,
   which gives you:

   ```
   https://nakshi-demo.netlify.app
   ```

---

## Option C — your own domain

If you want `nakshi.com.bd` or similar, buy the domain (roughly Tk 1,000–3,000 a year
for a `.com`) and point it at either host above. Both support custom domains free.
Only worth doing if you intend to keep working on this after the summit.

---

## Checking it before you submit

1. Open the URL in a **private / incognito window**. This is the important step —
   it proves the page works for someone who is not you.
2. Confirm the QR code renders in the panel on the right. It loads a small library
   from a CDN; if your network blocks it you will see a placeholder instead, which
   is harmless but worth knowing about before a judge sees it.
3. Try the interaction you will demo: push **GRS certificate expiry** past the ship
   date, raise **rPET covered by GRS certificate** to 5040, tick both chain boxes.
   The score should go from 26 to 100 and the badge should read *Export-ready*.

---

## Editing the page

Everything is in `index.html`. The parts you are most likely to change:

| What | Where to look |
|---|---|
| The seven compliance checks | `function runChecks(s)` — each check pushes one object onto `r` |
| Scoring weights | `function score(rs)` — currently `fail = -20`, `warn = -7` |
| Regulatory countdown dates | the `MILESTONES` array near the top of the script |
| Sample order defaults | the `value="..."` attributes on the inputs in the *Extracted values* panel |
| Pricing and market figures | the *Business model* section in the HTML |
| Colours and fonts | the `:root` block at the top of the `<style>` |

Adding an eighth check is about ten lines: push another
`{ k, t, m, w }` object onto `r` inside `runChecks`, where `k` is `"pass"`,
`"warn"` or `"fail"`. Everything else — the score, the tally, the passport
record — updates from that automatically.
