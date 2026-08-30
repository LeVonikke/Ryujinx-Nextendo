# ACNH Custom Designs Portal (experimental)

Animal Crossing: New Horizons does not create an `MO-…` code locally. It uploads a MessagePack
design to `api.hac.lp1.acbaa.srv.nintendo.net`; the response contains a numeric ID, which the game
formats as the 12-character base-30 `MO-XXXX-XXXX-XXXX` code. The emulator can redirect that hostname,
but a server must implement the portal API for codes to persist or be downloadable.

`services/acnh-designs` is a deliberately small self-hosted implementation of the known API surface:

- `POST /api/v1/auth_token`
- `POST`, `GET`, and `DELETE /api/v1/designs`
- `GET /api/v2/designs`

It preserves the original MessagePack body in SQLite, allocates non-null 12-character-compatible
numeric IDs, and returns the saved body on download. The server is intended for a private Nextendo
deployment; it does not communicate with Nintendo.

## Run it

Install the sole dependency and set a persistent random secret:

```powershell
cd services/acnh-designs
python -m pip install -r requirements.txt
$env:ACNH_DESIGNS_AUTH_SECRET = '<at-least-32-random-bytes>'
python server.py --host 0.0.0.0 --port 443 --certfile portal.pem --keyfile portal-key.pem
```

The Ryujinx fork accepts the server certificate for its redirected hosts. For production, terminate
TLS at the existing SNI router/reverse proxy and forward the ACNH hostname to this service instead;
in that case start it without `--certfile`/`--keyfile` on an internal port.

Set `NEXTENDO_ACNH_DESIGNS_IP` to the service or reverse-proxy address before starting Ryujinx. If it
is not set, that hostname retains the normal `NEXTENDO_SERVER_IP` route. A hosts-file exact entry still
has higher priority and is convenient for local testing:

```text
127.0.0.1 api.hac.lp1.acbaa.srv.nintendo.net
```

## Current validation boundary

The endpoints and payload layout are based on the public, community-documented portal API. Before
calling this production-ready, capture a portal session from the target ACNH version and compare its
request/response fields with the service log. In particular, test first-post creator creation, normal
design upload/download, Pro design upload/download, creator lookup, and deletion. This lets us add
any version-specific metadata without changing stored designs or their IDs.
