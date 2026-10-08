# ATA Trading Academy v26 - all source files
-e 

## ./.env.example

```
# Arc Mainnet
VITE_INFURA_PROJECT_ID=your_infura_project_id
VITE_API_BASE_URL=/api

# Circle Arc Onramp — SERVER ONLY (never prefix with VITE_)
CIRCLE_API_KEY=
CIRCLE_ONRAMP_REFERRER_DOMAIN=

# LI.FI — SERVER ONLY. Optional but recommended for higher API limits.
LIFI_API_KEY=
LIFI_INTEGRATOR=ata-trading-academy

# Stock prices - SERVER ONLY (free key at finnhub.io).
FINNHUB_API_KEY=

# ATA Mentor and Chart Analysis - SERVER ONLY (console.anthropic.com)
ANTHROPIC_API_KEY=
# optional model override
MENTOR_MODEL=
-e 
```
-e 

## ./.gitignore

```
node_modules
dist
.env
.env.local
.vercel
-e 
```
-e 

## ./ATA_INTEGRATION_STATUS.json

```
{
  "version": "v23",
  "arc_mainnet_configured": true,
  "arc_mainnet_chain_id": 5042,
  "arc_mainnet_chain_id_hex": "0x13b2",
  "production_faucet": false,
  "real_usdc_funding": true,
  "circle_onramp": "server-side Circle Onramp Kit session + connected wallet",
  "live_swap_provider": "LI.FI API quote + connected-wallet signing",
  "live_bridge_provider": "LI.FI API quote + connected-wallet signing",
  "bridge_source_wallet_locked_to_connected_wallet": true,
  "bridge_destination_wallet_locked_to_connected_wallet": true,
  "live_transaction_requires_wallet_signature": true,
  "api_keys_browser_exposure": false,
  "vercel_runtime": "Node.js 24.x"
}
-e 
```
-e 

## ./ATA_V12_README.md

```
# ATA Trading Academy v12

ATA is configured for real-USDC Arc Mainnet funding with separate bridge, swap, DeFi and trading execution domains.

Production rule: no faucet. Users use real USDC. Bridge and swap transactions require connected-wallet confirmation and confirmed transaction status before ATA displays success.
-e 
```
-e 

## ./ATA_V9_README.md

```
# ATA Trading Academy v9

ATA is structured as an independent trading platform with integrated DeFi wallet/funding/swap/earn capabilities.

The DeFi layer and real-trading execution layer are separate provider-adapter domains.
See `docs/ATA_DEFI_REAL_TRADING_INTEGRATION.md` for the production integration boundary.
-e 
```
-e 

## ./README.md

