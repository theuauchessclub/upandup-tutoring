# Up and Up Educational Services — Tutoring Booking Site

One-page virtual math tutoring site with live booking, split payments (PayPal), automatic second-payment billing, and Google Calendar + Meet integration.

## What's already live

The backend is **already deployed** to Supabase project `otdyhyzghaohnhtwzkvu`:

| Piece | Status |
|---|---|
| Database tables (`edu_packages`, `edu_sessions`, `edu_config`) | ✅ Deployed |
| `edu-availability` — returns booked slots so the calendar greys them out | ✅ Deployed & tested |
| `edu-create-package` — validates the 4 slots, applies discount server-side, creates PayPal order for Payment 1 with card vaulting | ✅ Deployed & tested |
| `edu-capture-payment` — captures Payment 1, saves the vault token, confirms sessions 1–2, creates Calendar events + Meet links | ✅ Deployed |
| `edu-charge-second-payment` — auto-charges Payment 2 within 3 days of session 3, confirms sessions 3–4 | ✅ Deployed |
| Daily cron (12:00 UTC) that runs the auto-charge | ✅ Scheduled |

The `supabase/functions/` folder in this repo is a **reference copy** of the deployed code.

## Publish the site (free, via GitHub Pages)

1. Create a new repository on github.com (e.g. `upandup-tutoring`)
2. Upload `index.html` (drag and drop works on github.com)
3. Repo → Settings → Pages → Source: "Deploy from a branch" → Branch: `main`, folder `/ (root)` → Save
4. Your site goes live at `https://<your-username>.github.io/upandup-tutoring/` in a minute or two

No build step, no cost.

## ⚠️ Before real parents can pay: switch PayPal to LIVE

The project's PayPal secrets are currently **sandbox** (test mode). To go live:

1. Log into https://developer.paypal.com with your PayPal Business account
2. Apps & Credentials → toggle **Live** → create (or open) an app → copy the **Client ID** and **Secret**
3. In Supabase dashboard → Project Settings → Edge Functions → Secrets, update:
   - `PAYPAL_CLIENT_ID` → your live client ID
   - `PAYPAL_CLIENT_SECRET` → your live secret
   - `PAYPAL_BASE_URL` → `https://api-m.paypal.com`
4. **Vaulting requirement:** the automatic second payment saves the parent's payment method ("vault"). Your live PayPal app must have **Vault / Save payment methods** enabled (Apps & Credentials → your app → Features → check "Vault"). If PayPal requires approval for this feature on your account, request it — it's standard for tutoring/subscription businesses.

## Changing prices or the discount code

Pricing lives in the database (server-side, so nobody can tamper with it from the browser). In Supabase → SQL Editor:

```sql
update edu_config set value = '65' where key = 'price_per_session';
update edu_config set value = 'FAMILY15' where key = 'discount_code';
update edu_config set value = '60' where key = 'discount_price_per_session';
```

Also update the matching display numbers near the bottom of `index.html` (`SESSION_PRICE`, `DISCOUNTED_PRICE`, `DISCOUNT_CODE`) so the pricing card shows the same values.

## How a booking flows

1. Parent picks 4 weekday slots (4–7 PM), fills the form, submits
2. `edu-create-package` re-validates everything, reserves the slots, and redirects to PayPal for **Payment 1** (sessions 1 & 2), with the payment method vaulted for later
3. On return, `edu-capture-payment` captures the money, confirms sessions 1 & 2, and fires your Google Apps Script webhook, which creates the Calendar events with Meet links and emails the parent
4. Every day at 12:00 UTC, the cron job finds packages whose session 3 is within 3 days and auto-charges **Payment 2** against the vaulted method, then confirms sessions 3 & 4 and creates their calendar events
5. If a charge fails, the package's `payment2_status` is marked `failed` — check the `edu_packages` table (or Supabase logs) periodically, or ask Claude to build an email alert for failures

## Viewing your bookings

Supabase dashboard → Table Editor → `edu_packages` (one row per family/package) and `edu_sessions` (one row per session). Or ask Claude to build you an admin page.
