AROS Prism
A multi-display management subsystem for AROS, supporting display mirroring, an extended desktop, independent Screens, and remote displays over wired and wireless networks.
Status: project definition and initial research. The features below describe intended capabilities, not an implemented or tested release.
Suggested repository name: aros-prism.
Overview
AROS Prism aims to provide a consistent way to discover, configure, and use multiple displays in AROS. Its scope covers local hardware outputs and network-connected displays, while preserving the Amiga-style concept of independent Screens.
The project is intended to serve AROS across architectures. AROS m68k running through Emu68 on Raspberry Pi is a particular development interest; support on that platform must be established against its actual graphics and network drivers.
Broad protocol coverage is an explicit project goal. Miracast, Google Cast, AirPlay, and Bluetooth-related mechanisms are included from the initial scope, alongside remote desktop and real-time streaming protocols. The backend model should allow additional protocols without redesigning the display manager. Inclusion in scope means intended research and development, not confirmed interoperability.
Intended capabilities
Capability	Intended behavior
Mirroring	Present the same Screen or desktop content on multiple outputs.
Extended desktop	Provide a larger logical desktop across outputs, with appropriate window placement and pointer movement.
Independent Screens	Assign different AROS Screens to different outputs, with independent display modes where supported.
Network displays	Use a remote receiver as an output over Ethernet or Wi-Fi.
Wireless display interoperability	Investigate compatibility with existing wireless display receivers and protocols.
Display preferences	Configure output arrangement, resolution, primary output, mirroring groups, and Screen assignments.
Connection management	Handle connection, disconnection, reconnection, and restoration of saved layouts.


An AROS Screen is a system-managed graphical surface that can contain windows. It is not synonymous with a physical monitor. Showing two independent Screens on separate monitors and extending one desktop across two monitors are distinct capabilities.
Initial research
AROS integration
The upstream OpenScreen implementation contains display-mode and monitor-selection logic. This is an existing integration point to investigate; it is not proof that arbitrary multi-output layouts or desktop extension work on every AROS port.
The implementation investigation should cover:
- Graphics HIDD objects, output enumeration, bitmaps, and display-mode selection.
- graphics.library display registration and presentation paths.
- Intuition Screen placement, window coordinates, focus, and pointer routing.
- Wanderer desktop behavior across outputs.
- ScreenMode preferences and persistent display configuration.
- Drivers that expose multiple physical outputs, and a possible virtual-display backend.
The exact upstream or downstream branch and commit must be recorded before implementation. API availability and driver behavior need verification on each target rather than assumptions based on another architecture.
Existing remote-display work
Icaros Desktop documentation describes a VNC remote Workbench facility. Existing AROS VNC implementations should therefore be inventoried before introducing another capture or transport implementation. Their source availability, licenses, supported architectures, and capture methods remain to be checked.
Raspberry Pi and Emu68
The Raspberry Pi 3 Model B specification lists a full-size HDMI port, a DSI display port, and composite video. These connectors do not establish simultaneous-output support in AROS. Each output combination requires firmware and driver investigation.
A local display plus a network-backed virtual output is a candidate configuration for this platform. Whether it can provide independent Screens or a continuous extended desktop must be demonstrated in AROS. Emu68 execution does not by itself provide a multi-display abstraction.
Network protocols and reference projects
Candidate	Relevance	Important qualification
RFB / VNC	Framebuffer updates and optional remote input; candidate for a first network display prototype.	Desktop extension requires an additional logical surface in AROS. Sending an existing framebuffer alone provides remote viewing or mirroring.
Miracast / Wi-Fi Display	Candidate interoperability with compatible TVs and receivers.	Requires investigation of discovery, Wi-Fi Direct support, session negotiation, encoding, and receiver compatibility.
AirPlay	Candidate interoperability with Apple-oriented display receivers.	Sender and receiver implementations are different roles; a receiver project does not provide an AROS sender.
Google Cast / Chromecast	Candidate transmission to Google Cast-enabled devices.	Media playback, live video delivery, and desktop mirroring are separate use cases. Official sender/receiver application support does not establish a native AROS screen-mirroring implementation.
Bluetooth	Investigate video distribution, discovery, pairing, control, and possible IP connectivity.	Bluetooth is a family of technologies and profiles, not one universal wireless-monitor protocol. Assess VDP/GAVDP/AVDTP video mechanisms and actual receiver support separately from discovery or control.
RDP	Candidate remote display and input interoperability.	An AROS client connecting to another OS is different from an AROS server exporting a Screen or virtual output. Neither alone establishes desktop extension.
WebRTC	Candidate live display delivery to a browser or dedicated receiver.	Requires a media pipeline, signaling, and input handling where desired; it is not a drop-in TV casting protocol.
RTP / RTCP	Candidate real-time audio/video transport for custom or compatible receivers.	Transport must be combined with codecs, session setup, and receiver behavior. RTP alone is not a complete wireless-display solution.
Project-specific receiver	Allows experiments with virtual outputs and display transport.	Requires a client on the receiving device and a documented protocol.