```
# ATA Trading Academy — Independent Real-Trading Architecture

This project starts from the original ATA source and restructures it around an independent trading-terminal model.

## Product layers

1. **ATA UI** — charts, order ticket, positions, balances, education, AI and journal.
2. **ATA application API** — authenticated server endpoints for accounts, orders and positions.
3. **Execution provider** — a server-side adapter for the exchange/broker/DEX selected by ATA.
4. **Market-data provider** — a separate adapter for live quotes/order books/candles.
5. **Arc wallet layer** — wallet connection and Arc/USDC operations where applicable.

## Critical security rule

Never place exchange/broker API secrets in React/browser code. Live execution must happen server-side through an authenticated provider adapter and should require explicit user authorization.

The included execution API deliberately refuses live orders until an approved provider is implemented and `ATA_LIVE_TRADING=true` is configured.

## Next implementation step

Choose the first real execution venue (for example a specific CEX API, broker API, or an on-chain DEX/contract). Then implement that venue in `server/providers/` and add authentication, signed requests/transactions, order-status reconciliation, websocket market data, rate limits, audit logs, and kill switches.


## Current implementation status

- The frontend contains an ATA-native trading terminal and order ticket.
- The backend exposes account, positions, orders, cancellation, and health routes.
- Live order submission is blocked until a real execution adapter is installed.
- Venue secrets remain server-side.
- `server/providers/VenueAdapter.template.js` is the clean integration boundary for the first approved venue.
- `server/market-data.js` keeps candles/order books/tickers independent from execution.

### Production checklist

1. Select the first authorized venue.
2. Implement its official API/SDK in a new server-side adapter.
3. Add user authentication and account-to-venue authorization.
4. Normalize venue order/position states into ATA's internal model.
5. Add websocket market data and order-status reconciliation.
6. Add idempotency keys, rate limits, audit logs, monitoring, and a kill switch.
7. Add explicit order-review/confirmation UX before live submission.
8. ATA is mainnet-only; paper trading is disabled.

## v3 execution safety layer

- Server-side order validation for quantity, prices, and leverage.
- Idempotency-Key required for live order submission to reduce duplicate-order risk.
- Append-only local audit log for API/order events (replace with a durable protected logging system in production).
- Provider registry keeps the first real venue as a separate adapter rather than coupling ATA to a specific exchange/broker.
- Live trading remains disabled until `ATA_LIVE_TRADING=true` and a real, authenticated provider adapter is installed.

**Important:** this project still does not execute real trades. A venue adapter and a real user authorization/account-linking system must be implemented before enabling live trading.

## v4 market-data layer

ATA now exposes a venue-neutral market-data boundary:
- `GET /api/market/ticker?symbol=BTC/USDC`
- `GET /api/market/candles?symbol=BTC/USDC&timeframe=1m&limit=500`
- `GET /api/market/orderbook?symbol=BTC/USDC&depth=20`

These endpoints intentionally return an unconfigured-provider error until an approved market-data adapter is implemented. This prevents ATA from displaying fabricated prices or implying that market data is live when it is not.

`server/events.js` defines normalized event types for future streaming market data and private order/position/account updates. A production adapter should translate the selected venue's WebSocket/stream payloads into these ATA events.

### Next production connection
1. Select the first execution venue and market-data source.
2. Implement its official API/SDK adapter on the server only.
3. Normalize symbols, balances, orders, positions, fills, and errors.
4. Add authenticated user-to-account authorization.
5. Add streaming reconciliation for ticker/order-book and private order/position updates.
6. Use Arc Mainnet production credentials and live-provider testing before enabling production execution.


## Market data
ATA can use Massive as the server-side market-data provider for real-time stocks and forex aggregates, trades and quotes. The provider key stays on the server. Massive does not provide Level-2/full order-book depth; ATA therefore exposes NBBO/top-of-book for stocks and forex. For full depth, connect a licensed depth provider such as dxFeed using its commercial integration details.


## ATA DeFi + Portal

The DeFi tab adds an independent ATA integration layer for Portal embedded wallets and supported EVM wallets. It supports: creating a Portal wallet, connecting an existing browser wallet, reading Portal balances/activity, sending assets, Li.Fi swap execution, and yield discovery when the Portal account has the yield integration enabled.

Portal's Web SDK supports `createWallet()`, balance/transaction retrieval, sending assets, and Li.Fi swaps. Portal's current public Web documentation uses Yield.xyz for yield discovery; the project therefore does not claim a direct Morpho execution path until that integration is enabled/confirmed for the ATA Portal account.

### Production security

For production, do not expose Portal client credentials unnecessarily. Portal documents Web OTP/authUrl authentication as the more secure production approach compared with putting a client API key in the DOM. Configure Portal's production environment, custom subdomain where required, RPC endpoints, and provider integrations before enabling live DeFi actions.
-e 
```
-e 

## ./VERCEL_DEPLOY.md

