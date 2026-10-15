# Unleashed Recompiled — D3D11 Backend Handoff

## GOAL
Run Sonic Unleashed Recompiled on lower-end GPUs (specifically NVIDIA NVS 4200M,
Fermi-era, D3D11 Feature Level 10.1, no D3D12, no Vulkan).

## WHO THIS IS FOR
A contributor with C++, D3D11, and ideally shader-cross-compilation experience.
Assume they have NOT read the previous chat logs.

## CURRENT STATE (as of handoff)
- ✅ Full recompiled build: `UnleashedRecomp.exe` (115 MB) builds from source
- ✅ 261 PPC recomp files generated (XenonRecomp output)
- ✅ All third-party libs build (plume, SDL2, fmt, msdfgen, etc.)
- ✅ Custom D3D11 backend exists at `thirdparty/plume/plume_d3d11.cpp`
- ⚠️ D3D11 backend is INCOMPLETE and UNTESTED at runtime
- ❌ Game still errors with "Unable to create a D3D12 or Vulkan backend" on NVS 4200M
- ❌ The D3D11 backend is not yet wired into the backend selection code

## WHAT'S ALREADY BEEN DONE
- Stubbed curl in `UnleashedRecomp/install/update_checker.cpp` (no network)
- Removed D3D12 CMake block (`UnleashedRecomp/CMakeLists.txt`)
- `/FORCE:MULTIPLE` added to allow patches to override recompiled stubs
- Plume CMake modified to compile `plume_d3d11.cpp`
- Multiple `__builtin_*` shims added to `UnleashedRecompLib/ppc/ppc_context.h`
- byteswap fixes in XenonRecomp/XenosRecomp/x_decompress

## WHAT'S NOT DONE — NEXT STEPS
1. **Wire D3D11 into the backend selection** (in `gpu/video.cpp`)
   - See `PATCHES.md` for the exact diffs — this was the next task
2. **Fix shader format mismatch** (THE HARD PART)
   - Game shaders are SPIR-V (from XenosRecomp) or DXIL — neither works on D3D11
   - D3D11 needs **DXBC bytecode** (SM4.x/5.0)
   - Options:
     a. Add SPIRV-Cross to build, convert SPIR-V → HLSL → compile with fxc/d3dcompiler
     b. Pre-convert all shaders offline, ship DXBC blobs
     c. (Best long-term) Get XenosRecomp to emit DXBC directly
   - Note: NVS 4200M maxes at **SM4.1 / FL10.1**, so even DXBC SM5.0 won't run.
     Shaders must target **SM4.1 or SM4.0**.
3. **Downgrade feature-level calls** in `plume_d3d11.cpp`
   - Search for: `UNORDERED_ACCESS`, `STRUCTURED_BUFFER`, `D3D11_FEATURE_LEVEL_11_*`
   - Each needs a FL10.1-compatible fallback
4. **Push constants need to become constant buffers** (D3D11 has no push constants)
   - Already partially stubbed in `D3D11CommandList::setGraphicsPushConstants`
   - Needs a real implementation using ID3D11Buffer + map/unmap
5. **Descriptor sets need real D3D11 implementation**
   - Current `D3D11DescriptorSet::apply` is a naive placeholder
6. **Query pools / timestamps** — currently stubbed, will need DXGI query objects
7. **Command fences / semaphores** — currently stubbed, will need D3D11 event queries
8. **MSAA resolve** — D3D11 has `ResolveSubresource`, but pipeline needs to match

## HOW TO BUILD
- Visual Studio 2026 Community, x64 Native Tools prompt
- Freetype: `C:\Users\User\Downloads\freetype-master\freetype-master\`
- Curl: STUBBED — no libcurl needed
- CMake config:
cmake .. -G "Visual Studio 18 2026" -A x64 -DUNLEASHED_RECOMP_D3D11=ON ^
-DFREETYPE_INCLUDE_DIRS="<path>" -DFREETYPE_LIBRARY="<path>"
- Build:
cmake --build . --config Release --target UnleashedRecomp > build.log 2>&1
findstr /I /C:"error" /C:"LNK" /C:"fatal" build.log


## KNOWN ISSUES / GOTCHAS
- Notepad++ sometimes silently fails to save — verify edits with `(Get-Content file)[N]`
- Stale `.obj` files — `rmdir /s /q CMakeFiles` when headers change
- `ppc_recomp.*.cpp` regenerate from `default.xex` + `default.xexp` + `SWA.toml`
- `/FORCE:MULTIPLE` is required for manual overrides to work
- The `ppc_context.h` MSVC shim block is fragile — add builtins there, never in generated files

## GAME ASSETS (NOT INCLUDED — USER MUST PROVIDE)
- Extracted `default.xex` from `game\default.xex`
- Extracted `default.xexp` from `update\default.xexp`
- Extracted `shader.ar` from `game\shader.ar`
- Place in `UnleashedRecompLib\private\`

## TEST HARDWARE
- NVIDIA NVS 4200M (FL10.1, no D3D12, no Vulkan) — the target
- Any modern GPU for regression testing (D3D12/Vulkan path must still work)

## PRIORITY IF YOU ONLY HAVE A FEW HOURS
1. Wire D3D11 into `video.cpp` (see `PATCHES.md`) — 30 min
2. Build and run — see what the FIRST runtime error is
3. Report back what that error is before starting shader work

## CONTACT / ORIGIN
Built by a solo dev over weeks with rotating AI assistance.
No formal team. The contributor who picks this up IS the team now.