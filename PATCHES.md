# Patches to Apply

## PATCH 1 — user/config.h
Add `D3D11` to `EGraphicsAPI` enum. See config.h line ~68.

## PATCH 2 — user/config.cpp
Add `{ "D3D11", EGraphicsAPI::D3D11 }` to the enum template at line ~309.

## PATCH 3 — gpu/video.cpp
Add `extern std::unique_ptr<RenderInterface> CreateD3D11Interface();`
to the `namespace plume` block.

## PATCH 4 — gpu/video.cpp
Replace:
    interfaceFunctions.push_back(CreateVulkanInterfaceWrapper);
with:
    interfaceFunctions.push_back(CreateD3D11Interface);
    interfaceFunctions.push_back(CreateVulkanInterfaceWrapper)