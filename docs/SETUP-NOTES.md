# docs/ Setup Notes

This file documents the integrations baked into the published Pages site (`docs/index.html` and `docs/leaders-memo.html`) that need a one-time activation by the maintainer. Both are free, privacy-respecting, and zero-cost at the volumes this site will see.

If you're not the maintainer, you can ignore this file — it doesn't affect reading the content.

---

## 1. Analytics — Cloudflare Web Analytics

**Why this one:** free forever, no cookies, no PII collection, no GDPR consent banner needed, no JavaScript besides one tiny script. Lighter than Plausible (which costs money) and far lighter than Google Analytics.

**Activation steps (~5 minutes):**

1. Sign in (or sign up — free) at https://www.cloudflare.com/web-analytics/
2. Click "Add a site"
3. Enter your site hostname (`lofigamerguy.github.io` or your custom domain)
4. Cloudflare returns a JavaScript snippet containing a beacon token. The token looks like a long hex string in the `data-cf-beacon` attribute.
5. Replace the placeholder `YOUR_CLOUDFLARE_BEACON_TOKEN_HERE` in BOTH `docs/index.html` and `docs/leaders-memo.html` with the real token.
6. Commit and push. Pages rebuilds in 1-2 minutes; analytics start within a few hours.

**What you get:**
- Page views per page
- Top referrers
- Top countries
- Visit duration distribution
- Browser / OS breakdown
- All without cookies or PII

**To verify it's working:** visit your published site, then check the Cloudflare Analytics dashboard within ~24 hours. You should see your visit logged.

**To remove it:** delete the `<script>` block in both HTML files. No other cleanup needed.

---

## 2. Feedback form — Formspree

**Why this one:** lets non-GitHub readers (your boss, executives, anyone sharing the link onward) send feedback without a GitHub account. Free tier supports 50 submissions/month, which is generous for a content site. No backend to operate.

**Activation steps (~5 minutes):**

1. Sign up (free) at https://formspree.io/
2. Click "New Form"
3. Set the form name (e.g., "AI Engineering Patterns feedback") and the destination email (where submissions get forwarded)
4. Formspree returns a form endpoint URL of the shape `https://formspree.io/f/xxxxxxxx`
5. Replace the placeholder `https://formspree.io/f/YOUR_FORMSPREE_ENDPOINT_HERE` in `docs/index.html` with the real endpoint
6. Commit and push

**What you get:**
- A simple feedback form at the bottom of the catalogue
- Submissions forwarded to your email
- Spam protection built in
- Anti-spam reCAPTCHA shown automatically when needed
- No backend code to maintain

**Alternatives if you don't want Formspree:**
- **Web3Forms** (https://web3forms.com/) — free, similar shape, no account needed for the basic plan
- **Formcarry** (https://formcarry.com/) — similar
- **A simple `mailto:` link** — zero-setup, but less professional and exposes your email to scrapers
- **Remove the form entirely** — just delete the `<section class="fo-feedback">` block in `docs/index.html`

**To remove it:** delete the `<section class="fo-feedback">` block in `docs/index.html`. No other cleanup needed.

---

## What's the cost?

Both services are genuinely free at the volumes this site will see:
- Cloudflare Web Analytics: free forever, no upper limit
- Formspree free tier: 50 submissions / month (you'll likely never hit this)

No credit card required for either.

---

## Why these specific tools?

I considered several alternatives:

**For analytics:**
- *Plausible* — excellent, but $9/month minimum
- *Umami* — open-source, but requires self-hosting
- *Google Analytics* — free, but invasive, slow, and requires cookie consent
- *Cloudflare Web Analytics* — chosen for: free, cookieless, fast (no consent banner), no PII

**For feedback:**
- *Formspree* — simple, mature, free tier sufficient
- *Tally* — also good, slightly heavier UI
- *Google Forms* — free but ugly inline; better as standalone link
- *Notion forms* — heavier, requires Notion account
- *A custom backend* — overkill for this

The choices reflect "lightest tool that does the job" rather than feature maximalism.
