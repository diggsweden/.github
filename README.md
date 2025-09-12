<!--
SPDX-FileCopyrightText: 2023 Digg - Agency for Digital Government

SPDX-License-Identifier: CC0-1.0
-->

# DiggSweden base .github-project

This is the standard Default .github repo for DiggSweden. It contains Pull Request-, Issue-templates and Community Health-files that will be applied for all projects in the whole GitHub-organisation "should the projects not override them with something more adjusted".

It also contains the front presentation text.

Contents:

## Docs:

- docs/how_to_work_github.adoc (A document describing how to work on Digg's GitHub)


- SECURITY.md (A general template for where to report Security Issues)
- CODE_OF_CONDUCT.md (Contributor Covenant)
- Templates for the Pull Requests, Bug and Feature Requests
- profile/README.md (GitHub-organisation fronttext)


Note that these files are mostly taken from the general [Open Source Project Template-project](https://github.com/diggsweden/open-source-project-template) and any Issues should most likely go there directly.

## Reusable Workflows 

Some GitHub Workflows can be reused in other projects, see the '.github/workflows-folder.

- CommitLint
- DependencyAnalysis
- LicenseLint
- MegaLint - Lint Containers, scripts, code (to be deprecated use misc lint instead)
- Misc Lint - Mise and Just lint setup 
- Public Code Lint - Verify that the project has a valid public code yaml
- OpenSSF-Scorecard - Projecthealth from a security perspective 
- Versionbump and Changelog - generate a Changelog, in a commit Release, bump project version


## GitCliff templates

Git-Cliff is a changelog solution, with a few builtin templates. 
These templates is the builtin ones but modified to remove 'chore(release):-lines from the output.

- keepachangelog
- minimal
- default

----

## License

Most, but not all of the files are licensed under the Creative Commons Zero v1.0 Universal License - see the file headers for details.

CODE_OF_CONDUCT.md is Copyright: [Contributor Covenant](https://www.contributor-covenant.org/)
 License: [CC-BY-4.0](https://creativecommons.org/licenses/by/4.0/)

Finally, the Digg Logo is *not* under any free usage License.

----

## Maintainers

Digg Open Source Guild
