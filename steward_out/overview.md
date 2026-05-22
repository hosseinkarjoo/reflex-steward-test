# reflex

**Version:** unknown  
**Modules:** 117 across 8 layers (Core, Entry, Infrastructure, Internal, Middleware, Models, Plugins, Routes)

## Architecture

**Core** (22 files)
- `reflex/components/component.py`
- `reflex/components/dynamic.py`
- `reflex/components/field.py`
- `reflex/components/literals.py`
- `reflex/components/props.py`
- `reflex/components/tags/cond_tag.py`
- `reflex/components/tags/iter_tag.py`
- `reflex/components/tags/match_tag.py`
- `reflex/components/tags/tag.py`
- `reflex/components/tags/tagless.py`
- `reflex/environment.py`
- `reflex/event.py`
- `reflex/istate/dynamic.py`
- `reflex/style.py`
- `reflex/vars/base.py`
- `reflex/vars/color.py`
- `reflex/vars/datetime.py`
- `reflex/vars/dep_tracking.py`
- `reflex/vars/function.py`
- `reflex/vars/number.py`
- `reflex/vars/object.py`
- `reflex/vars/sequence.py`

**Entry** (2 files)
- `reflex/__main__.py`
- `reflex/app.py`

**Infrastructure** (41 files)
- `reflex/_upload.py`
- `reflex/app_mixins/lifespan.py`
- `reflex/app_mixins/mixin.py`
- `reflex/assets.py`
- `reflex/compiler/templates.py`
- `reflex/compiler/utils.py`
- `reflex/istate/data.py`
- `reflex/istate/manager/disk.py`
- `reflex/istate/manager/memory.py`
- `reflex/istate/manager/redis.py`
- `reflex/istate/manager/token.py`
- `reflex/istate/proxy.py`
- `reflex/istate/shared.py`
- `reflex/istate/storage.py`
- `reflex/page.py`
- `reflex/reflex.py`
- `reflex/state.py`
- `reflex/testing.py`
- `reflex/utils/build.py`
- `reflex/utils/codespaces.py`
- `reflex/utils/console.py`
- `reflex/utils/exceptions.py`
- `reflex/utils/exec.py`
- `reflex/utils/export.py`
- `reflex/utils/format.py`
- `reflex/utils/frontend.py`
- `reflex/utils/frontend_skeleton.py`
- `reflex/utils/js_runtimes.py`
- `reflex/utils/misc.py`
- `reflex/utils/net.py`
- `reflex/utils/path_ops.py`
- `reflex/utils/prerequisites.py`
- `reflex/utils/processes.py`
- `reflex/utils/redir.py`
- `reflex/utils/registry.py`
- `reflex/utils/rename.py`
- `reflex/utils/tasks.py`
- `reflex/utils/telemetry.py`
- `reflex/utils/templates.py`
- `reflex/utils/token_manager.py`
- `reflex/utils/types.py`

**Internal** (29 files)
- `reflex/admin.py`
- `reflex/compiler/compiler.py`
- `reflex/config.py`
- `reflex/constants/base.py`
- `reflex/constants/colors.py`
- `reflex/constants/compiler.py`
- `reflex/constants/config.py`
- `reflex/constants/custom_components.py`
- `reflex/constants/event.py`
- `reflex/constants/installer.py`
- `reflex/constants/route.py`
- `reflex/constants/state.py`
- `reflex/constants/utils.py`
- `reflex/custom_components/custom_components.py`
- `reflex/experimental/client_state.py`
- `reflex/experimental/hooks.py`
- `reflex/experimental/memo.py`
- `reflex/plugins/_screenshot.py`
- `reflex/plugins/base.py`
- `reflex/plugins/shared_tailwind.py`
- `reflex/plugins/sitemap.py`
- `reflex/plugins/tailwind_v3.py`
- `reflex/plugins/tailwind_v4.py`
- `reflex/utils/compat.py`
- `reflex/utils/decorator.py`
- `reflex/utils/imports.py`
- `reflex/utils/lazy_loader.py`
- `reflex/utils/pyi_generator.py`
- `reflex/utils/serializers.py`

**Middleware** (4 files)
- `reflex/app_mixins/middleware.py`
- `reflex/istate/wrappers.py`
- `reflex/middleware/hydrate_middleware.py`
- `reflex/middleware/middleware.py`

**Models** (1 file)
- `reflex/model.py`

**Plugins** (2 files)
- `reflex/compiler/plugins/builtin.py`
- `reflex/compiler/plugins/memoize.py`

**Routes** (1 file)
- `reflex/route.py`

## Flow Rules

**Allowed:** `Core -> Internal`, `Entry -> Core`, `Entry -> Infrastructure`, `Entry -> Internal`, `Entry -> Middleware`, `Entry -> Models`, `Entry -> Routes`, `Infrastructure -> Entry`, `Infrastructure -> Internal`, `Infrastructure -> Models`, `Infrastructure -> Routes`, `Internal -> Infrastructure`, `Internal -> Plugins`, `Middleware -> Infrastructure`, `Plugins -> Infrastructure`, `Plugins -> Internal`
**Blocked:** `Core -> Entry`, `Core -> Infrastructure`, `Core -> Middleware`, `Core -> Models`, `Core -> Plugins`, `Core -> Routes`, `Entry -> Plugins`, `Infrastructure -> Core`, `Infrastructure -> Middleware`, `Infrastructure -> Plugins`, `Internal -> Core`, `Internal -> Entry`, `Internal -> Middleware`, `Internal -> Models`, `Internal -> Routes`, `Middleware -> Core`, `Middleware -> Entry`, `Middleware -> Internal`, `Middleware -> Models`, `Middleware -> Plugins`, `Middleware -> Routes`, `Models -> Core`, `Models -> Entry`, `Models -> Infrastructure`, `Models -> Internal`, `Models -> Middleware`, `Models -> Plugins`, `Models -> Routes`, `Plugins -> Core`, `Plugins -> Entry`, `Plugins -> Middleware`, `Plugins -> Models`, `Plugins -> Routes`, `Routes -> Core`, `Routes -> Entry`, `Routes -> Infrastructure`, `Routes -> Internal`, `Routes -> Middleware`, `Routes -> Models`, `Routes -> Plugins`
