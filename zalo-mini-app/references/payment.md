# Payments — Checkout SDK Integration

This document outlines a standard, secure integration pattern for Zalo Mini App payments via **Checkout SDK** and payment partners.

> **Security requirement**: private keys and MAC/signature generation must be server-side.

## Supported payment partners (examples)

- ZaloPay
- MoMo
- VNPay
- PayME
- Bank transfer
- COD (cash on delivery)

## Recommended high-level flow

```
Mini App → Your Backend → Checkout SDK → Payment Partner
                         ↘ Webhook callback ↙
```

1. User creates an order in the Mini App.
2. Mini App calls your backend to create the Checkout payload + MAC.
3. Mini App calls `createOrder()` (client-side) using the server-signed payload.
4. User completes payment on partner UI.
5. Your backend receives webhook callbacks and verifies MAC/signatures.
6. Your backend updates order status and optionally notifies the user.

## Client-side: start payment

```js
import { createOrder } from "zmp-sdk";

export async function startPayment(orderInput) {
  // 1) Ask backend for Checkout payload (includes MAC)
  const resp = await fetch("https://your-server.com/api/checkout/create", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(orderInput),
  });

  if (!resp.ok) throw new Error("Unable to create checkout order");
  const payload = await resp.json();

  // 2) Invoke Checkout SDK
  const result = await createOrder({
    orderId: payload.orderId,
    desc: payload.desc,
    amount: payload.amount,
    item: payload.item,
    extraData: payload.extraData,
    mac: payload.mac, // server-generated
  });

  return result;
}
```

## Server-side: create MAC (example)

```js
const crypto = require("crypto");

const CHECKOUT_PRIVATE_KEY = process.env.CHECKOUT_PRIVATE_KEY;
const MINI_APP_ID = process.env.MINI_APP_ID;

function createMac(data, privateKey) {
  // Sort keys to ensure a stable signature base string
  const sorted = Object.keys(data)
    .sort()
    .map((k) => `${k}=${data[k]}`)
    .join("&");

  return crypto.createHmac("sha256", privateKey).update(sorted).digest("hex");
}

function createCheckoutPayload(order) {
  const data = {
    appId: MINI_APP_ID,
    orderId: String(order.id),
    amount: order.totalAmount,
    desc: `Order #${order.id}`,
    item: JSON.stringify(order.items),
    extraData: JSON.stringify({ internalOrderId: order.id }),
    time: Date.now(),
  };

  return { ...data, mac: createMac(data, CHECKOUT_PRIVATE_KEY) };
}
```

## Webhook handler (server-side)

Always verify the MAC/signature before updating your database.

```js
app.post("/webhook/payment", async (req, res) => {
  const { orderId, status, amount, transactionId, mac } = req.body;

  const expectedMac = createMac(
    { orderId, status, amount, transactionId },
    CHECKOUT_PRIVATE_KEY,
  );

  if (mac !== expectedMac) {
    return res.status(400).json({ error: "Invalid MAC" });
  }

  // Update order status (idempotent update is recommended)
  await updateOrderStatus({ orderId, status, amount, transactionId });
  res.json({ success: true });
});
```

## Operational best practices

- **Idempotency**: ensure webhook processing is safe to retry.
- **Amount verification**: validate `amount` against your database, not client input.
- **Timeouts & retries**: handle partner delays gracefully.
- **Logging**: log transaction IDs, but never log full MAC or private keys.

