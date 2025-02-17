# Highlighter

Highlighter is a tool for creators and fans to interact with each other.

# Premises
* Nothing built on highlighter should be custom or "lock" users into highlighter. Creators should **always** have an exit strategy and it shouldn't attempt to become a central point of capture. If Highlighter is compelled to censor a creator, that creator should continue to be a first-class citizen by easily hosting their own Highlighter instance.
* There mustn't be any value capture where users are forced to interact with highlighter. Creators should be able to setup their offering on any client that supports the standards Highlighter is built on and user should be able to subscribe and consume the content on any client that supports them.

# Implementation

## NIP-29
Highlighter relies heavily on NIP-29 for content distribution and grouping.

- [x] When a new creator signs up (i.e. creates tiers), a NIP-29 group is created where the `h` tag matches the pubkey of the creator by publishing a kind:9006 event.

## NIP-88
NIP-88 provides the infrastructure for recurring subscriptions

# Components:
This repository is a monorepo that contains the following packages:

* [apps/web](./apps/web): The web frontend for Highlighter
* [apps/relay29](./apps/relay29): The relay that implements NIP-29, based on https://github.com/fiatjaf/relay29
* [packages/ui-common](./packages/ui-common): Common UI components used across the Highlighter ecosystem -- *there is a lot of preemptively extracted code from old projects here, most of this stuff should be moved to the web app*
* [packages/ndk](./packages/ndk): Monorepo for highlighter

# Running Highlighter
```
git clone --recursive https://github.com/pablof7z/highlighter
# Use --recursive to fetch all the git submodules!
cd highlighter
pnpm install
turbo build
cd apps/web
cp .env.example .env
```

Replace the placeholder values in `.env` with your own values. You'll need to create a private key for your creator relay identity.

* Run the relay
```
cd apps/relay/relay
RELAY_URL=ws://localhost:5577 RELAY_PRIVKEY=<your-relay-private-key> RELAY_NAME="some-name-for-your-relay" DOMAIN=localhost go run -ldflags=-compressdwarf=false .
```

* Run the web app
```
cd apps/web
pnpm dev
```
