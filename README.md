# SermonCraft — setup guide

Free AI tools for pastors and teachers: sermon outlines, Greek & Hebrew word study, sermon ideas, and Sunday school + youth lessons — generated in the user's chosen language (40+ supported).

You have three files:

- `index.html` — the whole website (real React, no build step needed)
- `api/generate.js` — a tiny backend that keeps your API key secret
- `README.md` — this guide

---

## 1. See it right now (no setup)

Double-click `index.html` to open it in your browser. Every tool works and you can click around. Until you connect an API key (below), generating shows a clearly-labeled **sample** so you can see the layout and output — the real, custom, multi-language results turn on once your key is connected.

## 2. Turn on live AI (about 15 minutes)

**a. Get a Claude API key.** Go to https://console.anthropic.com, sign in, and create an API key (starts with `sk-ant-`).

**b. SET A SPENDING CAP — do this first.** In the Anthropic console, set a monthly usage/budget limit (e.g. $40). This is the safety net we discussed: because the tool is free, your only real cost is the AI, and a cap means it can **never** cost you more than the number you choose. When it hits the cap, generation pauses until you raise it or the month resets.

**c. Publish it (free host).** The easiest is **Vercel**:
1. Put all three files in one folder (keep `api/generate.js` inside an `api` folder).
2. Create a free account at https://vercel.com and a free GitHub account.
3. Upload the folder to a GitHub repository, then in Vercel click **Add New → Project** and import that repo.
4. In the project's **Settings → Environment Variables**, add:
   - Name: `ANTHROPIC_API_KEY`
   - Value: your `sk-ant-...` key
5. Click **Deploy**. You'll get a live web address. Add your own domain later if you like (~$12/year).

Your key lives only on the server (in `api/generate.js`) and is never visible to visitors.

## 3. Cost, in plain numbers

- The API is pay-as-you-go — no subscription. It uses **Claude Haiku**, the cheapest model, at about **a penny per generation**.
- Roughly: 500 generations/month ≈ **$4–5**; 5,000 ≈ **$40–50**. Your spending cap is the hard ceiling on all of it.
- Hosting on Vercel's free tier is **$0** at this scale. Donations (Ko-fi, Buy Me a Coffee, PayPal) cost nothing but a small % of what comes in.

## 4. Make it yours

- **Donation link:** in `index.html`, search for `#donate` and replace the placeholder link with your Ko-fi / Buy Me a Coffee / PayPal URL (there's one in the footer and one on the Donate button).
- **Languages:** the language menu (top bar) controls the language of everything the AI writes. Add or remove languages in the `LANGUAGES` list near the top of the script in `index.html`.
- **Greek/Hebrew words:** common words (love, grace, faith, peace, mercy, holy, glory, spirit, and more) load instantly from a built-in, verified list — accurate and free. To add more, copy an entry in the `LEXICON` object and fill it in **from a trusted lexicon** (this avoids AI guessing at Strong's numbers).
- **Model quality:** to upgrade from Haiku, change the `model` line in `api/generate.js` to `claude-sonnet-4-6` (better, ~3× the cost) or `claude-opus-4-8` (best, ~5×).
- **Accounts & saved library:** visitors can sign up (email + password) and everything they generate is saved to a personal "My Library". Setup takes ~15 minutes — see `SETUP-ACCOUNTS.md`. Until then the site runs fine without accounts.
- **Prompts:** each tool builds its instructions in `index.html` (look for `const prompt =` inside each tool). Tweak wording freely.

## 5. When you want to "grow it"

This runs as a single file so you can launch today. When you're ready for a full multi-file React project (Vite) with separate component files, easier testing, and room to add accounts, saved history, etc. — just ask and it converts in one step.

---

### Notes

- **Netlify instead of Vercel?** Rename the function to `netlify/functions/generate.js` and change the first line to `exports.handler = async (event) => { ... }`, reading the body with `JSON.parse(event.body)` and returning `{ statusCode: 200, body: JSON.stringify({ text }) }`. Ask if you'd like this version written out.
- SermonCraft is a **starting point and study aid**, not a replacement for prayer, study, and your own voice — review everything before you preach or teach it, and verify any AI-generated original-language details against a trusted lexicon.
