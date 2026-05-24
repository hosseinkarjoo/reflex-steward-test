# reflex Architecture Health Report
Generated: 2026-05-22 13:45:05 UTC | Version: unknown | Status: DEGRADED | Score: 46/100

## Summary

| Area | Status | Detail |
|------|--------|--------|
| Architecture | FAIL | 23 violations (0 critical) |
| Documentation | OK | all sources current |
| Test Coverage | OK | N/A checks covered, 0 untested |
| AST Parse | OK | 119 files, 0 errors |

## Architecture Violations

### High
- `compiler.py` → `app.py` — Internal calls Entry (17 call sites)
- `utils.py` → `datetime.py` — Infrastructure calls Core (1 call site)
- `hydrate_middleware.py` → `app.py` — Middleware calls Entry (1 call site)
- `middleware.py` → `app.py` — Middleware calls Entry (1 call site)
- `state.py` → `dynamic.py` — Infrastructure calls Core (1 call site)
- `prerequisites.py` → `datetime.py` — Infrastructure calls Core (1 call site)

### Warn
- `telemetry.py` → `datetime.py` — Infrastructure calls Core (1 call site)

### Coupling Hotspots

| File | Lines | Fan-out | Fan-in | Flags |
|------|-------|---------|--------|-------|
| `reflex/app.py` | 1743 | 22 | 4 | high_fan_out, god_file |
| `reflex/compiler/compiler.py` | 1288 | 12 | 3 | high_fan_out, god_file |
| `reflex/compiler/utils.py` | 909 | 6 | 4 | high_fan_out, god_file |
| `reflex/experimental/memo.py` | 1160 | 0 | 9 | high_fan_in, god_file |
| `reflex/istate/manager/__init__.py` | 274 | 7 | 5 | high_fan_out, high_fan_in |
| `reflex/istate/manager/disk.py` | 387 | 5 | 1 | high_fan_out |
| `reflex/istate/manager/token.py` | 244 | 2 | 6 | high_fan_in |
| `reflex/reflex.py` | 948 | 11 | 1 | high_fan_out, god_file |
| `reflex/state.py` | 2655 | 15 | 14 | high_fan_out, high_fan_in, god_file |
| `reflex/testing.py` | 918 | 7 | 1 | high_fan_out, god_file |
| `reflex/utils/build.py` | 334 | 6 | 2 | high_fan_out |
| `reflex/utils/console.py` | 4 | 0 | 16 | high_fan_in |
| `reflex/utils/exec.py` | 831 | 5 | 7 | high_fan_out, high_fan_in, god_file |
| `reflex/utils/frontend_skeleton.py` | 510 | 6 | 5 | high_fan_out, high_fan_in |
| `reflex/utils/js_runtimes.py` | 760 | 7 | 6 | high_fan_out, high_fan_in |
| `reflex/utils/misc.py` | 133 | 0 | 6 | high_fan_in |
| `reflex/utils/path_ops.py` | 320 | 0 | 10 | high_fan_in |
| `reflex/utils/prerequisites.py` | 711 | 11 | 17 | high_fan_out, high_fan_in |
| `reflex/utils/processes.py` | 556 | 5 | 5 | high_fan_out, high_fan_in |
| `reflex/utils/telemetry.py` | 453 | 4 | 6 | high_fan_in |
| `reflex/utils/telemetry_accounting.py` | 331 | 5 | 1 | high_fan_out |
| `reflex/utils/templates.py` | 445 | 7 | 1 | high_fan_out |
| `reflex/utils/token_manager.py` | 463 | 5 | 1 | high_fan_out |

## Documentation Drift

| Source | Status | Mismatches |
|--------|--------|------------|

To apply corrections:
```bash
cp steward_out/skills-patch.json <skills-file>  # after review
```

## Test Coverage

### By Module

