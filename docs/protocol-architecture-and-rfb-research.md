# Protocol modularization and RFB-first research plan

Status: architectural proposal and research backlog (2026-10-28). **Not an implementation claim.**

## Motivation and scope

AROS Prism manages local and remote displays, including mirroring, independent Screens, and eventually virtual outputs and extended desktops. Protocol implementations should **not belong to Prism**: they should be reusable by standalone AROS clients, servers, and unrelated applications. Prism is a consumer and adapter, not the owner of RFB, RDP, WebRTC, or casting protocol logic.

The first architectural cut is **separation of responsibilities and dependencies**, not immediately creating a shared library for every protocol or moving each to its own repository. Investigate native AROS shared libraries with versioned ABI (`OpenLibrary`/`CloseLibrary`) where appropriate; static linkage can be an intermediate validation step. Avoid embedding Intuition, Screen, graphics HIDD, or Prism-specific types in public protocol APIs.

## Protocol classification and roles

| Candidate | AROS roles | Scope / caveat |
| --- | --- | --- |
| RFB/VNC | Viewer client; framebuffer server | First candidate. Existing-screen sharing is not desktop extension. |
| RDP | Client and potentially server | Substantial security, channel and platform dependencies; investigate FreeRDP/WinPR. |
| Miracast / Wi-Fi Display | Source (transmitter); Sink (receiver) | Wi-Fi Direct, RTSP, RTP, codecs; source/sink not equivalent to ordinary client/server. |
| AirPlay | Sender; receiver | Receiver implementations do not imply sender support; version/security constraints. |
| Google Cast | Controller/sender; receiver | Distinguish media casting, live video, and desktop mirroring. |
| WebRTC | Peer-to-peer endpoints | Signaling, ICE, DTLS/SRTP, codecs, optional data channels; browser interoperability. |
| RTP/RTCP | Sender/receiver transport | Infrastructure, not a complete display protocol. |
| RTSP | Session-control endpoints | Shared component for streaming stacks; not a display protocol by itself. |
| Bluetooth | Profile-specific roles | Discovery/control/connectivity or specific video profiles; not a generic wireless-monitor protocol. |
| Prism-native | Sender/receiver | Optional custom protocol with explicit interoperability and versioning contract. |
| Sunshine/Moonlight | Streaming host/client | Research latency-oriented desktop streaming; codec/capture requirements. |
| Open Screen Protocol | Presentation-related roles | Research relevance; not equivalent to remote framebuffer. |

Avoid imposing symmetry where specifications do not have it. Separate protocol, transport, discovery, media encoding, capture, input injection, authentication, and UI adapters where these boundaries are real. Reuse existing AROS networking and Bonami/DNS-SD where suitable, without coupling unrelated protocols (e.g. RTP must not depend on AppleMIDI).

## Proposed dependency direction

```text
Prism display/session managers          Independent AROS applications
   | (RFB server adapter)                  | (RFB viewer adapter)
   +-------------------+-------------------+
                       |
                RFB protocol API
                 client / server
                       |
           AROS sockets / Exec integration

AROS graphics / HIDD / Intuition --> Prism capture/virtual-output adapter
                                  (NOT dependencies of RFB protocol core)
```

A reusable protocol library should expose role-appropriate session APIs, independent concurrent sessions, explicit lifecycle and errors, callback/message-driven asynchronous integration, buffer ownership, and a stable versioned ABI. Distinguish reusable wire-protocol core from protocol-specific encodings and optional TLS/authentication modules. No ABI name or method signature is finalized; `rfb.library` is a working proposal, not an existing upstream component.

## Verified upstream AROS findings

- `rom/hidds/gfx/headless/` exists in upstream AROS and builds `headlessgfx` as a HIDD plus a Headless monitor loader.
- `headlessgfx_hiddclass.c` explicitly describes its display as a framebuffer intended to be read back and served over VNC. It registers indexed and true-color formats (8/15/16/24-bit depths; 24-bit depth in a 32bpp format) and a configurable mode (default 1024x768).
- `headlessgfx_bitmapclass.c` implements a bitmap HIDD class and visibility state; this **does not by itself establish a VNC server, efficient dirty-region reporting, or an export API**.
- `headlessgfx_init.c` integrates with `graphics.library` via `AddDisplayDriver` in the relevant startup mode.
- GitHub code searches of `aros-development-team/AROS` for `vncserver` and `LibVNCServer` returned no matches; a search for `headless vnc` found the headless driver comment. These searches are not proof of absence in all distributions, archives, or forks.

