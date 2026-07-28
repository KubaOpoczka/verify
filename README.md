# verify

A single self-contained page that takes a real DKIM-signed email and verifies its
2048-bit RSA signature **in the browser**, from first principles.

**Live:** https://kubaopoczka.github.io/verify/

## What it actually does

1. Canonicalises the message body per RFC 6376 (relaxed), hashes it with SHA-256,
   and compares against the `bh=` tag the signer committed to.
2. Rebuilds the signed header set — including the `DKIM-Signature` header with its
   own `b=` value emptied — and hashes that.
3. Performs the RSA operation `pow(s, e, n)` and recovers the PKCS#1 v1.5 block:
   `00 01 || FF… || 00 || DigestInfo || H`. The digest is *recovered from the
   signature*, then matched against the digest computed in step 2.

Edit the message body on the page and the body hash stops matching, while the
header signature stays valid — which is precisely the tampering DKIM exists to detect.

## Verify your own mail

The page also takes a pasted raw message (Gmail: **More -> Show original**) and verifies
it for real: it parses the `DKIM-Signature` header, canonicalises the headers and body,
fetches the signer's public key over DNS-over-HTTPS, parses the DER `SubjectPublicKeyInfo`,
and checks the RSA signature — all client-side.

**Privacy:** the message never leaves your browser. The only outbound request is a DNS
lookup for the *signing domain and selector* (never your address or the content), because
the public key has to come from somewhere. Cloudflare is tried first, then Google.

Cross-checked against the Python reference: 7/7 canonicalisation cases byte-identical
(folded headers, repeated headers, tabs and trailing whitespace, simple canon, a signed
header that is absent, empty body), and DER parsing agrees on two real published keys
(a 2048-bit and a 1024-bit).

Limitations, stated plainly: `rsa-sha256` only (not ed25519), the first `DKIM-Signature`
header is the one checked, and `l=` partial-body signatures are flagged but not
specially handled. A failure usually means the pasted source was re-wrapped by the
client — DKIM signs exact bytes.

## No crypto library

SHA-256 (FIPS 180-4) and modular exponentiation are implemented directly in the
page's JavaScript. There are no dependencies and no build step; `crypto.subtle` is
deliberately unused so the file works when opened straight from disk.

The signature being checked comes from a fixture signed by a third-party tool, so
verification is non-circular. Only the **public** modulus and the signature are
embedded here — no private key.

## An auditable CV

`cv/` is a CV that checks itself: <https://kubaopoczka.github.io/verify/cv/>

As it loads it queries the GitHub and npm registry APIs from the browser and fills in the real
values — repository count, package version, publish date, licence, last push. Claims a machine
cannot check are labelled **INSPECT** (read the code) or **NOT PUBLIC** rather than presented as
verified. If an API is unreachable the row says so and links to the endpoint; it never falls back
to a hard-coded number.

The CV also checks the three July 2026 portfolio systems directly against GitHub and links
to their live deployments:

- [Faultline](https://kubaopoczka.github.io/faultline/) — deterministic distributed-systems failure lab
- [Signal Garden](https://kubaopoczka.github.io/signal-garden/) — local-only audio-reactive generative instrument
- [Second Order](https://kubaopoczka.github.io/second-order/) — seeded Monte Carlo decision laboratory

## Where it comes from

The reference implementation is `analyzer/dkim.py` in DAEMON, an email
threat-intelligence console (28 modules, 130 passing tests, standard library at the
core). The JavaScript here is a second, independent implementation written to
cross-check it — the two agree byte for byte.

— [Kuba Opoczka](https://www.linkedin.com/in/kubaopoczka)
