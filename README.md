# CASH Transfer UI

One-click web UI to transfer CASH tokens on Solana via [AgentWallet](https://frames.ag) — no CLI, no SOL needed for gas.

🔗 **Live demo:** `https://YOUR-USERNAME.github.io/cash-transfer-ui`

---

## What it does

Automates the full transfer flow in a single click:

1. **Check balance** — fetches CASH balance from your AgentWallet account
2. **Query source token account** — finds your CASH token account on-chain
3. **Query destination token account** — finds recipient's CASH token account
4. **Encode instruction** — builds a `transferChecked` instruction for SPL Token 2022
5. **Broadcast** — sends transaction via AgentWallet's sponsored gas endpoint

---

## Deploy to GitHub Pages

```bash
# 1. Fork or clone this repo
git clone https://github.com/YOUR-USERNAME/cash-transfer-ui
cd cash-transfer-ui

# 2. Push to GitHub
git add .
git commit -m "init"
git push origin main

# 3. Enable GitHub Pages
# Go to: Settings → Pages → Source → Deploy from branch → main / (root)
```

Your UI will be live at `https://YOUR-USERNAME.github.io/cash-transfer-ui`

---

## Usage

Fill in three fields and click **Run All Steps**:

| Field | Description |
|-------|-------------|
| **Username** | Your AgentWallet username |
| **API Token** | Your `mf_...` token from `~/.agentwallet/config.json` |
| **Destination** | Solana wallet address of the recipient |

The tool transfers **all** available CASH balance to the destination.

---

## Requirements

- AgentWallet account ([sign up](https://frames.ag))
- CASH token in your wallet
- Destination wallet must have an existing CASH token account

> **Gas:** AgentWallet sponsors gas fees automatically after 24h from account creation.  
> If your account is new (< 24h), wait until sponsorship activates or send a small amount of SOL to your Solana wallet.

---

## Running locally

If GitHub Pages returns CORS errors, run locally:

```bash
# Python
python3 -m http.server 8080

# Node
npx serve .
```

Then open `http://localhost:8080`

---

## Manual curl reference

If you prefer the terminal, here are the equivalent curl commands:

### 1. Check balance
```bash
curl https://frames.ag/api/wallets/{USERNAME}/balances \
  -H "Authorization: Bearer {API_TOKEN}"
```

### 2. Get source token account
```bash
curl -s https://api.mainnet-beta.solana.com \
  -X POST -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"getTokenAccountsByOwner","params":["{SOLANA_ADDRESS}",{"mint":"CASHx9KJUStyftLFWGvEVf59SGeG9sh5FfcnZMVPCASH"},{"encoding":"jsonParsed"}]}'
```

### 3. Get destination token account
Same as above but replace `{SOLANA_ADDRESS}` with the recipient's address.

### 4. Transfer (replace values from steps above)
```bash
curl -X POST "https://frames.ag/api/wallets/{USERNAME}/actions/contract-call" \
  -H "Authorization: Bearer {API_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "chainType": "solana",
    "instructions": [{
      "programId": "TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb",
      "accounts": [
        {"pubkey": "{SOURCE_TOKEN_ACCOUNT}",      "isSigner": false, "isWritable": true},
        {"pubkey": "CASHx9KJUStyftLFWGvEVf59SGeG9sh5FfcnZMVPCASH", "isSigner": false, "isWritable": false},
        {"pubkey": "{DESTINATION_TOKEN_ACCOUNT}", "isSigner": false, "isWritable": true},
        {"pubkey": "{YOUR_SOLANA_ADDRESS}",       "isSigner": true,  "isWritable": false}
      ],
      "data": "{ENCODED_DATA}"
    }],
    "network": "mainnet"
  }'
```

### Encoding `data` (Node.js)
```js
function encodeTransferChecked(amount, decimals) {
  const buf = new Uint8Array(10);
  buf[0] = 12; // discriminator
  let amt = BigInt(amount);
  for (let i = 0; i < 8; i++) { buf[1+i] = Number(amt & 0xffn); amt >>= 8n; }
  buf[9] = decimals;
  return Buffer.from(buf).toString('base64');
}

console.log(encodeTransferChecked(1167000, 6)); // → DJjOEQAAAAAABg==
```

---

## Reference

| Item | Value |
|------|-------|
| CASH Mint | `CASHx9KJUStyftLFWGvEVf59SGeG9sh5FfcnZMVPCASH` |
| SPL Token 2022 Program | `TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb` |
| Solana RPC | `https://api.mainnet-beta.solana.com` |
| AgentWallet API | `https://frames.ag/api` |
| CASH decimals | `6` |
