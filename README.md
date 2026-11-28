# VIctory on an RX 7900 XTX

## Playability report: Linux, branch `linux-wine` (2026-11-28)

**Played, on Linux under Proton, through Lucia's prison prologue and into the first
mission, with a controlled character, at about 30 fps at 1920x1080.** One PC, one
session (run 242, about 12 minutes of game time), started from a save in the prison
cell. Ray tracing off. No game files are in this repository.

- **PC:** AMD Radeon RX 7900 GRE 16 GB, Ryzen 9 9900X, 46 GB RAM, CachyOS Linux,
  Mesa 26.3.1 (RADV), Proton Experimental (Wine 11), vkd3d-proton 3.2.0.
- **How it was run:** `scripts/play-1080p.sh` (Series X profile, the picture made at
  1920x1080 and run through FSR 4.1.1 `native` as anti-aliasing, the interface on its
  own plane). The character was steered with the layer's stand-in pad (W/A/S/D left
  stick, I/J/K/L camera, Z/C triggers, Enter for A), driven live through
  `VICTORY_KEYS_FILE` (diagnostics builds; `crates/victory-hle/src/modules/gameinput.rs`).

What happened, in order (the game's own objectives and subtitles, in the French
version played):

1. The prison cell: the move/camera hint shows, the character turns and walks.
2. Objective "approach a fellow inmate to interact with her": walked to an inmate at a
   table; the interaction plays.
3. Objective "go with the prison guard": followed C.O. Wilson out of the dormitory,
   through the common room and a barred gate, down the corridors.
4. The parole office cutscene ("Lucia Caminos?", "a supervised release programme").
5. Out of prison: the cutscene of Lucia loading a pistol, then outside by day in front
   of the shops.
6. **First mission: the hold-up of a cheque-cashing shop ("CAJERO")**, the clerk
   on his knees, the aiming tutorial on screen. Stopped there: the stand-in pad's left
   trigger did not satisfy the tutorial's aim step in this session.

