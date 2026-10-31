# SADEE PRO STORE — Gaming Store Update

Premium dark gaming storefront with neon-red styling, responsive product cards, wishlist, stock visibility, customer order lookup, eZ Cash manual verification, license delivery, promo codes, announcements, reviews moderation, website customization, and a separate admin console.

## Included in this build

- Gaming-style black / neon-red storefront with responsive layout and reduced-motion support.
- Product search, category filters, plans, product detail modal, stock visibility, and device-local wishlist.
- Customer order submission using a server-side function; the displayed order code is used with the customer's email in **My Orders**.
- eZ Cash manual verification: the admin must check the actual transaction before verifying. The admin enters the exact license key after verification; no key is auto-generated.
- License key appears in My Orders only after payment verification and after the admin saves the supplied key; email delivery is attempted using Resend when configured.
- Admin product add/edit/delete and product-image upload through a secret-protected Netlify Function.
- Admin overview cards for products, low stock, orders and verified sales (orders/sales cards are based on the most recent 100 orders returned by the function).
- Site customizer for logo image, hero image, hero video and accent color.
- Promo code creation and checkout discount validation.
- Announcements/maintenance notices and customer review submission with admin approval.
- PWA manifest and basic offline shell caching.

## Before deploying

**This ZIP has not been connected to your live Supabase or Netlify account, and has not been deployed.** Do not assume that payments, email, database policies or the live site have been verified. Run the setup below and test with a small test order before announcing it to customers.

### 1. Supabase

1. Open the Supabase SQL Editor for the project configured in `config.js`.
2. Run `supabase/schema.sql`.
3. Run `supabase/orders-schema.sql`.
4. Confirm the `product-images` storage bucket exists and is public-read only. The SQL removes anonymous upload/update/delete policies; uploads are done server-side.
5. Check that your products table uses the fields defined in `schema.sql` and edit the sample listing before publishing.

### 2. Netlify environment variables

Add these under Netlify site settings → Environment variables. Never add the service-role key or admin secret to `config.js` or commit them to GitHub.

- `SUPABASE_URL` — your Supabase project URL.
- `SUPABASE_SERVICE_ROLE_KEY` — the Supabase service-role key; server-side only.
- `STORE_ADMIN_SECRET` — a long, random private value. You enter it in the admin prompt when performing protected actions.
- `RESEND_API_KEY` — optional; required for automatic email delivery.
- `STORE_FROM_EMAIL` — optional; a sender address verified with Resend.

Redeploy after adding or changing environment variables. Functions require Node.js support; the included `netlify.toml` sets the functions directory to `netlify/functions`.

### 3. Store configuration

Edit `config.js` and set the correct public Supabase URL and publishable/anon key, the official `EZCASH_NUMBER`, and your WhatsApp number. The eZ Cash number is intentionally blank until the store owner provides the real number. Do not invent it. `SUPPORT_EMAIL` is a placeholder and should be replaced with a real support email or left unused.

### 4. eZ Cash and licenses

1. Customer submits an order code, email and payment reference.
2. Admin checks the actual incoming eZ Cash transaction outside the website.
3. Admin first verifies the actual payment. After verification, the admin enters the exact customer license key in the Orders panel and saves it; only then is the key shown in My Orders. The key is not auto-generated.
4. The customer uses **My Orders** with the order code and matching email to see the status and key. Email is sent only when Resend settings are valid.

A screenshot or payment reference alone is not proof of payment. Do not verify until the transaction is confirmed in the official account/merchant records.

### Netlify deployment (no GitHub Pages required)

This archive is arranged with `index.html` at the archive root and `netlify/functions/` at the root. Deploy it to the Netlify site using the Git-connected repository or Netlify manual deploy after extracting the ZIP.

1. Extract this ZIP. Upload the **contents** of the extracted `SADEE PRO STORE` folder to the GitHub repository root, so `index.html`, `config.js`, `netlify.toml`, `admin/`, and `netlify/functions/` are directly visible at the repository root.
2. In Netlify, open **Site configuration → Build & deploy → Continuous deployment** and connect the repository. Set the base directory to blank/root, build command to blank, and publish directory to `.`. The included `netlify.toml` also declares these settings.
3. Do not use GitHub Pages for this site. Netlify is the hosting provider; GitHub is only the source-code repository.
4. Add the required Netlify environment variables listed above, then trigger a fresh deploy.
5. After deploy, test `https://YOUR-SITE.netlify.app/.netlify/functions/validate-promo` in a browser. A JSON `Method not allowed` response is expected for GET and confirms the function route is deployed. An HTML 404 means the wrong deploy source or publish configuration is still active.
6. Do not use a plain static drag-and-drop deploy if you need the server-side checkout/admin functions. Use a Netlify deploy that includes the `netlify/functions/` directory.

### 5. Promo codes

Promo codes are validated server-side when an order is submitted. Discounts are applied to the order amount. This is a manual-payment checkout, not an automatic eZ Cash gateway. Verify the final amount received matches the discounted amount before issuing a key.

### 6. Testing checklist

- Test product add/edit/delete with the admin secret.
- Test image upload with a small JPG/PNG/WebP/GIF (maximum about 3 MB).
- Test promo code with valid, expired/invalid and repeated-code cases.
- Submit a test order, confirm it stays pending, then test reject and verify flows.
- Confirm stock decrements only after successful verification; restock products before verifying if stock is zero.
- Confirm My Orders does not reveal a key for pending/rejected orders.
- Configure Resend and test email delivery before using it for real orders.
- Test phone layout and PWA install on the deployed HTTPS domain.

## Security notes

- Product, settings, image upload and order-review mutations are routed through server-side Netlify Functions using `STORE_ADMIN_SECRET` and `SUPABASE_SERVICE_ROLE_KEY`.
- Never put `SUPABASE_SERVICE_ROLE_KEY`, `STORE_ADMIN_SECRET` or Resend secrets in browser code.
- The order lookup currently relies on the order code plus matching email. Add rate limiting/CAPTCHA at the edge before a high-traffic public launch.
- Review submissions are hidden until approved by an admin.
- This project uses manual eZ Cash verification; it does not automatically query eZ Cash transaction records.
- A promo code is applied to the order total; check the amount received before verifying payment.
