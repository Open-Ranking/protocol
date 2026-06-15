# ORE-A

## Authentication

`draft` `optional`

This document defines how clients authenticate with Open Ranking providers using **Nostr Web Tokens (NWT)**. Authentication is optional and provider-defined. Providers MAY require it for some or all requests, at their own discretion.

---

## Overview

Open Ranking uses [Nostr Web Tokens](https://github.com/Open-Ranking/nostr-web-tokens) for authentication. A NWT is a signed Nostr event of `kind 27519` that conveys signed claims from the client to the provider.

Clients MUST NOT assume authentication is required or not required for a given provider or endpoint. Clients SHOULD attempt requests unauthenticated and handle authentication errors as they arise.

## Transport

Clients MUST include the NWT in the `Authorization` header using the `Nostr` scheme, with the token encoded as Base64URL without padding:

```
Authorization: Nostr <base64url-no-padding-token>
```

## Audience

Clients MUST set the `aud` claim to the provider's domain name (e.g. `wot.example.com`)
Providers MUST reject tokens whose `aud` claim does not match their domain.

```json
["aud", "wot.example.com"]
```

## Errors

Authentication error codes and their semantics are defined in the [NWT Error Codes Specification](https://github.com/Open-Ranking/nostr-web-tokens#http-error-codes).  
Providers SHOULD include a human-readable description of the problem in the `X-Reason` response header.
