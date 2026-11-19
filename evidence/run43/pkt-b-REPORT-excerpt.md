# Excerpt: saleor-run43/pkt-b REPORT (finding 1.1, site-cache KeyError race)

Source: the pkt-b tester's first-look report (packet workdir, not in
version control), lines 23–52. Nothing else is changed.

> ### 1.1 Site cache clear races get_current return read → KeyError (race, bug)
>
> - Confirmation `4fda0fb3fb9a`, verdict `confirmed`, kind `race` ("with each
>   thread running whole calls, TLC finds no violation of NoKeyError in the
>   model, so the failure needs a thread switch inside a call"). Evidence:
>   statically confirmed. Branch conditions not checked.
> - Check: violated `NoKeyError`, bounds Threads=2, Instances=2, 33 states.
>   Profile: "default (UNCONFIRMED): instances unknown, 4 process(es) per
>   instance, checked with 2 in-process copies; Instances = 2: instances unknown:
>   checked with 2 (worst case of 1 or more)". Neighbours: none.
> - Runtime judged: cpython-gil ("python_runtime cpython-gil [inferred:
>   Dockerfile#FROM python:3.9-slim] of deployment model saleor/default
>   (UNCONFIRMED; passed service/profile)").
> - Assumptions: cache matched coarsely (module tag; the index skips the
>   annotated assignment inside a with-block); ghost `failed` not compared;
>   switch points from cpython 3.14.6 bytecode; threads share one object;
>   unanswered sharing question assumed yes (see depends_on).
> - Depends_on: "Do several threads use module `saleor.site.patch_sites` at
>   once? [sharing:saleor.site.patch_sites]" (open; assumed yes).
> - What breaks: `new_get_current` (`saleor/site/patch_sites.py:24-28`) checks
>   the key outside the lock and reads it back outside the lock, while
>   `new_clear_cache` (`:59-62`) rebinds the global under the lock. Trace:
>   thread 1 checks hit → thread 2 clears → thread 1's `return
>   THREADED_SITE_CACHE[site_id]` raises KeyError. The module docstring claims a
>   "thread-safe structure"; the return read is not. The two request-host paths
>   share the shape. Init premises checked by reading: `{}` is a cold/just-
>   cleared cache, `{S}` a warm one.
> - Fix direction: re-check under the lock, or return the fetched object instead
>   of re-reading the global.
