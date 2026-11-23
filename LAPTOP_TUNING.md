# Tuning for this laptop

Target: Ryzen 7 5700U (8C/16T, 15 W) · Radeon Vega 8 iGPU (Lucienne, Mesa radeonsi) · 14 GB shared RAM · Linux · 1080p 60 Hz · goal: balanced

## Code changes (base: upstream tag `mc26.3-0.9.3`)
- `ChunkBuilder.getOptimalThreadCount()`: SMT-aware. Uses physical cores − 2, so 10 → 6 chunk workers on this CPU.
  The CPU and iGPU share one power budget, so fewer workers leave more power for the GPU and cut heat/stutter.
- `RenderSectionManager.MIN_UPLOAD_DURATION_BUDGET`: 10ms -> 3ms. Upstream's 10ms floor (commented as "2ms")
  always won over the 30%-of-frame budget, so chunk uploads could take up to 10 of the 16.6ms in a 60Hz frame.
  The result: smoother frame pacing while flying or exploring. The trade-off is slightly slower chunk pop-in.
- `SodiumWorldRenderer`: translucency sort mode N0 -> N1. Sorting nearby water/glass is deferred by one
  frame instead of blocking the current one, so the main thread stalls less. One frame at 60Hz isn't visible.

## Build
    ./gradlew :fabric:build
    # jar: fabric/build/libs/

## In-game (balanced)
- Video Settings -> Graphics API: try **Vulkan**. The Linux RADV driver usually has less CPU overhead than
  OpenGL on Vega. If you see glitches, switch back to OpenGL. Sodium supports both.
- Max FPS 60 + VSync ON (matches the 60 Hz panel and avoids rendering frames you can't see)
- Render distance 8–10, simulation distance 6–8
- Graphics Fast, clouds Off or Fast, particles Decreased
- Leave Sodium's "Chunk Builder Threads" on Auto

## Launcher / system
- JVM: `-Xms3G -Xmx4G -XX:+UseZGC -XX:+ZGenerational` (don't go higher: the iGPU borrows system RAM)
- Plug in the charger and use the `balanced` power profile (`powerprofilesctl set balanced`)
- Keep Mesa up to date. Optional env var: `mesa_glthread=true`
