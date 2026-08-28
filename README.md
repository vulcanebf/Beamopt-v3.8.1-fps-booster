This mod was heavily influenced by chatgpt.

BeamOpt

BeamOpt is an experimental performance optimization mod for BeamNG.drive 0.39.4.0.20972 Win64. It reduces the cost of mirrors, vehicle reflections, planar reflections, and PSSM world shadows while preserving the dedicated vehicle-shadow path.

The mod uses a normal BeamNG Lua/UI component plus a Windows PowerShell native bridge. All native modifications are RAM-only and are guarded for the exact tested BeamNG executable build.

Current tested target: BeamNG.drive 0.39.4.0.20972 Win64
Tested renderers: Direct3D 11, Direct3D 12, Vulkan
Recommended renderer for maximum gain: Direct3D 11

Performance results

Performance varies by map, vehicle, camera, graphics settings, CPU/GPU balance, and renderer. These are measured results from the development/test system, not guaranteed results for every PC.

Renderer / view

BeamOpt off

BeamOpt on

Gain

DX11, interior

24.5 FPS

~36 FPS

+11.5 FPS / +46.9%

DX11, exterior

32.2–32.5 FPS

41–44 FPS

~+26% to +37%

DX12

35.8 FPS

38.6 FPS

+2.8 FPS / +7.8%

Vulkan

33.5 FPS

35.5 FPS

+2.0 FPS / +6.0%

The largest measured improvement was on Direct3D 11, especially in interior view where native mirrors and shadows were a major CPU/render-thread cost.

Shadow-rate testing

Selective world-shadow rates of 1/2, 1/3, 1/4, and 1/5 are available.

In current testing:

1/2 is the recommended setting.

1/5 remained visually stable in the tested scene.

Going below 1/2 did not produce a meaningful additional FPS increase in that scene.

Dedicated vehicle shadows continue updating at full rate while world PSSM shadows are selectively scheduled.

This means 1/2 currently provides most of the available shadow-scheduling benefit without unnecessarily lowering world-shadow update frequency.

Main optimizations

Selective PSSM world-shadow scheduling

BeamOpt does not simply skip the entire shadow pass.

It synchronizes the main world-shadow atlas clear and world-cascade render at the selected rate while keeping the dedicated VehicleShadowTex path running at full rate.

Available world-shadow rates:

Native

1/2

1/3

1/4

1/5

This avoids the severe black/flickering shadow failures that occurred with early whole-pass frame skipping.

PSSM 6.0 fast path

BeamOpt includes the recovered PSSM 6.0 optimization.

This changes the native PSSM behavior by holding the recovered runtime value at 6.0 and suppressing the native writer that normally replaces it.

During development this produced one of the largest CPU-side improvements. CPU wait time was observed dropping dramatically in some scenes.

Enabled by default.

World caster mask 0x100

The normal world PSSM cascade caster mask is changed from 0x300 to 0x100.

This produced an additional measured performance improvement while retaining the important tested world shadows.

Enabled by default.

Optional 3-split PSSM mode

BeamNG normally uses 4 PSSM splits.

BeamOpt includes an optional 3-split mode for an additional smaller performance reduction in PSSM work.

It is disabled by default because the final recommended configuration was validated with the native 4-split layout.

Mirror scheduling

Vehicle mirror CubeReflectors can run at:

Native

1/2

1/3

1/4

1/5

Mirror updates are phase-staggered to avoid updating every reflector on the same frame.

Automatic interior/exterior mirror profile

BeamOpt detects the camera mode from live mirror topology:

Mirrors detected: treated as interior view

0 mirrors detected: treated as exterior view

The mirror buttons store the user's interior preference.

Example:

Set mirrors to 1/2 in first-person/interior view.

Switch to third-person.

BeamOpt detects 0 mirrors and leaves the mirror path effectively Native.

Return to first person.

Mirrors are detected again and the saved 1/2 preference is restored automatically.

The automatic view profile is enabled by default.

Vehicle body reflections

Vehicle body/dynamic CubeReflector updates can be reduced independently from mirrors.

Available rates:

Native

1/2

1/3

1/4

1/5

Planar reflections

PlaneReflector updates can also be scheduled independently:

Native

1/2

1/3

1/4

1/5

Low-I/O bridge

The BeamOpt control/status bridge uses a low-frequency file interface instead of constantly polling/writing.

Current cadence is approximately 0.5 Hz, which produced a measurable improvement during development.

Recommended settings

For the current tested build:

Vehicle Mirrors:          1/2 interior preference
Auto mirror profile:      ON
Vehicle Body Reflections: test 1/2 if desired
Planar Reflections:       test 1/2 if desired

World Shadows:            1/2
PSSM 6.0:                 ON
World Caster Mask 0x100:  ON
3-Split PSSM:             OFF initially

Start from this configuration, then test the reflection controls individually for your vehicle/map.

Installation

Recommended install location

I recommend keeping the BeamNG user folder somewhere easy to access, such as on the Desktop, and placing BeamOpt in that user folder's mods directory.

Example layout:

Desktop
└── BeamNG.drive
    └── mods
        └── BeamOpt_v3_8_1_AUTO_MIRROR_PROFILE_EXPERIMENTAL.zip

If your BeamNG user folder is somewhere else, use that folder's normal mods directory instead.

You can find or move the BeamNG user folder through the BeamNG launcher/user-folder management options.

1. Install the BeamNG mod

Place:

