# Hype Feed

A pure peer-to-peer photo stream built on [Nostr](https://nostr.com), with Lightning tipping (zaps) built in. No server, no account, no build step: the whole app is a single HTML file.

**Live:** <https://mbitc.github.io/hype-feed/>

## What it does

- **Full-screen vertical feed.** Swipe through photos, one per screen, in a mobile-first PWA-style layout.
- **Local and global.** If you allow location access, the feed also pulls posts from your area, using a coarse geohash (precision 5, roughly a few kilometres). Without it, you see the global stream only.
- **Post in a few taps.** Take a photo or pick one from the gallery, add a caption and broadcast it.
- **Tip with Lightning.** Zap any post from the lightning button on its card.
- **Shareable links.** Every post has a link of the form `#nostr-<event id>` that opens that exact post.
- **Automatic identity.** On first load the app uses your Nostr browser extension (NIP-07) if there is one; otherwise it generates a local keypair.

## How it works

| Area | Details |
| --- | --- |
| Protocol | Nostr over raw WebSockets (no client library for transport) |
| Relays | `wss://nos.lol`, `wss://relay.nostr.band`, `wss://purplerelay.com`, `wss://relay.damus.io`, with automatic reconnection and back-off |
| Posts | Kind 1 events tagged `t=hypefeed`, plus a `g` geohash tag when location is available |
| Images | Compressed in the browser (max 1200 px, JPEG), uploaded to nostr.build; falls back to an embedded data URL if the upload fails |
| Signing | `nostr-tools` 1.17.0 for local keys, or `window.nostr` for extensions |
| Profile | Kind 0 metadata carrying your Lightning address (`lud16`); existing profile fields are preserved when you save |
| Zaps | NIP-57: kind 9734 zap request, LNURL-pay invoice, kind 9735 receipt |

The feed keeps at most 40 cards in the DOM at once to stay light on low-end phones.

## Zaps in detail

1. Tap the lightning button on a post and choose 21, 100, 500 or 1000 sats, or enter your own amount.
2. The app resolves the author's Lightning address from their Nostr profile and requests an invoice.
3. Before showing it, the app checks that the invoice amount matches what you chose, and respects the wallet's minimum and maximum limits.
4. Pay it by whichever route suits you:
   - **Open in Wallet App** (a `lightning:` link, best on mobile)
   - **QR code** or **copy invoice**
   - **WebLN** browser extension, where available
5. The app detects payment by, in order of preference:
   - WebLN confirming the payment;
   - the wallet provider's `verify` URL (LUD-21), polled every few seconds;
   - the signed zap receipt arriving via the relays;
   - a manual **I paid** button, when none of the above is available.

### The counters

- **Post counter** (on each card): sats received by that post, taken from zap receipts on the relays. Your own confirmed zaps are also shown immediately, and remembered on this device.
- **Top badge**: the total of zap receipts addressed to *your* key. It is not your wallet balance. Lightning addresses are receive-only, so the app cannot see what your wallet holds.

### Receiving zaps

Open the wallet panel (top right), enter your Lightning address (for example `you@example.com`) and save. It is published to your Nostr profile so that others can zap you.

## Known limitations

- **Zap receipts depend on the recipient's wallet provider.** If the provider does not support NIP-57, payments still arrive, but nobody sees a public counter. The zap sheet tells you when this is the case.
- **Locally confirmed tips are per device.** Other people cannot see them.
- **Receipts are not yet checked against the provider's `nostrPubkey`.** They are checked for a valid signature, a matching amount and the right recipient, but a determined attacker could still forge one. Treat the badge as informational.
- **The local private key lives in `localStorage`.** Use a Nostr extension if you care about the key. "Generate new identity" discards the local key permanently.
- **Posts depend on the relays.** Old events may be pruned, so a shared link to an old post may stop working.
- **Opened from a downloaded file (`content://` or `file://`), the browser may not persist local data.** Use the hosted version for anything that needs to be remembered.

## Running it

There is nothing to build.

```bash
# any static server will do
python3 -m http.server 8080
# then open http://localhost:8080/hype-feed.html
```

To deploy, publish `hype-feed.html` (as `index.html`) on any static host, such as GitHub Pages. If the browser keeps serving an old copy, append a query string before the hash, for example `?v=2#nostr-…`.

Third-party scripts are loaded from CDNs: Tailwind CSS, Lucide icons, `nostr-tools` 1.17.0 and `qrcodejs`. An internet connection is required.

## Roadmap

- **Nostr Wallet Connect (NIP-47)**: connect a wallet to show its real balance, pay zaps without leaving the app and list recent payments.
- Verify zap receipts against the provider's `nostrPubkey`.
- Sync confirmed tips across devices.

## Licence

No licence has been chosen yet.
