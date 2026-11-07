# Intel Mac compatibility assessment

## Summary

The existing MacResMon app could be adapted to run on Intel Macs, with most of
the current dashboard and app shell reused. It cannot provide equivalent
hardware readings on Intel Macs as-is: its hardware telemetry is built around
Apple Silicon and the `macmon` sensor source. Matching every reading across
Intel Mac models is also unlikely because sensor availability differs by
model.

This is a feasibility assessment, not a claim that Intel support is implemented
or tested.

## Current blockers

- `package.json` declares `arm64` as the supported CPU architecture.
- `native/build.sh` requires a `macmon` executable and bundles it into the app.
- `native/Sources/MacResMon/Sampler.swift` launches `macmon` for hardware
  samples and uses Apple Silicon `hw.perflevel*` sysctls for core tiers.
- `native/Sources/MonitorCore/Hardware.swift` parses `macmon` fields such as
  `cpu_active_ratio`, `pcpu_cores`, and `ecpu_cores`. There is no Intel
  telemetry provider or fallback.
- The UI includes Apple Silicon assumptions, including “unified” memory and
  Performance/Efficiency core-cluster labels (`src/Memory.jsx` and
  `src/App.jsx`).

## What can likely be reused

The Swift app shell, WebView, settings, reports, alerts, and much of the
dashboard are not intrinsically tied to Apple Silicon. Host metrics such as
processes, battery status, network traffic, and disk usage are already read
separately from `macmon`. The snapshot format also permits many hardware
readings to be absent.

## Work needed

1. Add an Intel-compatible telemetry provider and normalize its readings into
   the existing snapshot contract. Keep hardware readings optional when a
   model or sensor source does not provide them.
2. Verify CPU and per-core load collection on Intel Macs. Find and evaluate an
   appropriate source for SMC temperatures and fan RPM; sensor access and
   available fields may vary by model.
3. Treat GPU, component-level power, and other model-dependent readings as
   optional rather than promising exact parity.
4. Make the interface describe memory and core groups generically when the
   machine does not have unified memory or Apple-style performance/efficiency
   tiers.
5. Remove the `arm64`-only package restriction and update build/package logic
   for Intel binaries. Consider separate architecture builds or a universal
   app only after confirming every bundled native dependency supports both
   architectures.
6. Add parser and snapshot tests for Intel samples, then build and test on
   representative Intel Macs and macOS versions.

## Conclusion

A useful Intel version with a similar interface is feasible. A fully identical
set of CPU, GPU, power, temperature, and fan readings cannot be assumed across
Intel Mac models; it depends on an Intel sensor source and hardware-specific
validation.
