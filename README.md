# VrcLIraniCore

XHTTP relay برای V2Ray/Xray روی Vercel Edge Runtime با تمرکز روی:

- سرعت بالا و پینگ پایین
- کاهش مصرف منابع غیرضروری
- کاهش ریسک abuse با قفل‌کردن مسیر و احراز هویت

---

## Changelog v1.2

### What was added

- Upstream timeout control با ENV:
  - `UPSTREAM_TIMEOUT_MS`
  - مقدار پیش‌فرض نسخه 1.2: `20000`
- Relay authentication اختیاری:
  - `RELAY_KEY`
  - پذیرش از `x-relay-key` یا query `k`
- Method allowlist:
  - فقط `GET`, `HEAD`, `POST`
- Header filtering (allowlist) برای کم‌کردن هدرهای اضافه
- Header strip list سخت‌گیرانه‌تر برای هدرهای hop-by-hop/proxy
- Timeout/Error logging برای عیب‌یابی بهتر
- `RELAY_PATH` اختیاری:
  - اگر ست می‌شد فقط همان مسیر و subpathها عبور می‌کرد
  - اگر ست نمی‌شد همه مسیرها عبور می‌کردند
- بازگشت rewrite عمومی در `vercel.json`:
  - `/(.*) -> /api/index`

---

## Changelog v1.3 (Current Hardening)

### Security and stability hardening

- `RELAY_PATH` اجباری شد (از ENV خوانده می‌شود)
- `RELAY_KEY` اجباری شد و فقط از هدر `x-relay-key` پذیرفته می‌شود
- اگر `RELAY_KEY` ست نباشد یا کوتاه باشد (کمتر از 16 کاراکتر)، سرویس fail-closed می‌دهد (`500`)
- اگر `RELAY_PATH` ست نباشد یا `/` باشد، سرویس fail-closed می‌دهد (`500`)
- مقدار پیش‌فرض timeout برای جریان‌های طولانی XHTTP به `120000` افزایش یافت
- مسیر auth از query حذف شد (query token دیگر پذیرفته نمی‌شود)

---

## Deploy Guide (Vercel)

### 1) Import project

- ریپو را در Vercel import کن
- Branch: `main`

### 2) Configure Environment Variables

در `Project -> Settings -> Environment Variables` این‌ها را ست کن:

1. `TARGET_DOMAIN`
   - کار: مقصد واقعی upstream (سرور Xray/V2Ray)
   - فرمت: `https://host:port`
   - مثال:
     - `https://your-origin.example.com:2053`
     - `https://1.2.3.4:8443`

2. `RELAY_PATH` (Required)
   - کار: مسیر مجاز relay
   - نکته:
     - باید با `/` شروع شود
     - نباید `/` تنها باشد
   - مثال:
     - `/vc`
     - `/edge-vc-901`
     - `/r/ab92`
   - تنظیم کلاینت:
     - Path کلاینت باید دقیقا همین مقدار باشد

3. `RELAY_KEY` (Required)
   - کار: کلید احراز هویت برای هر درخواست
   - نحوه ارسال:
     - فقط از هدر `x-relay-key`
   - شرط:
     - حداقل 16 کاراکتر
   - توصیه:
     - 32 تا 64 کاراکتر، ترکیبی از حروف بزرگ/کوچک/عدد/نماد

4. `UPSTREAM_TIMEOUT_MS` (Optional)
   - کار: timeout برای اتصال به upstream
   - پیش‌فرض اگر نزنی:
     - `120000`
   - چی بزنیم؟
     - `90000` برای قطع سریع‌تر
     - `120000` حالت متعادل (پیشنهادی)
     - `180000` اگر اتصال‌های طولانی‌تر می‌خوای

### 3) Redeploy

- بعد از هر تغییر ENV حتما Redeploy بزن تا تغییر اعمال شود

---

## Client Configuration Notes

- `Path` در کلاینت = دقیقا `RELAY_PATH`
- `Host` = دامنه پروژه Vercel (یا دامنه شخصی)
- Request Header اضافه کن:
  - `x-relay-key: <RELAY_KEY>`
- اگر هدر نفرستی یا اشتباه باشد:
  - پاسخ `403` می‌گیری

---

## 5 Sample Random RELAY_KEY Values

این‌ها نمونه هستند؛ یکی را انتخاب کن یا مشابه‌شان تولید کن:

1. `nD7!vQ2@kL9#xP4$mT8^rF1&zH6*eJ3`
2. `A5q$W8m!R2z#N7y@C4u^K1p&L9d*X6s`
3. `tY3@bN8!hQ6#vM1$kR5^cP9&jL2*zF7`
4. `G2!sD9@pL4#xV7$mK1^qR8&nT5*zH3`
5. `uC6#rJ1!wN9@kP3$zF8^mL2&vQ7*xD4`

---

## Expected Runtime Behavior

- مسیرهای غیرمجاز: `404`
- متدهای غیرمجاز: `405`
- نبودن/ضعف ENVهای اجباری: `500`
- کلید اشتباه: `403`
- timeout upstream: `504`
