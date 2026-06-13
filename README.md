# easysupport_sdk

Official **Flutter** SDK for [EasySupport](https://easysupport.io) — drops a
voice-first help-desk + real-time chat surface into any Flutter app. Bootstraps
a support channel, manages the customer session, and connects a live chat
socket to your EasySupport workspace.

> Looking to embed the **web** widget on a website? That's a separate artifact
> (the JS widget shipped from the EasySupport portal) and has its own
> Content-Security-Policy guidance. This package is the **Dart/Flutter** client.

## Install

```yaml
# pubspec.yaml
dependencies:
  easysupport_sdk:
    git:
      url: https://github.com/eFone/easysupport_sdk.git
      ref: main   # or pin a tag
```

```dart
import 'package:easysupport_sdk/easysupport_sdk.dart';
```

## Quick start

```dart
// 1. Initialize once (e.g. in main() or after login).
await EasySupport.init(
  const EasySupportConfig.essentials(
    baseUrl: 'https://api.easysupport.io', // YOUR workspace API host (HTTPS!)
    channelToken: 'pk_live_xxx',           // channel/public token from the portal
    name: 'Jane Doe',                      // optional — pre-fills the customer
    email: 'jane@example.com',             // optional
  ),
);

// 2. Open the support surface from any widget with a BuildContext.
ElevatedButton(
  onPressed: () => EasySupport.open(context),
  child: const Text('Get help'),
);
```

`init()` is resilient: if channel bootstrap fails it logs (via `debugPrint`) and
leaves the SDK un-ready rather than throwing. Check readiness before opening:

```dart
if (EasySupport.isReady) EasySupport.open(context);

// Or react to state changes:
EasySupport.stateListenable.addListener(() {
  debugPrint('EasySupport ready: ${EasySupport.isReady}');
});
```

`open()` accepts optional `heightFactor` (0–1, default `0.9`), `useSafeArea`
(default `true`), and an `onError` callback:

```dart
await EasySupport.open(
  context,
  heightFactor: 0.85,
  onError: (error) => debugPrint('EasySupport error: $error'),
);
```

## Configuration — `EasySupportConfig`

| Field | Type | Default | Notes |
|---|---|---|---|
| `baseUrl` | `String` | **required** | Your EasySupport workspace API origin. **Always use your real HTTPS host** — never a LAN IP / `http://` address (a hard-coded `192.168.x` baseUrl is the classic integration footgun). |
| `channelToken` | `String` | **required** | Public channel token from the portal. Must be non-empty. |
| `apiBaseUrl` | `String?` | `null` | Override the REST host if it differs from `baseUrl`. |
| `channelKey` | `String?` | `null` | Optional channel key. |
| `name` / `email` | `String?` | `null` | Pre-fill the customer identity. |
| `widgetTitle` | `String?` | `null` | Header title override. |
| `webViewUrl` | `String?` | `null` | Optional web-view fallback URL. |
| `autoOpen` | `bool` | `true` | Auto-open behavior. |
| `isEmojiEnabled` | `bool` | `true` | Emoji picker. |
| `isMediaEnabled` | `bool` | `true` | Media/attachment upload. |
| `useWebSocketChannel` | `bool` | `false` | Use the raw WebSocket channel instead of socket.io. |
| `webSocketChannelUrl` | `String?` | `null` | URL for the raw WebSocket channel. |
| `webSocketChannelSocketIoMode` | `bool` | `false` | socket.io-compatible framing over the WS channel. |
| `socketPath` | `String?` | `/socket.io/` | socket.io path. |
| `socketNamespace` | `String?` | `null` | socket.io namespace. |
| `socketTransports` | `List<String>` | `['websocket', 'polling']` | Allowed transports. |
| `socketQuery` | `Map<String, dynamic>` | `{}` | Extra connection query params. |
| `socketAuth` | `Map<String, dynamic>` | `{}` | socket.io auth payload. |
| `additionalHeaders` | `Map<String, String>` | `{}` | Extra HTTP headers on REST calls. |

A `fromJson` factory accepts both `snake_case` and `camelCase` keys, so you can
hydrate the config from a remote-config payload.

## Network & data

- **Hosts contacted:** your `baseUrl` / `apiBaseUrl` over HTTPS (REST) and the
  same origin for the chat socket (socket.io path `/socket.io/` by default, or
  the raw WebSocket URL when `useWebSocketChannel: true`). No third-party
  endpoints are contacted by the SDK itself.
- **Local persistence (`SharedPreferences`):** only non-sensitive identifiers —
  `easy_support_customer_id`, `easy_support_chat_id`, `easy_support_channel_id`.
  No tokens or message bodies are persisted on-device by the SDK.

## See also

- [`EMBEDDING.md`](EMBEDDING.md) — host/egress checklist for app-review and
  network-policy reviewers, plus the HTTPS-not-LAN baseUrl rule.

## License

MIT — see [LICENSE](LICENSE).
