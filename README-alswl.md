# alswl/go-openapi-runtime

Fork of [go-openapi/runtime](https://github.com/go-openapi/runtime) maintained by [@alswl](https://github.com/alswl).

## Background

This fork exists to carry patches that are not yet accepted upstream, plus project-specific modifications
needed by alswl's internal services. The primary additions are:

### 1. `SetPathParamEscaped` API (`v0.26.0`)

Added per-parameter control over URL path escaping. By default, all path parameter values are
URL-escaped via `url.PathEscape`. With `SetPathParamEscaped(name, false)`, callers can opt out
of escaping on a per-parameter basis — useful when parameter values are already escaped or when
the parameter value intentionally contains characters that should remain literal.

```go
// All params escaped (default)
req.SetPathParam("id", "foo/bar") // → /flats/foo%2Fbar/

// Opt out of escaping for a specific param
req.SetPathParam("id", "foo/bar")
req.SetPathParamEscaped("id", false) // → /flats/foo/bar/
```

### 2. Denco Router Fork

`middleware/denco/` is a vendored fork of [naoina/denco](https://github.com/naoina/denco), a fast
Double-Array URL router. It was forked because the upstream lacks features required by OpenAPI
routing. See [FORK.md](middleware/denco/FORK.md) for details.

Key modifications over upstream:
- Escaped path support (uses `r.URL.EscapedPath()` instead of decoded `r.URL.Path`)
- RESTCONF path parameter support (`=` prefix notation, RFC 8040)
- Colon-as-parameter-marker only at segment boundaries (avoids false matches)
- Dash handling in path parameters

## Versioning

This fork follows upstream's versioning with an increment:
- `v0.26.0` — based on upstream `v0.25.0` + `SetPathParamEscaped`

Tags are annotated and include a changelog summary in the tag message.

## Upgrade Workflow

When a new upstream release is available:

### Step 1: Fetch upstream

```bash
git remote add upstream https://github.com/go-openapi/runtime.git
git fetch upstream --tags
```

### Step 2: Review the diff

```bash
# List commits new to upstream since our base tag
git log v0.25.0..upstream/master --oneline

# Review full diff
git diff v0.25.0..upstream/master
```

### Step 3: Create an upgrade branch and merge

Always use a dedicated upgrade branch — never merge upstream directly into `master`.
This keeps `master` clean if the merge goes wrong.

```bash
git checkout master
git checkout -b upgrade/upstream-v<NEXT_UPSTREAM_VERSION>
git merge upstream/master
# Resolve conflicts. Common conflict areas:
#   - client/request.go (SetPathParamEscaped)
#   - client_request.go (ClientRequest interface)
#   - middleware/denco/ (forked router — prefer our version)
```

### Step 4: Resolve Denco conflicts

When denco files conflict, **always prefer this fork's version**. The upstream project normally
does not touch `middleware/denco/` because it is a vendored copy — conflicts here typically mean
upstream adopted a denco change we already have, or restructured the middleware layer.

Strategy:
```bash
# During merge, for conflicts in middleware/denco/:
git checkout --ours middleware/denco/
git add middleware/denco/

# For conflicts in middleware/router.go that touch denco usage:
# manually compare — upstream may have added new routing features
# that need to work with our forked denco
```

### Step 5: Run tests

```bash
go test ./...
```

### Step 6: Merge to master and tag

```bash
git checkout master
git merge upgrade/upstream-v<NEXT_UPSTREAM_VERSION>

# Determine the new version. If upstream moved from v0.25.0 → v0.26.0:
#   our version becomes v0.27.0 (upstream + 1)
# If upstream only had patch changes:
#   keep our version ahead, e.g. v0.26.1

git tag -a v<VERSION> -m "v<VERSION>: merge upstream <UPSTREAM_TAG>"
git branch -d upgrade/upstream-v<NEXT_UPSTREAM_VERSION>
```

### Step 7: Push

```bash
git push origin master --tags
git push alswl master --tags
```

## Branch Management

### Branch Strategy

```
upstream/master ──────────────────────────────────────────►
                  \               \                \
                   \               \                \
master (fork) ─────●────────────────●────────────────●────►
                    \              / \              /
                     \            /   \            /
feat/* ───────────────●──────────/─────●──────────/───►
                               /                  /
upgrade/upstream-* ───────────●──────────────────/───►
```

### Branches

| Branch | Purpose | Lifetime |
|--------|---------|----------|
| `master` | Tracks upstream `master` + our patches, always ready to tag | Permanent |
| `feat/*` | New features developed locally, merged to `master` when done | Delete after merge |
| `upgrade/upstream-*` | Merge upstream releases, resolve conflicts, then merge to `master` | Delete after merge |

### Rules

1. **Never commit directly to `master`**. All changes go through feature or upgrade branches.
2. **One feature per branch**. `feat/add-path-param-escaped-support`, not `feat/misc-changes`.
3. **Feature branches branch off and merge back to `master`**. Keep the history linear.
4. **Upgrade branches handle upstream merges in isolation** — if the merge goes badly, just delete the branch and try again. `master` stays clean.
5. **Delete merged branches**. Avoid branch clutter. A merged `feat/*` branch has no remaining value — the commits are in `master`.

### Example: Adding a new feature

```bash
git checkout master
git checkout -b feat/my-new-feature
# ... develop, commit, test ...
git checkout master
git merge feat/my-new-feature
git branch -d feat/my-new-feature
git push origin master --tags
```

### Example: Merging a new upstream release

```bash
git fetch upstream --tags
git checkout master
git checkout -b upgrade/upstream-v0.26.0
git merge upstream/master
# ... resolve conflicts, test ...
git checkout master
git merge upgrade/upstream-v0.26.0
git tag -a v0.27.0 -m "..."
git branch -d upgrade/upstream-v0.26.0
git push origin master --tags
```

### Current State

```
master:  da56347 → 2cb321f → 69a4564
         (upstream  (feat:     (docs:
          v0.25.0)   pathParam  FORK.md)
                     escaped)

feat/add-path-param-escaped-support: (merged to master, can be deleted)
```

## Upstream PR Status

| Feature | Upstream PR | Status |
|---------|-------------|--------|
| `SetPathParamEscaped` | TBD | Landing in this fork first |

---

Last updated: 2026-07-05. Based on upstream `v0.25.0`.
