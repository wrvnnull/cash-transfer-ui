# cash-transfer-ui

> One-click bash script generator for transferring CASH tokens on Solana via [AgentWallet](https://frames.ag).

🔗 **Live:** https://wrvnnull.github.io/cash-transfer-ui/

---

## What is this?

A static web UI that generates a ready-to-run bash script for transferring CASH tokens (SPL Token 2022) using the AgentWallet API — no wallet extension needed, no SOL for gas.

**Why a script and not a direct web app?**  
GitHub Pages can't directly call `frames.ag` due to browser CORS restrictions. Instead, this tool generates a bash script you paste into your terminal — same result, no CORS issue.

---

## How to use

1. Open https://wrvnnull.github.io/cash-transfer-ui/
2. Fill in:
   - **Username** — your AgentWallet username
   - **API Token** — your `mf_...` token
   - **Destination** — recipient's Solana wallet address
3. Click **Generate Script**
4. Click **Copy Script**
5. Paste into terminal → Enter

The script handles everything automatically:

```
[1/5] Checking balance...
  ✓ Solana: H2SGfc...dvRR
  ✓ CASH  : 1167000 (raw)
[2/5] Source token account...
  ✓ 2PAGDh...nc6u
[3/5] Destination token account...
  ✓ 3P5jhz...EBGV
[4/5] Encoding instruction...
  ✓ data: DJjOEQAAAAAABg==
[5/5] Broadcasting transaction...

╔══════════════════════════════════╗
║  ✓ Transfer Confirmed!           ║
╚══════════════════════════════════╝

  Gas free : true
  Explorer : https://solscan.io/tx/...
```

---

## Requirements

- `curl` — for API calls
- `node` — for encoding the transfer instruction
- An [AgentWallet](https://frames.ag) account with CASH balance
- Recipient wallet must already have a CASH token account

---

## Gas sponsorship

AgentWallet sponsors Solana gas fees automatically.

| Account age | Gas status |
|-------------|------------|
| < 24 hours  | Not sponsored yet |
| ≥ 24 hours  | Free ✓ |

If your account is new and you see this error:

```
⚠ Gas not active yet (account <24h). Wait or send SOL to: ...
```

**Option A:** Wait until 24h after account creation, then re-run the same script.  
**Option B:** Send a small amount of SOL to your Solana wallet address, then re-run.

---

## Technical notes

CASH is an **SPL Token 2022** with a `transferHook` extension. The standard AgentWallet transfer endpoint (`transfer-solana`) does not support `cash` as an asset — it only accepts `sol` and `usdc`.

This tool works around that by:
1. Fetching token accounts directly from the Solana RPC
2. Building a `transferChecked` instruction manually (10-byte encoded)
3. Sending it via AgentWallet's `contract-call` endpoint

### Instruction encoding (Node.js)
```js
const buf = Buffer.alloc(10);
buf[0] = 12;                          // transferChecked discriminator
let amt = BigInt(rawAmount);
for (let i = 0; i < 8; i++) {
  buf[1+i] = Number(amt & 0xffn);
  amt >>= 8n;
}
buf[9] = 6;                           // decimals
console.log(buf.toString('base64')); // e.g. DJjOEQAAAAAABg==
```

### Key addresses
| Item | Address |
|------|---------|
| CASH Mint | `CASHx9KJUStyftLFWGvEVf59SGeG9sh5FfcnZMVPCASH` |
| SPL Token 2022 Program | `TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb` |
| Solana RPC | `https://api.mainnet-beta.solana.com` |
| AgentWallet API | `https://frames.ag/api` |

---

## Run locally

```bash
git clone https://github.com/wrvnnull/cash-transfer-ui
cd cash-transfer-ui
python3 -m http.server 8080
# open http://localhost:8080
```

---

## License

MIT