Relevant reference implementations:
- LibVNCServer / LibVNCClient: portable C libraries for VNC server and client functionality. The README specifies GPL version 2 or later. AROS portability and integration need assessment.
- MiracleCast: a Miracast reference project. Its README explicitly states that the Display-Source side is not implemented. Its Linux-oriented requirements include systemd, GLib, GStreamer, and wpa_supplicant. It is a research reference, not a ready-to-use AROS sender. The README describes LGPL licensing; individual files and bundled licenses need inspection before reuse.
- UxPlay: an AirPlay mirroring and audio receiver under GPLv3. It is relevant to protocol research and potential external receiver experiments, not an implementation of transmission from AROS to an Apple TV.
Wi-Fi connectivity and Miracast support must be tracked separately. Transmission over an ordinary IP network does not establish Miracast interoperability.
Bluetooth investigation should include the Bluetooth SIG Video Distribution Profile (VDP), Generic A/V Distribution Profile (GAVDP), and A/V Distribution Transport Protocol (AVDTP). Discovery and control over Bluetooth with image transport over Wi-Fi is another candidate design. Compatibility with the AROS Bluetooth stack should be assessed, including possible integration with the separate Bluzing project. No existing Bluzing profile support is assumed.
For every backend, document sender/receiver roles, mirroring, independent virtual-output support, desktop extension, audio, remote input, discovery, authentication, dependencies, and tested receiver models. Sender functionality from AROS is the primary display-output use case; receiver functionality can be explored as an additional capability.
Proposed architecture
The following decomposition is a design proposal, not an existing AROS API:
Component	Responsibility
Display manager	Track outputs, capabilities, topology, assignments, and saved layouts.
AROS integration	Connect display management to HIDD, graphics, Intuition, and desktop behavior.
Local output backends	Present content through supported graphics drivers.
Virtual output backend	Provide a renderable surface for a remote display.
Network backends	Discover or connect receivers, negotiate sessions, and send updates.
Preferences and diagnostics	Configure displays and report driver, network, and performance information.


Physical outputs, virtual outputs, and transport sessions should have distinct identities. A network reconnection should not silently change the meaning of a saved desktop layout.
Cursor handling, pixel formats, changed-region tracking, buffering, and scaling are part of the investigation. Video encoding and hardware acceleration are optional backend capabilities to evaluate, rather than prerequisites assumed to exist.
Initial development milestones
- [ ] Record the target AROS branch, commit, architecture, hardware, and toolchain.
- [ ] Inventory existing monitor APIs, driver capabilities, and AROS VNC software.
- [ ] Build a diagnostic utility that reports displays and available modes.
- [ ] Demonstrate capture and mirroring of one Screen to a network receiver.
- [ ] Demonstrate a virtual output containing content independent of the local output.
- [ ] Demonstrate independent Screens on two available outputs.
- [ ] Implement and validate continuous desktop extension and input routing.
- [ ] Add display preferences, saved layouts, and disconnect recovery.
- [ ] Evaluate local multi-output hardware configurations.
- [ ] Investigate Miracast and AirPlay sender interoperability as separate backends.
- [ ] Investigate Google Cast media delivery and desktop mirroring separately.
- [ ] Investigate Bluetooth video profiles, discovery, control, and AROS stack integration.
- [ ] Evaluate RDP, WebRTC, and RTP/RTCP backends and document their supported roles.
- [ ] Maintain an extensible protocol compatibility matrix and investigate further relevant protocols.
Validation goals
Tests and demonstrations should identify the specific platform and backend used. Relevant scenarios include different resolutions, indexed and true-color formats, cursor transitions, focus changes, Screen switching, disconnects, and reconnects.
Performance reporting should include CPU load, memory use, update rate, bandwidth, and end-to-end latency. No resolution, frame rate, or protocol compatibility guarantee is established yet.
For desktop extension, validation must show that the secondary output contains independently rendered content and that the desktop and input model behave correctly. A duplicated framebuffer is not sufficient.
Network display sessions should define authentication, access control, and transport protection. Remote input should be an explicit capability that can be disabled independently of viewing.
Licensing
The license for original Prism code has not yet been selected. This README does not grant a license to future source code.
AROS code is governed by its applicable file notices, including the AROS Public License 1.1. Third-party code retains its own licenses. Before adoption, record the exact revision, license, dependencies, and intended integration method of each component. No blanket compatibility conclusion is made for combining APL, GPL, and LGPL components.
Contributing
Contributions are welcome in graphics-driver research, Intuition and desktop integration, network protocols, receiver development, and platform testing.
Research reports should include reproducible steps, hardware details, source revisions, and observed behavior. Proposed features should clearly distinguish implementation results from architectural ideas.
References
Initial public-source review: September 30, 2026. This review did not include compilation or testing on AROS hardware.
- AROS upstream repository
- AROS OpenScreen implementation
- AROS Intuition autodocs
- AROS Public License 1.1
- Icaros Desktop manual
- RFC 6143 — Remote Framebuffer Protocol
- LibVNCServer / LibVNCClient
- MiracleCast
- UxPlay
- Raspberry Pi 3 Model B specifications
- Google Cast overview
- Bluetooth Video Distribution Profile 1.1
- Bluetooth Generic A/V Distribution Profile 1.3
- Bluetooth A/V Distribution Transport Protocol 1.3
- Microsoft Remote Desktop Protocol overview
- RFC 8835 — Transports for WebRTC
- RFC 3550 — RTP