Sources:
- https://github.com/aros-development-team/AROS/tree/master/rom/hidds/gfx/headless
- https://github.com/aros-development-team/AROS/blob/master/rom/hidds/gfx/headless/headlessgfx_hiddclass.c
- https://github.com/aros-development-team/AROS/blob/master/rom/hidds/gfx/headless/headlessgfx_bitmapclass.c
- https://github.com/aros-development-team/AROS/blob/master/rom/hidds/gfx/headless/headlessgfx_init.c

## Existing ecosystem VNC: investigate before porting

Icaros Desktop documentation describes VNC remote Workbench functionality and an AROS VNC client. A `vnc-client-server.lha` archive has also been reported, but its source, license, lineage, target compatibility, and actual contents **have not been verified**. Do not assume these implementations share code or are suitable for upstream.

Research tasks:
1. Locate the precise Icaros client/server binaries, sources, original authors, revisions, and licenses; inspect the archive rather than relying on its filename.
2. Identify whether any existing AROS RFB implementation already separates wire protocol from graphics capture, input, networking, and UI.
3. Compare with LibVNCServer/LibVNCClient (portable C, GPL-2.0-or-later per upstream project); determine whether reuse is legally and technically viable with AROS's applicable licenses. Do **not** assume APL/GPL compatibility.
4. Determine how to read back the Headless HIDD framebuffer, handle dirty rectangles, cursor, palette/pixel conversion, resize, synchronization, and multiple clients; measure copy costs.
5. Determine whether a viewer can use the same protocol core without linking Prism or graphics capture code.

Reference: https://github.com/LibVNC/libvncserver

## First vertical slice: RFB

Prefer RFB because it is a comparatively tractable bidirectional client/server test without mandatory video codecs. **Do not port LibVNCServer or implement RFB from scratch before the existing AROS code audit.**

Acceptance criteria:
1. Reusable RFB core builds independently of Prism and has documented licensing and dependencies.
2. A standalone AROS server and standalone AROS viewer use the proposed API; interoperability with an external VNC peer is demonstrated.
3. Prism's adapter can serve a pre-existing Screen (mirroring) without putting Prism/Intuition types in the protocol core.
4. A later experiment connects an independently rendered virtual output (potentially using Headless HIDD); do not describe framebuffer mirroring as desktop extension.
5. Test on recorded AROS branch/commit/architecture with authentication/access controls, remote-input enable/disable, disconnection/reconnection, resolution and pixel-format variations, and performance measurements.
6. Document concurrent sessions, ownership, callbacks/Exec message semantics, ABI versioning, and error handling.

## Subsequent protocol research

- RTP/RTCP and RTSP: identify reusable transport/session pieces, keeping them separate from specific applications.
- WebRTC: evaluate libdatachannel (C API, C++ core and crypto dependencies) and browser-compatible signaling.
- Miracast: study Wi-Fi Direct availability and Source implementations; MiracleCast is primarily a Sink reference and is Linux-oriented.
- AirPlay: UxPlay is a Receiver reference, not an AROS sender; examine protocol/security and licensing.
- Cast: distinguish control, receiver app and screen-mirroring paths.
- RDP: investigate FreeRDP client architecture and realistic server requirements.
- Sunshine/Moonlight and Open Screen Protocol: research only until target use cases and dependencies are validated.

References:
- https://www.rfc-editor.org/rfc/rfc6143
- https://www.rfc-editor.org/rfc/rfc3550
- https://github.com/albfan/miraclecast
- https://github.com/FDH2/UxPlay
- https://github.com/FreeRDP/FreeRDP
- https://github.com/paullouisageneau/libdatachannel
- https://github.com/LizardByte/Sunshine
- https://github.com/moonlight-stream/moonlight-qt

## Open Screen: updated protocol and dependency research (2026-10-28)

The W3C Open Screen work should be tracked as **Open Screen Network Protocol** and **Open Screen Application Protocol**, rather than assuming that a single historical Open Screen Protocol document remains the whole specification. Verify exact working-draft revisions and interoperability requirements before implementation.

References:
- https://www.w3.org/TR/openscreen-network/
- https://www.w3.org/TR/openscreen-application/
- https://chromium.googlesource.com/openscreen/
- https://github.com/chromium/openscreen

