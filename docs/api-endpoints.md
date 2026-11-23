# Elagi API — Endpoint Reference

**Base URL:** `https://elagi.semicorner.com/api`

The project brief listed several routes that do not exist on the server. The real routes below were found from the API documentation, the website's network traffic (DevTools → Network → Fetch/XHR) and the front-end bundle. Every route was then confirmed with requests.

## Routes: requirements vs. reality

| Feature | Route in the brief | Real route | Auth |
|---|---|---|---|
| Register | `POST /register` | `POST /register` | — |
| Verify email | `POST /verify-email` | **`POST /verify-otp`** `{email, otp}` | — |
| Login | `POST /login` | `POST /login` | — |
| Reset request | `POST /password-reset/request` | **`POST /forgot-password`** `{email}` | — |
| Reset confirm | `POST /password-reset/confirm` | **`POST /reset-password`** `{email, otp, newPassword}` | — |
| Get profile | `GET /user/profile` | **`GET /user`** | Bearer |
| Update profile | `POST /user/profile` | **`POST /user/update`** `{name, phone}` | Bearer |
| Logout | — | `POST /logout` | Bearer |
| Products (paged) | — | `GET /medicines?page=N&sort=price_asc\|price_desc` | — |
| Category | — | `GET /medicines/category/{name}?page=N` | — |
| Product details | — | `GET /medicines/{id}` | — |
| Rare medicine | `POST /features/rare medicine` | **`POST /store-rare-medicine`** | Bearer |
| Wishlist add | `POST /Wishlist` | **`POST /favorites/add/{productId}`** | Bearer |
| Wishlist get | `GET /Wishlist` | **`GET /favorites`** | Bearer |
| Wishlist remove | `DELETE /Wishlist` | **`DELETE /favorites/remove/{productId}`** | Bearer |
| Cart | — | `POST /cart/add` · `GET /cart` · `PATCH /cart/{cartItemId}` · `DELETE /cart/{cartItemId}` | Bearer |
| Checkout | — | `POST /checkout` `{address, paymentMethod, cartItems[]}` | Bearer |
| Order history | `GET /user/profile/orders` | **`GET /orders`** | Bearer |
| Feedback | — | `GET /feedback` (public) · `POST /feedback` (Bearer) | mixed |
| Contact us | — | `POST /contact/save` `{name, email, message}` | — |

> Cart update/delete take the **cart item id**, not the product id.

## Response formats

| Situation | Status | Shape |
|---|---|---|
| Success | 200 / 201 | `{ "message": "..." }` (+ `token` on login, `user` on update) |
| Register validation | 400 | `{ "errors": { "<field>": ["..."] } }` — custom messages |
| Other validation | 422 | `{ "message": "...", "errors": { ... } }` — Laravel defaults |
| Contact-us validation | 400 | fields at top level, no wrapper |
| Not logged in | 401 | `{ "message": "Unauthenticated." }` |
| Lists | 200 | Laravel paginator: `current_page, data[], from, to, last_page, per_page, total, next_page_url, prev_page_url` |

## Rate limit

60 requests / minute (`X-RateLimit-Limit: 60`), returns **429** when exceeded. The collection handles this in code (see [test-strategy.md](test-strategy.md#rate-limit-handling)), so no manual delay is needed in the Runner.