Frame times over the same session (median of the layer's report lines): the café
cutscene 37.3 ms (27 fps), the prison 31.0 ms (32 fps), its corridors 26.9 ms (37 fps),
the shop 44.0 ms (23 fps); in the prison scene measured on its own, 28.2 ms (35 fps,
runs 237-240). **Stability:** no crash, no device lost, nothing in the kernel log over
this session, the 420 s run through the intro to the "PROLOGUE" card (run 215) and
some sixty measured runs since 2026-11-27. One launch on 2026-11-26 hung the GPU
(amdgpu ring timeout); not seen since, cause not found.

**Rendering, honestly:** faces, hair, skin, clothes, rooms and daylight streets are
drawn and lit; FSR 4 at the output size keeps faces and fabric sharp. Fixed since the
playthrough (2026-11-28): **eyes and glasses no longer glow white indoors**. Without ray
tracing, reflection rays found nothing and took the sky's light, 100 to 1,000 times the
room's; they now take the room's light along the reflected direction
(`VICTORY_AMBIENT_SPECULAR`, after VIctory-NVIDIA). And 4,213 shaders that branched the
wrong way on two AMD compare instructions now take the right side (s_min/s_max, after
Vice-Tay and leotux), which took the last glints out of the eyes. Still open: in bright
interiors the player and lit surfaces can blow out to white, some rooms are too dark,
and textures can arrive late after a scene change.

Every figure is from `docs/linux-wine.md` on this branch, with the run behind it.

---

A fork of [VIctory](https://github.com/Marvisak/VIctory), a compatibility layer
that runs an Xbox Series title natively on Windows from files the user supplies.

## The goal

**The Series X version at 4K output (drawn at 1440p) with ray tracing on,
looking the way it does on the Xbox**, on an AMD RX 7900 XTX first.

In this order:

1. **A correct picture.** Textures and models reach full detail and stay there
   (no pop-in or pop-out, no flickering trees), and ray-traced lighting is right:
   not too dark, not too bright, no flicker.
2. **Then the frame rate:** 30 fps or more with ray tracing on, then as high a
   frame rate and as low frame times as the PC allows.
3. **Then smaller PCs:** 32 GB (later 16 GB) of RAM and smaller cards.

The way there is **porting, not emulating**: patching the title's renderer so it
behaves the way a PC renderer does, rather than having the layer watch and copy
the console's shared memory every frame. Shaders are translated ahead of time
into a versioned store that is kept between builds. The Series S profile and
ray tracing off stay available.

**This repository contains no game files, code or assets.** You need your own
copy of the game. It is not affiliated with or endorsed by the game's developer
or publisher.

## Where it stands (2026-11-26)

Tested on one machine only: Windows 11, Intel i9-12900K, 64 GB RAM, Radeon
RX 7900 XTX, renderer patches 1, 2, 5, 6 and 7 on. Runs of 270 s in a city
street: standing still, then walking and turning the camera.

| Version | Standing | Walking | RAM |
|---|---|---|---|
| **Series X, 4K drawn at 1440p, ray tracing on (the goal)** | 99 ms (10 fps) | ~110 ms (9 fps) | 45 GB |
| Series X, 4K drawn at 1440p, ray tracing off | 49 ms (20 fps) | ~62 ms (16 fps) | 45 GB |
| Series X, 1080p output, ray tracing off | 40 ms (25 fps) | ~43 ms (23 fps) | 42 GB |
| Series S, ray tracing off | 43 ms (23 fps) | ~64 ms (16 fps) | 37 GB |

Fixed 2026-11-26: trees and palms dropping out for a few frames, and most of the
lighting swinging between too dark and too bright in lamp-lit rooms. Both came from
indirect draws whose arguments sit in renderer patch 1's in-place memory being
prepared before the title's culling had written them; they are now prepared one at
a time right before they run (`VICTORY_PORT_CONSTANTS_ORDER`, on by default, about
2-5 ms). Rooms lit only by bounced light are no longer black with ray tracing
(`VICTORY_AMBIENT_RT=0.7,1.5`, which `Play-Watched.ps1 -RayTracing` sets).

Still open: models and textures stay at their lowest detail for a long time while
the title streams (worst in the intro): the title paces its streaming by its own
frames, so it loads about three times slower at 10 fps than at 30; ray-traced
structures of moving objects are partly corrupt (the rays follow some 450 stray
pointers a frame).

Earlier measurements with ray tracing off:

| | |
|---|---|
| City street, Series S profile, renderer patches 1, 2, 5, 6 and 7 on | about 45 ms a frame (about 22 fps, 44.9 ms in run merge-5) |
| Same, patches 1, 2, 5 and 6 | 45.6 ms, p95 47 ms |
| Same, patches 1, 2 and 5 | 47.3 ms (about 21 fps), p95 49 ms |
| Series X profile, patches 1+2 | about 53 ms (about 19 fps) |
| Same scene before the patches | 57-75 ms (13-17 fps) |
| Shaders the title uses that translate | all of them (25,790 of 25,985; the rest are unused geometry shaders) |
| Memory | about 30 GB of private memory, 10.5 GB of VRAM: **64 GB of RAM recommended** for now |

It started at 280 ms a frame in the same scenes three days earlier. What sets
the frame now is the one thread that submits the title's work to the card
(about 40 ms); the card itself needs about 36 ms and now waits about 7 ms a frame with nothing queued.

### Renderer patches (off by default while they are tested)

| Switch | What it does |
|---|---|
| `VICTORY_PORT_CONSTANTS=1` | The title's 174 MB constants heap is read by the card where the title writes it: no uploads, no fills |
| `VICTORY_PORT_DESCRIPTORS=1` | The title's descriptor-table stores are patched (byte-checked) to report the slots they write, instead of a scan of ~600,000 slots a frame |
| `VICTORY_PORT_CBV=1` | Translated shaders read constant buffers in place as raw bytes (~22,000 fills a frame to none; -4.7 ms) |
| `VICTORY_PORT_GUARD=1` | The write guards' lock is shared by faults and refreshes, with a claim per 64 KiB block; the submission's planning runs beside it (title threads' waits 29 ms to ~2 ms a frame; -2 ms) |
| `VICTORY_PORT_JOURNAL=1` | The heaps' records are asked for at the title's submit, and the journal's notes are split into 64 parts with locks of their own (the submit thread's waits ~5 ms + 1 ms to ~0.3 ms; -1 to -3 ms) |

### Other switches

| Switch | What it does |
|---|---|
| `--console series-s` | The Series S profile: 1080p output, about 5 ms faster, a softer picture |
| `VICTORY_OUTPUT=1920x1080` | The title renders to that output size itself (Series X profile; about 16 ms less card time) |
| `VICTORY_FRAME_INTERVAL_MS=16.667` | Lets the title go above 30 fps (menus and videos reach 60; the game's clock stays right) |
| `VICTORY_AMBIENT_SETTLE` | How fast light settles after walking indoors (on by default, `250,3`; `0` turns it off) |
| `VICTORY_TEXTURE_STORE=1` | Converted textures kept between runs (about 8 ms a frame for now: off) |

What is known to be wrong:

- Trees, signs and far objects flicker in the distance in some places.
- Lighting is too bright or too dark after walking indoors, until the title has
  run for a while; rooms with little direct light are too dark without `-RayTracing`.
- Textures of what just came into view appear late while the title streams.
- The first run of a new build stutters while the driver makes pipelines
  (`victory warm` makes them ahead).
- Recording a frame in the world makes the title's own watchdog end it.

## How to run it

You need Windows 10/11 x64, a CPU with AVX2, a Direct3D 12 GPU with shader
model 6.6 and mesh shaders, Rust (the toolchain is pinned), the MSVC build
tools, and your own dump of the title.

```powershell
git clone https://github.com/avatarmanz/VIctory-7900XTX
cd VIctory-7900XTX
copy victory.example.toml victory.toml      # then set your paths, outside this folder
cargo build --locked --profile diag -p victory --features diagnostics
scripts\play.ps1 -Exe <path of your copy's executable>
```

`scripts\play.ps1 -Safe` leaves out the newest, faster ways; try it first when
something is wrong. `-RayTracing` turns the title's ray-traced indirect light on.

For PC FSR 4.1.2, set `[paths].fidelityfx_dir` in `victory.toml` or
`VICTORY_FIDELITYFX_DIR` to the folder containing AMD's signed
`amd_fidelityfx_upscaler_dx12.dll`. Use the DirectX 12 upscaler in
`Kits/FidelityFX/signedbin`. [AMD FSR SDK 2.4.0](https://github.com/GPUOpen-LibrariesAndSDKs/FidelityFX-SDK/tree/v2.4.0/Kits/FidelityFX/signedbin)

`scripts\play.ps1 -Exe <path> -Fsr4K quality` selects 2560×1440 input for
the retail 1.0.2.1 Series X 4K SDR profile. `balanced` selects 2258×1270
and `performance` selects 1920×1080. All three output 3840×2160. With
`victory run` directly, set `VICTORY_FSR_4K` to the preset name; `native`
(not in the launcher) renders at the output's own size, FidelityFX as
anti-aliasing. With `VICTORY_OUTPUT=<width>x<height>` beside it (one of the
title's modes, as without a profile) that size is put out instead and the
preset's share of it rendered: `VICTORY_OUTPUT=1920x1080` with `quality`
renders 1280×720. `VICTORY_FSR_VERSION=<version>` asks the library for
another upscaler than 4.1.2 (`4.1.1`, `3.1.5`: what it or the driver offers).
The log reports the provider at context creation and the actual input and
output sizes at the first successful temporal dispatch. The title profile
and PC FidelityFX bridge were ported from [DexNom/VIctory](https://github.com/DexNom/VIctory/commit/ab34a8b21477c94da2a6cf07249138e8bc84602e).

On a hybrid-graphics PC, the runtime and offline pipeline checks prefer a
high-performance adapter with Shader Model 6.6. The runtime logs its adapter
name, PCI vendor and device IDs, and dedicated VRAM. Set
`VICTORY_D3D12_ADAPTER` to a zero-based DXGI index or part of the adapter's
name to select it explicitly. With the launcher, pass
`-More @{VICTORY_D3D12_ADAPTER='NVIDIA'}`.

Before the first run, ahead of time (both can be stopped and go on later):

```powershell
target\diag\victory.exe shaders <the shader archives>   # every shader translated, minutes
target\diag\victory.exe warm --used                     # pipelines earlier runs drew with, for a new driver or build
```

Commands, debugging switches and upstream's own description:
[docs/README-fxceless.md](docs/README-fxceless.md). Every `VICTORY_...` switch is
described in a comment where it is read.

## Linux (Wine)

The same `victory.exe` runs under Wine with vkd3d-proton, started by
`scripts/play.sh`, the counterpart of `play.ps1`. Measured on one PC (Ryzen 9
9900X, Radeon RX 7900 GRE 16 GB with RADV of Mesa 26.3, Proton Experimental's
Wine 11, kernel 7.3), Series X profile, ray tracing off, in the title's first
scene: 2.3 frames a second before this work, 21 at 3840x2160, and 36 to 37 with
`scripts/play-1080p.sh` (the title rendering at 1920x1080, renderer patches 1,
2 and 5, the title let above its 30 frames a second). Everything that is
Wine's alone is behind `on_wine()` or a switch and leaves Windows as it was.

You need a Vulkan driver with mesh shaders, Proton (its Wine, vkd3d-proton and
DXVK's `dxgi.dll` are copied into a prefix of their own; `PROTON=<folder>`
names another than Steam's Proton Experimental), Microsoft's 64-bit
`d3dcompiler_47.dll` (`D3DCOMPILER=<path>`; Wine's own cannot compile the
runtime's shaders), Rust with the `x86_64-pc-windows-msvc` target and
[cargo-xwin](https://github.com/rust-cross/cargo-xwin), and your own dump of
the title.

```sh
git clone https://github.com/avatarmanz/VIctory-7900XTX
cd VIctory-7900XTX
cp victory.example.toml victory.toml        # then set your paths as Wine sees them (Z:\home\...), outside this folder
cargo xwin build --locked --profile diag -p victory --features diagnostics --target x86_64-pc-windows-msvc
scripts/play.sh --cmd shaders                        # every shader of the title translated, once
scripts/play.sh <your copy's executable>             # as play.ps1 does
scripts/play-1080p.sh                                # GAME_EXE=<executable>: 1920x1080, the fastest settings measured
```

`scripts/play.sh --safe`, `--rt`, `--start` (into the first scene without the
keyboard) and `KEY=VALUE` settings are described at the top of the script;
`VICTORY_SCREEN=<output>` puts the window on another monitor than the primary
one.

What was found under Wine, with the runs behind each figure, the switches and
what is still open: [docs/linux-wine.md](docs/linux-wine.md). In short:

- Wine's server walked its list of views at every map and unmap: the title's
  pages are shown with the host's `mmap` instead (`VICTORY_WINE_MAP`).
- Programs with loops of several back edges ran vkd3d-proton's dxil-spirv out
  of stack; they get one latch a loop (`VICTORY_SINGLE_LATCH`), and a pipeline
  whose making ends its worker is given up (`VICTORY_PIPELINE_WATCH`).
- Renderer patch 1 gave a black picture: amdgpu takes memory by address only as
  one anonymous mapping, so the title's graphics pages are shown as one
  (`VICTORY_PORT_HOME`).
- People other than the heroine had no face in every second or third run: a
  compute pass the title runs once was dropped while its pipeline was being
  made. Compute passes wait for theirs (`VICTORY_COMPUTE_WAIT_MS`), and the
  pipeline library is kept under Wine as on Windows (`VICTORY_LIBRARY_NULL`).
- The waits around the layer's own clamps of indirect arguments emptied the
  card under RADV; they name what they wait for (`VICTORY_CLAMP_SYNC`).

Not looked at: any scene but the first, play, ray tracing, other cards and
drivers.

## What is different here

In short; the reasons and the measurements are in [docs/findings.md](docs/findings.md).

- **List calls are written down, not carried out, on the title's threads**
  (`VICTORY_THIN`): eight threads of ours translate them.
- **Submissions are prepared by a thread of ours and joined**
  (`VICTORY_ASYNC_SUBMIT`, `VICTORY_JOIN_PARTS`), with what can be spread over
  threads spread: refreshes, compares (AVX2), descriptor records, landings.
- **What shaders wrote comes back beside the write guards**, by the dwords that
  changed, on a thread of its own (`VICTORY_RELAY`, `VICTORY_LAND`).
- **Buffer fills that need bytes still on the card are done on the card**
  (`VICTORY_CARD_STEPS`) instead of waiting for it.
- **The title's writes are caught more cheaply**: one view opened at a fault,
  blocks that rest, places in the title's code that look before they write.
- **Textures go up on a copy queue of their own** and a slot shows one only after
  its upload ran, which also ended a device loss that came every 15 minutes or so.
- **Boats float**: textures that shaders write and the title's CPU reads go back
  to title memory (from fxceless and Geckomacabre, on by default here).
- **Frame record and replay** with per-pass times and shader variants compared
  turn by turn ([docs/replay.md](docs/replay.md)).

## Credits

This is a tree of other people's work with more on top. The history is kept, and
commits taken from elsewhere name the commit and its author.

- **[VIctory](https://github.com/Marvisak/VIctory)** by Marvisak and its
  contributors: the loader, the runtime services, the Direct3D 12 back end and
  the shader recompiler.
- **[fxceless/victory-fork](https://github.com/fxceless/victory-fork)**: the fork
  this one started from, and the people whose pull requests it carries.
- **[DexNom/VIctory](https://github.com/DexNom/VIctory)**: ray tracing fixes,
  cube gradients, sparse textures.
- **[Belobug/VIctory-Belobug-AMD](https://github.com/Belobug/VIctory-Belobug-AMD)**:
  the ambient light idea, pipeline workers, and work by romainhedouin.
- **[FQY55kai/ZJOU-VICTORY](https://github.com/FQY55kai/ZJOU-VICTORY)**.
- **[myokex/victory-amd](https://github.com/myokex/victory-amd)**: the guard over
  every view of a page (with Spexivus, joroyt and tightfall), and Joshua Odom's
  reads into title memory.
- **Geckomacabre** and **cxy0072019-sketch**: the texture read-back.
- **[XCL](https://github.com/ivall/xcl)** (GPL-2.0-only) was read for how the same
  title behaves under another layer; no code of it is here.
- This fork's changes were made with [Claude Code](https://claude.com/claude-code).

## License

GPL-3.0-or-later, as VIctory. See [LICENSE](LICENSE); the one file that is not
covered is named in [docs/asset-policy.md](docs/asset-policy.md).
