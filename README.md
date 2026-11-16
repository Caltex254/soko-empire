# Soko Empire

A premium, mobile-first Kenyan thrift-fashion storefront and separate admin area. Built with Express + SQLite + vanilla HTML/CSS/JS to keep the storefront lightweight and straightforward to deploy.

## Included
- Responsive luxury storefront, category browsing, search, product filters, product detail modal, cart, checkout and WhatsApp links.
- Seed catalogue with sample Kenyan thrift pricing. Replace demo photos and product descriptions with your actual inventory before launch.
- Admin login at `/admin`; overview, product CRUD and multi-photo uploads, order management/status changes, delivery-note printing, customer list, promo code creation, shop settings and CSV export.
- SQLite persistence, password hashing, signed admin tokens, rate limiting, Helmet security headers and server-side stock validation.
- M-Pesa STK endpoint in **simulation mode** by default.

## Run locally
1. Install Node.js 20+.
2. Copy `.env.example` to `.env`.
3. Set a unique `JWT_SECRET`, `ADMIN_EMAIL`, and a strong `ADMIN_PASSWORD`.
4. Run `npm install` then `npm start`.
5. Open `http://localhost:3000`; admin: `http://localhost:3000/admin`.

The first boot creates the admin account using `ADMIN_EMAIL` and `ADMIN_PASSWORD`. Changing these environment variables after the first boot does not automatically reset the existing admin password. Change it through a secure password-reset procedure before production (or remove the admin row only while intentionally resetting the installation).

## Deployment notes
- Deploy the Node service to a host that supports persistent disk storage. Set `DB_PATH` to a persistent volume path and mount persistent storage for `uploads/`. Ephemeral serverless filesystems are not suitable for SQLite and product uploads.
- Set `BASE_URL`, `JWT_SECRET`, admin credentials, WhatsApp number, M-Pesa Till/Paybill display value and delivery settings in the host environment.
- Put the site behind HTTPS. Restrict CORS to your actual domain if the frontend/API are split across origins.
- Back up the SQLite database and `uploads/` directory regularly.

## M-Pesa: do not enable live collection yet
`POST /api/payments/mpesa/stk` intentionally returns a simulation response unless `MPESA_MODE=live`. The live branch currently returns HTTP 501. This is deliberate: real Daraja STK Push requires valid Safaricom Daraja app credentials, a production shortcode/passkey, a publicly reachable HTTPS callback, callback signature/metadata validation, idempotent payment reconciliation and testing with the correct Daraja environment. Do not set `MPESA_MODE=live` until a tested Daraja adapter has been implemented. The simulation never sends an STK request and never collects money.

## Before public launch
- Replace all demo inventory, images, testimonials and social handles with genuine store content.
- Add an explicit returns/refunds policy and complete delivery price matrix by county and courier.
- Implement and test real Daraja STK Push + callback verification; never mark an order paid based only on the browser response.
- Add admin password reset/MFA, audit logs, CSRF protections if cookie auth is introduced, backup/restore testing and production monitoring.
- Review consumer protection, privacy/data protection and tax obligations applicable to your Kenyan business.

## API overview
Public: `GET /api/products`, `GET /api/products/:id`, `GET /api/settings`, `POST /api/orders`, `POST /api/payments/mpesa/stk`.
Admin: `POST /api/admin/login`; authenticated `/api/admin/overview`, `/products`, `/orders`, `/customers`, `/coupons`, `/settings`, `/export/orders`.

## Deploying to Vercel
The repository includes `index.js` and `vercel.json` so the whole app runs as a single serverless Node function on Vercel. Import the repo in Vercel (no build command needed) and set the environment variables from `.env.example` in the project settings.

Serverless caveats: SQLite lives in the function's ephemeral `/tmp` (`/tmp/sokoempire.sqlite`) and photo uploads in `/tmp/uploads`. Data resets on cold starts and is not shared between function instances, and the request body limit is ~4.5 MB. That is fine for demos and previews; for real trading use a host with persistent disk (see Deployment notes above) or move the database and image storage to managed services.
