# Elagi API Testing 💊

[![API Tests](https://github.com/asmaaebrahim18/elagi-api-testing/actions/workflows/api-tests.yml/badge.svg)](https://github.com/asmaaebrahim18/elagi-api-testing/actions/workflows/api-tests.yml)
![Postman](https://img.shields.io/badge/Postman-FF6C37?logo=postman&logoColor=white)
![Newman](https://img.shields.io/badge/Newman-CLI-orange)
![JavaScript](https://img.shields.io/badge/Tests-JavaScript%20%2F%20Chai-F7DF1E?logo=javascript&logoColor=black)
![License](https://img.shields.io/badge/license-MIT-blue)

End-to-end API test automation for **Elagi (علاجي)**, an online healthcare and pharmacy web app (Laravel REST API).
Built with **Postman** and run from the command line and CI with **Newman**.

> Graduation project — Software Testing course, Semi Corner (2026).

---

## 📊 At a glance

| | |
|---|---|
| **Requests** | 239 (33 end-to-end + 206 validation, negative, boundary and security) |
| **Assertions** | 1,195 per full run |
| **Features covered** | 15 (auth, profile, products, wishlist, cart, checkout, feedback…) |
| **Bugs found** | 35 — automated as `BUG-xx` tests ([bug report](docs/bug-report.md)) |
| **Hard-coded test data** | None — every run creates its own user and data |

**Latest full run:** all functional tests passed. The 62 failing assertions all belong to the 32 `BUG-xx` tests, which are **expected to fail** until the bugs are fixed. 2 further failures were response-time spikes (2.9 s against a 2 s limit) on the shared host.

---

## 🗂️ Project structure

```
elagi-api-testing/
├── collection/
│   └── Elagi-API-Testing.postman_collection.json   # the test suite
├── environment/
│   └── Elagi.postman_environment.json              # base_url
├── docs/
│   ├── test-strategy.md     # scope, techniques, naming, data strategy
│   ├── api-endpoints.md     # real routes vs. the project brief
│   └── bug-report.md        # all findings with severity
├── reports/                 # Newman HTML / JUnit reports (generated)
├── .github/workflows/
│   └── api-tests.yml        # runs the tests on every push
└── package.json             # npm scripts for Newman
```

## 🧪 What is tested

### Main Flow — chained end-to-end scenario

```
Register → Verify OTP → Login → Get profile → Forgot password → Reset password
→ Old password refused → Login with new password → Update profile
→ Browse page 3 → Category → Product details → Sort by price → Rare-medicine request
→ Wishlist add ×2 / get / remove → Cart add ×2 → Update quantity → Remove item
→ Checkout → Order history → Feedback → Contact us
```

Each request passes its token, ids and prices to the next one through variables.

### Validation & Negative Scenarios

| Feature | Requests | Examples |
|---|---|---|
| Registration & OTP | 34 | missing / invalid fields, Gmail-only rule, duplicate email, OTP reuse |
| Login & logout | 13 | wrong password, unverified user, token revoked after logout |
| Profile & password reset | 33 | boundary lengths, mass assignment (`is_admin`), old password refused |
| Products & categories | 18 | pagination edges (page 0, −1, last+1), sort across pages, unknown id |
| Rare medicine & wishlist | 28 | quantity limits, duplicates (409), another user's items (IDOR) |
| Cart, checkout & orders | 49 | quantity rules, cart-item vs product id, price tampering, totals |
| Feedback & contact us | 31 | rating 1–5 boundaries, max lengths, script tags |

### Checks on every request

✅ Status code ✅ Response time ✅ JSON content type ✅ JSON schema ✅ Returned values match the sent data ✅ Business rules ✅ No password or sensitive data leaked

More detail: [docs/test-strategy.md](docs/test-strategy.md)

---

## 🚀 How to run

### Option 1 — Postman

1. Postman → **Import** → select both files from `collection/` and `environment/`.
2. Select the **Elagi** environment.
3. Right-click the collection → **Run collection**.
   - Run **Main Flow** first (it creates the user that later tests use), or run the whole collection.

### Option 2 — Command line (Newman)

Requires [Node.js](https://nodejs.org/) 18+.

```bash
git clone https://github.com/asmaaebrahim18/elagi-api-testing.git
cd elagi-api-testing
npm install

npm run test:main   # Main Flow only (~35 s)
npm run test:all    # full regression (~10 min, includes known-bug tests)
```

Reports are saved to `reports/` (open the `.html` file in a browser).

### Option 3 — GitHub Actions

- Every push to `main` runs the **Main Flow**.
- **Actions → API Tests → Run workflow → full-regression** runs everything.
- The HTML and JUnit reports are attached to each run as **artifacts**.

---

## 🐞 Key findings

| Severity | Finding |
|---|---|
| 🔴 High | Old login tokens keep working after a password reset |
| 🔴 High | Checkout accepts items that are not in the cart (even with an empty cart) |
| 🔴 High | No stock check — 9 products have negative stock |
| 🔴 High | Failed card payment still creates the order and empties the cart |
| 🔴 High | Script tags saved as-is in profile name and feedback (stored XSS risk) |
| 🟠 Medium | Several inputs crash the server (500 + SQL error) instead of returning 422 |

Full list (35 findings): [docs/bug-report.md](docs/bug-report.md)

> 3 critical access-control findings were reported privately and are not published here.

---

## 🛠️ Skills demonstrated

`REST API testing` · `Postman` · `Newman CLI` · `JavaScript / Chai assertions` · `JSON Schema validation` · `Request chaining` · `Dynamic test data` · `Pre-request scripts` · `Boundary value analysis` · `Equivalence partitioning` · `Negative & security testing (IDOR, XSS, mass assignment)` · `Rate-limit handling` · `CI with GitHub Actions` · `Bug reporting`

---

## 👩‍💻 Author

**Asmaa Ebrahim** — QA Engineer

[LinkedIn](https://www.linkedin.com/in/asmaaebrahim98/) · [GitHub](https://github.com/asmaaebrahim18)

Instructor: Galal Torkey — Semi Corner
