# Udyam Digital

**Where Small Businesses Meet Big Possibilities**

A working prototype that lets a small business owner discover, buy, verify (KYC) and track telecom and digital services in one sitting, on a phone or a laptop. Built for Airtel SYNC 2026, Enterprise Track A.

> Hackathon prototype on synthetic data. It is not a live telecom service. Payments are simulated, and KYC checks run against mock registries, not government systems.


More screenshots of every page, in both sizes, are in `docs/screenshots/`.

---

## 1. Run it (beginner steps)

You need one thing installed: **Node.js version 18 or newer** (download the "LTS" version from nodejs.org). To check, open a terminal and type `node -v`.

Open a terminal in this folder (`udyam-digital`) and run these, one at a time.

**Step 1. Install everything (only the first time, takes a few minutes)**

```bash
npm run setup
```

**Step 2. Start the server (terminal 1)**

```bash
npm run server
```

Wait until you see `API ready at http://localhost:4000/api/health`.

**Step 3. Start the website (open a second terminal in the same folder)**

```bash
npm run client
```

**Step 4. Open the app**

Go to **http://localhost:5173** in your browser.

To see the phone layout on a laptop: press `F12` in Chrome, then click the phone/tablet icon at the top left of the panel that opens.

To open it on your real phone: connect phone and laptop to the same Wi-Fi, and open the "Network" address that terminal 2 prints (for example `http://192.168.1.5:5173`).

### About the database (MongoDB)

You do **not** have to install MongoDB. When the server starts it picks the first of these that works:

1. The `MONGODB_URI` in `server/.env`, if you set one (for example a free MongoDB Atlas cluster).
2. A MongoDB already running on your computer.
3. A temporary built-in MongoDB. The first run downloads it once (needs internet, a few hundred MB), and its data is cleared whenever the server stops.

For demo day, option 1 or 2 is safer than relying on a download over venue Wi-Fi. To use option 1: copy `server/.env.example` to `server/.env` and paste your connection string after `MONGODB_URI=`.

### Optional: one address instead of two terminals

```bash
npm run demo
```

This builds the website and serves everything from **http://localhost:4000**.

### Optional: switch on the LLM

The advisor works fully without any AI key (rule-based engine). To let an LLM reword the explanation, put a key in `server/.env`:

```
LLM_API_KEY=your-key
LLM_BASE_URL=https://api.openai.com/v1
LLM_MODEL=gpt-4o-mini
```

Any OpenAI-compatible API works. The LLM never chooses products or prices: if its text contains a number the pricing engine did not produce, the text is thrown away and the rule-based explanation is shown.

### If something goes wrong

| What you see | What to do |
| --- | --- |
| `Port 4000 is already in use` | Close the other server window, or add `PORT=4001` to `server/.env` (then also start the client with `VITE_API_TARGET=http://localhost:4001`). |
| `Cannot reach the server` on the website | Terminal 1 is not running. Run `npm run server`. |
| `Data file not found` | The CSV files must be inside the `data` folder. |
| Server stuck on "starting a temporary built-in one" | It is downloading MongoDB. Wait, or use `MONGODB_URI` as described above. |
| First document upload is slow | OCR is warming up. Later uploads take 1 to 3 seconds. |

---

## 2. Demo script for judges (about 3 minutes)

Scenario: a medical store in Lucknow, 5 employees, 1 location, wants reliable internet, UPI/POS billing and CCTV, and has a GSTIN.

1. Open the home page and click **Play the medical-store demo**. The advisor asks its questions and answers them by itself, then recommends **Pharmacy Smart Connect** (Biz Fibre 100, Cloud POS & Billing, 2 Cloud CCTV cameras) at ₹1,996 a month plus ₹6,100 setup.
2. Click **See bundle and full price**. Switch on the **Backup 4G/5G Failover** add-on and watch the total change.
3. Click **Proceed to checkout**. In the yellow demo box choose **D01: Evergreen Medicals, Lucknow** to fill the form. Tick the demo payment box and place the order.
4. On the confirmation screen, continue to KYC. Read the consent notice, tick the box and agree.
5. Click **Fill sample GSTIN**, then **Check GSTIN**. The legal name, address and Active status appear from the mock registry.
6. Click **Fill sample Udyam number**, then **Check Udyam**.
7. Under Documents click **Use sample document** on each of the four slots. OCR reads each one and shows the extracted fields and checks. KYC is approved automatically.
8. Click **Choose installation date**, pick a date and time, and book it.
9. Click **Simulate next stage** twice. The order reaches **Service activated**.
10. Open **Partner** (access code `partner-demo`) to show lead ranking and the business metrics.

