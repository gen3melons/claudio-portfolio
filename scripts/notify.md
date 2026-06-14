# WhatsApp Alerts via CallMeBot

Claudio sends one WhatsApp message **per transaction**. CallMeBot is a free personal WhatsApp relay.

## One-time setup (user does this once)
1. Add the CallMeBot number **+34 621 34 34 03** to your phone contacts. (CallMeBot rotates this number periodically — if WhatsApp says it's invalid, check the latest at the docs link below.)
2. From your WhatsApp, send the message: **`I allow callmebot to send me messages`** to that number.
3. You'll receive a reply with your personal **API key** (e.g. `123456`).
4. Note your phone in international format **without `+`** (e.g. `54911...` for Argentina, or your actual number).

Docs: https://www.callmebot.com/blog/free-api-whatsapp-messages/

## Secret handling
**Do NOT commit your phone/API key to this repo.** They are passed to the scheduled routine's prompt/config at setup time. The repo stays public so GitHub Pages is free.

- `PHONE` = your number, no `+`
- `APIKEY` = the key CallMeBot gave you

## Send call (what Claudio runs per trade)
URL-encode the `text`. Then GET:
```
https://api.callmebot.com/whatsapp.php?phone=<PHONE>&text=<URLENCODED_TEXT>&apikey=<APIKEY>
```

Example with curl:
```bash
curl -G "https://api.callmebot.com/whatsapp.php" \
  --data-urlencode "phone=$PHONE" \
  --data-urlencode "apikey=$APIKEY" \
  --data-urlencode "text=🤖 Claudio BUY 0.5 BTC @ \$64000 (\$32000) | Risk-on; core crypto allocation | NAV \$100000"
```

## Message template
```
🤖 Claudio <BUY|SELL> <qty> <SYMBOL> @ $<price> ($<value>) | <reasoning> | NAV $<nav>
```

## Optional: daily no-trade digest
If enabled, on days with no trades send one line:
```
📊 Claudio daily: no trades. Regime <regime>. NAV $<nav> (<+/-x%> since inception).
```
