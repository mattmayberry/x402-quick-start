# x402 Quick Start

## IDENTITY

This skill teaches me the five-step payment handshake pattern for x402-gated APIs: make the initial request, parse the 402 response, construct the payment, submit it to a facilitator, and retry the request with payment proof. It covers the payment flow only — nothing about revenue analytics, pipeline configuration, or multi-endpoint management.

---

## CORE RULES

- Never attempt to construct or sign a transaction without a wallet address and USDC amount confirmed from the 402 response. Do not guess or infer payment parameters.
- Always parse the `WWW-Authenticate` or `X-Payment-Details` header (or equivalent facilitator field) before attempting payment. Payment parameters are server-provided, not operator-assumed.
- Never include a real private key or seed phrase in any output, log, or template. Payment construction requires a wallet address and signed transaction — signing happens in the operator's wallet tooling, not in my output.
- If the payment proof header format is ambiguous or the facilitator's scheme differs from the standard pattern, stop and ask the operator for the exact field name and encoding before proceeding.
- Treat a second 402 (after payment) as a payment failure, not a retry loop. Stop, report the error, and ask the operator whether to re-attempt.

---

## PROCEDURES

### Procedure 1: Make the Initial Request and Parse the 402 Response

1. Issue the HTTP request to the x402-gated endpoint using the operator's specified method (GET, POST, etc.) and any required headers or body.
2. Receive the 402 Payment Required response.
3. Parse the payment details from the response. Standard location is the `WWW-Authenticate` header or a JSON body under the `payment` key — confirm with the operator if the facilitator uses a different field.
4. Extract and record the following fields:
   - `amount` — USDC value required (confirm units: some facilitators return in base units, e.g., 1000000 = 1 USDC at 6 decimals)
   - `address` — destination wallet address for payment
   - `network` — Base or Solana (determines transaction format)
   - `expires_at` — timestamp after which this payment quote expires (if present)
   - `facilitator` — the x402 facilitator endpoint that will verify payment (if separate from the API server)
   - `payment_id` or `quote_id` — unique identifier for this payment session (required for retry)
5. Report the parsed payment details to the operator before proceeding. Do not initiate payment autonomously.

### Procedure 2: Construct and Submit the Payment

1. Confirm the operator has a wallet with sufficient USDC balance on the specified network.
2. Using the operator's wallet tooling (not this skill), construct a USDC transfer transaction to the `address` for the `amount` specified in the 402 response.
   - For Base: ERC-20 USDC transfer on Base mainnet (contract: operator-confirmed, not hardcoded here)
   - For Solana: SPL token transfer on Solana mainnet
3. Sign and broadcast the transaction using the operator's wallet. Record the resulting transaction hash.
4. If a facilitator endpoint is specified, submit payment proof to the facilitator:
   ```
   POST [facilitator_url]/verify
   Content-Type: application/json

   {
     "payment_id": "[quote_id from 402 response]",
     "tx_hash": "[transaction hash from wallet]",
     "network": "[base|solana]"
   }
   ```
5. The facilitator returns a `payment_token` (JWT or signed credential). Record it.
6. If no separate facilitator verification step exists and the API server verifies payment directly, proceed to Procedure 3 with the raw transaction hash.

### Procedure 3: Retry the Request with Payment Proof

1. Repeat the original HTTP request with the payment proof included in the header.
2. Standard header format:
   ```
   X-Payment: [payment_token]
   ```
   Some facilitators use `Authorization: Bearer [payment_token]` or a custom header — confirm with operator if the field name differs.
3. If the server returns 200: payment succeeded. Report the response to the operator.
4. If the server returns 402 again: the payment was not accepted. Do not retry automatically. Report the error response, the payment_id used, and the transaction hash. Ask the operator how to proceed.
5. If the server returns 401 or 403: the payment was accepted but authorization failed for a different reason (e.g., wrong scope, expired token). Report the response and ask the operator.

---

## TEMPLATES

### Payment Details (parsed from 402 response)

```
x402 Payment Required — [endpoint]

Amount:       [value] USDC ([raw_units] in base units if applicable)
Network:      [Base | Solana]
Pay to:       [first_6]...[last_4] (truncated wallet address)
Expires:      [expires_at or "not specified"]
Payment ID:   [quote_id or payment_id]
Facilitator:  [facilitator_url or "direct — server verifies"]

Awaiting operator confirmation to proceed with payment.
```

### Payment Proof Header (for retry request)

```
X-Payment: [payment_token]
```

Or if the facilitator uses Authorization Bearer:

```
Authorization: Bearer [payment_token]
```

### Payment Failure Report

```
x402 Payment Failure — [endpoint]

Payment ID:    [id]
Tx Hash:       [hash]
Network:       [network]
Server Response: [status code] — [response body excerpt]

Payment was submitted but the server returned a second 402.
Do not retry without operator instruction. Possible causes:
- Transaction not yet confirmed on-chain (latency)
- Payment amount or address mismatch
- Quote expired before retry

Awaiting operator instruction.
```

---

## ESCALATION

Stop and confirm with the operator before:
- Initiating any payment (always — payment is never autonomous)
- Retrying after a second 402 response
- Proceeding when the 402 response is missing required fields (amount, address, network)
- Proceeding when the `expires_at` timestamp has passed

Handle and report without stopping:
- Parsing and displaying the 402 payment details
- Formatting the retry request with a provided payment token
- Reporting a successful 200 response after retry

---

## LIMITATIONS

This skill covers the payment handshake only: parse 402, pay, retry. It does not provide:

- Revenue tracking, call volume analysis, or payment success rate reporting across endpoints
- Pricing optimization or endpoint health monitoring
- Multi-endpoint orchestration or pipeline configuration
- Wallet management, key signing, or on-chain transaction construction (signing requires the operator's wallet tooling)
- Cross-session transaction history or cumulative USDC spend tracking
- Facilitator-specific implementation details beyond the standard pattern — if your facilitator deviates, supply the exact field names

For revenue analytics across x402 endpoints, see **x402 Revenue Dashboard** on ClawMart ($29). For building and managing multi-step x402 data pipelines, see **Data Pipeline Builder** on ClawMart ($89).
