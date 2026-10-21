# Netrunner Linux VTuber integration

Initial preparation: 2026-10-21.

## Scope and upstream

This is a public integration fork of <https://github.com/tasokait/spout2pw>,
based on upstream commit `5b22ec96617ad04c02a197859afcc09152436899`.
The original project is <https://github.com/hoshinolina/spout2pw>.
The fork initially added documentation. It now also includes the small launcher
registry-persistence fix described below. Dependency revisions and license
terms are preserved; no binaries are published by this fork.

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

## Acceptance criteria

- [ ] Audit pinned dependencies and produce reproducible source/build inputs.
- [x] Run a pinned upstream bridge release with recorded Proton/GPU/application versions.
- [ ] Build and audit a reproducible bridge binary from this fork.
- [x] Enumerate sender names without confusing unrelated video nodes (synthetic tests and real VTube Studio).
- [x] Receive tested RGBA DMA-BUF layouts and acquire/release timelines on Intel/NVIDIA.
- [ ] Extend coverage to other formats/modifiers and GPU combinations.
- [ ] Verify transparency, premultiplied alpha, color, orientation and resizing.
- [ ] Handle sender loss/restart, multiple senders and name/identity changes.
- [x] Validate basic VTube Studio reception with clean avatar output on the recorded combination.
- [ ] Validate VSeeFace individually.
- [ ] Measure latency, resource use and stability under streaming load.
- [ ] Validate the packaged bridge and its corresponding source/notices.

No item is marked complete merely because the bridge was forked or a PipeWire
node exists. Application validation is limited to the real VTube Studio smoke described below.

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
validation of Spout2PW with VTube Studio or VSeeFace. The later real-application milestone below updates these initial pending items. No receiver source or binary is included here,
and this update changes neither the bridge implementation nor its licensing.


## Real VTube Studio smoke, 2026-10-21

The Steam application (1325860, build 23574781), with Spout output enabled,
was received through upstream Spout2PW 0.3.0 and
GE-Proton11-1-spout2pw-fixes-V2 on RTX 5050. The GE archive SHA256 was verified
against the release's SHA256SUMS. Stock Proton Experimental 11.0-20261014
found the sender but failed to export its shared texture handle as an FD.

The private receiver added independent DRM SyncTimeline acquire/release
negotiation, with bounded waits and completion before returning each buffer.
It received 1920x1080 GPU frames without source errors. The normal Widgets
entry presented 298 Main and 297 Vertical frames; the source published 233
frames. Avatar orientation was visually checked; composition over green
verified transparent corners and visible model pixels. These are smoke results,
not a latency benchmark or certification of long streaming sessions.

VSeeFace, real-app restart/resizing, physical GPU changes and extended load
remain pending. Earlier pending statements above describe the initial milestone.
No proprietary receiver source or binary is published in this repository.

This fork also waits up to five seconds after setupapi installation for Wine's
registry persistence before reporting failure. The race was observed during
the real smoke. This small bridge launcher change retains LGPL-2.1 terms.


Post-integration validation: the private native suite passed 81/81 tests,
including 33 GPU tests. Eleven focused pixel/receiver/timeline scenarios cover
implicit and explicit synchronization. Four Python Spout tests passed. A shell
probe verified delayed service-registration visibility through the installer
retry; `bash -n` passed. These checks do not certify VSeeFace or long workloads.


## VSeeFace environment preparation, 2026-10-21

The test machine now has system Wine 11.20, Winetricks and UMU Launcher 1.4.3.
A separate win64 prefix uses GE-Proton11-1-spout2pw-fixes-V2 through UMU, with
Arial and Visual C++ 2010/2015 installed. Bridge installation and an end-to-end
launcher check using `cmd.exe /c ver` succeeded. This is environment preparation:
the VSeeFace executable, camera, tracking and real Spout output have not been
validated. No dependency binary or proprietary launcher/receiver is included
in this public repository.
