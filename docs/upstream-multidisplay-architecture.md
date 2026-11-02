# Upstream AROS multi-display architecture: Kal's design note

**Recorded:** 2026-11-02  
**Source:** Kal's statement addressed to CoolCat (project maintainer), supplied in discussion. No upstream commit or public discussion URL has been verified for this statement.

> "CoolCat multi-display support will be done through intuitions monitor classes, and a special bitmap class that forwards operations to the relevant gfx hidds ... it is something my gfx subsystem work has been working towards."

## What Kal described

- Multi-display support is intended to use **Intuition monitor classes**.
- A **special bitmap class** will forward graphics operations to the relevant **Gfx HIDDs**.
- Kal indicates that his ongoing graphics-subsystem work has been moving toward this architecture.

This is a statement of architectural direction, **not confirmation of a completed, merged, or publicly documented API**.

## Implications for Prism

Prism should integrate with the upstream multi-display infrastructure rather than independently recreate monitor management, bitmap dispatch, or graphics-driver routing.

Potential separation of concerns (proposal, not confirmed upstream behavior):

1. **Intuition monitor classes:** monitor-level integration and Screen/output association, subject to actual API inspection.
2. **Special bitmap class:** upstream operation forwarding to the appropriate graphics HIDD.
3. **Gfx HIDDs:** device-specific graphics implementation.
4. **Prism:** remote-display discovery, connection/session management, transport protocols, and preferences that complement upstream display management.

A virtual or network-backed Gfx HIDD **might** allow a remote receiver to function as an independent display destination. This remains a hypothesis; mirroring an existing framebuffer is not equivalent to adding a distinct monitor or extending a desktop.

## Questions to resolve with Kal / upstream code

- Are the monitor classes and special bitmap class already implemented, under development, or only planned? In which branch/commits?
- Can a Gfx HIDD represent a **virtual, headless, or remote output**, or must it correspond to physical display hardware?
- What are the bitmap forwarding rules for operations involving two displays or HIDDs (copies, blits, ownership, pixel formats, synchronization)?
- How will Screen placement, window migration, pointer routing, and extended desktop coordinates be handled?
- What lifecycle events and APIs expose output enumeration, hotplug, removal, and mode changes?
- What is the preferred integration boundary for Prism's network transports without coupling network code to Intuition or the Gfx HIDD implementation?

## Recommended next steps

- [ ] Identify Kal's relevant upstream branches, PRs, commits, and API definitions.
- [ ] Map the monitor and bitmap classes to the existing AROS graphics/Intuition paths.
- [ ] Confirm the virtual-output extension point with Kal before designing a Prism display HIDD.
- [ ] Prototype one virtual display and test independent Screen output before desktop extension.
- [ ] Keep network protocols modular and reusable by clients and servers, independent of display routing.

**Design constraint:** Do not implement competing monitor or bitmap dispatch abstractions until upstream extension points have been investigated.
