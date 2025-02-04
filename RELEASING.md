# Releasing

<div class=pagetoc>
<!-- toc -->
</div>

Tips for hledger release managers and maintainers.

## Glossary

|                       |                                                                                                                                                            |
|-----------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------|
| *Full release*        | A release of all the core packages (hledger-lib, hledger, hledger-ui, hledger-web)                                                                         |
| *Partial release*     | A release of only some of the core packages                                                                                                                |
| *Mixed release*       | A release where some of the core packages have different versions (due to previous partial release)                                                        |
| *Test release*        | A release that is published on github, but not hackage/stackage. These test the release process (and secondly, generate fresh binaries for early adopters) |
|                       |                                                                                                                                                            |
| *hledger repo*        | The `hledger` git repository, containing the core hledger tools and docs. Official public copy: github.com/simonmichael/hledger                            |
| *"master"*            | The `master` branch in the hledger repo; the main line of hledger development                                                                              |
| *release&nbsp;branch* | Branches named `MAJORVERSION-branch` in the hledger repo, eg `1.24-branch`. Releases are made from these.                                                  |
|                       |                                                                                                                                                            |
| *site repo*           | The `hledger_website` git repository, containing the hledger.org website and additional docs. Usually checked out under the hledger repo as `site/`.       |
| *"site"*              | The `master` branch in the site repo, used to generate <https://hledger.org>.                                                                              |
|                       |                                                                                                                                                            |
| *version*             | the 2 or 3-part dotted version number that names a hledger release: MA.JOR or MA.JOR.MINOR.                                                                |
| *MAJORVER*            | Just the MA.JOR part, eg 1.24                                                                                                                              |
| *OLD*, *NEW*          | Previous and new pending release versions. Examples: 1.24 -> 1.24.1, 1.24.1 -> 1.24.2, 1.24.2 -> 1.25.                                                     |
|                       |                                                                                                                                                            |

## 2021-12 

Starting over, again:

- All the stuff below the horizontal rule needs review and cleanup already; consider it old.
- Don't try to write down, let alone automate, every step of releasing; it's too much and too unstable.
- Practice releasing as often as possible.
- Keep making things a little better each time through. Simpler, more reliable, easier, faster, cheaper, higher quality.
- The different aspects of releasing have complex interdependencies and sequencing constraints.
  Chunk and separate them as far as possible:
  - **Software** - selecting changes, packages, release dates; coordinating contributions; ensuring release readiness
  - **Branch Management** - coordinating main and release branch, local and remote repos, CI branches
  - **Version Bumping** - choosing and applying new version numbers and related things like tags, github releases, urls, ghc and dep versions, stackage resolvers, everywhere needed
  - **Docs** - command help, manuals, changelogs, release notes, github release notes, install page, install scripts, announcements, process docs
  - **Testing** - local testing, CI testing, extra release-specific testing
  - **Artifacts** - generating binaries, zip files, github releases etc.
  - **Publishing** - uploading, pushing, making visible, finalising
  - **Announcing** - various announcement stages and channels
- All releases must now be made from a release branch, for uniformity and to avoid mishaps like uploading unreleased code to hackage.

## Some next goals

- Update/consolidate release process docs.
- Develop a process for making test releases at any time.
- Establish routine weekly test releases.

----

## Review/update/consolidate:

## Phases of release cycle:


### 0. Dev
  
Normal development, on master and PR branches.


### 1. Pre-release
  
Preparations to make just before a release.

#### Resolve issues

Review, select, resolve PRs and issues.

#### Polish changelogs

Complete and polish changelogs.

#### Plan release

Plan the release number and any extra release-time activities.


### 2. Release

The sequence of steps to follow when making a release.

#### Freeze

- Set version.
- Finalise changelogs.
- Generate release notes.
- Prepare announcement.
- Tag.
- Generate CI release binaries.
- Draft github release.
- 24 hour release countdown with no changes.
- If any problems found, return to Pre-release.

#### Publish

- Website changes.
  - release notes
  - install page
  - manuals
  - webserver redirects
- Publish hackage packages.
- Push tags.
- Publish github release.
- Publish website changes.
- Announce

### 3. Post-release

Monitor, support, respond.


## Release preparation detail

### Any time before release

1. create release branch when needed:\
  `git branch MAJORVER-branch BRANCHPOINT`\
   Sometimes when convenient we make major releases on master (adapt the steps below as needed).
   And make the release branch later, when a minor release becomes necessary, eg:\
  `git branch 1.22-branch 1.22`

1. update changelogs in master

1. review changes so far, estimate which packages will be released