To run the same journey by hand, click **Start with the advisor** instead, and try typing answers such as "Medical store" or "nahi hai", or switch to हिंदी at the top right.

### Show that Smart KYC catches problems

At checkout, pick one of the hackathon's sample applicants instead of D01, then use the sample GSTIN and documents on the KYC page.

| Sample | What happens |
| --- | --- |
| K01 Vindhya Tours & Travels | GSTIN is **Cancelled** in the registry, so it is rejected |
| K02 Harmony Dental Care | GST certificate is a tilted scan; OCR straightens it and it verifies |
| K04 Saffron Dental Care LLP | GST certificate is blurred; rejected with a "retake the photo" message |
| K05 Balaji Realty LLP | No Udyam certificate in the pack; skip the Udyam step and it approves |
| K06 Saffron Unisex Salon | GSTIN printed on the certificate differs by one digit; mismatch |
| K07 Neelgiri Physio Clinic | Legal name on the certificate differs from the registry; mismatch |
| K11 Trinity Bake House Pvt Ltd | Electricity bill is 257 days old; rejected (limit is 90 days) |
| K03, K08, K09, K10, K12 | Clean; approved automatically |

Document dates are checked against **15 July 2026**, the "today" the dataset README asks for (`KYC_REFERENCE_DATE` in `.env`).

---

## 3. What is inside

| Module | Where to look | Notes |
| --- | --- | --- |
| A. AI Business Advisor | `/advisor` | 7 questions, chips, typing indicator, Hindi/English, voice input, change any answer, rule-based with optional LLM wording |
| B. Bundle, quotation, checkout | `/bundle`, `/catalogue`, `/checkout` | Up to three bundle options, quantity steppers, add-on toggles, live price, printable quotation, demo payment |
| C. Smart KYC | `/kyc/:orderId` | Consent, GSTIN check, Udyam check, OCR on four document types |
| D. Order tracker | `/track/:orderId` | Six-stage timeline with timestamps, pending actions, self-booked installation, help request |
| Stretch: partner dashboard | `/partner` | Lead ranking with explainable score, filters, follow-up updates, baseline metrics |

**Extras beyond the brief**

- Auto-play demo for the medical-store scenario.
- OCR works offline (language data ships with the project) and repairs look-alike characters in a GSTIN using its check digit.
- Checkout can fill business name and address from the GSTIN.
- The customer books their own installation date. "Visit not scheduled" and "installation delayed" are among the drop-off reasons in the lead data.
- What-if calculator on the metrics tab (clearly marked as an illustration, not a forecast).
- Document images are never stored (see `docs/PRIVACY_NOTE.md`).

---

## 4. How the important rules work

### Eligibility and pricing (never decided by the LLM)

All of it is in `server/src/services/catalogue.js` and reads `product_catalogue.csv`:

- city tier, minimum employees, minimum sites, GSTIN required
- `requires_one_of` dependencies (for example CCTV needs an internet link in the same bundle)
- setup fee × quantity, contract = longest minimum contract in the bundle, time to go live = slowest product's `typical_install_days`

**One assumption you should know about.** The catalogue gives a price *range* for each product but no rule for where a customer lands in it. We chose:

| Different services in the bundle | Price used |
| --- | --- |
| 1 | top of the range (list price) |
| 2 | middle of the range |
| 3 or more | bottom of the range |

"Savings" is list price minus quoted price, so it always comes from catalogue values. To change the rule, edit `PRICE_POLICY` at the top of that file. Prices exclude taxes because the catalogue does not list any.

Cities that are not in the dataset are treated as tier 2, and the advisor says so.

### Recommendation

`server/src/services/recommender.js` maps each need to products, sizes quantities from employees and locations, adds a required internet link when something depends on one, and re-checks the result through the pricing engine. Products that were considered but not allowed are shown to the customer with the reason.

### Lead priority score (rule-based, not machine learning)

`server/src/services/leadScoring.js`. Each lead gets up to 100 points and the dashboard shows the breakdown:

