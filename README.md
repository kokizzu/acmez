acmez - ACME client library for Go
==================================

[![godoc](https://pkg.go.dev/badge/github.com/mholt/acmez/v3)](https://pkg.go.dev/github.com/mholt/acmez/v3)

ACMEz ("ack-measy" or "acme-zee", whichever you prefer) is a fully-compliant [RFC 8555](https://tools.ietf.org/html/rfc8555) (ACME) implementation in pure Go. It is lightweight, has an elegant Go API, and its retry logic is highly robust against external errors. ACMEz is suitable for large-scale enterprise deployments. It also supports common IETF-standardized ACME extensions.

**NOTE:** This module is for _getting_ certificates, not _managing_ certificates. Most users probably want certificate _management_ (keeping certificates renewed) rather than to interface directly with ACME. Developers who want to use certificates in their long-running Go programs should use [CertMagic](https://github.com/caddyserver/certmagic) instead; or, if their program is not written in Go, [Caddy](https://caddyserver.com/) can be used to manage certificates (even without running an HTTP or TLS server if needed).

This module has two primary packages:

- **`acmez`** is a high-level wrapper for getting certificates. It implements the ACME order flow described in RFC 8555 including challenge solving using pluggable solvers.
- **`acme`** is a low-level RFC 8555 implementation that provides the fundamental ACME operations, mainly useful if you have advanced or niche requirements.

In other words, the `acmez` package is **porcelain** while the `acme` package is **plumbing** (to use git's terminology).


## Features

- Simple, elegant Go API
- Thoroughly documented with spec citations
- Robust to external errors
- Structured error values ("problems" as defined in RFC 7807)
- Smart retries (resilient against network and server hiccups)
- Challenge plasticity (randomized challenges, and will retry others if one fails)
- Context cancellation (suitable for high-frequency config changes or reloads)
- Highly flexible and customizable
- External Account Binding (EAB) support
- Tested with numerous ACME CAs (more than just Let's Encrypt)
- Implements niche aspects of RFC 8555 (such as alt cert chains and account key rollover)
- Efficient solving of large SAN lists (e.g. for slow DNS record propagation)
- Utility functions for solving challenges
	- Device attestation challenges ([draft-acme-device-attest-02](https://datatracker.ietf.org/doc/draft-acme-device-attest/))
	- [RFC 8737](https://www.rfc-editor.org/rfc/rfc8737.html) (tls-alpn-01 challenge)
	- [RFC 8823](https://www.rfc-editor.org/rfc/rfc8823.html) (email-reply-00 challenge; S/MIME)
- ACME Renewal Information (ARI) support ([RFC 9773](https://datatracker.ietf.org/doc/html/rfc9773)
- ACME profiles ([draft-aaron-acme-profiles](https://datatracker.ietf.org/doc/draft-aaron-acme-profiles/))


## Install

```
go get github.com/mholt/acmez/v3
```


## Examples

See the [`examples` folder](https://github.com/mholt/acmez/tree/master/examples) for tutorials on how to use either package. **Most users should follow the [porcelain guide](https://github.com/mholt/acmez/blob/master/examples/porcelain/main.go) to get started.**


## Challenge solvers

The `acmez` package is "bring-your-own-solver." It provides helper utilities for http-01, dns-01, and tls-alpn-01 challenges, but does not actually solve them for you. You must write or use an implementation of [`acmez.Solver`](https://pkg.go.dev/github.com/mholt/acmez/v3#Solver) in order to get certificates. How this is done depends on your environment/situation.

However, you can find [a general-purpose dns-01 solver in CertMagic](https://pkg.go.dev/github.com/caddyserver/certmagic#DNS01Solver), which uses [libdns](https://github.com/libdns) packages to integrate with numerous DNS providers. You can use it like this:

```go
// minimal example using Cloudflare
solver := &certmagic.DNS01Solver{
	DNSManager: certmagic.DNSManager{
		DNSProvider: &cloudflare.Provider{APIToken: "topsecret"},
	},
}
client := acmez.Client{
	ChallengeSolvers: map[string]acmez.Solver{
		acme.ChallengeTypeDNS01: solver,
	},
	// ...
}
```

If you're implementing a tls-alpn-01 solver, the `acmez` package can help. It has the constant [`ACMETLS1Protocol`](https://pkg.go.dev/github.com/mholt/acmez/v3#pkg-constants) which you can use to identify challenge handshakes by inspecting the ClientHello's ALPN extension. Simply complete the handshake using a certificate from the [`acmez.TLSALPN01ChallengeCert()`](https://pkg.go.dev/github.com/mholt/acmez/v3#TLSALPN01ChallengeCert) function to solve the challenge.



## History

In 2014, the ISRG was finishing the development of its automated CA infrastructure: the first of its kind to become publicly-trusted, under the name Let's Encrypt, which used a young protocol called ACME to automate domain validation and certificate issuance.

Meanwhile, a project called [Caddy](https://caddyserver.com) was being developed which would be the first and only web server to use HTTPS _automatically and by default_. To make that possible, another project called lego was commissioned by the Caddy project to become of the first-ever ACME client libraries, and the first client written in Go. It was made by Sebastian Erhart (xenolf), and on day 1 of Let's Encrypt's public beta, Caddy used lego to obtain its first certificate automatically at startup, making Caddy and lego the first-ever integrated ACME client.

Since then, Caddy has seen use in production longer than any other ACME client integration, and is well-known for being one of the most robust and reliable HTTPS implementations available today.

A few years later, Caddy's novel auto-HTTPS logic was extracted into a library called [CertMagic](https://github.com/caddyserver/certmagic) to be usable by any Go program. Caddy would continue to use CertMagic, which implemented the certificate _automation and management_ logic on top of the low-level certificate _obtain_ logic that lego provided.

Soon thereafter, the lego project shifted maintainership and the goals and vision of the project diverged from those of Caddy's use case of managing tens of thousands of certificates per instance. Eventually, [the original Caddy author announced work on a new ACME client library in Go](https://github.com/caddyserver/certmagic/issues/71) that satisfied Caddy's harsh requirements for large-scale enterprise deployments, lean builds, and simple API. This work exceeded expectations and finally came to fruition in 2020 as ACMEz. It is much more lightweight with zero core dependencies, has a simple and elegant code base, and is thoroughly documented and easy to build upon.

> [!NOTE]
> This is not an official repository of the [Caddy Web Server](https://github.com/caddyserver) organization.

---

(c) 2020 Matthew Holt
