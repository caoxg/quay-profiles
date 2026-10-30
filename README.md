# quay-profiles

Profiles for [Quay](https://github.com/caoxg/quay), the Arduino® UNO Q app manager, published separately from Quay so that they can change without a Quay release. [中文说明](README.zh-CN.md)

- **App profiles** (`apps/`) tell Quay what an app needs — one-time system setup, steps before start, health and safety checks, policy slots, shortcuts — for apps that do not ship a [`quay.yaml`](https://github.com/caoxg/quay/blob/main/docs/quay-yaml.md) of their own.
- **System profiles** (`systems/`) recognise board systems other than the stock Arduino image (Duck OS, …), and say what they do differently that matters to Quay.

Quay carries a copy of this repository (taken when it was built) and fetches a newer one about twice a day, from GitHub or through jsDelivr (`cdn.jsdelivr.net/gh/caoxg/quay-profiles@main`). It uses whichever copy has the later `updated` date. `quay profiles` shows the copy in use, `quay profiles --refresh` fetches now.

> Quay is an independent project, not affiliated with or endorsed by Arduino or Luwu Dynamics. The profiles here were written for Quay; the vendors do not maintain them.

## Layout

```text
index.yaml                schema, updated, and the lists of profile files
apps/xgoduck/<commit>.yaml XGO-Duck (Luwu Dynamics), one version per runtime commit it starts from
systems/duck-os.yaml      Duck OS (Luwu Dynamics' image for XGO-Duck)
```

`index.yaml`:

```yaml
schema: 2              # the layout version; a Quay that reads an older schema keeps its own copy
updated: "2026-10-31"  # bump on every change: Quay takes the copy with the later date
apps:                  # the versions of one profile oldest first
  - apps/xgoduck/5f1954a.yaml
  - apps/xgoduck/5f511b9.yaml
systems: [systems/duck-os.yaml]
```

**Bump `updated` with every change**, or Quays that already have a copy of the same date will not take it.

## App profiles

The same format as an app's own `quay.yaml`, plus `id`, `source` (offered under **Custom** in the app) and `match` (which repositories or `app.yaml` names it is for). See [docs/quay-yaml.md](https://github.com/caoxg/quay/blob/main/docs/quay-yaml.md). An app's own `quay.yaml` always wins.

### Versions

An app changes: XGO-Duck's runtime had 3 policy models at first and has 8 now. So a profile comes in versions, one file per commit of the app's repository it starts from, named after it (`apps/xgoduck/5f511b9.yaml`) and saying so in `from` (the full hash). A version is for that commit and every later one, until the next version; a commit older than every version gets no profile. All versions share `id`, and `index.yaml` lists them oldest first.

When the app changes something a profile describes, add a version from the commit that changed it, rather than editing the old one: apps already installed at older commits keep the version that fits them. Quay picks the version from the app's git history when it installs or updates the app, and records it on the board (`.quay.json`, `profileFrom`).

Only apps installed through Quay get a profile. An app installed some other way is a plain app to Quay: the profile it matches only names where **Reinstall with Quay** comes from.

## System profiles

```yaml
id: duck-os
name: Duck OS
vendor: Luwu Dynamics
homepage: https://github.com/LuwuDynamics
description: …
description_zh: …
detect:                      # the system is this one when every file exists
  files: [/etc/duck-release.json]
version:                     # shown on the System card: a top-level key of a JSON file
  file: /etc/duck-release.json
  json: release
base:                        # what it is based on (shown as "based on Arduino image …")
  file: /etc/duck-release.json
  json: base_version
hotspot:                     # the system's own hotspot, set up by its own tools: Quay shows whether
  nm_profile:                #   it starts by itself ("Away hotspot: controlled by Duck OS, …"), since it
    id_prefix: Duck-Setup-hotspot-   # then decides how the board is reached away from Wi-Fi
    mode: ap
notes:                       # what it does differently that the user should know
  - id: duck-setup-hotspot
    show_in: network         # network: the Wi-Fi and hotspot dialog; overview: the board's page
    when:                    # all the parts given hold (none: always)
      files: […]             #   these files exist
      nm_profile:            #   a saved NetworkManager profile matches every field given
        id_prefix: Duck-Setup-hotspot-   # or id
        mode: ap             #   infrastructure | ap
        autoconnect: true
        # ssid: …
    text: Duck Setup's hotspot "{ssid}" comes up at every boot …   # {ssid} {id} {count}: from the match
    text_zh: …
    action:                  # optional: one of Quay's built-in fixes, applied to what nm_profile matched
      do: nm_autoconnect_off
      label: Don't open it at boot
      label_zh: …
      confirm: …             # shown before Quay goes ahead
      confirm_zh: …
```

Built-in actions (`do`):

| `do` | What Quay does to the matched NetworkManager profiles |
|---|---|
| `nm_autoconnect_off` | keeps them, but they no longer come up by themselves |
| `nm_delete` | deletes them, except the one in use |

### What a profile cannot do

A system profile only describes. Quay reads the files it names (and from JSON files only the keys it names, nothing else), matches saved network profiles, and offers the built-in action — after the user clicks it and confirms. A profile cannot make Quay run a command of its own, so a mistake here, or a tampered copy, can at worst show a wrong note. If a system changes in a way the built-in actions cannot handle, Quay needs a new action, in a Quay release.

## Trying a change

Point Quay at your checkout:

```bash
QUAY_PROFILES=~/src/quay-profiles quay profiles
QUAY_PROFILES=~/src/quay-profiles quay system     # on a board running the system
```

In Quay's repository, `make profiles` takes `../quay-profiles` as the built-in copy (and runs the profile tests).

## License

Apache-2.0, like Quay. Copyright 2026 Xiaogang Cao.