**Existing code:** Chromium's Open Screen Library contains relevant discovery, casting and Open Screen work, including OSP demonstration code. Its existence does **not** establish a complete, current, interoperable implementation of both W3C specifications. Audit the current code, build requirements, C++ toolchain, dependency licenses, protocol coverage and maintenance status before attempting a port.

**Protocol layers to validate separately:**
1. Service discovery and resolution: compare the Network Protocol requirements with the actual AROS Bonami DNS-SD implementation, including service-type browsing, PTR/SRV/TXT/A/AAAA and IPv6 behavior.
2. Authentication and pairing: identify the precise password-authenticated key-exchange variant and its binding to authenticated transport. Do not conflate SPAKE2, SPAKE2+ and EC J-PAKE.
3. Secure transport: investigate QUIC requirements and their TLS 1.3 integration, UDP/event-loop integration, timers, IPv6, certificate validation and platform entropy.
4. Application semantics: map the Application Protocol's presentation roles and lifecycle to Prism sessions without putting W3C protocol types in Prism's graphics/HIDD interfaces.
5. Actual display semantics: distinguish URL/media presentation from desktop mirroring and extended desktop; Open Screen is not automatically a generic remote framebuffer or monitor protocol.

**AROS baseline:** `workbench/libs/mbedtls/` currently builds mbedTLS 3.6.7 and has AROS adapters for `entropy.resource` and `bsdsocket.library`. Its current configuration enables TLS 1.3 client support; do not assume QUIC-TLS integration or SPAKE2 is available. The mbedTLS 3.6.7 PSA PAKE source includes EC J-PAKE; that is not evidence of SPAKE2. The ngtcp2 upstream documentation lists TLS backends such as BoringSSL/AWS-LC, wolfSSL and OpenSSL; it does not establish a ready-to-use mbedTLS integration. Research crypto/backend feasibility before choosing a QUIC port.

Sources:
- https://github.com/aros-development-team/AROS/blob/master/workbench/libs/mbedtls/mmakefile.src
- https://github.com/aros-development-team/AROS/blob/master/workbench/libs/mbedtls/mbedtls_aros_config.h
- https://github.com/aros-development-team/AROS/blob/master/workbench/libs/mbedtls/mbedtls_entropy_aros.c
- https://github.com/Mbed-TLS/mbedtls/blob/mbedtls-3.6.7/library/psa_crypto_pake.c
- https://github.com/ngtcp2/ngtcp2

### Relationship to AROS Matter

[AROS Matter](https://github.com/JJDSNT/aros-matter) is a **separate project** focused initially on a Matter controller. Matter commissioning uses SPAKE2+ and its operational network stack does not depend on QUIC. Do not add Matter or the connectedhomeip SDK as Prism dependencies. The projects can independently motivate reusable upstream improvements to AROS networking, IPv6, multicast, DNS-SD/Bonami, cryptography, entropy, timers and credential handling, where applicable.

Shared infrastructure should live upstream only after its actual API consumers, portability, security properties and licensing are validated. Avoid prematurely introducing one AROS shared library per algorithm or protocol.

### Parallel tracks and decision gates

- **Display vertical slice:** keep the existing RFB-first plan, starting with the upstream Headless HIDD and historical AROS VNC audit. An actual display demonstration must not be blocked on Open Screen/QUIC research.
- **Open Screen feasibility:** pin W3C drafts and Open Screen Library revision; audit exact network/application requirements; produce a dependency and interoperability matrix.
- **Generic AROS connectivity:** audit IPv6, multicast, Bonami, mbedTLS, SPAKE2 variant availability, QUIC-compatible TLS backends, event loops and timers. Only propose independently useful, testable upstream patches.
- **Boundary acceptance:** no coupling between Prism and Matter, between RTP and AppleMIDI, or between reusable wire-protocol APIs and Prism-specific Screen/HIDD structures.

The first deliverable for this research track is an **evidence-backed gap report**, not an unconditional decision to implement QUIC or Open Screen before RFB.

## Agent instructions / guardrails

- Treat this as a research/architecture record, not authorization to rewrite upstream AROS or invent APIs.
- Verify all third-party claims against exact source revisions, README/LICENSE files, and build dependencies.
- Inventory before extracting; propose minimal patches with independent tests.
- Keep Prism focused on display topology, outputs, sessions, and integration with AROS graphics; keep protocol implementations reusable outside Prism.
- Record unanswered questions and evidence explicitly. Do not conflate display mirroring, independent Screens, virtual outputs, or continuous extended desktop.
