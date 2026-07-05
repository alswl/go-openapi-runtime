# alswl/go-openapi-runtime

Fork of [go-openapi/runtime](https://github.com/go-openapi/runtime) maintained by [@alswl](https://github.com/alswl).

## Why this fork

This fork carries patches that are not yet upstream, primarily:

### `SetPathParamEscaped` API

Per-parameter control over URL path escaping. Default behavior is unchanged —
all path param values are still `url.PathEscape`'d. Call `SetPathParamEscaped(name, false)`
to opt out for pre-escaped or literal values.

```go
req.SetPathParam("id", "foo/bar")           // → /flats/foo%2Fbar/
req.SetPathParamEscaped("id", false)        // → /flats/foo/bar/
```

### Denco Router (vendored in `middleware/denco/`)

Fork of [naoina/denco](https://github.com/naoina/denco) with modifications
for OpenAPI routing. See [FORK.md](middleware/denco/FORK.md).

## Versioning

All tags use `-fork` suffix to avoid collision with upstream:

```
Upstream:  v0.32.4  →  v0.33.0  →  v0.34.0
Fork:      v0.33.0-fork   v0.34.0-fork
```

- Version number = upstream base + 1 minor (e.g. `v0.33.0-fork` = upstream `v0.32.4` + our patches)
- `-fork` suffix makes fork releases explicit
- Annotated tags with changelog summary

## Remotes

```bash
origin  git@github.com:go-openapi/runtime.git          # upstream
alswl   git@github.com:alswl/go-openapi-runtime.git    # this fork
```

## Branch strategy

```
origin/master (upstream) ─────────────────────────────────►
                           \
fork ───────────────────────●────────────────────────────────►
                             \
release/v* ──────────────────●── (feature work, then force-push to fork)
```

| Branch | Purpose |
|--------|---------|
| `fork` | Main fork branch: tracks upstream + our patches |
| `release/v*` | Release preparation: feature work happens here, then force-pushed to `fork` |
| `feat/*` | Feature branches (merge to `release/v*` or directly to `fork`) |

### Rules

1. **`fork` receives force-pushes from `release/v*` branches** — it's the release target, not a development branch
2. **Feature work happens on `release/v*`** — once everything is ready and tested, force-push to `fork` and tag
3. **Keep history clean** — squash/rebase before force-pushing

## Upgrade workflow

When upstream releases a new version:

```bash
# 1. Fetch upstream
git fetch origin --tags

# 2. Create a release branch from upstream
git checkout -b release/v<NEXT>-fork origin/master

# 3. Cherry-pick our patches from the previous fork tag
git cherry-pick <commit-range>

# 4. Adapt to any upstream changes, run tests
go test ./...
go test -race ./...

# 5. Tag and force-push
git tag -a v<NEXT>-fork -m "v<NEXT>-fork: based on upstream v<UPSTREAM>"
git push --force-with-lease alswl fork
git push alswl v<NEXT>-fork
```

## Cherry-picking

After merging, source commits live on `fork`. Find and pick them by SHA:

```bash
git log fork --oneline
git cherry-pick <sha>
```

## Upstream PR Status

| Feature | Status |
|---------|--------|
| `SetPathParamEscaped` | TBD |

---

Based on upstream `v0.32.4`. Last updated: 2026-07-05.
