---
name: publish-to-jigsaw
description: Use when the person wants to publish, host, deploy or share a small web app, or wants an app that named people sign in to with shared data. Covers how to build an app that works on Jigsaw and how to publish it with the Jigsaw connector.
---

# Publish to Jigsaw

Every Jigsaw app is private: only the owner and the people they add by email can open it. If the person wants a public website that anyone can open without signing in, Jigsaw is the wrong place; say so.

Read this before writing the app, not after: an app built to these rules works on Jigsaw the first time.

Jigsaw hosts small apps and shares them by email, like a doc. To publish:
- If you can run shell commands, call get_upload_link. It gives an upload address and a ticket. Zip the app's folder, leaving out node_modules and .git, and POST it to that address with the ticket as a Bearer token, writing the command for the machine you are on. The files go from disk, so you do not retype them. Always prefer this.
- If you cannot run commands, use publish_app and pass every file.

Build a PAGE APP unless the app truly needs its own server:
- A page app is static files with an index.html at the top level. No build step, no server code, no login code.
- Jigsaw signs every visitor in before the page loads. Call GET /_jigsaw/me for { email, role }. Role is "owner", "editor" or "viewer". The visitor can change data when the role is "owner" or "editor".
- The page may use inline or external scripts and styles; Jigsaw adds no content policy to an app's own files. Refer to Jigsaw's API with paths that start with a slash, as written here.
- Store data with Jigsaw's data API, on the app's own address, using fetch with no extra headers:
    GET    /_jigsaw/data/<list>            -> { records: [{ id, data, createdBy, createdAt, updatedAt }] }
    POST   /_jigsaw/data/<list>            body: any JSON object -> the new record
    GET    /_jigsaw/data/<list>/<id>
    PUT    /_jigsaw/data/<list>/<id>       body: the replacement JSON object
    DELETE /_jigsaw/data/<list>/<id>
  <list> is a lowercase name you choose, like "tasks". Send JSON with Content-Type: application/json.
  A record is { id, data, createdBy, createdAt, updatedAt }: data is exactly the object you sent, createdBy is an email, the dates are ISO strings.
  POST answers 201 with the new record. PUT answers 200 with the updated record; it replaces the whole object, so to change one field send the old data with that field changed. DELETE answers 204 with no body.
  A list returns records oldest first, up to 1000 at a time (or ?limit=N), with "next": when it is not null, ask again with ?after=<next> for the following page. A record can be up to 100 KB.
  The data belongs to the app: everyone with access sees the same lists.
  Errors answer with a status of 400 or above and { "error": "what went wrong" }.
  Jigsaw enforces roles: viewers can read, editors and the owner can write. A viewer's write gets 403, so hide edit controls for viewers.
- To call an outside API that needs a secret key, never put the key in the page. Call:
    POST /_jigsaw/fetch  body: { "url": "https://api.example.com/v1/...", "method": "POST", "headers": { "Content-Type": "application/json" }, "body": "..." }
  Do not send the key or any placeholder for it. The owner pastes the key into Jigsaw (app settings, Keys) and says which host it is for and how that service expects it (usually the Authorization: Bearer header). Jigsaw adds it and returns the outside response. Calls to a host with no key saved are refused with 403, so tell the owner the exact host to save the key for.

SERVER APP (only when a server is needed): include a package.json with a "start" script.
- The server must listen on process.env.PORT and process.env.HOST.
- Jigsaw installs the packages (install hooks do not run), runs "npm run build" if there is a "build" script, then "npm start". Do not upload node_modules.
- Keep files and databases in process.env.JIGSAW_DATA_DIR. It outlives restarts and new versions; the app's own folder does not.
- Every request has already passed sign-in. Who the visitor is arrives in headers: X-Jigsaw-Email, X-Jigsaw-Role ("owner", "editor" or "viewer") and X-Jigsaw-User-Id (stable per person). The same facts are in X-Jigsaw-Pass, a signed JWT you can verify with the keys at process.env.JIGSAW_JWKS_URL (issuer process.env.JIGSAW_ISSUER, audience process.env.JIGSAW_APP_ID).
- The app's code must respect the role itself. The app sees the address visitors use (Host), available as process.env.JIGSAW_APP_URL too.
- WebSockets work. Cookies the app sets are its own.
- Keys the owner pasted arrive as environment variables.
- The first start after publishing can take a minute or two while packages install.

To publish a new app into a team the person builds for, pass the team's handle as "team". list_apps shows which team an app belongs to.

After publishing, tell the person the app's address and the sharing address, which is https://jigsawapps.com/apps/<app name>/share.
