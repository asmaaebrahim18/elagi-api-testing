# Bug Report — Summary

35 issues were found and automated as `BUG-xx` tests. Each test asserts the **expected** behaviour, so it fails while the bug exists and will pass once it is fixed.

> **Responsible disclosure:** 3 access-control findings (BUG-32, BUG-33, BUG-34) expose other users' personal data. They were reported privately. Their details and their tests are **not** included in this public repository.

## By severity

| Severity | Count |
|---|---|
| 🔴 Critical / High | 10 |
| 🟠 Medium | 18 |
| 🟡 Low | 7 |

## Security

| ID | Finding | Expected | Actual | Severity |
|---|---|---|---|---|
| BUG-32/33/34 | Access-control issues on listing endpoints | Data available to authorised users only | — *withheld* — | 🔴 Critical |
| BUG-05 | Old token still works after a password reset | All sessions revoked after reset | Old token → 200 | 🔴 High |
| BUG-09 | Update profile saves `<script>` in the name | 422 / sanitised | Saved as-is (stored XSS risk) | 🔴 High |
| BUG-28 | Feedback saves `<script>` tags | Rejected or sanitised | Saved as-is, shown on homepage after approval | 🔴 High |
| BUG-04 | Errors expose a full stack trace | Generic error message | Debug mode on: server paths, SQL details | 🟠 Medium |
| BUG-27 | Feedback can be posted under any email | Uses the logged-in user | Uses name/email from the body | 🟠 Medium |

## Business logic

| ID | Finding | Expected | Actual | Severity |
|---|---|---|---|---|
| BUG-16 | Product stock is negative (e.g. −1383) | Stock ≥ 0 | 9 products below 0 | 🔴 High |
| BUG-19 | Cart accepts more than the stock | 422 | Added (and can be ordered) | 🔴 High |
| BUG-23 | Checkout with an **empty** cart creates an order | Only cart items accepted | Any `cartItems` in the body are ordered | 🔴 High |
| BUG-25 | Failed card payment still creates the order | No order, cart kept | 500, but order saved and cart emptied | 🔴 High |
| BUG-24 | Ordering some items empties the whole cart | Remaining items kept | All items removed | 🟠 Medium |
| BUG-18 | Cart quantity above 3 accepted (site limit is 3) | 422 | 4 and 100000 accepted | 🟠 Medium |
| BUG-20 | Cart quantity can be updated to 0 / negative | 422 | 200, item stays with qty 0 / −1 | 🟠 Medium |
| BUG-11/12 | Rare medicine quantity 0 or 4 accepted (site says 1–3) | 422 | 200 | 🟠 Medium |
| BUG-10 / BUG-35 | Duplicate phone accepted (update profile / register) | 400 / 422 | Accepted | 🟠 Medium |

## Validation & error handling

| ID | Finding | Expected | Actual | Severity |
|---|---|---|---|---|
| BUG-13 | Rare medicine name > 255 chars | 422 | 500 with SQL error | 🟠 Medium |
| BUG-21/22 | Cart update with text / no quantity | 422 | 500 with SQL error | 🟠 Medium |
| BUG-29/30 | Very long feedback / contact message | 422 | 500 with SQL error | 🟠 Medium |
| BUG-06/07/08 | Update profile skips registration rules (phone with letters, 10 digits, name with digits) | 422 | 200 | 🟠 Medium |
| BUG-26 | Checkout address > 255 chars | 422 | 500 | 🟡 Low |
| BUG-14 | Rare medicine accepts phone `abc123` | 422 | 200 | 🟡 Low |
| BUG-31 | Contact-us errors use a different format | 422 `{message, errors}` | 400 with fields at top level | 🟡 Low |

## Usability / data

| ID | Finding | Expected | Actual | Severity |
|---|---|---|---|---|
| BUG-01 | Arabic full name rejected at registration (Arabic store) | Accepted | 400 "letters and spaces only" | 🟠 Medium |
| BUG-02 | Landline `020…` accepted as a mobile number | 400 | Accepted (only length checked) | 🟡 Low |
| BUG-03 | `@GMAIL.COM` rejected | Case-insensitive | 400 | 🟡 Low |
| BUG-15 | Removing a product not in the wishlist | 404 | 200 "removed" | 🟡 Low |
| BUG-17 | Pagination links drop the `sort` parameter | Sort kept | Next/prev links unsorted | 🟡 Low |

## Other observations (not automated)

- Login returns translation keys instead of text: `messages.login_success`, `messages.no_account`.
- User enumeration: login returns 404 vs 401, and forgot-password says "User does not exist".
- `GET /user` returns internal fields (`otp`, `otp_code`, `is_admin`).
- `email_verified_at` stays `null` after successful OTP verification.
- Register returns 400 with custom messages; other endpoints return 422 with Laravel defaults.
- Several routes in the project brief do not exist (see [api-endpoints.md](api-endpoints.md)).
