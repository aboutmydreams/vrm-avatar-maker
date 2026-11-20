# Avatar handoff (v1)

Any website can let its users make or edit their avatar in VRM Avatar Maker and get the result back
directly: no download, no upload. The site opens the maker in a popup; the two pages pass the avatar
between them in the browser (`postMessage`). Nothing goes through a server, and the maker needs no
account or token: your site saves the avatar with its own login, as it would an upload.

Try it: `web/handoff/demo.html` (on the live site: https://avatars.somanyrooms.com/handoff/demo.html).

## How it goes

1. Your page opens the maker, with your page's origin in `host`:
   `https://avatars.somanyrooms.com/editor/?host=https://your.site`
2. The maker says `ready`. You answer with `start`, optionally with the user's current avatar.
3. The maker shows a **Send to your.site** button. When the user presses it, you get `avatar`
   with the VRM file.
4. You answer `received` (or an error); the maker shows it and closes the popup.

## Messages

Every message is an object with `type` and `version: 1`. The maker only sends to the exact origin
given in `host`, and only accepts messages from that origin and from the window that opened it. Check
`event.origin` on your side the same way.

| Direction | `type` | Other fields |
|---|---|---|
| maker → site | `avatar-handoff:ready` | `maker` (name), `formats: ["vrm1"]`. Repeated every 0.5 s (10 s at most) until you send anything. |
| site → maker | `avatar-handoff:start` | All optional. `vrm`: ArrayBuffer of a VRM made in the maker (it carries its recipe and reopens fully editable); or `recipe`: the recipe object; `name`: the avatar's name; `label`: your site's name for the button ("Send to *label*", default: your host name). |
| maker → site | `avatar-handoff:avatar` | `vrm`: ArrayBuffer (VRM 1.0, `.vrm`); `format: "vrm1"`; `name`. |
| site → maker | `avatar-handoff:received` | `ok`: true, or false with `error` (shown to the user). |
| maker → site | `avatar-handoff:closed` | The maker's window is closing. |

A VRM made elsewhere can't be edited in the maker (it doesn't know its parts); if you send one, the
maker says so and starts from its usual avatar.

## Your side (about 20 lines)

```js
const MAKER = "https://avatars.somanyrooms.com";
function editAvatar(currentVrm /* ArrayBuffer or null */) {
  const popup = window.open(`${MAKER}/editor/?host=${encodeURIComponent(location.origin)}`, "avatar-maker", "width=1200,height=800");
  let started = false;
  async function onMessage(e) {
    if (e.origin !== MAKER || e.source !== popup) return;
    const msg = e.data;
    if (msg.type === "avatar-handoff:ready" && !started) {
      started = true;
      const start = { type: "avatar-handoff:start", version: 1, label: "Your Site" };
      if (currentVrm) start.vrm = currentVrm.slice(0);  // a copy: it's transferred
      popup.postMessage(start, MAKER, start.vrm ? [start.vrm] : []);
    } else if (msg.type === "avatar-handoff:avatar") {
      try {
        await saveAvatar(new Blob([msg.vrm], { type: "model/gltf-binary" }), msg.name);  // your upload
        popup.postMessage({ type: "avatar-handoff:received", version: 1, ok: true }, MAKER);
      } catch (err) {
        popup.postMessage({ type: "avatar-handoff:received", version: 1, ok: false, error: err.message }, MAKER);
      }
    } else if (msg.type === "avatar-handoff:closed") window.removeEventListener("message", onMessage);
  }
  window.addEventListener("message", onMessage);
}
```

Call `editAvatar` from a click (browsers block popups otherwise). An iframe works the same way
(`window.parent` instead of `window.opener`), but a popup gives the maker the room it needs.

## Rules the maker keeps

- `host` must be an `https:` origin (`http:` only for localhost). Anything else and the handoff is off.
- It sends only to that origin, only when the user presses Send, and the button names the
  destination.
- It reads only the recipe from a VRM you send (never runs anything from it), and replaces item ids
  it doesn't know with defaults.
- The avatar stays within the usual budget (40,000 triangles); Send is off when it's over, like
  Download.

## Not yet: apps that aren't web pages

A VR headset app or a game can't receive a `postMessage`. A later version may add
`?upload=<one-time upload URL>&return=<URL>`: the maker sends the VRM to that single-use, short-lived
address (it must allow our origin, CORS) and then goes to `return`. Ask if you need it.