```
# ATA Trading Academy — Vercel production setup

## Build
- Framework: Vite
- Build command: `npm run build`
- Output directory: `dist`
- Node.js: `24.x`

## Required server environment variables

Set these in Vercel **Project → Settings → Environment Variables → Production**:

```text
CIRCLE_API_KEY=YOUR_CIRCLE_ONRAMP_API_KEY
CIRCLE_ONRAMP_REFERRER_DOMAIN=YOUR_VERCEL_HOSTNAME
LIFI_API_KEY=YOUR_LIFI_PARTNER_API_KEY
LIFI_INTEGRATOR=ata-trading-academy
```

Optional frontend variable:

```text
VITE_INFURA_PROJECT_ID=YOUR_INFURA_PROJECT_ID
```

Never prefix `CIRCLE_API_KEY` or `LIFI_API_KEY` with `VITE_`. They must stay server-side.

## Circle Onramp
ATA creates a short-lived Circle Onramp session on the server and scopes the session to USDC on Arc and the connected wallet. The long-lived Circle API key never reaches the browser.

Production iframe embedding requires the exact deployed hostname to be authorized with Circle. If Circle has not authorized the hostname yet, the Onramp widget must be opened using Circle's supported top-level/popup flow instead of an iframe.

## LI.FI swap
ATA requests live quotes from the server, with Arc chain ID `5042`, and returns the transaction request to the browser. The connected wallet signs approvals and the swap transaction.

## LI.FI bridge
ATA requests a live route with:
- source chain: Arc `5042`
- source wallet: the currently connected wallet
- destination wallet: the same connected wallet

ATA rejects a route if LI.FI returns a transaction whose `from` address does not equal the connected wallet. The browser also verifies the wallet again before signing.

## Important
Do not put real API keys into `.env.example`, source files, Git, or the browser bundle. After adding production environment variables, redeploy the Vercel project.

## v25 market data
Add `FINNHUB_API_KEY` (server-only) for stocks and economy news. Crypto (CoinGecko) and forex (ECB via Frankfurter, daily) need no key.

## v26
Add `ANTHROPIC_API_KEY` (server-only) for ATA Mentor and Chart Analysis. `FINNHUB_API_KEY` is now only for stock prices. News was removed.
-e 
```
-e 

## ./api/_claude.js

```
export async function askClaude({ system, messages, max_tokens = 1400 }) {
  const key = process.env.ANTHROPIC_API_KEY
  if (!key) {
    const e = new Error('ATA Mentor needs ANTHROPIC_API_KEY set in Vercel.')
    e.status = 503
    throw e
  }
  const r = await fetch('https://api.anthropic.com/v1/messages', {
    method: 'POST',
    headers: { 'content-type': 'application/json', 'x-api-key': key, 'anthropic-version': '2023-06-01' },
    body: JSON.stringify({ model: process.env.MENTOR_MODEL || 'claude-sonnet-5-5', max_tokens, system, messages }),
  })
  const data = await r.json().catch(() => ({}))
  if (!r.ok) {
    const e = new Error(data?.error?.message || `AI provider returned ${r.status}`)
    e.status = 502
    throw e
  }
  return (data.content || []).filter(b => b.type === 'text').map(b => b.text).join('\n').trim()
}

export const MENTOR_RULES = `You are ATA Mentor, a patient, technical trading mentor inside ATA Trading Academy. Teach market structure, entries, exits, risk management and trading psychology for forex, crypto and stocks. Talk like a mentor walking a student through a setup: explain the reasoning step by step, name key levels, state what would invalidate the idea, and size risk as a percentage of account. Give scenarios (if price does X, then Y), never certainties or guarantees. Never tell the user to buy or sell right now, never promise profit, and remind them briefly that this is education, not financial advice. All real funds on this platform are Arc USDC only. Write plain text with short headings in capitals and short lists using hyphens. Do not use markdown symbols like ** or #. Keep answers focused and under about 350 words unless asked for more.`
-e 
```
-e 

## ./api/bridge/quote.js

```
const ARC_CHAIN_ID = 5042
const TOKENS = {
  USDC: "0x3600000000000000000000000000000000000000",
  EURC: "0xbEf5f6d51CB62b58e6A8f77868681825C6fe21c1",
}
function isAddress(value) { return /^0x[a-fA-F0-9]{40}$/.test(value || "") }
function isPositiveDecimal(value) { return /^\d+(?:\.\d+)?$/.test(value || "") && Number(value) > 0 }
function toUnits(value, decimals = 6) {
  const [whole, fraction = ""] = String(value).split(".")
  if (fraction.length > decimals) throw new Error(`Amount supports at most ${decimals} decimals.`)
  return (BigInt(whole || "0") * (10n ** BigInt(decimals)) + BigInt((fraction + "0".repeat(decimals)).slice(0, decimals) || "0")).toString()
}
export default async function handler(req, res) {
  if (req.method !== "GET") return res.status(405).json({ error: "Method not allowed" })
  try {
    const { wallet, destinationChain, asset = "USDC", amount } = req.query || {}
    if (!isAddress(wallet)) throw new Error("Connect your wallet before bridging.")
    if (!destinationChain || Number(destinationChain) === ARC_CHAIN_ID) throw new Error("Choose a destination chain other than Arc.")
    if (!TOKENS[asset]) throw new Error("Unsupported Arc bridge asset.")
    if (!isPositiveDecimal(amount)) throw new Error("Enter a valid bridge amount.")
    const params = new URLSearchParams({
      fromChain: String(ARC_CHAIN_ID), toChain: String(destinationChain),
      fromToken: TOKENS[asset], toToken: asset,
      fromAddress: wallet, toAddress: wallet, fromAmount: toUnits(amount),
      order: "FASTEST", slippage: "0.005", integrator: process.env.LIFI_INTEGRATOR || "ata-trading-academy",
      maxPriceImpact: "0.15",
    })
    const headers = { accept: "application/json" }
    if (process.env.LIFI_API_KEY) headers["x-lifi-api-key"] = process.env.LIFI_API_KEY
    const upstream = await fetch(`https://li.quest/v1/quote?${params.toString()}`, { headers })
    const data = await upstream.json().catch(() => ({}))
    if (!upstream.ok) return res.status(upstream.status).json({ error: data?.message || data?.error || `LI.FI returned ${upstream.status}` })
    // Never allow ATA to sign for a different wallet. The user must sign the returned transaction.
    if (data?.transactionRequest?.from && data.transactionRequest.from.toLowerCase() !== wallet.toLowerCase()) {
      return res.status(409).json({ error: "LI.FI returned a transaction for a different wallet. Route rejected." })
    }
    return res.status(200).json({ ...data, ata: { connectedWallet: wallet, destinationWallet: wallet, requiresConnectedWalletSignature: true } })
  } catch (e) {
    return res.status(400).json({ error: e?.message || "Unable to get a live bridge quote." })
  }
}
-e 
```
-e 

## ./api/cctp/transfer.js

```
const ARC_DOMAIN = 26
const ARC_USDC = '0x3600000000000000000000000000000000000000'
const ARC_EURC = '0xbEf5f6d51CB62b58e6A8f77868681825C6fe21c1'

