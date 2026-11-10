# HTTPS, TLS, and Web Security

Why plaintext HTTP is dangerous, and how TLS (Transport Layer Security) turns it into HTTPS. TLS sits between HTTP and TCP and answers three concrete threats: eavesdropping, tampering, and impersonation. The core mental model: certificates prove identity, asymmetric crypto safely agrees on a secret, and symmetric crypto does the fast bulk encryption.

## The three threats of plaintext HTTP

HTTP sends everything in plaintext, like a postcard. Every intermediate node — home router, ISP, submarine cable, edge router — can read and modify the bytes. The threat model decomposes into three questions:

- **Eavesdropping (竊聽)** — can someone see what I say? Breaks confidentiality. On a shared Wi-Fi, a packet sniffer (Wireshark) can capture a login body. Inside a Kubernetes cluster, plaintext service-to-service traffic is visible to anyone with Pod-level access — why zero-trust designs encrypt even east-west traffic.
- **Tampering (竄改)** — can someone change what I say? Breaks integrity. A malicious middle node can inject ads, swap a download's binary for a trojan, or rewrite a transfer account number. HTTP has no way to verify received content matches what the server sent.
- **Impersonation (冒充)** — is the other party who they claim to be? Breaks authenticity. DNS spoofing can point `mybank.com` at an attacker's server that looks identical; encryption is meaningless if you are encrypting to the wrong party.

TLS fixes all three: encryption (confidentiality), MACs/signatures (integrity), and certificates (authenticity). This is the CIA triad applied to transport.

## Certificates and the chain of trust

A digital certificate is a server's ID card. If we cannot confirm identity, even encrypted conversations can be with a fraud. The standard is **X.509**, containing:

- **Subject** — who the certificate is for: Common Name (CN) and, in modern certs, Subject Alternative Names (SAN) listing supported domains.
- **Public key** — binds identity (domain) to a public key used in the handshake.
- **Issuer** — the Certificate Authority (CA) that signed it (DigiCert, GlobalSign, Let's Encrypt).
- **Validity** — start and expiry; modern practice shortens to ~90 days or a year.
- **Digital signature** — the CA's private key over a hash of the certificate contents; this is what makes the document unforgeable.

Why can't an attacker just mint a certificate saying `Subject: google.com`? Because the digital signature must verify: the browser hashes the certificate, decrypts the signature with the CA's public key, and compares. Any alteration breaks the match. Browsers ship with a built-in set of trusted root CAs (the trust anchor); chains of intermediate CAs let roots stay offline while certificates get issued at scale. This is how we get from "this cert is signed by someone" to "this cert proves the server really is Google."

## Symmetric vs asymmetric encryption

TLS is a mixed doubles pair, not a competition:

- **Symmetric crypto** — one key for both encrypt and decrypt (e.g., AES). Extremely fast, often hardware-accelerated (AES-NI), the workhorse for bulk data. Fatal weakness: the key exchange problem — how do two strangers share the key over an insecure channel?
- **Asymmetric crypto** — a public/private key pair. Slow, but solves key exchange: encrypt to the public key, only the private key holder can decrypt. Also enables digital signatures (private key signs, public key verifies).

The handshake uses asymmetric crypto to protect the small, precious secret (the pre-master secret), then switches to symmetric crypto for the actual data — like mailing someone an open padlock (public key), they lock the secret list in a box, only you can open it, and you both derive the same session password from the list.

## TLS 1.2 handshake (2-RTT)

Four exchanges to authenticate the server and agree on a session key before any HTTP data flows:

1. **Hello & negotiation** — `ClientHello` (max TLS version, cipher suite list, Client Random) → `ServerHello` (chosen suite, Server Random). The two random numbers prevent replay attacks: every handshake derives a different key.
2. **Identity check** — Server sends its certificate (and optionally key-exchange params). Client verifies: not expired, domain matches, CA is in the trust list. Failure → the "your connection is not private" red screen.
3. **Secret agreement** — Client generates a third secret, the pre-master secret, encrypts it with the server's public key and sends it. Only the private key holder can decrypt — even a sniffer sees only ciphertext.
4. **Derivation & done** — Both sides mix Client Random + Server Random + Pre-master Secret into a Master Secret, then a symmetric Session Key. `ChangeCipherSpec` + `Finished` both ways confirm the tunnel works. From then on, HTTP is wrapped in encrypted TLS records.

## TLS 1.3 (1-RTT)

TLS 1.2's 2-RTT is latency the DevOps world dislikes. TLS 1.3 (2018) collapses this: the client sends its key-share guess in the first flight (since most deployments use a handful of algorithms), so a full handshake takes one round trip. It also removed obsolete/insecure cipher suites (RSA key exchange, CBC, etc.) — fewer moving parts, forward secrecy by default.

## Related pages

- [[http2-multiplexing]] — the modern HTTP layer that runs on top of TLS.
- [[computer-networking-tcp-ip]] — the layered model TLS slots into (Application layer over TCP), and where TLS failures sit in the troubleshooting split.
- [[kubernetes-networking]] — TLS termination at the Ingress is the Kubernetes deployment of these ideas.

## References
