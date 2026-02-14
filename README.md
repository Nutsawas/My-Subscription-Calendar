# Subscription Calendar

A single-page web app that shows your subscriptions on a calendar. It fetches data from an **n8n webhook** (POST) and displays the current month with due dates, icons, and total monthly spend (฿).

- **Tech:** Plain HTML/CSS/JS, no build step.
- **Data source:** Configure your n8n workflow to respond with an array of subscription items; the app parses several response shapes (see below).

---

## Running the app (required: local server)

Opening `index.html` directly (e.g. double-click) uses **origin `null`**, so the n8n webhook will block the request (CORS). You must run the app through a local server.

**Option 1 – npx (default port 3000):**

```bash
cd subscription-calendar
npx serve .
```

Then open **http://localhost:3000** in your browser.

**Option 2 – Python:**

```bash
cd subscription-calendar
python3 -m http.server 8000
```

Then open **http://localhost:8000**.

---

## n8n setup

- Use the **Production URL** for the webhook (not the Test URL).
- The path must match the URL used in the app. By default the app calls:  
  `https://n8n.willy.moda/webhook/my-subscription` (POST, `Content-Type: application/json`).
- Change `WEBHOOK_URL` in `index.html` if your webhook path is different.

The app sends a POST with an empty JSON body `{}` and expects a JSON response (see “Expected response format” below).

---

## Expected response format

The app expects the webhook to return **an array of subscription items**. It will also accept:

- A top-level **array** → used as-is.
- An object with one of: `data`, `body`, `body.data`, `subscriptions`, `items`, `result`, `output` (each an array) → that array is used.
- The first **array** found among the response object’s values.
- A single object with `name` / `title` / `subscriptionName` → wrapped into a one-item array.
- n8n “First Entry” style: `raw.json` → wrapped into a one-item array.

**Important:** To show **all** subscriptions, the n8n **Respond to Webhook** (or equivalent) must return the **full array**, not only “First Entry”. If you respond with a single item, only one subscription will appear.

---

## Field mapping (columns / properties)

Each subscription item can use the following property names (the app checks in this order where relevant):

| Purpose        | Accepted property names |
|----------------|-------------------------|
| **Name**       | `name`, `subscriptionName`, `subscription_name`, `title`, `service`, `product` |
| **Due day**    | **Day of month (1–31):** `day`, `payment_day`, `day_of_month`, `date_day` |
|                | **Full date (YYYY-MM-DD or parseable):** `paymentDate`, `payment_date`, `date`, `dueDate`, `due_date`, `next_payment`, `nextPayment` — the app uses the day only for the **current month** |
| **Price**      | `price`, `amount` (used for “Monthly spend”) |
| **Icon**       | `icon`, `iconSvg`, `icon_svg`, `svgIcon`, `svg_icon`, `Icon`, `IconSvg`, `svg`, `image`, `imageUrl`, `image_url` — **string**: inline SVG (e.g. `<svg ...>...</svg>`) or image URL |
| **Icon color** | `iconColor`, `icon_color`, `color`, `dotColor`, `dot_color` — e.g. `#22c55e` or `green` |

If **Icon** is missing, the app uses built-in icons for: **github**, **netflix**, **spotify**, **amazon**, **linkedin** (matched by subscription name, case-insensitive, no spaces); otherwise a default icon is used.

---

## What the app shows

- **Calendar:** Current month only; each day shows subscriptions whose due day (or parsed date) falls on that day in the current month.
- **Header:** Month and year; **Monthly spend** = sum of all `price`/`amount` for the parsed list (in ฿).
- **Day cells:** Subscription icons (with optional color); today is highlighted. Hover/title shows subscription name and price when available.

---

## Troubleshooting

### “Only one subscription appears but I send more”

1. **n8n is returning a single item**  
   In **Respond to Webhook**, do not respond with “First Entry” only. Respond with the **full array** of items (e.g. use a node that aggregates all items and respond with that array, or `{{ $json }}` in a way that returns the whole list).

2. **Other items are in another month**  
   The calendar shows **only the current month**. If one subscription is due in January and another in February, you will see one in January and the other in February when you view that month. There is no month switcher in the UI yet.

### Load error / CORS

- Open the app via **http://localhost:3000** (or your chosen port), not via `file://`.
- Ensure the webhook URL in the app matches your n8n **Production** webhook URL and that n8n allows requests from your origin.

---

## Changing the webhook URL

Edit `index.html` and set:

```js
const WEBHOOK_URL = 'https://your-n8n-host/webhook/your-path';
```

Then run the app through a local server as above.
