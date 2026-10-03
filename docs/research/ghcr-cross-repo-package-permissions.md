# GHCR cross-repo package push permissions

Research for taskmedia/container-images issue #2 (repo-merge effort).
Question: after merging `kubectl-yq`, `kubectl-gpg-ncftp`, `curl-jq`, `scratch-success`
into `taskmedia/container-images`, can the new repo's workflow `GITHUB_TOKEN` push new
versions to the **pre-existing** `ghcr.io/taskmedia/<name>` packages that were created by
the old repos?

Sources: GitHub Docs only (rendered pages plus the `github/docs` markdown sources, which
are the canonical text behind those pages). Researched 2026-10-03.

---

## Bottom line

**No — not by default. It will fail with a permission error until someone explicitly grants
`container-images` write access to each existing package.** Each of those four GHCR packages
is org-scoped (granular permissions) and is currently linked to, and inheriting permissions
from, its original repository. The `container-images` workflow's `GITHUB_TOKEN` is scoped to
`container-images` only, and GitHub's docs state flatly that uploading a new version to an
*existing* package is allowed only for "workflows running in repositories that are given
write permission on that package" — regardless of whether the package is public, internal,
or private. The fix is a one-time, per-package manual step: add `container-images` under
**Manage Actions access** on each package's settings page with the **Write** role. There is
no REST/GraphQL API for this, so it cannot be scripted.

---

## 1. Default scoping

### GHCR packages are org-scoped, not repo-scoped

- The Container registry is listed under "Granular permissions for user/organization-scoped
  packages": "Packages with granular permissions are scoped to a personal account or
  organization. You can change the access control and visibility of the package separately
  from a repository that is connected (or linked) to a package."
  — <https://docs.github.com/en/packages/learn-github-packages/about-permissions-for-github-packages>
- So `ghcr.io/taskmedia/kubectl-yq` belongs to the **taskmedia org namespace**, not to the
  `kubectl-yq` repo. Good news: the package coordinates survive the merge and are not
  tied to the old repo's existence.

### But access is still inherited from the linked repo by default

- "By default, if you publish a package that is linked to a repository, the package
  automatically inherits the access permissions (but not the visibility) of the linked
  repository. [...] When a package automatically inherits access permissions,
  GitHub Actions workflows in **the linked repository** also automatically get access to
  the package."
  — <https://docs.github.com/en/packages/learn-github-packages/configuring-a-packages-access-control-and-visibility>
- Publishing from a workflow is exactly the case that creates that link: "The easiest way to
  connect a repository to a container package is to publish the package from a workflow
  using `${{secrets.GITHUB_TOKEN}}`, as the repository that contains the workflow is linked
  automatically."
  — <https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry>
- And: "The package inherits the visibility and permissions model of the repository where
  the workflow is run. Repository admins where the workflow is run become the admins of the
  package once the package is created."
  — <https://docs.github.com/en/packages/managing-github-packages-using-github-actions-workflows/publishing-and-installing-a-package-with-github-actions>

Net effect: the four packages are each linked to their old repo and inherit write access
from it. `container-images` is not in that picture.

### What `packages: write` on `GITHUB_TOKEN` actually grants

- `packages: read|write|none` is a valid `permissions` scope.
  — <https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#permissions>
- The token itself is repo-bound: "The `GITHUB_TOKEN` secret is a GitHub App installation
  access token. [...] **The token's permissions are limited to the repository that contains
  your workflow.**"
  — <https://docs.github.com/en/actions/concepts/security/github_token>
- Confirmed again on the packages side: "To publish, install, delete, and restore packages
  **associated with the workflow repository**, use `GITHUB_TOKEN`."
  — <https://docs.github.com/en/packages/learn-github-packages/about-permissions-for-github-packages>

So `packages: write` is not "write to all org packages" — it is "write to packages this
repository has been granted write on". Declaring `packages: write` in the new workflow is
necessary but not sufficient.

### The decisive quote

The docs table for Actions workflow tasks says, for "Upload a new version to an existing
package":

> If the package is private, internal, or public, **only workflows running in repositories
> that are given write permission on that package** can upload new versions to the package.

— <https://docs.github.com/en/packages/managing-github-packages-using-github-actions-workflows/publishing-and-installing-a-package-with-github-actions>

And the container-registry page names our exact failure mode:

> Note that the `GITHUB_TOKEN` **will not have permission to push the package if you have
> previously pushed a package to the same namespace, but have not connected the package to
> the repository.**

— <https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry>

