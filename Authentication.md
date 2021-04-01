# Authentication

- **authentication**: verifying identity (`401 Unauthorized`)
  - note status code says unauthorized, but this really means authentication
    failed
- **authorization**: verifying permission (`403 Forbidden`)

> username/password scheme

- **stateful**: using a session with cookies
- **stateless**: using a token via JWT/OAuth

## Sessions

### Flow

- user submits login credentials (email/password)
- server verifies credentials against DB
- server creates a temporary user `session`
- server issues a cookie with a `session ID`
  - `session ID` is used to identify a `session`
  - `session ID` is a randomly generated string
- user sends the cookie with each request
- server validates it against the session store & grants access
- when user logs out, server destroys the session & clears cookie

### Features

- every user session is stored server-side (**stateful**). possible storage
  options include:

  - memory (file system)
  - cache (redis of memcached)
  - DB (postgresql or Mongo)

- each user is identified by a `session ID`
  - `opaque` ref
    - no 3rd party can extract data out, (only a random string that is only
      relevant to server)
    - only issuer (server) can map back to data
  - stored in a cookie
    - signed with a secret that only server has
    - protected with flags

## Cookies

- `Cookie` is a header, just like `Authorization` or `Content-Type`
- used in session management, personalization, tracking
- consists of name, value and (optional) attributes / flags
- set with `Set-Cookie` header by server, appended with `Cookie` by browser

### Security

- signed (`HMAC`) with a secret by the server to mitigate tampering
- _rarely_ encrypted (`AES`) to be protected from being read
  - no security concerns if read by 3rd party
  - carries no meaningful data
  - even if encrypted, still a 1-1 match
- encoded (`URL`) - not for security, but compatibility

### Attributes

- `Domain` and `Path` (can only be used on a given site & route)
- `Expiration` (can only be used until expiry)
  - when omitted, becomes a _session cookie_: a special type of cookie
  - gets deleted when browser is closed

### Flags

- `HttpOnly`: cannot be read with JS on client-side
- `Secure`: can only be sent over encrypted `HTTPS` channel)
- `SameSite`: can only be sent from same domain (no CORS sharing)

### Cross-site request forgery (CSRF)

- exposing ourselves when dealing with session based authentication using
  cookies
- unauthorized actions on behalf of authenticated user
- mitigated with CSRF token (e.g. sent in a separate `X-CSRF-TOKEN` cookie)

## Tokens

### Flow

- user submits login credentials (email/password)
- server verifies the credentials against DB
- server generates a temporary _token_ and embeds user data into it
- server responds back with token (in body or header)
- user stores the token in client storage
- user sends the token along with each request
- server verifies the token & grants access
- when user logs out, token is cleared from client storage

### Features

- tokens are not stored server-side, only on the client (stateless)
- signed with a secret against tampering
  - verified and can be trusted by the server
- tokens can be opaque or self-contained
  - carries all required use data in its payload
  - reduces database lokups, but exposes data to XSS
- typically sent in the `Authorization` header
- when a token is about to expire, it can be refreshed
  - client is issued both access & refresh tokens
- used in SPA web apps, web API's, mobile apps

## JSON Web Tokens (JWT)

- open standard for authorization & info exchange
- compact, self-contained, URL-safe tokens
- signed with symmetric (secret) or asymmetric (public/private) key

```
HTTP/1.1 200 OK
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI1YmQ2MWFhMWJiNDNmNzI0M2EyOTMxNmQiLCJuYW1lIjoiSm9obiBTbWl0aCIsImlhdCI6MTU0MTI3NjA2MH0.WDKey8WGO6LENkHWJRy8S0QOCbdGwFFoH5XCAR49g4k
```

- contains _header_ (meta), _payload_ (claims), and _signature_ delimited by `.`

### Security

- signed (`HMAC`) with a secret
  - guarantees that token was not tampered
  - any manipulation invalidates token
- rarely encrypted(JWE)
  - web clients need to read token payload
  - can't store the secret in client storage securely