export default function handler(req, res) {
  if (req.method !== 'POST') return res.status(405).json({ error: 'Method not allowed' })
  try {
    const { sourceDomain, amount, sourceWallet, destinationWallet, destinationDomain, asset = 'USDC' } = req.body || {}
    if (Number(sourceDomain) !== ARC_DOMAIN) throw new Error('Bridge source must be Arc Mainnet domain 26.')
    if (!amount || !sourceWallet || !destinationWallet || destinationDomain === undefined) throw new Error('amount, sourceWallet, destinationWallet and destinationDomain are required.')
    if (Number(destinationDomain) === ARC_DOMAIN) throw new Error('Choose a destination chain other than Arc.')
    if (!['USDC', 'EURC'].includes(asset)) throw new Error('Unsupported bridge asset.')
    return res.status(200).json({
      status: 'prepared',
      message: `Arc ${asset} bridge intent prepared. The user must sign the live CCTP transaction in their wallet.`,
      sourceDomain: ARC_DOMAIN,
      destinationDomain: Number(destinationDomain),
      amount,
      asset,
      tokenAddress: asset === 'USDC' ? ARC_USDC : ARC_EURC,
      sourceWallet,
      destinationWallet,
      requiresWalletSignature: true,
    })
  } catch (e) {
    return res.status(400).json({ error: e.message || 'Unable to prepare bridge transfer.' })
  }
}
-e 
```
-e 

## ./api/chart-analysis.js

```
import { askClaude, MENTOR_RULES } from './_claude.js'
export const config = { maxDuration: 60 }
const TYPES = ['image/jpeg', 'image/png', 'image/webp']
export default async function handler(req, res) {
  if (req.method !== 'POST') return res.status(405).json({ error: 'Method not allowed' })
  try {
    const { image, mediaType, context } = req.body || {}
    if (typeof image !== 'string' || !TYPES.includes(mediaType)) return res.status(400).json({ error: 'Upload a PNG, JPEG or WebP chart screenshot.' })
    if (image.length > 4_000_000) return res.status(413).json({ error: 'Image is too large. Try a smaller screenshot.' })
    const note = typeof context === 'string' ? context.slice(0, 600) : ''
    const prompt = `Walk me through this chart like a mentor talking a student through a setup. Cover, in order: 1) MARKET STRUCTURE (trend, swing highs/lows, breaks of structure), 2) KEY LEVELS (support, resistance, liquidity zones you can actually see), 3) POSSIBLE SCENARIOS (bullish and bearish, with the trigger for each), 4) ENTRY, STOP AND TARGET IDEAS with the invalidation level, 5) RISK (suggested risk per trade and approximate reward-to-risk), 6) WHAT TO WATCH next. Only describe what is visible; if the timeframe, instrument or levels are unclear, say so.${note ? `\nStudent context: ${note}` : ''}`
    const reply = await askClaude({
      system: MENTOR_RULES + ' You are reviewing an uploaded chart screenshot.',
      max_tokens: 1800,
      messages: [{ role: 'user', content: [{ type: 'image', source: { type: 'base64', media_type: mediaType, data: image } }, { type: 'text', text: prompt }] }],
    })
    return res.status(200).json({ reply })
  } catch (e) {
    return res.status(e.status || 500).json({ error: e.message || 'Chart analysis unavailable.' })
  }
}
-e 
```
-e 

## ./api/health.js

```
export default function handler(req, res) {
  res.status(200).json({
    ok: true,
    network: 'Arc',
    chainId: 5042,
    chainIdHex: '0x13b2',
    nativeGasCurrency: 'USDC',
    explorer: 'https://explorer.arc.io',
    onrampConfigured: Boolean(process.env.CIRCLE_API_KEY && process.env.CIRCLE_ONRAMP_API_BASE_URL && process.env.CIRCLE_ONRAMP_SESSION_PATH),
    cctpDomain: 26,
  })
}
-e 
```
-e 

## ./api/market/quotes.js

```
const CRYPTO = { bitcoin: 'BTC', ethereum: 'ETH', solana: 'SOL', ripple: 'XRP', cardano: 'ADA' }
const FX = ['EUR', 'GBP', 'JPY', 'CHF', 'AUD', 'CAD']
const STOCKS = ['AAPL', 'MSFT', 'NVDA', 'TSLA', 'AMZN', 'SPY']

