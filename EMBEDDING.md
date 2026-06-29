# Embedding EasySupport (Flutter SDK) — integrator checklist

A one-page reference for engineers and security/app-review reviewers integrating
`easysupport_sdk` into a Flutter app. For the public API, see [README.md](README.md).

## 1. Network egress (what to allow)

The SDK only talks to **your own EasySupport workspace host** — the `baseUrl`
(and `apiBaseUrl`, if set) you pass to `EasySupport.init`. There are no
hard-coded third-party endpoints.

| Traffic | Destination | Protocol |
|---|---|---|
| REST (bootstrap, customer session, message history, uploads) | `apiBaseUrl ?? baseUrl` | HTTPS |
| Live chat | same origin, socket.io path `/socket.io/` (or `webSocketChannelUrl` when `useWebSocketChannel: true`) | WSS (websocket; falls back to polling) |

If your app or MDM enforces an egress allowlist (App Transport Security on iOS,
network-security-config on Android), allowlist **only your HTTPS workspace
origin**. No CDN or analytics host is required by this SDK.

> **CSP note:** Content-Security-Policy is a *web* concept and does not apply to
> the native Flutter SDK. If you are embedding the **JS web widget** on a
> website instead, allow `connect-src` to your EasySupport API host (and any
> socket.io host) per that widget's own docs — it is a different artifact from
> this package.

## 2. The `baseUrl` rule (most common mistake)

`baseUrl` is **required** and has **no default**. Always pass your real,
publicly-resolvable **HTTPS** host (e.g. `https://api.easysupport.io`).

- ❌ A LAN IP (`http://192.168.x.x:3000`) — works only on the dev's network and
  ships mixed-content / unreachable builds to customers.
- ❌ Plain `http://` — blocked by ATS / cleartext policies on modern OSes.
- ✅ `https://<your-workspace-host>`.

## 3. On-device data

The SDK persists only non-sensitive identifiers via `SharedPreferences`:
`easy_support_customer_id`, `easy_support_chat_id`, `easy_support_channel_id`.
No auth tokens or message contents are written to local storage by the SDK.

## 4. Permissions

If `isMediaEnabled` is `true` (default), the in-SDK attachment flow uses the
host app's image-picker / file-access permissions. Declare the relevant
`NSPhotoLibraryUsageDescription` (iOS) / storage or photo permissions (Android)
in your app if you keep media enabled; set `isMediaEnabled: false` to disable.

## 5. Minimum integration

```dart
await EasySupport.init(
  const EasySupportConfig.essentials(
    baseUrl: 'https://api.easysupport.io',
    channelToken: 'pk_live_xxx',
  ),
);
// later, from a widget:
if (EasySupport.isReady) EasySupport.open(context);
```