BeamOpt_v3_8_1_AUTO_MIRROR_PROFILE_EXPERIMENTAL.zip

directly in the active BeamNG user mods folder.

Do not unpack the BeamNG mod ZIP unless you are intentionally developing/editing it.

2. Install the native bridge

Extract the standalone bridge bundle somewhere convenient, for example:

Desktop\BeamOpt Bridge\

The bridge folder should contain:

BeamOpt-NativeBridge-v3.8.1-AUTO-MIRROR.ps1
Start-BeamOpt-NativeBridge-v3.8.1-AUTO-MIRROR-FIXED.cmd

Keep those two files together.

3. Start BeamNG

Launch BeamNG.drive normally.

For the exact currently supported build, the game executable should report:

0.39.4.0.20972

BeamOpt has exact-build guards and should refuse to patch an unexpected executable.

4. Start the BeamOpt native bridge

After BeamNG is running, run:

Start-BeamOpt-NativeBridge-v3.8.1-AUTO-MIRROR-FIXED.cmd

A PowerShell window should remain open and report the bridge status.

If Windows displays a security prompt, remember that this bridge opens the BeamNG process and modifies runtime memory. Review the source before running it if desired.

5. Add the BeamOpt UI app

In BeamNG:

Open the UI Apps editor.

Find BeamOpt.

Add it to the current UI layout.

Use the BeamOpt panel to select mirror, reflection, shadow, and PSSM settings.

Normal shutdown

Before closing the bridge, press:

Q

in the PowerShell bridge window.

BeamOpt will attempt to restore every native modification and verify restoration.

All native changes are RAM-only. A complete BeamNG process exit also clears them.

If the bridge is force-closed, crashes, or reports that restoration failed, fully exit BeamNG before starting another BeamOpt/native test session.

Important compatibility notes

Exact BeamNG version

The native bridge currently targets:

BeamNG.drive 0.39.4.0.20972 Win64

Do not assume native offsets remain valid after a BeamNG update.

The bridge contains build/signature validation and should fail rather than patch an unexpected build, but a new BeamNG release should still be treated as unsupported until validated.

Renderer compatibility

The current implementation has been tested successfully with:

Direct3D 11

Direct3D 12

Vulkan

The optimization gain is renderer-dependent.

Measured results:

DX11:   ~+47% in the tested interior case
DX12:   ~+8%
Vulkan: ~+6%

DX11 currently benefits the most from BeamOpt on the development system.

Do not run old experimental hooks simultaneously

Do not combine the release bridge with old standalone development scripts that patch the same BeamNG functions.

Before switching from development scripts to the packaged mod:

Press Q in every standalone test window.

Close them.

If unsure, fully exit BeamNG.

Relaunch BeamNG.

Run only the BeamOpt bridge.

What was tested and rejected

A number of ideas were tested during development and intentionally not used as final/default optimizations.

Mirror face-count cap

Vehicle mirrors and dynamic reflections were mapped, but reducing the selected cube-face count did not provide enough useful performance/quality benefit to keep.

Reflection resolution reduction

Native values observed during testing included approximately:

Vehicle mirrors:          512
Vehicle body reflection:  128
Planar reflection:        256

Mirror resolution was intentionally left at 512 for image quality.

Vehicle body reflections were already low-resolution, so further reduction was not considered worthwhile.

Shadow budget reduction

Reducing the native shadow scheduler budget from roughly 3 ms to 1 ms produced little improvement because the expensive view-dependent PSSM path is processed separately from later budget-controlled shadow maps.

lastSplitCastersEnabled

The native runtime value was already disabled (0), so there was nothing useful to optimize there.

Coarse whole-shadow frame skipping

Early versions skipped the whole ShadowMapPass.

That gave large FPS improvements but caused:

black/flashing world shadows,

stale near-cascade coverage,

vehicle-shadow judder,

high-speed/low-render-distance flicker.

The final selective PSSM scheduler was developed specifically to avoid those problems.

How the selective shadow scheduler works

BeamOpt's final shadow optimization was developed by mapping the BeamNG 0.39.4 PSSM render path.

Relevant recovered behavior includes:

PSSMLightShadowMap::_render
    world PSSM cascade work
    world shadow atlas operations
    world render-state cleanup
    dedicated VehicleShadowTex path

The production scheduler keeps the dedicated vehicle-shadow path full-rate while pairing the expensive world atlas clear and world cascade rendering on the same update phase.

This is why it can reduce world-shadow work without simply making every shadow—including the vehicle shadow—update at the reduced rate.

Safety / experimental status

BeamOpt is an experimental reverse-engineered performance mod.

It:

opens the BeamNG process,

reads native object state,

allocates small executable trampoline pages,

patches selected native instructions in RAM,

restores the original bytes when disabled or closed normally.

It does not permanently modify BeamNG.drive.x64.exe on disk.

Use it at your own risk and keep normal backups of your BeamNG user folder/mod configuration.

Current release

BeamOpt v3.8.1
Target: BeamNG.drive 0.39.4.0.20972 Win64

Current recommended baseline:

DX11
World Shadows 1/2
PSSM 6.0 ON
World Caster Mask 0x100 ON
4 PSSM splits
Auto interior/exterior mirror profile ON
Interior mirrors 1/2

On the development system, this configuration increased interior DX11 performance from approximately 24.5 FPS to 36 FPS, while the complete optimization stack produced substantial gains in multiple test scenes.

Results will vary by system.
