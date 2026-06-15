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

## Character rules (IMPORTANT)
CallMeBot **rejects messages with emojis and some symbols** (`$`, `%`, etc.) with
"The message has invalid charecters." Keep alert text **plain ASCII**:
- write amounts as `100002 USD` (not `$100,002`)
- write changes as `+1.5 pct` (not `+1.5%`)
- no emojis, no `|` is fine, no curly quotes.

## Send call (what Claudio runs per trade)
URL-encode the `text`. Then GET:
```
https://api.callmebot.com/whatsapp.php?phone=<PHONE>&text=<URLENCODED_TEXT>&apikey=<APIKEY>
```

Example with curl:
```bash
curl -s -G "https://api.callmebot.com/whatsapp.php" \
  --data-urlencode "phone=$PHONE" \
  --data-urlencode "apikey=$APIKEY" \
  --data-urlencode "text=Claudio BUY 0.5 BTC at 64000 USD (32000 USD) - risk-on, core crypto allocation - NAV 100000 USD"
```
A success response contains "Message queued".

## Message template (ASCII only)

Multi-line message, one per trade. Replace all `$` with nothing (just the number + USD) and all `%` with `pct`. Use `->` for arrows. Newlines are fine via `--data-urlencode`.

```
Trade executed:
* <BUY|SELL> <qty> <SYMBOL> at <price> USD -> <value> USD deployed
* Reasoning: <one-line reasoning>
* New <SYMBOL>: <new_total_qty> <SYMBOL> at avg cost <avg_cost> USD

Portfolio after trade:
* NAV: <nav> USD (<+/->nav_change USD / <+/->nav_pct pct from yesterday <prev_nav> USD)
* Cash: <cash> USD (<cash_pct> pct) - <cash_note>
* Crypto sleeve: <sleeve_pct> pct of NAV (under <sleeve_limit> pct <regime>-cap)

Next catalyst: <catalyst>
```

For SELL: replace the "New <SYMBOL>" line with "Remaining <SYMBOL>: <qty> <SYMBOL> (fully exited)" or the remaining qty + realized P&L.

## Optional: daily no-trade digest
If enabled, on days with no trades send one line:
```
Claudio daily: no trades. Regime <regime>. NAV <nav> USD (<+/-x> pct since inception).
```
