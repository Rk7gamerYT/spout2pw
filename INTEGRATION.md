# Netrunner Linux VTuber integration

Initial preparation: 2026-10-21.

## Scope and upstream

This is a public integration fork of <https://github.com/tasokait/spout2pw>,
based on upstream commit `5b22ec96617ad04c02a197859afcc09152436899`.
The original project is <https://github.com/hoshinolina/spout2pw>.
This initial change adds documentation only. It does not change bridge source,
build scripts, dependency revisions or license terms, and publishes no binaries.

## Licensing and architecture boundary

- The inherited Spout2PW bridge and derivative modifications retain LGPL-2.1
  terms in `COPYING.txt`. This fork does not relicense them as MIT or Apache.
- Existing third-party notices and licenses must be retained. Dependency
  licenses remain their own; audit the actual build artifacts before distribution.
- Netrunner Studio / NetEngine's receiver is proprietary, implemented
  independently in its own codebase, without copying bridge implementation code
  or linking the Studio / NetEngine executable against this bridge.
- The intended interface is a standard PipeWire video stream, with separately
  running bridge and receiver. Any third-party code used by the receiver must
  be assessed under its own license.
- Distribution of bridge binaries must satisfy the bridge's source, notice and
  license obligations. This repository alone is not evidence that a future
  binary package meets those obligations.

The data path is:

`VTube Studio / VSeeFace (Wine/Proton) -> Spout2PW -> PipeWire -> NetEngine receiver`

The Windows Spout2/D3D11 receiver remains a separate, existing backend.

## Acceptance criteria (pending)

- [ ] Audit pinned dependencies and produce reproducible source/build inputs.
- [ ] Build and run the bridge with recorded Wine/Proton, driver and GPU versions.
- [ ] Enumerate sender names without confusing unrelated video nodes with Spout senders.
- [ ] Receive negotiated buffers, including required DMA-BUF modifiers and synchronization.
- [ ] Verify transparency, premultiplied alpha, color, orientation and resizing.
- [ ] Handle sender loss/restart, multiple senders and name/identity changes.
- [ ] Validate VTube Studio and VSeeFace individually, with clean avatar output.
- [ ] Measure latency, resource use and stability under streaming load.
- [ ] Validate the packaged bridge and its corresponding source/notices.

No item is marked complete merely because the bridge was forked or a PipeWire
node exists. The fork is currently unvalidated with these applications.

## Receiver milestone, 2026-10-21

The independent, proprietary NetEngine receiver has been implemented in its
own codebase. Tests there cover shared-memory input, linear and tiled DMA-BUF
import through EGL, alpha conversion, vertical flip, capture-rate limits,
multiple receivers, sender restart and resizing. A further receiver milestone
now copies DMA-BUF input into owned GPU slots and imports those into Vulkan,
without CPU pixel readback or texture upload. This uses one GPU copy, bounded
frame leases and completion before returning the PipeWire producer buffer;
it is not strict zero-copy. Color/alpha conversion and flip run on the GPU.

The GPU path passed native synthetic tests on Intel UHD 770 and NVIDIA RTX
5050, including modifier compatibility, pixels, alpha, flip, resize, retained
storage and forced import failure with CPU fallback. Intel-to-NVIDIA input used
CPU fallback on the tested machine. The normal Widgets entry also passed on
RTX 5050 with both previews reporting GPU transport. The native suite passed
78/78 tests, including 30 GPU tests. Direct/flat capture-to-encoder shortcuts
are not enabled for these Linux DMA-BUF frames; encoding uses composition.

The Studio Widgets entry was exercised with a synthetic DMA-BUF producer.
A separate libfunnel probe using this fork's pinned libfunnel revision and
synchronous mode also delivered frames. These receiver-side results are not
validation of Spout2PW with VTube Studio or VSeeFace. All app-specific acceptance
criteria above remain pending. No receiver source or binary is included here,
and this update changes neither the bridge implementation nor its licensing.
