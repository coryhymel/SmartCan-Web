# SmartCan-Web

A single-page browser "middle-man" for the SmartCan project. It listens for
events published by a Spark Core device and saves each scanned barcode to a
[Parse](https://parseplatform.org/) database.

## How it works

`JavascriptListener.html` does three things when you open it in a browser:

1. Opens a [Server-Sent Events](https://developer.mozilla.org/en-US/docs/Web/API/EventSource)
   stream to the Spark Cloud for one device:
   `https://api.spark.io/v1/devices/<deviceID>/events/?access_token=<accessToken>`
2. Listens for events named `Uptime`. Each event's `data` field is JSON, and
   its `data` property holds the barcode value.
3. Saves that value as a new `Barcode` object (field: `barcode`) in Parse.

The `#output` paragraph on the page shows the connection and save status. If
the save fails, an error message appears.

```
Spark Core ──publish "Uptime"──▶ Spark Cloud ──SSE──▶ JavascriptListener.html ──▶ Parse (Barcode)
```

## Setup

1. Open `JavascriptListener.html` and replace the `XXXXXXXXXXX` placeholders:

   | Placeholder | Where | Value |
   | --- | --- | --- |
   | `Parse.initialize(appId, jsKey)` | first line of the script | Parse application ID and JavaScript key |
   | `deviceID` | script | ID of the Spark Core |
   | `accessToken` | script | Spark Cloud access token |

2. Open the file in a browser that supports `EventSource` and leave the tab
   open. The page relays events only while it stays open.

There is no build step. jQuery 1.7.2 and the Parse JS SDK 1.2.18 load from
their CDNs.

> **Security:** these credentials sit in plain text in the page. Don't commit
> real values, and don't host a filled-in copy anywhere public.

## Known limitations

This is legacy code, and some of the services it depends on have changed or
shut down since it was written:

- **Spark is now Particle.** `api.spark.io` has been replaced by
  `api.particle.io`, and Particle currently expects the access token in an
  `Authorization: Bearer` header rather than a query parameter.
- **Parse's hosted service shut down in 2017.** You would need a self-hosted
  [Parse Server](https://github.com/parse-community/parse-server) and a current
  Parse JS SDK, pointed at it with `Parse.serverURL`.
- **The scripts load over `http://`.** Browsers block these as mixed content
  when the page is served over HTTPS.
- **Some referenced files and elements are missing.** `css/reset.css` and
  `css/styles.css` are not in the repo, and the success handler shows a
  `.success` element that the page doesn't contain.
- **The event name is `Uptime`**, even though the payload is a barcode. The
  device firmware must publish under that name.
