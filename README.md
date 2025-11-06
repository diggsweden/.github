<!--
SPDX-FileCopyrightText: 2023 Digg - Agency for Digital Government

SPDX-License-Identifier: CC0-1.0
-->

# DiggSweden base .github-project

This is the standard Default .github repo for DiggSweden.
It contains Pull Request-, Issue-templates and Community Health-files that will be applied for all projects in the whole GitHub-organisation "should the projects not override them with something more adjusted".

It also contains the front presentation text, and a few important docs.

Contents:

## Docs:

- [How to work on GitHub](docs/how_to_work_github.adoc)
- [Checklist before a version 1.0.0 release](docs/release1.0.checklist.md)

## General Organisation files

- SECURITY.md (A general template for where to report Security Issues)
- CODE_OF_CONDUCT.md (Contributor Covenant)
- Templates for the Pull Requests, Bug and Feature Requests
- profile/README.md (GitHub-organisation fronttext)

Note that these files are mostly taken from the general [Open Source Project Template-project](https://github.com/diggsweden/open-source-project-template) and any Issues should most likely go there directly.

## General Renovate Configuration

This repository contains a shared Renovate base configuration that all DiggSweden projects extend.

### renovate-base.json

Centralized Renovate configuration providing:
- **Standardized settings** across all projects (7-day minimum release age, weekend schedule, Europe/Stockholm timezone)
- **Security presets** (OpenSSF Scorecard, vulnerability alerts)
- **Automerge policies** (minor/patch updates after tests pass)
- **GitHub Actions** digest pinning and grouping
- **Semantic commits** with git sign-off (build(deps) format)

Projects extend this base using:
```json
{
  "extends": ["local>diggsweden/.github:renovate-base"]
}
```

This ensures consistency while allowing project-specific overrides (managers, dependency grouping, etc.).

## Reusable Workflows 

Obsolete here. Have moved to reuseable-ci project.

## GitCliff templates

Obsolete here. Have moved to reuseable-ci project.

----

## License

Most, but not all of the files are licensed under the Creative Commons Zero v1.0 Universal License - see the file headers for details.

CODE_OF_CONDUCT.md is Copyright: [Contributor Covenant](https://www.contributor-covenant.org/)
 License: [CC-BY-4.0](https://creativecommons.org/licenses/by/4.0/)

Finally, the Digg Logo is *not* under any free usage License.

----

## Maintainers

Digg Open Source Guild
