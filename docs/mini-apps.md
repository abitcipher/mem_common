# Building a MEM Mini App

A **Mini App** is a small app that runs inside MEM and talks to your **service**, end to end.
Users install it from a signed package file. Its screen shows what your service sends, and every
button on it becomes a request to your service. Requests and answers travel like MEM messages:
over the local network, Bluetooth, other MEM phones (couriers) or the hub. A phone without
Internet can still use your Mini App.

```
phone: MEM ── Mini App screen ──┐
                                │  end to end, any channel MEM has
                                ▼
your server: mem-botd ── local WebSocket (JSON) ── your service (any language)
```

This guide builds one from scratch. The complete working example is the lunch poll in
`samples/miniapp-poll/` (in the MEM source repository). Read this guide first, then use
the sample as a template.

> Mini Apps are being tested. In the app they are off by default (Settings → Experimental →
> Mini Apps). The package format and the local link may still change before 1.0.

## Contents

1. [How it works](#1-how-it-works)
2. [What you need](#2-what-you-need)
3. [Step 1: the service's MEM node](#3-step-1-the-services-mem-node)
4. [Step 2: actions and their schemas](#4-step-2-actions-and-their-schemas)
5. [Step 3: the screen](#5-step-3-the-screen)
6. [Step 4: the service](#6-step-4-the-service)
7. [Step 5: pack and sign](#7-step-5-pack-and-sign)
8. [Step 6: try it on one machine](#8-step-6-try-it-on-one-machine)
9. [Step 7: install it on a phone](#9-step-7-install-it-on-a-phone)
10. [Optional: a web page](#10-optional-a-web-page)
11. [Updating a Mini App](#11-updating-a-mini-app)
12. [Limits](#12-limits)
13. [Errors](#13-errors)
14. [Privacy rules you get for free, and must not work around](#14-privacy-rules-you-get-for-free-and-must-not-work-around)
15. [Checklist before you publish](#15-checklist-before-you-publish)

## 1. How it works

- **The package** (`.memext`) is one file signed with your developer key. It holds:
  - the Mini App's id, name, version and description;
  - its screen, written in [Remote Flutter Widgets](https://pub.dev/packages/rfw) text
    (`.rfwtxt`);
  - its actions: the name of each request it can send, with a JSON Schema for the request's
    data;
  - optionally, a JSON Schema for the state your service sends back;
  - your service's pairing card (`memcard0:…`), which tells phones where to send requests;
  - optionally, web origins for a web page version;
  - a WebAssembly component. The package format requires one, but a Mini App's screen does
    not run it today, so any valid component will do; step 5 shows how to use the sample's.
- **The persona.** When a user installs your Mini App, MEM creates an identity for that user
  and that Mini App only. Your service sees a 64-hex-digit persona id, never the user's MEM
  identity. Two services cannot link a user, because they share no identity, key or address.
- **Linking.** On install, the persona introduces itself to your service through the card. Your
  node accepts at once. Until it has, requests wait on the phone ("Linking…").
- **Requests** go from the phone to your service. The phone checks each request's data against
  the action's schema before anything is sent.
- **Updates** go from your service to the phone. An update either answers a request or is a
  push, which nobody asked for. A `state` update merges into the screen's `data`; a `data`
  update is delivered as is, to a web page.
- **Your service** never handles keys or channels. A small node, `mem-botd`, runs next to it
  and does all of MEM. Your service talks JSON with it over a WebSocket on the same machine.

## 2. What you need

- **Rust** (the repository's pinned toolchain), to build the tools:

  ```sh
  cargo build --release -p mem-botd -p mem-extpack
  # target/release/mem-botd, target/release/mem-extpack
  ```

- **The WebAssembly target**, for the package's component:
  `rustup target add wasm32-unknown-unknown`.
- **A server** for your service and `mem-botd`. They run on the same machine, because the
  link between them is loopback only. Any language works if it has a WebSocket client and
  JSON. The sample is Elixir/Phoenix and includes a ready client, `MemBot`.
- **A hub address** your node and your users can reach, for example the operator's
  `wss://meshm.artcomplect.com/v1`. With `--lan`, phones on the same network also reach your
  node directly.
- **A phone with MEM**, or the command-line `phone` example (step 6) for testing.

## 3. Step 1: the service's MEM node

Create a profile, a token for the local link, and the card that goes into your package:

```sh
mkdir -p ~/mem-service && cd ~/mem-service
head -c 24 /dev/urandom | base64 > botd.token
chmod 600 botd.token
mem-botd card --dir profile --name "Lunch poll" > card.txt   # prints memcard0:…
```

The profile directory holds your service's MEM identity. **Back it up and keep it.** The card
is derived from it, and every installed copy of your Mini App sends requests to that identity.
If you lose the profile, a new package with a new card is needed and users must reinstall.

Run the node:

```sh
mem-botd run --dir profile --token-file botd.token \
  --hub wss://meshm.artcomplect.com/v1 --name "Lunch poll" [--lan] [--listen 127.0.0.1:4100]
```

It listens on `127.0.0.1:4100` by default, accepts one connection at a time, and admits only
`Authorization: Bearer <token>`. Run it as a service (systemd or similar) with the same profile
every time.

## 4. Step 2: actions and their schemas

Decide what the screen can ask your service. Each **action** is a name and a JSON Schema for
its data. The phone checks the data before sending and refuses anything that does not match,
so your service gets only well-formed input. Still treat it as untrusted.

`vote.schema.json`:

```json
{
  "type": "object",
  "required": ["choice"],
  "properties": { "choice": { "enum": ["soup", "pasta", "salad"] } },
  "additionalProperties": false
}
```

`results.schema.json`, for an action without data:

```json
{ "type": "object", "maxProperties": 0 }
```

Optionally, a **state schema** describes what your answers must look like. The phone checks
every `state` update against it and drops updates that do not match.

```json
{
  "type": "object",
  "required": ["votes"],
  "properties": {
    "votes": {
      "type": "object",
      "properties": {
        "soup":  { "type": "integer", "minimum": 0 },
        "pasta": { "type": "integer", "minimum": 0 },
        "salad": { "type": "integer", "minimum": 0 }
      },
      "additionalProperties": false
    },
    "mine": { "enum": ["soup", "pasta", "salad"] }
  },
  "additionalProperties": false
}
```

Rules for all data:

- **No floating-point numbers.** MEM's wire format (CBOR) carries integers, text, booleans,
  null, lists and objects. Send prices as cents, ratings as integers, and so on.
- **At most eight levels of nesting.**
- **Size:** request data at most 16 KiB, an update's body at most 30 KiB, at most 1024 items
  in any list or object.
- **Lifetime.** An action may give its requests a shorter lifetime, from one hour to 30 days
  (`vote=vote.schema.json@3600` in step 5). Use it when a stale request makes no sense, such as
  "order lunch" the next day. Without one, requests live as long as MEM messages do.

Schemas use standard JSON Schema; draft 2020-12 is the default. Nothing is fetched from the
network, so `$ref` works only inside the same schema.

## 5. Step 3: the screen

The screen is a Remote Flutter Widgets library in text form. MEM renders it with the latest
state from your service.

`ui.rfwtxt`:

```
import core.widgets;
import core.material;

widget root = Padding(
  padding: [16.0, 16.0, 16.0, 16.0],
  child: Column(
    crossAxisAlignment: "start",
    children: [
      Text(text: "What's for lunch?"),
      SizedBox(height: 12.0),
      choice(name: "soup",  label: "Soup",  count: data.votes.soup),
      choice(name: "pasta", label: "Pasta", count: data.votes.pasta),
      choice(name: "salad", label: "Salad", count: data.votes.salad),
    ],
  ),
);

widget choice = Padding(
  padding: [0.0, 4.0, 0.0, 4.0],
  child: ElevatedButton(
    onPressed: event "vote" { choice: args.name },
    child: Text(text: [args.label, ": ", args.count]),
  ),
);
```

How it fits together:

- **`root`** is the widget MEM shows.
- **`data`** is the state your service sent. Each key of a `state` update replaces that key in
  `data`, and a key sent as `null` removes it. State lives in memory only, so your service
  should send the whole state it wants shown when it answers a request (the sample's
  `results` action does that when the screen opens).
- **`event "name" { … }`** sends a request: the event's name must be one of your actions, and
  its arguments become the request's data. Events with other names are ignored.
- **`Text`** also shows numbers and yes/no values (as ✓ and ✗) from the state, and joins a
  list such as `[args.label, ": ", args.count]`.

Widget libraries you may import:

| Library | What it has |
|---|---|
| `core.widgets` | rfw's core widgets (layout, `Text`, `SizedBox`, `Padding`, `Row`, `Column`, …) except `Image` and `Container` |
| `core.material` | rfw's Material widgets (`ElevatedButton`, `Card`, `ListTile`, …) except `DrawerHeader` |
| `mem.inputs` | `TextInput` (`label`, `hint`, `number`, `maxLength`, `onSubmitted`, `onChanged`), `Checkbox` (`label`, `value`, `onChanged`), `Switch` (`label`, `value`, `onChanged`), `Progress` (`value` 0–1, or none for an ongoing one). Each reports what the user entered as `value` in its event's arguments. |

`Image`, `Container` and `DrawerHeader` are left out because they can load an image from any
URL. That would tell a server the phone's address without the user's consent. A Mini App's
screen cannot reach the network at all.

An input example: a text field that sends `{"value": "…"}` with the action `rename`:

```
import mem.inputs;
…
TextInput(label: "Name", maxLength: 40, onSubmitted: event "rename" {}),
```

Keep the screen under 64 KiB.

## 6. Step 4: the service

Your service connects to `mem-botd` and exchanges JSON objects, one per WebSocket text frame.

**Connect** to `ws://127.0.0.1:4100/` with the header `Authorization: Bearer <token>`. The node
greets you:

```json
{"type": "hello", "card": "memcard0:…"}
```

**Requests** arrive like this:

```json
{"type": "request", "user": "<64 hex>", "id": "<32 hex>", "action": "vote",
 "data": {"choice": "pasta"}}
```

`user` is the persona of that user for your service. Use it as the user's key in your
database. It stays the same for as long as they keep your Mini App installed.

**Answer** with an update that names the request:

```json
{"type": "update", "user": "<64 hex>", "in_reply_to": "<32 hex>", "kind": "state",
 "body": {"votes": {"soup": 0, "pasta": 1, "salad": 0}, "mine": "pasta"}, "ref": 1}
```

**Push**, with no `in_reply_to`, to tell a user something nobody asked for, for example
"your order is ready". MEM allows at most 32 pushes a day to each user. Users can mute a Mini
App.

```json
{"type": "update", "user": "<64 hex>", "kind": "state", "body": {"status": "ready"}, "ref": 2}
```

`kind` is `state` (merged into the screen's `data`, checked against the state schema) or `data`
(delivered as is, for a web page). `ref` is yours: the node answers each update with
`{"type": "ok", "ref": 1}` once it has accepted it for sending, or
`{"type": "error", "ref": 1, "reason": "…"}`, for example when a body is too large or has
floats.

An answer is not delivered at once if the phone is offline. It travels like any MEM message
and arrives when the phone is reachable.

**In Elixir**, copy `samples/miniapp-poll/service/lib/mem_bot.ex` into your project, add it to
your supervision tree and implement one callback:

```elixir
children = [
  {MemBot, url: "ws://127.0.0.1:4100/", token: File.read!("botd.token") |> String.trim(),
   handler: MyService.MiniApp}
]

defmodule MyService.MiniApp do
  @behaviour MemBot

  @impl true
  def handle_request(user, "vote", %{"choice" => choice}) do
    {:reply, %{"votes" => MyService.Poll.vote(user, choice), "mine" => choice}}
  end

  def handle_request(user, "results", _data),
    do: {:reply, %{"votes" => MyService.Poll.tally()}}

  def handle_request(_user, _action, _data), do: :noreply
end

# A push:
MemBot.push(user, %{"status" => "ready"})
```

`MemBot` reconnects by itself when the link drops. **In other languages**, implement the four
messages above with your WebSocket client: wait for `hello`, answer each `request`, check the
`ok`/`error` for each update, and reconnect with a short back-off.

## 7. Step 5: pack and sign

Make a developer key once and keep it safe, like an app signing key. Every update must be
signed with the same key: phones refuse an update signed by another (`developer_changed`), so
**losing the key means users must remove your Mini App and install a new one**. The install
screen shows the key's fingerprint, which is how users recognise your packages.

```sh
mem-extpack keygen dev.key        # prints the developer key's fingerprint
```

Build a component. Until Mini App screens run components, the sample extension's is enough:

```sh
(cd extensions && CARGO_TARGET_DIR=../target/extensions \
  cargo build --release --target wasm32-unknown-unknown)
# target/extensions/wasm32-unknown-unknown/release/mem_ext_poll.wasm
```

Pack. The id is reverse-DNS style and never changes; the version is an integer that rises with
every release:

```sh
mem-extpack pack dev.key mem_ext_poll.wasm ui.rfwtxt lunch.memext \
  com.example.lunch 1 "Lunch poll" \
  --description "Vote for today's lunch." \
  --service-card card.txt \
  --action vote=vote.schema.json@3600 \
  --action results=results.schema.json \
  --state-schema state.schema.json
mem-extpack show lunch.memext     # checks the signature, prints what is inside
```

`pack` refuses schemas that do not compile, sizes over the limits, and a Mini App without
`--service-card`.

## 8. Step 6: try it on one machine

The `phone` example is a MEM phone on the command line. It installs the package, sends one
request and prints the answer. Without a hub address it starts a test hub and prints its
address:

```sh
cargo build -p mem-botd --examples
target/debug/examples/phone lunch.memext vote '{"choice": "pasta"}'
# … test hub at ws://127.0.0.1:<port>/v1
```

In a second terminal, run your node on that hub, then your service:

```sh
mem-botd run --dir profile --token-file botd.token --hub ws://127.0.0.1:<port>/v1
# then start your service
```

The phone prints the answer, for example
`{"mine":"pasta","votes":{"pasta":1,"salad":0,"soup":0}}`. The phone's introduction and its
request waited on the hub until your node came up, exactly as they would for a real offline
user.

## 9. Step 7: install it on a phone

1. In MEM: **Settings → Experimental → Mini Apps** on.
2. Get the `.memext` file onto the phone: a download, a file share, a cable.
3. Open the top-right menu, then **Mini Apps → Install from file**, and choose the file.
4. MEM shows what the package is and may do before installing: its name and version, the
   developer's fingerprint, your service, what each action sends and how long its requests
   live, and the web origins. The user confirms.
5. The Mini App appears in the list, "Linking…" until your node has accepted its persona, then
   "Linked". Open it: the screen shows your state, and the requests on their way are listed
   under it with how the last answer travelled.

The user can mute pushes, open the web page (if any), and remove the Mini App from its menu.

## 10. Optional: a web page

A Mini App may also declare web pages, for richer interfaces than rfw allows:

```sh
mem-extpack pack … --web-origin https://poll.example.com
```

- The first origin is opened, only when the user taps it, and only after a one-time
  disclosure: the site's operator sees the phone's address and what is done on the page.
- The page may navigate only within the declared origins (at most 8). Its site data is deleted
  before and after.
- After each page load MEM provides:
  - `mem.request(action, data)`: sends a request exactly as the screen does. The data is checked
    against the action's schema; the promise gives the request id, or fails with an
    [error key](#13-errors).
  - `mem.onUpdate(handler)`: called with `{kind, body, inReplyTo}` for each update from your
    service (`data` updates included).
- MEM fires `memready` on `window` once both are in place:

```js
window.addEventListener('memready', () => {
  mem.onUpdate(({ kind, body }) => { if (kind === 'state') render(body); });
  mem.request('results', {});
});
```

`samples/miniapp-poll/web/index.html` is the poll as such a page. Requests from the page still
travel through MEM end to end; only loading the page itself uses the Internet.

## 11. Updating a Mini App

- Sign with the **same developer key**, the **same id** and a **higher version**. Phones refuse
  an update signed by another key (`developer_changed`) and a lower version (`downgrade`).
  Before installing, they show "Updates version N to M" and the developer fingerprint.
- Keep old actions working for a while: phones update when their users install the new file,
  not all at once.
- Keep the same service card, which means the same `mem-botd` profile. A new card means a new
  service identity for every user.

## 12. Limits

| What | Limit |
|---|---|
| Package id | 64 bytes |
| Name / description | 48 / 280 bytes |
| Component | 2 MiB |
| Screen (`ui.rfwtxt`) | 64 KiB |
| Actions | 32, names up to 64 bytes |
| One schema / all schemas | 16 KiB / 64 KiB |
| Request lifetime | 1 hour to 30 days (default: as MEM messages) |
| Request data | 16 KiB, no floats, ≤ 8 levels, ≤ 1024 items per list or object |
| Update body | 30 KiB, same rules |
| Pushes | 32 a day to each user |
| Web origins | 8, each up to 256 bytes, `https://` |
| Mini Apps per phone | 64 |

## 13. Errors

What the phone (and `mem.request` on a web page) reports:

| Key | Meaning |
|---|---|
| `invalid` | The data does not match the action's schema. |
| `malformed` | Not JSON, or it has floats or nests deeper than eight levels. |
| `unknown_action` | The package declares no such action. |
| `too_large` | The data is over the size limit. |
| `awaiting_approval` | Your node has not accepted this user's persona yet: is `mem-botd` running and reachable through the hub? |
| `not_installed` | The Mini App was removed. |
| `mini_apps_off` | The user has Mini Apps turned off. |
| `unavailable` | This build of MEM has no Mini Apps. |

When installing:

| Key | Meaning |
|---|---|
| `malformed` | Not a package, or out of bounds. |
| `bad_signature` | The signature does not verify. |
| `not_mini_app` | An extension without a service card. |
| `bad_schema` | A schema does not compile. |
| `downgrade` | A lower version than the installed one. |
| `developer_changed` | Signed by another developer key than the installed version. |

## 14. Privacy rules you get for free, and must not work around

- **Users are personas.** Do not ask users for their MEM identity, phone number or contacts to
  "link accounts". If you need an account, let the user sign in inside your Mini App, and keep
  in mind that this links the persona to that account.
- **No tracking from the screen.** The screen cannot load anything from the network. Do not try
  to smuggle URLs into it. On the web page, the user was told what the site sees; do not add
  third-party trackers there.
- **Pushes are limited and can be muted.** Use them for what the user is waiting for.
- **Treat every request as untrusted.** It passed the schema on an honest phone, but anyone can
  write a client.
- **Your node holds your service's identity.** Protect the profile directory and the token like
  any server secret. The token gives full control over what your service says to its users.

## 15. Checklist before you publish

- [ ] The `mem-botd` profile is backed up; the node runs as a service with a reachable hub.
- [ ] Every action has a strict schema (`additionalProperties: false`, `required`, enums).
- [ ] A state schema matches every answer you send; no floats anywhere.
- [ ] Opening the screen sends a request that returns the full state.
- [ ] The service answers every request, including unknown actions (`:noreply` is fine).
- [ ] The developer key is backed up offline; the version is higher than the last release.
- [ ] `mem-extpack show` verifies the package; the `phone` example gets an answer.
- [ ] Tested on a real phone with Mini Apps on, including offline and through the hub.

## Licence

MEM, its tools and the sample are source-available under the terms in
[`LICENSE.md`](../LICENSE.md). Read them before distributing a package or running a service for
others.
