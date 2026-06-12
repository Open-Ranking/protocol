# Open Ranking

Open Ranking is a protocol for reputation, ranking, discovery, and trust-aware profile services on top of Nostr. It defines a standard HTTP interface that specialized providers can implement, giving clients a simple and interoperable way to ask for:

- reputation scores for one or more pubkeys
- ranked recommendations
- profile search results
- provider capabilities and supported algorithms

The detailed behavior of each endpoint is defined in separate **OREs** (Open Ranking Enhancements).

## Endpoints

| Endpoint | ORE |
|---|---|
| `POST /stats/pubkey` | [ORE-02](02.md) |
| `POST /rank/pubkeys` | [ORE-03](03.md) |
| `POST /recommend/pubkeys` | [ORE-04](04.md) |
| `POST /search/pubkeys` | [ORE-05](05.md) |
| `POST /followers` | [ORE-06](06.md) |

---

## Why it exists

### How Nostr clients handle ranking today

Most Nostr clients compute ranking directly in the app using what is commonly called the **2-hop rule**: take the user's follows, then find *their* follows, build a small local graph, and rank within it.

This approach has fundamental problems:

- **New users have an empty graph.** If you just joined Nostr and follow nobody, you get nothing back.
- **It is computationally expensive.** Building and traversing a graph on-device is slow, drains battery, and blocks the UI.
- **It is limited in scope.** 2 hops will cover a smaller and smaller fraction of the network as Nostr grows, making it increasingly blind to relevant content and people.

### Existing alternatives fall short

Several attempts have been made to move this work off the client:

- **DVMs (Data Vending Machines)** never gained meaningful adoption. They are poorly suited for this use case: they use relays as both transport and storage, putting unnecessary load on the network, and they have no standard for capabilities discovery or algorithm selection.

- **NIP-85 Trusted Assertions** covers only a narrow slice of what Open Ranking addresses, a single reputation endpoint. It stores large volumes of events on relays, which is expensive and doesn't scale. It also cannot be used for search, discovery, or efficient batch queries, and gives clients no way to select an algorithm at request time.

- **NIP-50 relay search** only covers event search, and quality is generally low. Relays do not take into account the reputation of event authors or other heuristics, because doing so would require maintaining a large, continuously updated index - exactly the kind of heavy infrastructure that relays should not need to run.

- **Primal Caching Server** is a more capable solution, but it is tightly coupled to Primal's own client and internal needs. Its API surface is large and evolving, making independent re-implementation by third parties very hard in practice. It is a product, not a protocol.

### What Open Ranking does instead

Reputation metrics, rankings, and search all require indexing a vast number of events and running a robust pipeline to stay up-to date. This is heavy work that should not live on relays. Relays should stay simple, cheap to run, and easy to self-host, because that fosters decentralization.

Open Ranking moves this heavy work to **dedicated providers** that can specialize in it and compete on quality. The protocol defines a shared standard interface so that providers are interoperable: clients talk to any provider the same way, and users can switch providers freely.

---

## Design principles

### JSON over HTTP

Open Ranking uses plain HTTP with JSON for both requests and responses. HTTP is stateless, widely understood, and comes with well-defined semantics for error codes, authentication, and caching. JSON keeps the client-side integration simple: no custom encodings, no binary formats, no special-purpose libraries required.

### Interoperable providers

Because all providers speak the same protocol, users can switch providers without requiring app developers to write provider-specific integrations. This is the same model that makes Nostr relays and Blossom media servers work well: the protocol is the interface, not the specific server.

### Provider-defined algorithms

Providers are free to define and name their own ranking and search algorithms. The algorithm is their core value, their secret sauce. A provider can expose multiple algorithms, each offering different tradeoffs between speed, precision, personalization, and coverage. Clients can request a specific algorithm or fall back to the provider's default.

### Machine-discoverable capabilities

Every provider exposes a capabilities document at a standard well-known location (`/.well-known/open-ranking`). That document tells clients which endpoints and algorithms the provider supports. Clients can adapt to any provider programmatically, without relying on human-readable documentation.

### No unnecessary bloat

Open Ranking defines only what needs to be standardized. Things like pricing, rate limits, subscriptions, and business model are intentionally left outside the protocol. Providers are free to handle those however they want and document them on their own terms.

---

## Core concepts

### Provider
A server that implements one or more Open Ranking endpoints. Providers are free to optimize internally (indexing strategy, caching, pipeline design) as long as they conform to the protocol surface.

### Algorithm
A provider-defined ranking or search strategy, identified by an opaque string. Algorithms are the provider's differentiator. A provider may expose multiple algorithms for the same endpoint.

### Capability document
A small discovery document served at `/.well-known/open-ranking`. It tells clients which endpoints the provider supports and which algorithms are available for each, including the default. Business model, pricing, and other provider-specific information are intentionally not part of this document.

### ORE
An **Open Ranking Enhancement**. OREs are the specification documents that define the exact behavior of endpoints, discovery mechanisms, and protocol rules, similar to how NIPs work for Nostr. Providers are not required to implement every ORE. Some OREs will be required for compliance, others optional.

## Relationship to Nostr

Open Ranking does not replace Nostr.

It complements Nostr by providing a standardized way to query providers for derived data. Raw Nostr events remain on relays. Open Ranking providers consume this data, process it, and provide derived views via a simple HTTP interface. Therefore, it goes against the philosophy of the protocol to return events that could be fetched from relays.

## Status

Draft.

---

## License

To be defined.
