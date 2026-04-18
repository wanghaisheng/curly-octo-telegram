
```
The user wants to migrate to a Monorepo-style architecture, but the guidance explicitly notes that in Lovable projects, we must keep everything inside src/ to avoid breaking the Vite + Supabase type-gen toolchain. So this is a "Modular Monolith" — logical monorepo structure mirrored inside src/.

Let me explore current state quickly to ground the plan.

Key facts I already know from context:

src/storage/ adapter layer exists (Supabase/sqljs/d1)

src/services/ exists as logic layer

src/components/, src/screens/, src/pages/, src/hooks/ exist

src/integrations/supabase/ is auto-generated, must NOT move

4 files still import supabase directly (audit found)

Hardcoded URLs in data/, colors in components

The plan: build src/packages/ and src/apps/ mirror inside src/, move logic into "packages", keep src/pages + entry as the "app". No tRPC for now (overkill — keep Action layer as typed function calls inside packages/api). Phase work to avoid breaking Vite/Supabase.

Keep it concise per instructions.

<lov-plan>

Modular Monolith Migration (Inside src/)

Mirror a monorepo structure inside src/ to preserve Lovable's Vite + Supabase auto-generation toolchain. Logically monorepo, physically transparent to the compiler.

Target Structure
code
Text
download
content_copy
expand_less
src/
  apps/
    web/                       ← The React SPA (current pages + entry)
      pages/                     (moved from src/pages)
      screens/                   (moved from src/screens)
      App.tsx, main.tsx          (moved from src/)
  packages/
    ui/                        ← Shared UI primitives
      components/                (moved from src/components, minus app-specific)
    core/                      ← Platform-agnostic utilities
      utils/                     (moved from src/lib + src/utils)
      config/                    NEW: AppConfig (URLs, CDN, feature flags)
    api/                       ← Action Layer (contract center)
      actions/                   NEW: typed action functions (getUser, createPost…)
      services/                  (moved from src/services — internal to api)
    storage/                   ← Storage Adapter (moved from src/storage)
    types/                     ← Shared TS types (moved from src/types + character.ts)
    hooks/                     ← React hooks (moved from src/hooks)
    i18n/                      ← Language provider (moved from components/ui/language-provider)
  integrations/                ← UNCHANGED (Supabase auto-gen)
  index.css                    ← UNCHANGED (Vite/Tailwind entry)

Path aliases (tsconfig.json + vite.config.ts):

@/* → src/* (kept for back-compat)

@app/* → src/apps/web/*

@ui/* → src/packages/ui/*

@core/* → src/packages/core/*

@api/* → src/packages/api/*

@storage/* → src/packages/storage/*

@types/* → src/packages/types/*

@hooks/* → src/packages/hooks/*

@i18n/* → src/packages/i18n/*

Action Layer (API Contract Center)

Replace direct service calls from hooks with typed actions. Actions are the public contract; services become internal implementation.

code
Ts
download
content_copy
expand_less
// src/packages/api/actions/blog.ts
import { fetchPosts as _fetchPosts } from '../services/blogService';
import type { Result, BlogPost } from '@types';

export const blogActions = {
  list: async (filters?): Promise<Result<BlogPost[]>> => {
    try { return { ok: true, data: await _fetchPosts(filters), source: 'network' }; }
    catch (e) { return { ok: false, error: e as Error }; }
  },
  // get, create, update, delete…
};

Future: same blogActions shape can be backed by tRPC, REST, or local SQLite — UI doesn't change.

Phased Execution

Phase 1 — Scaffold + aliases (no moves yet)

Create empty src/apps/web/, src/packages/{ui,core,api,storage,types,hooks,i18n}/ with READMEs

Add path aliases to tsconfig.json, tsconfig.app.json, vite.config.ts

Verify build still passes

Phase 2 — Move leaf packages (low-risk)

src/storage/ → src/packages/storage/

src/types/ + src/data/characters.ts types → src/packages/types/

src/lib/ + src/utils/ → src/packages/core/utils/

Create src/packages/core/config/ with AppConfig (SITE_DOMAIN, CDN_BASE, external links)

Update imports project-wide via search/replace

Phase 3 — Move services + introduce Actions

src/services/ → src/packages/api/services/ (internal)

Create src/packages/api/actions/ wrapping each service with Result<T> pattern

Hooks call actions, not services directly

Phase 4 — Move hooks, i18n, UI

src/hooks/ → src/packages/hooks/

src/components/ui/language-provider.tsx → src/packages/i18n/

src/components/ui/* → src/packages/ui/components/ui/

Domain components stay where used or move to src/apps/web/components/

Phase 5 — Move app shell

src/pages/ → src/apps/web/pages/

src/screens/ → src/apps/web/screens/

src/App.tsx, src/main.tsx, src/App.css → src/apps/web/

Update index.html script src to /src/apps/web/main.tsx

Phase 6 — Cleanup

Remove now-empty old directories

Remove temporary @/ shims if all imports migrated

Update docs/ARCHITECTURE.md + READMEs

What Does NOT Move

src/integrations/supabase/ (auto-generated)

src/index.css (Vite/Tailwind entry)

src/vite-env.d.ts

supabase/, public/, index.html, tailwind.config.ts, vite.config.ts (root configs)

Risks & Mitigations
Risk	Mitigation
Vite HMR breaks	Keep src/ as Vite root; only add aliases, don't move outside src/
Supabase type-gen breaks	src/integrations/supabase/ untouched
Import-path churn	Do moves package-by-package, run tsc --noEmit after each phase
Lovable auto-fix confusion	Keep @/* alias as transitional fallback
Recommended Scope for THIS Implementation Pass

```
