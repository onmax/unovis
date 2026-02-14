# Vite Graph Expansion Analysis

This package keeps barrel imports (for example `import { CurveType } from '@unovis/ts'`) for convenience.

At the same time, Vite may still parse a large module graph during dependency processing because root barrels re-export many modules.

## Re-export fan-out

1. Root barrel:
   - `src/index.ts` re-exports `./components`, `./types`, and other namespaces.
2. Component barrel:
   - `src/components.ts` re-exports all component entrypoints, including graph and map components.
3. Type barrel:
   - `src/types.ts` re-exports many component type modules.

Even when only one enum is imported from the root barrel, Vite can traverse re-export targets while constructing the module graph.

## Why this is not runtime eager-loading

Graph and map layout runtimes are still lazy-loaded where intended (for example dynamic imports in graph layout and map modules).
The observed overhead is mostly from build-time graph traversal, not from runtime eager execution.

## Mitigation strategy in this package

1. Preserve root barrels.
2. Add a lightweight additive entrypoint:
   - `@unovis/ts/enums`
3. Keep granular imports available:
   - `@unovis/ts/types/curve`
4. Convert type-only imports to `import type` where applicable to avoid unnecessary runtime dependency edges.

## Repro harness

Run:

```bash
pnpm --filter @unovis/ts forcebuild
pnpm --filter @unovis/ts check:vite-graph
```

Strict assertions:

```bash
pnpm --filter @unovis/ts check:vite-graph --assert
```

The harness reports transformed module counts and whether heavy packages (`elkjs`, `three`, `leaflet`, `maplibre-gl`) are present for each fixture.
