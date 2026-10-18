# Xstore Telegram Mini App — Render deployment

## What is included
- Telegram-like responsive Mini App at `/`
- Telegram Web App `initData` HMAC verification on the server (no trusting `initDataUnsafe` for auth)
- Product/category search, profile, wallet balance, recent orders and wallet transactions
- Existing bot commands/admin panel/localdb.js/ZapUPI flow retained
- Product purchase and wallet top-up buttons hand off to the existing Telegram bot checkout. This intentionally avoids creating a second payment verification path; ZapUPI verification and delivery remain in the existing bot.

## 1. Upload / update your GitHub repository
Upload the files in this ZIP to the same repository as your current bot. Keep `bot.js`, `adminPanel.js`, `fampay.js`, and `localdb.js` together. Commit and push the changes.

## 2. Render settings
Use a **Web Service** with:
- Build command: `npm install`
- Start command: `npm start`
- Runtime: Node.js 18 or newer

Add these Environment Variables in Render (never put secrets in the frontend):
- `BOT_TOKEN` — your existing Telegram bot token
- `BOT_USERNAME` — bot username without `@`
- `MINI_APP_URL` — exact public HTTPS URL of this Render service, e.g. `https://xstore-bot.onrender.com`
- Keep any existing ZapUPI/FamPay environment variables you already use. Do not replace working values blindly.

Redeploy after setting variables. Visit the service URL; the Mini App should open. `/api/miniapp/bootstrap` will return 401 in a normal browser because it requires Telegram's signed `initData`; that is expected.

## 3. Open Mini App in Telegram
The bot's main menu adds **🚀 Open Xstore Mini App** when `MINI_APP_URL` is set. Send `/start` to the bot again and tap that button. No BotFather menu setup is required for this in-chat button. You can optionally configure a Main Mini App in BotFather later, using the same HTTPS URL.

## 4. Important Render storage warning
`localdb.js` stores JSON in `./data`. Render's ordinary filesystem is ephemeral, so a redeploy/restart can lose users, products, orders, and wallet balances. For production, attach a Render Persistent Disk mounted at the app's expected data location or migrate the JSON database to durable storage. Do not run multiple instances against the same JSON files. Back up data before redeploying.

## 5. Payment behavior
The Mini App displays wallet/orders and product listings. The Buy/Get product and Add money buttons open the existing bot, which runs the current order, ZapUPI payment, verification, and product delivery logic. Do not mark an order paid from frontend JavaScript or from a redirect page; only the server-side verified payment flow should do that.

## 6. Quick troubleshooting
- Button not shown: confirm `MINI_APP_URL` is set and redeploy.
- Mini App says verification failed: open it from the bot, not from a normal browser; confirm `BOT_TOKEN` is the same token used by this bot.
- Bot won't start: check Render logs and confirm `BOT_TOKEN` is valid.
- Product/order data disappears after deploy: configure persistent storage as above.
- Payments fail: check the existing ZapUPI settings and logs; Mini App does not bypass the existing gateway.
