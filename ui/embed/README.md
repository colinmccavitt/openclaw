# OpenClaw Embed Widget

A self-contained chat widget you can drop into any website (Squarespace, Webflow, custom HTML, etc.) that connects visitors directly to your OpenClaw AI agent.

## How it works

Each website visitor gets a private session stored in their browser's `localStorage`. When they send a message, it goes through your OpenClaw Gateway via a WebSocket connection and is handled by your AI agent. Responses stream back in real time.

## Quick start

### 1. Make your gateway publicly reachable

Your Gateway runs locally on port `18789`. Visitors' browsers need to reach it over **WSS** (secure WebSocket — required because Squarespace is HTTPS).

Options (pick one):

| Method | Command | Notes |
|---|---|---|
| **Cloudflare Tunnel** (recommended, free) | `cloudflared tunnel --url http://localhost:18789` | Stable HTTPS URL, no port forwarding needed |
| **ngrok** | `ngrok http 18789` | Free tier = random URL on each restart |
| **VPS + nginx** | Point your domain at the server, proxy to port 18789 | Best for production |

### 2. Get your auth token

```bash
pnpm openclaw config get gateway.auth.token
```

If the output is empty, generate one:

```bash
pnpm openclaw config set gateway.auth.token "$(openssl rand -hex 32)"
```

### 3. Allow your Squarespace origin

Replace `yoursite.squarespace.com` with your actual domain:

```bash
pnpm openclaw config set gateway.controlUi.allowedOrigins '["https://www.yoursite.squarespace.com"]'
```

If you have a custom domain on Squarespace, add both:

```bash
pnpm openclaw config set gateway.controlUi.allowedOrigins \
  '["https://www.yoursite.squarespace.com","https://www.yourcustomdomain.com"]'
```

Then restart the gateway via the **OpenClaw Mac app** (click the menu bar icon → Restart).

### 4. Edit and paste the widget

Open `squarespace-widget.html` and fill in the `CFG` block near the top:

```js
const CFG = {
  gatewayUrl:      'wss://abc123.trycloudflare.com',  // from step 1
  authToken:       'your-token-here',                 // from step 2
  botName:         'Assistant',
  welcomeMessage:  'Hi! How can I help you today?',
  primaryColor:    '#4F46E5',
};
```

Copy the **entire file contents** and paste into:

> Squarespace → Settings → Advanced → Code Injection → Footer

Save, then visit your site — a chat bubble will appear in the bottom-right corner.

## Customization

All visual properties are CSS custom properties. You can override them by adding a `<style>` block anywhere on the page:

```css
:root {
  --ocw-primary:      #0ea5e9;   /* button / header / user bubbles */
  --ocw-primary-dark: #0284c7;   /* hover state */
  --ocw-bg:           #ffffff;   /* panel background */
  --ocw-surface:      #f8fafc;   /* message area background */
  --ocw-border:       #e2e8f0;
  --ocw-text:         #0f172a;
  --ocw-text-muted:   #64748b;
  --ocw-bot-bubble:   #f1f5f9;   /* AI reply bubbles */
  --ocw-radius:       16px;      /* panel corner radius */
}
```

## Browser requirements

The widget uses the **Web Crypto Ed25519** API for device identity (same as the Control UI). This requires:

- Chrome 113+ (May 2023)
- Firefox 130+ (Sep 2024)
- Safari 17+ (Sep 2023)

Older browsers fall back to token-only authentication. The gateway must have `gateway.controlUi.allowInsecureAuth` set to `true` to accept those connections.

## Security note

The `authToken` is visible in your page source. Anyone who inspects the source can extract it and connect to your gateway. For a public chat widget this is intentional — you _want_ visitors to be able to chat. To restrict who can connect, consider IP allowlisting at the reverse proxy / Cloudflare Tunnel level.
