# Domain Setup & Deployment Guide — AI Agent Atlanta

All steps are performed in the Cloudflare dashboard. You'll need access to the Cloudflare account where the Pages project lives.

---

## 1. Add the Primary Domain to Cloudflare Pages

**Where:** Cloudflare Dashboard → Workers & Pages → your Pages project → Settings → Custom domains

1. Click **Add a custom domain**
2. Enter `aiagentatlanta.com` → click Continue
3. Cloudflare will prompt you to add a CNAME record. If `aiagentatlanta.com` is already on Cloudflare DNS, it does this automatically. If not, follow the DNS instructions shown.
4. Repeat for `www.aiagentatlanta.com` — Cloudflare will auto-redirect www → root for you.
5. Wait for the SSL certificate to provision (usually under 5 minutes).

**Verify:** Visit `https://aiagentatlanta.com` and `https://www.aiagentatlanta.com` — both should load the homepage.

---

## 2. Set Up the City Redirect Domains

For each of the five city domains below, you want all traffic to 301-redirect to the matching city page on aiagentatlanta.com. Do this with Cloudflare Redirect Rules (not Page Rules — those are deprecated).

**For each domain, repeat steps A through D:**

### City domains to configure:
| Domain | Redirect target |
|---|---|
| aiagentsmarietta.com | https://aiagentatlanta.com/marietta/ |
| aiagentscobb.com | https://aiagentatlanta.com/cobb/ |
| aiagentsalpharetta.com | https://aiagentatlanta.com/alpharetta/ |
| aiagentsbuckhead.com | https://aiagentatlanta.com/buckhead/ |
| aiagentspeachtree.com | https://aiagentatlanta.com/peachtree/ |

### Steps A–D (repeat for each domain):

**A. Ensure the domain is on Cloudflare**
- Go to the main Cloudflare dashboard (not Pages) → Add a Site → enter the domain name
- Choose the Free plan if prompted
- Update the domain's nameservers at your registrar to point to Cloudflare (shown on screen)
- Wait for nameserver propagation (usually under 30 minutes)

**B. Add a proxied DNS record (required to enable redirect rules)**
- In that domain's Cloudflare zone → DNS → Records → Add record
  - Type: `A`
  - Name: `@` (root)
  - IPv4 address: `192.0.2.1` (this is a reserved/dummy IP — Cloudflare proxies the traffic before it ever reaches this address)
  - Proxy status: **Proxied** (orange cloud — required)
  - TTL: Auto
- Also add:
  - Type: `CNAME`
  - Name: `www`
  - Target: `@`
  - Proxy status: Proxied

**C. Create the Redirect Rule**
- In that domain's Cloudflare zone → Rules → Redirect Rules → Create rule
- **Rule name:** e.g., `Redirect all to aiagentatlanta.com/marietta/`
- **When incoming requests match:**
  - Field: `Hostname`
  - Operator: `equals`
  - Value: `aiagentsmarietta.com` (or whichever domain you're configuring)
  - (Add a second "OR" condition for `www.aiagentsmarietta.com`)
- **Then:**
  - Type: `Static`
  - URL: the target URL (e.g., `https://aiagentatlanta.com/marietta/`)
  - Status code: `301`
- Click **Deploy**

**D. Verify**
- Visit `http://aiagentsmarietta.com` in a browser
- You should be 301-redirected to `https://aiagentatlanta.com/marietta/`
- Check with `curl -I http://aiagentsmarietta.com` to confirm the `301` status and `Location` header

---

## 3. fractionalai.net Redirect

This domain should eventually redirect to `https://aiagentatlanta.com/` — but **do not switch it until aiagentatlanta.com is confirmed live and stable**.

### Phase 1 — While aiagentatlanta.com is being confirmed:
- Keep `fractionalai.net` as a **custom domain on the Pages project** (same setup as step 1 above)
- This ensures the site stays accessible on fractionalai.net as a fallback

### Phase 2 — Once aiagentatlanta.com is live and confirmed:
1. Remove `fractionalai.net` from the Pages project custom domains (Settings → Custom domains → remove)
2. In the `fractionalai.net` Cloudflare zone, add a proxied A record `@` → `192.0.2.1` (same as city domains above)
3. Create a Redirect Rule: all traffic → `https://aiagentatlanta.com/` (301)
4. Verify with a browser and curl

---

## 4. Email Routing for hello@aiagentatlanta.com

Cloudflare Email Routing lets you receive email at your custom domain and forward it to an existing inbox — no mail server required.

1. In the `aiagentatlanta.com` Cloudflare zone → **Email** → **Email Routing**
2. Click **Enable Email Routing** (follow the prompt to add the required MX and TXT records — Cloudflare does this automatically if the zone is on Cloudflare)
3. Under **Routing rules** → **Custom addresses** → **Create address**:
   - Custom address: `hello`
   - Action: **Send to an email** → enter your receiving inbox (e.g., ryan@brandlifemarketing.com)
4. Cloudflare will send a verification email to the destination address — confirm it
5. Test: send an email to `hello@aiagentatlanta.com` and confirm it arrives in your inbox

---

## Quick Reference: Current Domain Map

| Domain | Destination | Method |
|---|---|---|
| aiagentatlanta.com | Pages project (primary) | Pages custom domain |
| www.aiagentatlanta.com | aiagentatlanta.com | Pages auto-redirect |
| aiagentsmarietta.com | /marietta/ | Redirect Rule (301) |
| aiagentscobb.com | /cobb/ | Redirect Rule (301) |
| aiagentsalpharetta.com | /alpharetta/ | Redirect Rule (301) |
| aiagentsbuckhead.com | /buckhead/ | Redirect Rule (301) |
| aiagentspeachtree.com | /peachtree/ | Redirect Rule (301) |
| fractionalai.net | Pages project (interim), then / redirect | Phase 1→2 |
| hello@aiagentatlanta.com | ryan@brandlifemarketing.com | Email Routing forward |
