# verify

A single self-contained page that takes a real DKIM-signed email and verifies its
2048-bit RSA signature **in the browser**, from first principles.

**Live:** https://smellybricc.github.io/verify/

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

## No crypto library

SHA-256 (FIPS 180-4) and modular exponentiation are implemented directly in the
page's JavaScript. There are no dependencies and no build step; `crypto.subtle` is
deliberately unused so the file works when opened straight from disk.

The signature being checked comes from a fixture signed by a third-party tool, so
verification is non-circular. Only the **public** modulus and the signature are
embedded here — no private key.

## Where it comes from

The reference implementation is `analyzer/dkim.py` in DAEMON, an email
threat-intelligence console (28 modules, 130 passing tests, standard library at the
core). The JavaScript here is a second, independent implementation written to
cross-check it — the two agree byte for byte.

— [Kuba Opoczka](https://www.linkedin.com/in/kubaopoczka)