async function getJson(url, headers = {}) {
  const r = await fetch(url, { headers: { accept: 'application/json', ...headers } })
  if (!r.ok) throw new Error(`Provider returned ${r.status}`)
  return r.json()
}

export default async function handler(req, res) {
  if (req.method !== 'GET') return res.status(405).json({ error: 'Method not allowed' })
  const type = String(req.query?.type || 'crypto')
  try {
    let items = []
    let source = ''
    if (type === 'crypto') {
      source = 'CoinGecko'
      const d = await getJson(`https://api.coingecko.com/api/v3/simple/price?ids=${Object.keys(CRYPTO).join(',')}&vs_currencies=usd&include_24hr_change=true`)
      items = Object.entries(CRYPTO).map(([id, sym]) => ({ symbol: `${sym}/USD`, price: d[id]?.usd, change: d[id]?.usd_24h_change }))
    } else if (type === 'forex') {
      source = 'ECB reference rates via Frankfurter (updated once per business day)'
      const d = await getJson(`https://api.frankfurter.dev/v1/latest?base=USD&symbols=${FX.join(',')}`)
      items = FX.map(c => {
        const rate = d.rates?.[c]
        const eurStyle = ['EUR', 'GBP', 'AUD'].includes(c)
        return { symbol: eurStyle ? `${c}/USD` : `USD/${c}`, price: rate ? (eurStyle ? 1 / rate : rate) : null, change: null }
      })
    } else if (type === 'stocks') {
      source = 'Finnhub'
      const key = process.env.FINNHUB_API_KEY
      if (!key) return res.status(503).json({ error: 'Stocks need FI
