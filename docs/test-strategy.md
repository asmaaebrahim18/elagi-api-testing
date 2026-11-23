# Test Strategy

## Scope

Functional, negative, boundary and basic security testing of the Elagi REST API: authentication, profile, products and categories, wishlist, rare-medicine requests, cart, checkout, orders, feedback and contact us.

**Out of scope:** UI testing, load/performance testing (only response time is checked), payment-gateway internals.

## Collection design

```
Elagi API Testing
├── Main Flow                           ← end-to-end, chained, runs first
│   ├── 1. Authentication   (10)        register → OTP → login → profile → reset password → update
│   ├── 2. Categories       (10)        pagination → category → details → sort → rare medicine → wishlist
│   ├── 3. Purchase          (9)        cart add → update qty → remove → checkout → order history
│   └── 4. Homepage          (4)        feedback → contact us
└── Validation & Negative Scenarios     ← one folder per feature
    ├── Registration - Negative, OTP - Negative, Login - Negative, Profile
    ├── Password reset, Update profile, Logout
    ├── Products, Rare medicine, Wishlist
    ├── Cart, Update - remove cart item, Checkout & orders
    └── Feedback, Contact us
```

Every feature folder ends with its `BUG-xx` tests (see [bug-report.md](bug-report.md)).

## Test design techniques

| Technique | Example |
|---|---|
| Equivalence partitioning | valid / invalid / missing email, phone, password |
| Boundary value analysis | password 7 ✗ / 8 ✓ chars · name 255 ✓ / 256 ✗ · rating 0 ✗, 1 ✓, 5 ✓, 6 ✗ · page 0, -1, last, last+1 |
| State transition | old password refused after reset · token refused after logout · OTP refused after reuse |
| Business rules | prices sorted across pages · cart line total = price × qty · order total = sum of items · stock decreases after checkout |
| Authorization / IDOR | another user's cart item or wishlist item cannot be changed · no token / invalid token → 401 |
| Input security | script tags, mass-assignment fields (`is_admin`, `email`, `id`) in the body, very long strings |

## Test case naming

| Prefix | Meaning |
|---|---|
| `REG` `OTP` `LOG` `PRO` `RST` `UPD` `LOGOUT` | Authentication & profile |
| `MED` `SORT` `DET` `CAT` `RARE` `WISH` | Products & features |
| `CART` `UPDC` `DELC` `CHK` `ORD` | Purchase |
| `FB` `CON` | Homepage |
| `-H` happy · `-N` negative · `-B` business rule · `-BV` boundary value · `-S` security | |
| `BUG-xx` | Asserts the **expected** behaviour → fails until the bug is fixed |

## What each request checks

1. Status code
2. Response time (< 2 s)
3. `Content-Type` is JSON
4. JSON schema (types, required fields, formats)
5. Values match what was sent (email, name, quantity, product id…)
6. Business rules (sorting, pagination, totals)
7. No sensitive data leaked (password not in the response)
8. Saves the values the next request needs (token, ids, prices)

## Dynamic test data

No hard-coded data is passed between requests:

- **Unique user per run:** `elagi.qa.<timestamp>@gmail.com` + random Egyptian mobile number.
- **Password** generated once per run at collection level.
- **Products** found by name from the API response (CeraVe, Dermovate, Elica-m), not by fixed ids.
- **Wishlist products** picked at random from page 3.
- Random address, rare medicine, quantity, rating and feedback text.

Because every run creates a fresh user, the collection can be run any number of times.

## Rate-limit handling

The API allows 60 requests per minute.

- A **collection-level** script reads `X-RateLimit-Remaining` and `Retry-After`, and pauses before the next request when the limit is almost used, or after a 429.
- The **Validation & Negative Scenarios** folder keeps a ~1 s gap between requests because its tests make extra setup calls.

The Main Flow runs at full speed. The full suite takes about 9–10 minutes.

## Entry / exit criteria

- **Entry:** API reachable; base URL configured.
- **Exit:** all Main Flow tests pass; every failure in the full suite is a known `BUG-xx` test.