- encoded (`Base64URL`) - not for security, but transport
  - payload can be decoded and read
  - no sensitive/private info should be stored
  - access tokens should be short-lived: if stolen, time-boxed to minimize
    damage

### Cross-Site Scripting (XSS)

- XSS attacks enable attackers to inject client-side scripts into web pages
  viewed by other users
- client-side script injections
- malicious code can access client storage to
  - steal user data from the token
  - initiate AJAX requests on behalf of user
- mitigated by sanitizing and escaping user input

## Client Storage

- JWT can be stored in client storage, `localStorage` or `sessionStorage`
  - `localStorage` has no expiration time
  - `sessionStorage` gets cleared when page is closed

### `localStorage`

- Browser key-value store with simple JS API _Pros_
- domain-specific, each site has its own, other sites can't read/write
- max size higher than cookie (`5 MB vs 4KB`) _Cons_
- plaintext, hence not secure by design
- limited to string data, need to serialize
- can't be used by web workers
- stored permanently, unless removed explicity
- accessible to any JS code running on the page (including XSS)
  - scripts can steal tokens or impersonate users _Best For_
- public, non-sensitive, string data _Worst For_
- private sensitive data
- non-string data
- offline capabilities

## Sessions vs JWT

### Sessions + Cookies

_Pros_

- session ID's are opaque and carry no meaningful data
- cookies can be secured with flags (same origin, HTTP-only, HTTPS)
- HTTP-only cookies can't be compromised with XSS exploits
- battle-texted 20+ years in many langs and frameworks

_Cons_

- Server must stored each user session in memory
- session auth must be secured against CSRF
- horizontal scaling is more challenging
  - risk of single point of failure
  - need stickky sessions with load balancing

### JWT Auth

_Pros_

- server does not need to keep track of user sessions
- horizontal scaling is easier (any server can verify the token)
- CORS is not an issue if `Authorization` header is used instead of `Cookie`
- FE and BE architecture is decoupled, can be used with mobile apps
- operational even if cookies are disabled

_Cons_

- server still has to maintain a blacklist of revoked tokens
  - defeats the purpose of stateless tokens
  - whitelist of active user seessions is more secure
- when scaling, the secret must be shared between servers
- data stored in token is "cached" and can go _stale_ (out of sync)
- tokens stored in client storage are vulnerable to XSS
  - if JWT token compromise, attacker can,
    - steal user info, permissions, metadata etc
    - access website resources on user's behalf
- requires javascript to be enabled

## Options for Auth in SPAs / APIs

1. Sessions
2. Stateless JWT
3. Stateful JWT

### Stateless JWT

- use payload embedded in token
- token is signed & `base64url` encoded
  - sent via `Authorization` header
  - stored in `localStorage` / `sessionStorage` (in plaintxt)
- server retrieves use info from the token
- no user session are stored server side
- only revoked tokens are persisted (to maintain the blacklist)
- refresh token sent to renew the access token

### Stateful JWT

- only user ref (e.g. ID) embedded in the token
- token is signed & `base64url` encoded
  - sent as an HTTP-only cookie (`Set-Cookie` header)
  - sent along with a non-HTTP `X-CSRF-TOKEN` cookie
- server uses ref. (ID) in the token to retrieve user from the db
- no user session stored on the server either
- revoked tokens still have to be persisted

### Sessions

- sessions are persisted server-side and linked by sess. ID
- session ID is signed and stored in a cookie
  - sent via `Set-Cookie` header
  - `HttpOnly`, `Secure` & `SameSite flags
  - scoped to the origin with `Domain` & Paths attrs
- another cookie can hold CSRF token

### Verdict

- sessions are (probably) better suited for web apps and websites

### Why not JWT?

- server sate needs to be maintained either way (black-lists for jwt)
- sessions are easily extended or invalidated
- data is secured server sdie & does leak through XSS
- CSRF is easier to mitigate than XSS (still a concern)
- data never goes stale (always in sync with DB)
- sessions are generally easier to setup & manage
- most apps/sites don't require enterprise scaling
