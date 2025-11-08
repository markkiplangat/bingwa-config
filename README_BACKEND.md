Bingwa STK Backend — Quick Ops & Playbook
========================================

Purpose
-------
This backend receives STK Push requests from the app, calls Safaricom Daraja APIs, and writes transaction & balance docs to Firestore (admin). It also exposes read-only endpoints the app uses.

Key Files
---------
- server.js               - main Express server
- firebase.js             - Firebase admin initialization (keep service account private)
- balanceController.js    - applyPurchase / updateBalance helpers
- package.json            - dependencies
- config.json (public)    - (optional) frontend config list (see client)

Environment (Render)
--------------------
Required env vars (set in Render / hosting environment):
- CONSUMER_KEY
- CONSUMER_SECRET
- BUSINESS_SHORTCODE
- PASSKEY
- CALLBACK_URL        (e.g. https://bingwa-stk-backend-v2.onrender.com/stkpush/callback)
- CALLBACK_SECRET     (random strong secret for callback validation)
- DARAJA_BASE_URL     (default: https://api.safaricom.co.ke)

Security Notes
--------------
- Never embed Daraja keys or CALLBACK_SECRET in the Flutter app.
- Do NOT store service account key or credentials in a public repo.
- Use Callback secret validation (header or ?secret) to filter callbacks.
- Optional: IP whitelist check against Safaricom IP ranges.

Common Tasks
------------
1. To change primary backend URL used by apps without an app update:
   - Edit `config.json` in the public repo and update backendUrls priority.
   - Commit & push. Apps that fetch config will adopt the new primary URL.

2. To rotate CALLLBACK_SECRET:
   - Generate new secret.
   - Set env var on Render and update `CALLBACK_URL` query param if necessary.
   - If you must switch secret immediately, update apps fetchable config for emergency.

Troubleshooting
---------------
- "Element at index 0 is not a valid array element" => Use `admin.firestore.Timestamp.now()` for array elements. (See commit.)
- "X-Forwarded-For header is set but trust proxy is false" => ensure `app.set('trust proxy', true)` for hosts behind proxies.

Endpoints
---------
- POST /stkpush
- POST /stkpush/callback
- GET  /api/get-balance?deviceId=...
- GET  /api/transactions?deviceId=...
- GET  /api/tx-status?checkoutRequestId=...

Deploying
---------
- Push to GitHub; Render configured to auto-deploy.
- Ensure environment variables set in Render dashboard.
- Use UptimeRobot to keep free Render instance awake if you want (free hack).

Support workflow (when a user reports a missing tokens)
-------------------------------------------------------
1. Ask for deviceId and CheckoutRequestID if available.
2. Search Render logs for callback events around the timestamp.
3. Use `/api/tx-status?checkoutRequestId=...` to find the transaction doc.
4. If transaction present: verify `status`, `mpesaReceipt`, `amountPaid`.
5. If callback found but tokens not applied: check `applyPurchase` logs and `token_balances` doc.
6. Issue refund or manually apply `update-balance` if required.