1. cherry pick changes to release
    - cherry pick release-worthy commits 
        - from: magit, `l o MAJORVER-branch..master`, `M-x magit-toggle-buffer-lock`, `M-x toggle-window-dedicated` (`C-c D`)
        - to:   magit, `l o master..MAJORVER-branch`, `M-x magit-toggle-buffer-lock`, `M-x toggle-window-dedicated`
        - ignore commits already seen in previous cherry picking sessions
        - ignore changelog commits / other boring commits 
          ("dev: doc: update changelogs")

    - update changelogs in master (move corresponding change items under pending release heading)

### On release day

In master:

- `./Shake.hs` to ensure `Shake` is current, review commands

- finalise [changelogs](CHANGELOGS.html), copy each new section (emacs `C-x r s a` ... `b` ... `c` ... `d`)

In release branch:

- paste new sections into changelogs

- `./Shake setversion NEWVER [-c]` (first without `-c` to review, then with `-c` to commit).
  <!-- Also `touch hledger/Hledger/Cli/Version.hs` ? -->

- `./Shake cmdhelp [-c]`

- `./Shake mandates`

- `./Shake manuals -B [-c]`  (using up to date doctool versions)

- `make tag`  (signed annotated tags for each package and the overall release)

- `stack clean && stack install && hledger --version && hledger-ui --version && hledger-web --version`
  to build locally and check version strings

- push to CI branches to test and to build release binaries
  - `git push -f origin master:ci-windows`  (or magit `P -f e origin/ci-windows`)
  - `git push -f origin master:ci-mac`
  - `git push -f origin master:ci-linux`
  - `git push -f origin master:ci-linux-static` (at release time only))
  - Tips:
    - build these release binaries at the very last possible moment
    - last commit should be a notable one - not docs only, not beginning with ;

In site repo:

- update `release-notes.md`
  - copy template, uncomment
  - replace date
  - replace XX with NEW
  - add new changelog sections, excluding hledger-lib
  - remove any empty sections
  - add contributors, `git shortlog -sn OLD..NEW`
  - add summary (major release) or remove it (minor release)
  - check preview in vs code
  - commit: `relnotes: NEW`

- update `install.md`
  - query-replace OLD -> NEW in 
    - "current hledger release"
    - CI binaries badges/links, including linux-static-arm32v7 if built
    - "building from source"
    - stack install command
    - cabal install command
  - query-replace OLD-brightgreen -> OLD-red
  - only after release binaries are built (preferably after release is published):
    update --version outputs (search: hledger --version)
  - final output line from `hledger test` (run in terminal for normal speed)
  - Total count from `make functest`
  - commit: `download: NEW`

In release branch:

- update `doc/ANNOUNCE` (major release)
  - summary, contributors from release notes
  - any other edits
  - commit: `;doc: ANNOUNCE`

In master:

- cherry pick useful release branch changes
  - `;doc: ANNOUNCE`

- update `hledger-install/hledger-install.sh`
  - HLEDGER_INSTALL_VERSION
  - RESOLVER
  - HLEDGER_*_VERSION
  - EXTRA_DEPS

- wait for CI binaries, https://ci.hledger.org

- pre-release pause: take a break away from keyboard

## Release detail

In release branch:

- review status, reflect

- `make hackageupload`

- push tags: magit `P t origin`

- create github release
  - https://github.com/simonmichael/hledger/releases, copy last release's body
  - create new release from NEW tag if CI workflow has failed
  - tag NEW, title NEW, body similar to previous release
  - at https://ci.hledger.org download CI binary artifacts, check sizes look similar to previous
  - select downloaded artifacts in Finder, drag into github release
  - publish release

- announce
  - push site download/relnotes updates
  - push master hledger-install update
  - share release notes link, then markdown content, in #hledger chat
  - send ANNOUNCE to hledger@googlegroups.com, haskell-cafe@googlegroups.com (major release)
    or brief announcement to hledger@googlegroups.com (minor release)
    - ANN: hledger NEW
    - release notes link, summary
    - release notes html (copied from browser)
  - tweet at https://twitter.com/simonkwmichael ?
  - toot at https://fosstodon.org/web/accounts/106304084994827771 ?

## Post release detail

- merge/check/update download page changes
  - docker - expect/merge PR
  - homebrew - expect badge to update soon
  - nix - expect `make nix-hledger-version` to update after a few days, find and update to that commit hash
  - linux distros - once in a while, follow the links & search for newer versions, update

- support

- handle issues

- update procedures, tools, docs

## Add major release to website

In site: 

- js/site.js: add NEW, update currentrelease
- Makefile: add NEW, two places
- make snapshot-NEW
- (cd src; rm current; ln -s NEW current)

In hledger.org caddy config:

- add `path` and `redir`s for NEW
- `systemctl reload caddy`

On hledger.org:

- make clean all

## Tips

- During pre/post release phases, update RELEASING.md in a copy,
  RELEASING2.md, to reduce commit noise and git interference.

