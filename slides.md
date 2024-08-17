---
theme: default
background: /ostriches-cover.jpg
title: Software Design Patterns for Nostr Applications
favicon: /favicon.png
class: text-center
drawings:
  persist: false
transition: slide-left
mdc: true
---

# Software Design Patterns for Nostr Applications

A Practical Workshop by <a href="https://primal.net/p/npub18vay956v7zs5qtgc65mvn54v96cuvqv6j9fmu4cgfjqkt5vjuvjsc47nzf" target="_blanl">victor@refugio.com.br</a>

<div class="abs-bl mx-14 my-12 flex">
  <img src="/images/casa21.png" class="h-8">
  <div class="ml-3 flex flex-col text-left">
    <div><b>Casa21</b></div>
    <div class="text-sm opacity-50">Aug. 17th, 2024</div>
  </div>
</div>

<div class="abs-br m-6 flex gap-2">
  <a href="https://github.com/vstabile/workshop-nostr-design-patterns" target="_blank" alt="GitHub"
    class="text-xl icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

<style>
  h1 {
    font-weight: 600;
    background: linear-gradient(to right, #fb923c, #ea580c);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
  }
</style>

---
transition: fade
layout: fact
---

# NOSTR

Notes and Other Stuff Transmitted by Relays

<span style="font-size: 1.5rem">&nbsp;</span>

---
transition: fade-out
layout: fact
---

# NOSTR

<s>Notes and Other Stuff Transmitted by Relays</s>

<span class="highlight">Signed Messages on Relays</span>

<style>
  .highlight {
    font-size: 1.5rem;
    font-weight: 700;
  }
</style>

---
transition: fade
---

# <span class="highlight">Signed</span> Messages on Relays

Each user generate his own keypair. No permission needed from any entity. Signatures, public key, and encodings are done according to the Schnorr signatures standard for the curve secp256k1, like Bitcoin.

Software should always store ids and keys in either hex or binary format, but they are usually diplayed to users, copy-pasted, shared and rendered to QR codes as bech32-formatted strings according to [NIP-19](https://github.com/nostr-protocol/nips/blob/master/19.md).

<v-click>

To prevent confusion between private keys, public keys and event ids, they are encoded with differente prefixes:

- <span class="highlight">Public Keys</span>: `npub`
- <span class="highlight">Private Keys</span>: `nsec`
- <span class="highlight">Note IDs</span>: `note`

</v-click>

<v-click>

For example, the hex public key:

- <span class="highlight">HEX Public Key</span>: `3bf0c63fcb93463407af97a5e5ee64fa883d107ef9e558472c4eb9aaaefa459d`
- <span class="highlight">Bech32 Public Key</span>: `npub180cvv07tjdrrgpa0j7j7tmnyl2yr6yr7l8j4s3evf6u64th6gkwsyjh6w6`

</v-click>

---
transition: fade
---

# Signed <span class="highlight">Messages</span> on Relays

The only object type that exists is the event, which has the following format:

```js {all|2|3|4|5|6,7,8,9|10|11|all}
{
  "id": <32-bytes lowercase hex-encoded sha256 of the serialized event data>,
  "pubkey": <32-bytes lowercase hex-encoded public key of the event creator>,
  "created_at": <unix timestamp in seconds>,
  "kind": <integer between 0 and 65535>,
  "tags": [
    [<arbitrary string>...],
    // ...
  ],
  "content": <arbitrary string>,
  "sig": <64-bytes lowercase hex of the signature of the sha256 hash of the serialized event data>
}
```

<div v-click style="margin-top: 1rem">
  <span class="highlight">Common event kinds</span>

  <div style="margin-top: 0.75rem">
    <ul>
      <li><strong>Kind 0</strong>: Metadata <small>(<a href="https://github.com/nostr-protocol/nips/blob/master/01.md" target="_blank">NIP 01</a>)</small></li>
      <li><strong>Kind 1</strong>: Short Text Note <small>(<a href="https://github.com/nostr-protocol/nips/blob/master/01.md" target="_blank">NIP 01</a>)</small></li>
      <li><strong>Kind 3</strong>: Follows <small>(<a href="https://github.com/nostr-protocol/nips/blob/master/02.md" target="_blank">NIP 02</a>)</small></li>
    </ul>
    <ul>
      <li><strong>Kind 5</strong>: Event Deletion <small>(<a href="https://github.com/nostr-protocol/nips/blob/master/09.md" target="_blank">NIP 09</a>)</small></li>
      <li><strong>Kind 14</strong>: Private Direct Messages
        <small>(<a href="https://github.com/nostr-protocol/nips/blob/master/17.md" target="_blank">NIP 17</a>)</small></li>
      <li><a href="https://github.com/nostr-protocol/nips?tab=readme-ov-file#event-kinds" target="_blank">Among others...</a></li>
    </ul>
  </div>
</div>

<style>
  ul {
    width: 50%;
    float: left;
  }
</style>

---
transition: slide-left
---

# Signed Messages <span class="highlight">on Relays</span>

Relays expose a websocket endpoint to which clients can connect, store and subscribe to events.

Clients can send 3 types of messages, which must be JSON arrays, according to the following patterns:

- `["EVENT", <event JSON as defined above>]`<small>, used to publish events.</small>
- `["REQ", <subscription_id>, <filters1>, ...]`<small>, used to request events and subscribe to new updates.</small>
- `["CLOSE", <subscription_id>]`<small>, used to stop previous subscriptions.</small>

<div v-click><br />Filters determine what events will be sent in that subscription, they can have the following attributes:

```js {all|2|3|4|5|6|7|8|all}{at:'2'}
{
  "ids": <a list of event ids>,
  "authors": <a list of lowercase pubkeys, the pubkey of an event must be one of these>,
  "kinds": <a list of a kind numbers>,
  "#<single-letter (a-zA-Z)>": <a list of tag values, for #e — a list of event ids, for #p — a list of pubkeys, etc.>,
  "since": <an integer unix timestamp in seconds. Events must have a created_at >= to this to pass>,
  "until": <an integer unix timestamp in seconds. Events must have a created_at <= to this to pass>,
  "limit": <maximum number of events relays SHOULD return in the initial query>
}
```

</div>

---
transition: fade-out
layout: fact
---

# Design Patterns

Event Sourcing & CQRS

---
transition: fade
---

# <span class="highlight">Event Sourcing</span> & CQRS

Event sourcing (ES) is an architectural design pattern that uses an append-only store to record the actions taken on the data.

<span class="highlight">ES avoids some of CRUD's shortcomings:</span>

<ol>
  <li>
    Data updates may cause conflicts, need extra logic to handle concurrency and locking.
    <ul v-click><li>Relays are very <i>dumb</i> and act as a distributed <i>Event Source</i></li></ul>
  </li>
  <li>
    Assumes that data is structured and predictable.
    <ul v-click><li>Specification evolves organically. Implementations' degree of compatibility may vary... and that's ok</li></ul></li>
  <li>
    Only the current state is recorded, operation history is lost.
    <ul v-click><li>Deletion and replaceable events break this, but there is no assurance relays will implement them, so clients need to handle the whole history of events regardless.</li></ul></li>
</ol>

<v-click>

<span class="highlight">Event Sourcing</span> allows Nostr to be both <span class="highlight">simple</span> and <span class="highlight">decentralized</span>. Whether you like it or not, there is no way around it when developing a Nostr application.

</v-click>

<style>
  ul {
    list-style-type: none;
    opacity: 0.7  ; 

    li {
      margin-left: 0;
      padding-left: 0;
    }
  }
</style>

---
transition: slide-left
---

# Event Sourcing & <span class="highlight">CQRS</span>

Command and Query Responsibility Segregation (CQRS) is a pattern that separates write (command) and read (query) operations for a data store, often used together with Event Sourcing

<div style="display: flex; justify-content: center; align-items: center; margin-top: 2rem; margin-bottom: 2rem">
  <img alt="CQRS Diagram" src="/images/cqrs.webp" style="max-width: 75%; height: auto;" />
</div>

<v-clicks>

- <span class="highlight">Command Model</span>: Validates inputs for data consistency and integrity. Builds and publishes Nostr events.
- <span class="highlight">Query Model</span>: Builds the current (denormalized) state of domain entities by projecting Nostr events.
- <span class="highlight">Write DB</span>: Set of Nostr relays used by the application.
- <span class="highlight">Read DB</span>: Caches the Query Model, optimizing for read performance, usually stored in-memory.

</v-clicks>

---
transition: fade-out
layout: fact
---

# Tooling

Sveltekit, NDK and Nostr-Login

---
transition: slide-left
---

# <span class="highlight">Tooling</span>

We're going to use a web framework, a Nostr development kit and login library.

<v-clicks>

- <span class="highlight">Sveltekit</span><br />A framework for web applications using Svelte, a compiler that takes declarative components and converts them into JavaScript that updates the DOM. <small><a href="https://kit.svelte.dev/" target="_blank">kit.svelte.dev</a></small>
- <span class="highlight">NDK</span><br />A Nostr Development Kit that makes building Nostr-related applications, as relays, clients, or anything in between, better, more reliable and overall nicer to work with. <small><a href="https://nostr-dev-kit.github.io/ndk/" target="_blank">nostr-dev-kit.github.io/ndk/</a></small>
- <span class="highlight">Nostr-Login</span><br />A `window.nostr` provider with nice UI for users to login with Nostr Connect (nip46), extension, read-only login, account switching, OAuth-like sign up, etc. <small><a href="https://github.com/nostrband/nostr-login" target="_blank">github.com/nostrband/nostr-login</a></small>

</v-clicks>

<style>
  li {
    margin-top: 1rem;
  }
</style>

---
transition: fade-out
layout: fact
---

# Let's Code

---
transition: slide-left
level: 2
---

# <span class="highlight">Let's code</span>

<a href="https://github.com/vstabile/nostr-blog" target="_blank">https://github.com/vstabile/nostr-blog</a>

```bash {1,2|1-4|1-7|1-10|all}
# clone the project
git clone git@github.com:vstabile/nostr-blog.git

cd nostr-blog

# install the dependencies
pnpm i

# copy the environment variables
cp .env.example .env

# run the web app locally
pnpm run dev
```

---
layout: center
class: text-center
---

# Thank You