| Factor | Max |
| --- | --- |
| Funnel stage reached | 30 |
| Drop-off reason (how recoverable it is) | 30 |
| Registration readiness (GSTIN active in registry, Udyam) | 15 |
| Segment fit (that segment's activation rate last quarter) | 10 |
| Engagement (products asked about, follow-up calls, lead source) | 15 |

High is 65 and above, Medium 45 to 64, Low below 45. The points are hand-set business rules. The score has not been trained or validated against outcomes.

### Baseline metrics

Computed from `smb_leads.csv` at start-up: 5,000 leads, 368 activated, 7.36% conversion, 17.6 days average activation time, and about **₹15,053 acquisition cost per activated customer** (total acquisition spend ÷ 368).

---

## 5. Project layout

```
udyam-digital/
├── data/                      hackathon CSVs, kyc_documents/, demo_applicants.json
├── server/                    Node.js + Express + Mongoose
│   ├── src/
│   │   ├── index.js           start-up
│   │   ├── app.js             routes, security middleware
│   │   ├── db.js              MongoDB connection (3 fallbacks)
│   │   ├── models/            User, Order, Kyc, Lead
│   │   ├── routes/            advisor, products, kyc, orders, dashboard, meta
│   │   ├── services/          dataService, catalogue, recommender, advisor, llm,
│   │   │                      gstin, ocr, kycDocs, orderService, leadScoring
│   │   └── middleware/        errors, validate, upload
│   ├── scripts/e2e.js         full journey + all sample applicants through the API
│   └── test/engine.test.js    rule-engine tests
├── client/                    React + Vite + Tailwind + React Router + Axios + Lucide
│   └── src/pages/             Landing, Advisor, Bundle, Catalogue, Checkout,
│                              Confirmation, Kyc, TrackLookup, Tracker, Partner
├── tools/                     make_demo_documents.py, ui_walkthrough.py
└── docs/                      PRIVACY_NOTE.md, screenshots/
```

## 6. API

| Method and path | Purpose |
| --- | --- |
| `POST /api/advisor/chat` | One turn of the conversation |
| `POST /api/advisor/recommend` | Recommendation from a profile |
| `GET /api/products`, `GET /api/products/:id` | Catalogue (with eligibility when a profile is passed) |
| `POST /api/products/quote` | Price and validity of a set of items |
| `POST /api/orders`, `GET /api/orders/:orderId` | Place and read an order |
| `PATCH /api/orders/:orderId/status` | Book installation, or simulate the next stage (demo mode) |
| `POST /api/orders/:orderId/support` | Simulated help request |
| `POST /api/kyc/consent` | Record consent |
| `POST /api/kyc/gstin/verify`, `POST /api/kyc/udyam/verify` | Registry checks |
| `POST /api/kyc/upload` | Document upload and OCR |
| `GET /api/kyc/:orderId/status` | KYC progress |
| `GET /api/dashboard/summary`, `GET /api/dashboard/leads`, `GET /api/dashboard/leads/:id`, `PATCH /api/dashboard/leads/:id` | Partner dashboard (needs the `x-partner-code` header) |
| `GET /api/meta`, `GET /api/lookup/gstin/:gstin`, `GET /api/demo/applicants`, `GET /api/demo/documents/:file` | Helpers for the UI and the demo |

Responses look like `{ "ok": true, "data": ... }` or `{ "ok": false, "error": { "message": "...", "details": [...] } }`.

## 7. Checks you can run

```bash
npm test        # rule-engine tests, no database needed
npm run e2e     # with the server running: full journey + every sample applicant
```

---

## 8. Honest notes

- **Team-made demo documents.** The data pack's sample documents (K01 to K12) include no pharmacy, so `tools/make_demo_documents.py` creates four specimen images (`D01_*.png`) for Evergreen Medicals, lead L02338. They contain only values from the mock registries, plus a made-up bill date, consumer number and ID number, and are watermarked as demo specimens.
- **A seventh advisor question (city)** was added, because product eligibility depends on city tier.
- **Order access** uses the order ID alone, and the partner dashboard uses a shared code. Both are demo-grade, not real authentication.
- **Tested with:** the API end-to-end script, the rule tests, and an automated browser walkthrough at 1366 px and 390 px wide. During development the database was a MongoDB-compatible stand-in; the built-in MongoDB download and the LLM option could not be exercised in the build environment, so try both once before demo day.
- **Hindi** covers the advisor fully and the main customer screens. Product names, some KYC detail text and the partner dashboard are in English.
- Changing the scoring rules re-scores the leads at the next start and resets follow-up statuses.
