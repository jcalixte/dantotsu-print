# Central print service routing to networked label printers

**Status:** accepted

Cards reach board printers through a small internal print service: the CLI renders client-side, then POSTs the card image + board-id to the service, which forwards the raster to that board's networked Brother QL-1110NWB by IP. We chose this over direct CLI→printer and over corporate CUPS queues because at 50+ boards the dominant cost is operational — a board→printer registry, retries, and "did it print / is it out of labels" visibility — which a central router provides and 50 hand-configured connections do not (serves G2/F4).

## Considered options

- **Direct-to-printer (T-A).** No server, but 50 addresses to track, cross-floor reachability issues, and no central status.
- **Central service (T-B, chosen).** One registry + `--board` routing + status; also the future home of the F5 short-link redirector.
- **Corporate CUPS (T-C).** Reuses IT ops, but label media/routing is awkward in generic queues and couples the project to IT's roadmap.

## Consequences

- The service is a single point of failure: if it is down, printing stalls. Mitigation (watched, not built for v1): keep it simple/HA and let the CLI cache the registry to fall back to direct-print.
- Printers must be networked (QL-1110NWB) rather than USB-only — slightly pricier per unit, but avoids a Raspberry Pi agent (and its patching burden) on every board.
- Rendering stays client-side; the service is a thin router, not a renderer.
