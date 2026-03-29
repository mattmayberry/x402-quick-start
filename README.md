# x402 Quick Start

**The five-step pattern for making your first x402-gated API call. Parse, pay, retry.**

Free skill for Claude Code, Claude Desktop, and any OpenClaw-compatible agent.

---

## What This Does

Installs the x402 payment handshake pattern on your agent so it can automatically pay for API calls that return HTTP 402 Payment Required responses.

The five-step pattern:

1. **Parse** — Read the `X-Payment-Required` header from the 402 response
2. **Display** — Show the payment details for user review before paying
3. **Construct** — Build the signed payment transaction (Base USDC or Solana SPL)
4. **Retrieve** — Get the payment token from the x402 facilitator
5. **Retry** — Resubmit the original request with the payment proof

---

## Install

Add `SKILL.md` to your project's `.claude/` directory, or paste the contents into your Claude system prompt.

```bash
curl -O https://raw.githubusercontent.com/themeridianlab/x402-quick-start/main/SKILL.md
```

---

## Usage

Once installed, your agent handles 402 responses automatically:

```
GET /api/data
→ 402 Payment Required
→ Agent parses X-Payment-Required header
→ Agent displays: "This API costs 0.001 USDC. Pay to proceed?"
→ User confirms
→ Agent constructs payment, retrieves token, retries request
→ 200 OK with data
```

Works with any x402-compliant API. Currently supports Base (ERC-20 USDC) and Solana (SPL token).

---

## What Is x402?

x402 is an open protocol for machine-to-machine payments over HTTP. A server returns 402 with a signed payment request; the client pays and retries with proof. No subscriptions, no API keys, no rate limits — just pay per call.

Learn more: [x402.org](https://x402.org)

---

## Want More?

This quick start is a sample of the full **[x402 Revenue Dashboard](https://www.shopclawmart.com/listings/cd120260-1d20-4205-aad9-14fd6aa5becc)** skill ($29 on ClawMart) — adds real-time revenue tracking, per-endpoint analytics, and payment failure monitoring for APIs you're running on x402.

---

## License

MIT. Free to use, fork, and extend.

A [Meridian Lab](https://themeridianlab.com) product.
