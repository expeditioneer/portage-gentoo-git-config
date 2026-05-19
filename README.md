> **⚠️ Archived — no longer maintained.**
> I no longer use Gentoo, so this repository is archived and will not receive
> further updates. The configuration is left here for reference and may become
> outdated as Portage evolves. Use at your own risk.

---

# portage-gentoo-git-config

Minimal Portage configuration that syncs the Gentoo ebuild repository via the
official [GitHub mirror][mirror] instead of rsync, plus [`repo.postsync.d`][postsync]
hooks that regenerate the parts the git mirror does not ship.

The git mirror contains only ebuilds and core metadata — no pre-generated
`md5-cache`, no DTDs, no GLSAs, no news items, no `projects.xml`. The hooks
fetch or regenerate them after every `emerge --sync`.

[mirror]: https://github.com/gentoo/gentoo
[postsync]: https://wiki.gentoo.org/wiki/Project:Portage/Sync

## What it does

* Configures Portage to sync the `gentoo` repository via the GitHub mirror.
* Fetches the pre-generated `md5-cache` from `rsync.gentoo.org` and runs
  [`egencache`][egencache] to keep it in sync with the local tree.
* Mirrors `metadata/dtd` from `anongit.gentoo.org` (via git, rsync fallback).
* Mirrors `metadata/glsa` — Gentoo Linux Security Advisories.
* Mirrors `metadata/news` — `emerge --sync` news items.
* Fetches `metadata/projects.xml` from `api.gentoo.org`.

[egencache]: https://wiki.gentoo.org/wiki/Egencache

## Requirements

* `sys-apps/portage` — provides the [`repo.postsync.d`][postsync] hook mechanism
* `sys-apps/gentoo-functions` — provides `/lib/gentoo/functions.sh`
  (`ebegin`, `eend`, `einfo` used by every hook)
* `dev-vcs/git` — sync itself plus dtd/glsa/news clones
* `net-misc/rsync` — md5-cache and dtd/glsa/news fallback
* `net-misc/wget` — `projects.xml`

## Installation

Copy the files into `/etc/portage`:

```bash
install -m 0644 repos.conf/gentoo.conf       /etc/portage/repos.conf/gentoo.conf
install -m 0755 repo.postsync.d/sync_gentoo_* /etc/portage/repo.postsync.d/
install -m 0644 repo.postsync.d/sync_overlay_cache /etc/portage/repo.postsync.d/
```

The next `emerge --sync` will use the git mirror and run all hooks.

## Optional: overlay cache regeneration

To also regenerate the metadata cache of any **other** (non-`gentoo`) repository
you have configured, make `sync_overlay_cache` executable:

```bash
chmod +x /etc/portage/repo.postsync.d/sync_overlay_cache
```

It runs `egencache` on every synced repo except `gentoo`, which has its own
optimized hook.