That is precisely the merge scenario: the namespace `taskmedia/kubectl-yq` already has a
package, pushed from a different repo, and `container-images` is not connected to it.

Note the asymmetry worth being aware of: **pulling** is not a problem. "If the package is
public, any workflow running in any repository can download the package" (same table). All
four of these are presumably public. Only the push side is gated.

---

## 2. Granting cross-repo access

### This is explicitly supported

- "For packages scoped to a personal account or an organization, to ensure that a GitHub
  Actions workflow has access to your package, you must give explicit access to the
  repository where the workflow is stored. **The specified repository does not need to be
  the repository where the source code for the package is kept. You can give multiple
  repositories workflow access to a package.**"
  — <https://docs.github.com/en/packages/learn-github-packages/configuring-a-packages-access-control-and-visibility#ensuring-workflow-access-to-your-package>

That sentence is the whole answer to "is this a supported pattern?" — yes, and it's the
documented way to run a monorepo of images publishing to pre-existing packages.

### UI flow (current, per docs)

For an org-scoped package:

1. On GitHub, navigate to the main page of the **taskmedia** organization.
2. Under the org name, click the **Packages** tab.
3. Search for and click the name of the package (e.g. `kubectl-yq`).
4. On the package's landing page, on the right-hand side, click **Package settings** (gear icon).
5. Under **"Manage Actions access"**, click **Add repository** and search for the repository
   you want to add (`container-images`).
6. Use the **Role** drop-down menu to select the default access level that repository should
   have to the package — choose **Write** (see role table below).

— steps assembled from
<https://docs.github.com/en/packages/learn-github-packages/configuring-a-packages-access-control-and-visibility#github-actions-access-for-packages-scoped-to-organizations>
and the underlying doc source reusables (`package-settings-from-org-level`,
`package-settings-option`, `package-settings-add-repo`,
`package-settings-actions-access-role-repo`) in
<https://github.com/github/docs/tree/main/data/reusables/package_registry>.

Role semantics:

| Role | Grants |
|---|---|
| Read | Can download package. Can read package metadata. |
| Write | Can upload and download this package. Can read and write package metadata. |
| Admin | Can upload, download, delete, and manage this package. Can grant package permissions. |

— <https://docs.github.com/en/packages/learn-github-packages/about-permissions-for-github-packages#visibility-and-access-permissions-for-packages>

**Write** is the minimum for pushing new versions. **Admin** would additionally be needed if
the workflow should ever delete/prune old versions via the REST API: "You can use a
`GITHUB_TOKEN` in a GitHub Actions workflow to delete or restore a package using the REST
API, if the token has `admin` permission to the package."
(<https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry>)
If a retention/cleanup job is in scope for this merge, grant Admin instead of Write.

### REST API: there is none for this