| Module | Layer | Test Files | Level |
|--------|-------|-----------|-------|
| `app.py` | Entry | 11 | well_tested |
| `compiler.py` | Internal | 2 | well_tested |
| `memoize.py` | Plugins | 3 | well_tested |
| `utils.py` | Infrastructure | 3 | well_tested |
| `environment.py` | Core | 6 | well_tested |
| `event.py` | Core | 4 | well_tested |
| `memo.py` | Internal | 3 | well_tested |
| `data.py` | Infrastructure | 4 | well_tested |
| `disk.py` | Infrastructure | 2 | well_tested |
| `memory.py` | Infrastructure | 7 | well_tested |
| `redis.py` | Infrastructure | 6 | well_tested |
| `token.py` | Infrastructure | 8 | well_tested |
| `proxy.py` | Infrastructure | 2 | well_tested |
| `storage.py` | Infrastructure | 3 | well_tested |
| `model.py` | Models | 4 | well_tested |
| `reflex.py` | Infrastructure | 2 | well_tested |
| `route.py` | Routes | 2 | well_tested |
| `state.py` | Infrastructure | 32 | well_tested |
| `testing.py` | Infrastructure | 51 | well_tested |
| `prerequisites.py` | Infrastructure | 6 | well_tested |
| `token_manager.py` | Infrastructure | 2 | well_tested |
| `lifespan.py` | Infrastructure | 1 | lightly_tested |
| `assets.py` | Infrastructure | 1 | lightly_tested |
| `templates.py` | Infrastructure | 1* | lightly_tested |
| `state.py` | Internal | 1 | lightly_tested |
| `client_state.py` | Internal | 1 | lightly_tested |
| `hydrate_middleware.py` | Middleware | 1 | lightly_tested |
| `page.py` | Infrastructure | 1 | lightly_tested |
| `frontend.py` | Infrastructure | 1 | lightly_tested |
| `frontend_skeleton.py` | Infrastructure | 1 | lightly_tested |
| `js_runtimes.py` | Infrastructure | 1 | lightly_tested |
| `precompressed_staticfiles.py` | Unknown | 1 | lightly_tested |
| `processes.py` | Infrastructure | 1 | lightly_tested |
| `rename.py` | Infrastructure | 1 | lightly_tested |
| `tasks.py` | Infrastructure | 1 | lightly_tested |
| `telemetry.py` | Infrastructure | 1 | lightly_tested |
| `base.py` | Core | 1 | lightly_tested |
| `__main__.py` | Entry | 0 | **UNTESTED** |
| `_upload.py` | Infrastructure | 0 | **UNTESTED** |
| `admin.py` | Internal | 0 | **UNTESTED** |
| `middleware.py` | Middleware | 0 | **UNTESTED** |
| `mixin.py` | Infrastructure | 0 | **UNTESTED** |
| `builtin.py` | Plugins | 0 | **UNTESTED** |
| `component.py` | Core | 0 | **UNTESTED** |
| `dynamic.py` | Core | 0 | **UNTESTED** |
| `field.py` | Core | 0 | **UNTESTED** |
| `literals.py` | Core | 0 | **UNTESTED** |
| `props.py` | Core | 0 | **UNTESTED** |
| `cond_tag.py` | Core | 0 | **UNTESTED** |
| `iter_tag.py` | Core | 0 | **UNTESTED** |
| `match_tag.py` | Core | 0 | **UNTESTED** |
| `tag.py` | Core | 0 | **UNTESTED** |
| `tagless.py` | Core | 0 | **UNTESTED** |
| `config.py` | Internal | 0 | **UNTESTED** |
| `base.py` | Internal | 0 | **UNTESTED** |
| `colors.py` | Internal | 0 | **UNTESTED** |
| `compiler.py` | Internal | 0 | **UNTESTED** |
| `config.py` | Internal | 0 | **UNTESTED** |
| `custom_components.py` | Internal | 0 | **UNTESTED** |
| `event.py` | Internal | 0 | **UNTESTED** |
| `installer.py` | Internal | 0 | **UNTESTED** |
| `route.py` | Internal | 0 | **UNTESTED** |
| `utils.py` | Internal | 0 | **UNTESTED** |
| `custom_components.py` | Internal | 0 | **UNTESTED** |
| `hooks.py` | Internal | 0 | **UNTESTED** |
| `dynamic.py` | Core | 0 | **UNTESTED** |
| `shared.py` | Infrastructure | 0 | **UNTESTED** |
| `wrappers.py` | Middleware | 0 | **UNTESTED** |
| `middleware.py` | Middleware | 0 | **UNTESTED** |
| `_screenshot.py` | Internal | 0 | **UNTESTED** |
| `base.py` | Internal | 0 | **UNTESTED** |
| `shared_tailwind.py` | Internal | 0 | **UNTESTED** |
| `sitemap.py` | Internal | 0 | **UNTESTED** |
| `tailwind_v3.py` | Internal | 0 | **UNTESTED** |
| `tailwind_v4.py` | Internal | 0 | **UNTESTED** |
| `style.py` | Core | 0 | **UNTESTED** |
| `build.py` | Infrastructure | 0 | **UNTESTED** |
| `codespaces.py` | Infrastructure | 0 | **UNTESTED** |
| `compat.py` | Internal | 0 | **UNTESTED** |
| `console.py` | Infrastructure | 0 | **UNTESTED** |
| `decorator.py` | Internal | 0 | **UNTESTED** |
| `exceptions.py` | Infrastructure | 0 | **UNTESTED** |
| `exec.py` | Infrastructure | 0 | **UNTESTED** |
| `export.py` | Infrastructure | 0 | **UNTESTED** |
| `format.py` | Infrastructure | 0 | **UNTESTED** |
| `imports.py` | Internal | 0 | **UNTESTED** |
| `lazy_loader.py` | Internal | 0 | **UNTESTED** |
| `misc.py` | Infrastructure | 0 | **UNTESTED** |
| `net.py` | Infrastructure | 0 | **UNTESTED** |
| `path_ops.py` | Infrastructure | 0 | **UNTESTED** |
| `pyi_generator.py` | Internal | 0 | **UNTESTED** |
| `redir.py` | Infrastructure | 0 | **UNTESTED** |
| `registry.py` | Infrastructure | 0 | **UNTESTED** |
| `serializers.py` | Internal | 0 | **UNTESTED** |
| `telemetry_accounting.py` | Unknown | 0 | **UNTESTED** |
| `templates.py` | Infrastructure | 0 | **UNTESTED** |
| `types.py` | Infrastructure | 0 | **UNTESTED** |
| `color.py` | Core | 0 | **UNTESTED** |
| `datetime.py` | Core | 0 | **UNTESTED** |
| `dep_tracking.py` | Core | 0 | **UNTESTED** |
| `function.py` | Core | 0 | **UNTESTED** |
| `number.py` | Core | 0 | **UNTESTED** |
| `object.py` | Core | 0 | **UNTESTED** |
| `sequence.py` | Core | 0 | **UNTESTED** |

\* import-only — not functionally tested

### Checks Coverage: 0/0 (N/A)


## Intentional Overrides

_No intentional overrides registered._

## Architectural Alignment

Natural path: **Clean** | Current: 33%

| Pattern | Score | Cost to 100% |
|---------|-------|--------------|
| Clean ✓ | 33% | — |
| Mvc | 29% | — |
| Hexagonal | 21% | — |
| Layered | 0% | — |

## Recommendations

1. **Consider splitting app.py**
   God file with high_fan_out, god_file

2. **Consider splitting compiler.py**
   God file with high_fan_out, god_file

3. **Consider splitting utils.py**
   God file with high_fan_out, god_file

4. **Consider splitting memo.py**
   God file with high_fan_in, god_file

5. **Consider splitting reflex.py**
   God file with high_fan_out, god_file

6. **Consider splitting state.py**
   God file with high_fan_out, high_fan_in, god_file

7. **Consider splitting testing.py**
   God file with high_fan_out, god_file

8. **Consider splitting exec.py**
   God file with high_fan_out, high_fan_in, god_file
