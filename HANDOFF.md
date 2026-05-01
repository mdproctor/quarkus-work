# casehub-work — Session Handover
**Date:** 2026-05-01

## Project Status

637 runtime tests passing. All known intermittent failures fixed and pushed. Working tree clean.

## What Was Done This Session

### Four intermittent failures fixed

**NotificationDeliveryTest** (`casehub-work-notifications`):
- `Thread.sleep()` → Awaitility `await().atMost(5s).untilAsserted()`
- No cleanup between tests → `@AfterEach @Transactional` deletes rules, audit entries, and work items in FK order
- Awaitility added to `casehub-work-notifications/pom.xml`

**WorkItemNativeIT.reports_slaBreaches_e2e_breach_appears**:
- Smoke test cached a 0-breach result at production TTL; e2e test hit same cache key
- Fixed: `?from=now-5m` creates a distinct `CompositeCacheKey`

**WorkItemGroupLifecycleEventTest.completedEventFiresExactlyOnceAtThreshold** (root: production bug):
- `WorkItemService.claim()` loaded `WorkItemSpawnGroup` (@Version entity) read-only, then `persistAndFlush()` flushed it alongside WorkItem. `MultiInstanceCoordinator` concurrently updated the spawn group version → OCC
- Fixed: `em.detach(group)` immediately after reading — `EntityManager` injected into `WorkItemService`

**WorkItemEventBroadcaster** (log noise, not test failure):
- `BroadcastProcessor.onNext()` throws `BackPressureFailure` with no subscribers — "lack of requests" means zero consumers, not slow consumer
- Fixed: catch `BackPressureFailure` silently in `onEvent()`

### CLAUDE.md updated with two new gotchas
- `persistAndFlush()` flushes entire session — detach read-only `@Version` entities
- `BroadcastProcessor` throws on no subscribers

## Open / Next

| Priority | What |
|---|---|
| 1 | #150 — broadcaster SPI (do before #93) |
| 2 | #93 — Distributed SSE (SPI + CDI design agreed) |
| 3 | #148–#152 — refinement epic |

## Key References

- Bug fix commits: `95bf119` (notifications), `2f3ff2c` (sla cache), `654117a` (OCC detach), `74403de` (BackPressure)
- Blog: `blog/2026-05-01-mdp01-three-bugs-wrong-error.md`
- Garden entries: GE-20260501-ab68c1 (Hibernate flush-all OCC), GE-20260501-fbd68d (BroadcastProcessor), GE-20260501-29e3b8 (WireMock port reuse)
- Previous context: `git show HEAD~10:HANDOFF.md`