I enumerated the full Packages REST API surface
(<https://docs.github.com/en/rest/packages/packages>). It covers only:

- `GET /orgs/{org}/packages`, `GET`/`DELETE /orgs/{org}/packages/{package_type}/{package_name}`,
  `POST .../restore`
- the same for `/user/packages/...` and `/users/{username}/packages/...`
- package *versions*: list / get / delete / restore
- `GET /orgs/{org}/docker/conflicts` (Docker-registry migration helper)

**There is no documented endpoint for managing a package's repository access, its "Manage
Actions access" list, or for connecting/linking a package to a repository.** So step 5 above
is UI-only — it cannot be put in a bootstrap script or Terraform-style automation. Treat it
as a manual one-time prerequisite in issue #3.

### Linking vs. Actions access — two different things

These are easy to conflate; the docs separate them deliberately:

> Syncing your package with a repository by using the **Add Repository** button under
> "Manage Actions access" in the package's settings **is different than connecting your
> package to a repository.**

— <https://docs.github.com/en/packages/learn-github-packages/configuring-a-packages-access-control-and-visibility#ensuring-workflow-access-to-your-package>

- **Actions access** (what we need): a *list* of repositories whose workflows may read/write
  the package. Multiple repos allowed. This is the grant that unblocks the push.
- **Connected / linked repository** (cosmetic + inheritance): exactly **one** repo, drives
  the package landing page's README/source display and the permission-inheritance behavior.
  Changing it is "unlink from the current repository, then link to a new one."
  (<https://docs.github.com/en/packages/learn-github-packages/connecting-a-repository-to-a-package>)

Optionally also re-point the **linked** repo to `container-images` so the package pages show
the right source. Two ways:

- UI: package → **Connect repository** / unlink-then-relink, per
  <https://docs.github.com/en/packages/learn-github-packages/connecting-a-repository-to-a-package>
- Dockerfile label (recommended by the docs for containers):
  `LABEL org.opencontainers.image.source=https://github.com/taskmedia/container-images`
  — "To connect a repository when publishing an image from the command line, and to ensure
  your `GITHUB_TOKEN` has appropriate permissions when using a GitHub Actions workflow, we
  recommend adding the label `org.opencontainers.image.source` to your `Dockerfile`."
  (<https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry>)

Caveat on the label: the docs present it as the mechanism that links a package *at publish
time* and note "A package only inherits the access permissions of a linked repository
automatically **if you link the repository to the package before you publish the package**,
such as by adding the `org.opencontainers.image.source` Docker label." For packages that
*already exist* with a different link, the label alone is not documented to re-point the link
or to confer push permission — see Ambiguities.

---

## 3. Same-org factor

Being in one org (taskmedia) helps with *who can perform the grant*, but does **not** make
the push work automatically.

- Org owners already have the needed admin rights on every package: "If you publish a
  package to an organization, anyone with the `owner` role in the organization also gets
  admin permissions to the package."
  — <https://docs.github.com/en/packages/learn-github-packages/about-permissions-for-github-packages>
  So no cross-account coordination is needed; a taskmedia owner can do all four grants.
- Same-org does **not** grant push. The upload rule is visibility-independent: "If the
  package is private, internal, or public, only workflows running in repositories that are
  given write permission on that package can upload new versions."
  (publishing-and-installing doc, cited above). `internal` visibility widens *download*
  across the enterprise, not upload.
- Org-level setting that can make things *worse*: Settings → **Packages** → "Default Package
  Settings" → **Inherit access from source repository** can be deselected by an org owner,
  which disables automatic inheritance for **new** packages org-wide. "If you disable
  automatic inheritance of access permissions, new packages scoped to your organization will
  not automatically inherit the permissions of a linked repository."
  — <https://docs.github.com/en/packages/learn-github-packages/configuring-a-packages-access-control-and-visibility#disabling-automatic-inheritance-of-access-permissions-in-an-organization>
  Worth checking this setting's current state in taskmedia; if it's off, nothing in the merge
  will work "by accident" and the explicit Actions-access grant is unconditionally required.
  (It affects new packages only, so it does not change the status of the four existing ones.)
- Also org-level: Settings → **Packages** → "Package Creation" controls whether members may
  create public/private/internal packages. Not a blocker for pushing to existing packages,
  but relevant if the merged workflow ever adds a *new* image name.
- **PAT vs `GITHUB_TOKEN`**: GitHub explicitly prefers `GITHUB_TOKEN` here — "All workflows
  accessing registries that support granular permissions should use the `GITHUB_TOKEN`
  instead of a personal access token."
  (<https://docs.github.com/en/packages/managing-github-packages-using-github-actions-workflows/publishing-and-installing-a-package-with-github-actions>)
  A classic PAT with `write:packages` held by a user who has write/admin on the packages
  *would* work as a fallback, since PAT permissions follow the user not the repo — note
  "packages" PAT scopes are **classic-PAT-only**
  (<https://docs.github.com/en/packages/learn-github-packages/about-permissions-for-github-packages#about-scopes-and-permissions-for-package-registries>).
  Treat that as an escape hatch, not the plan: it reintroduces a long-lived secret the
  merge is a good opportunity to be rid of.

### Out of scope (noted in passing)

`scratch-success` also publishes to Docker Hub (`fty4/<name>`). That path authenticates with
a Docker Hub access token stored as a repo/org secret and is entirely outside GHCR's
permission model — nothing in this document applies to it. The only merge-related action
there is making sure the Docker Hub credential secrets are available to
`taskmedia/container-images` (repo secret, or an org secret whose repository-access list
includes the new repo).

---

## 4. What to do — concrete checklist

Prerequisite for the new workflow's first push. Feeds issue #3.

**Per package** — repeat for all four: `kubectl-yq`, `kubectl-gpg-ncftp`, `curl-jq`,
`scratch-success`. Must be done by a taskmedia org owner or a package admin.

1. `https://github.com/orgs/taskmedia/packages` → click the package.
2. **Package settings** (gear, right-hand side of the package landing page).
3. Under **Manage Actions access** → **Add repository** → `container-images`.
4. Set its **Role** to **Write** (or **Admin** if the workflow will also prune old versions
   via the REST API).
5. Leave the existing old-repo entries alone for now; remove them only after the new
   workflow has pushed successfully, as a rollback cushion.

**Optional but recommended, same pass:**

6. Add `LABEL org.opencontainers.image.source=https://github.com/taskmedia/container-images`
   to each Dockerfile, so package pages point at the merged repo and future links are
   correct.
7. Re-point each package's **connected repository** to `container-images`
   (unlink → **Connect repository**) for correct README/source display on the package page.
   Independent of step 3 — step 3 is what actually grants the push.

**In the new workflow:**

8. Declare the scopes explicitly per job:
   ```yaml
   permissions:
     contents: read
     packages: write
   ```
9. Log in with `${{ secrets.GITHUB_TOKEN }}` (not a PAT) — e.g. `docker/login-action` against
   `ghcr.io` with `username: ${{ github.actor }}`, `password: ${{ secrets.GITHUB_TOKEN }}`.
10. Make Docker Hub credentials for `scratch-success` reachable from `container-images`
    (separate auth path, see above).

**Verification:**

11. Before migrating everything, prove the grant works: push a throwaway tag (e.g.
    `ghcr.io/taskmedia/curl-jq:merge-test`) from a branch workflow in `container-images`.
    A `denied: installation not allowed to Write organization package` / 403 at push time
    means step 3 was not applied to that package. Delete the test tag afterwards.

**Org-level check (once, not per package):**

12. taskmedia Settings → **Packages** → confirm the state of "Default Package Settings →
    Inherit access from source repository" and the "Package Creation" visibility options, so
    there are no surprises when a *new* image name is added later.

---

## 5. Ambiguities / not confirmed from primary sources

Flagged rather than guessed:

- **Whether "Manage Actions access" is editable while the package still inherits permissions
  from the old repo.** The docs say granular *people/team* permissions require removing
  inherited permissions first ("To access the package's granular permissions settings, you
  must remove the package's inherited permissions"), but they never state whether that also
  gates the separate **Manage Actions access** section. The docs treat the two as distinct
  concepts, which suggests Actions access is independently editable — but this is not
  confirmed. If **Add repository** is unavailable in the UI, the likely workaround is to
  disable inheritance on that package first (package settings → inheritance toggle), which
  carries the documented warning: "If you change how a package gets its access permissions,
  any existing permissions for the package are overwritten." Decide deliberately before
  flipping it.
- **Whether the `org.opencontainers.image.source` label re-points an already-linked
  package.** The docs describe the label as establishing the link at publish time and are
  silent on what happens when the package already exists with a *different* linked repo.
  Do not rely on the label alone to grant push permission — use step 3.
- **What happens to these packages if the old repos are deleted (rather than archived).**
  The docs cover *account* deletion ("GitHub deletes the packages and container images when
  you delete your account") and *repository transfer* ("If you have linked a package to a
  repository, the link is removed when you transfer the repository") but I found no primary
  statement about plain repository deletion for a granular-permission registry. Since the
  package is org-scoped it should survive, but this is unverified — **archive the old repos
  rather than delete them**, at least until the new workflow has pushed successfully to all
  four packages.
- **The exact error string on a denied push.** The error text in step 11 is a plausible
  reconstruction, not a quote from GitHub docs. Match on the HTTP 403 / `denied:` prefix, not
  on exact wording.
- **Whether an org-level policy can blanket-grant a repo write access to all org packages.**
  No such setting is documented; the per-package grant appears to be the only mechanism.
  Doing it four times is the expected cost.

---

## Source index

- About permissions for GitHub Packages — <https://docs.github.com/en/packages/learn-github-packages/about-permissions-for-github-packages>
- Configuring a package's access control and visibility — <https://docs.github.com/en/packages/learn-github-packages/configuring-a-packages-access-control-and-visibility>
- Connecting a repository to a package — <https://docs.github.com/en/packages/learn-github-packages/connecting-a-repository-to-a-package>
- Working with the Container registry — <https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry>
- Publishing and installing a package with GitHub Actions — <https://docs.github.com/en/packages/managing-github-packages-using-github-actions-workflows/publishing-and-installing-a-package-with-github-actions>
- About the `GITHUB_TOKEN` — <https://docs.github.com/en/actions/concepts/security/github_token>
- Workflow syntax — `permissions` — <https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#permissions>
- Packages REST API — <https://docs.github.com/en/rest/packages/packages>
- Doc source reusables (verbatim UI strings) — <https://github.com/github/docs/tree/main/data/reusables/package_registry>
