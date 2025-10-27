---
Version: 0.9.0
Last Updated: 2025-10-27
Author: Open Source Guild
---

# Checklista för en första Release 1.0.0 (Svenska)

Detta dokument fungerar som en guide för att säkerställa att ditt projekt uppfyller alla krav för en första 1.0-release.
Använd denna checklista för att validera teknisk efterlevnad, säkerhetsefterlevnad och anpassning till bästa praxis för öppen källkod.

För mer information se [Digg Open Source Checklista](https://github.com/diggsweden/open-source-project-template/blob/main/docs/Open_Source_Checklist.md) samt även [Digg Sweden Open Source Project Template](https://github.com/diggsweden/open-source-project-template)

## ✅ Krav

### Dokumentation
- [ ] **README.md** med:
  - [ ] Badges (version, openssf, licens, reuse-status)
  - [ ] Projektbeskrivning
  - [ ] Installationsinstruktioner
  - [ ] Användningsexempel
  - [ ] Förvaltare (Maintainer)
- [ ] **LICENSE** fil
- [ ] **SECURITY.md** med process för sårbarhetsrapportering
- [ ] **CONTRIBUTING.md** med riktlinjer för bidrag
- [ ] **CODE_OF_CONDUCT.md**
- [ ] **docs/DEVELOPMENT.md** med utvecklingsinstruktioner

### Juridik & Efterlevnad
- [ ] REUSE-efterlevnad (SPDX-headers i alla källfiler)
- [ ] Licenskompatibilitet verifierad
- [ ] Namn-/varumärkeskontroll utförd
- [ ] Inga SNAPSHOT-beroenden

### Kodkvalitet
- [ ] Testtäckning ≥ 50%
- [ ] Publika API:er har dokumentation
- [ ] Koden har granskats

### Git Hosting och säkerhet - CI/CD
- [ ] Pull request-CI-arbetsflöde
- [ ] Test-CI-arbetsflöde
- [ ] Release-CI-arbetsflöde
- [ ] Branch protection aktiverad
- [ ] Branch ruleset aktiverad - endast auktoriserade har skrivåtkomst
- [ ] Sårbarhetsskanning av beroenden i CI aktiverad
- [ ] Linter konfigurerad
- [ ] Secret scanning aktiverad
- [ ] GPG-signering konfigurerad för releaser
- [ ] SBOM-generering konfigurerad
- [ ] OpenSSF Scorecard-integration
- [ ] Signerade commits dokumenterade
- [ ] CI-linter och kontroller kan köras lokalt utan CI

### Releasekrav
- [ ] API stabilt (eller implementerar stabil specifikation)
- [ ] Inga planerade brytande ändringar
- [ ] Version följer semantisk versionering

### Java Library Maven/POM-konfiguration (om tillämpligt)
- [ ] groupId, artifactId, version (semantisk versionering)
- [ ] name, description, url
- [ ] licenses-block
- [ ] scm-block
- [ ] maven-enforcer-plugin konfigurerad
- [ ] central-release profil med:
  - [ ] maven-gpg-plugin
  - [ ] maven-source-plugin
  - [ ] maven-javadoc-plugin
- [ ] central-publishing-maven-plugin konfigurerad

### Java App Maven/POM-konfiguration (om tillämpligt)
- [ ] groupId, artifactId, version (semantisk versionering)
- [ ] name, description, url
- [ ] licenses-block
- [ ] scm-block
- [ ] maven-enforcer-plugin konfigurerad

### JS/TS Lib-konfiguration (om tillämpligt)
- [ ] name, version (semantisk versionering)
- [ ] description
- [ ] license
- [ ] repository-block

## 🔵 Rekommenderat

### Ytterligare kvalitet
- [ ] Exempel i dokumentation
- [ ] Ändringslogg-flöde i CI
- [ ] **CHANGELOG.md** (Keep-a-Changelog format)
- [ ] Säkerhetsgranskning genomförd och dokumenterad

### Utvecklingspraxis
- [ ] Conventional commits format används
- [ ] Issue-mallar konfigurerade
- [ ] PR-mall konfigurerad

## Kriterier

✅ **Redo för 1.0.0 när**:

- Alla tillämpliga punkter avklarade
- Inga SNAPSHOT-beroenden
- Tester passerar med bra täckning
- API stabilt (inga brytande ändringar planerade)

⚠️ **Stanna i 0.x när**:

- Implementerar utkastspecifikationer
- API utvecklas fortfarande baserat på återkoppling från användare
- Brytande ändringar förväntas

---

# Release 1.0.0 Readiness Checklist

This document serves as a guide to ensure your project meets all requirements for a first 1.0-release.
Use this checklist to validate technical readiness, security compliance, and adherence to open source best practices.

Also see [Digg Open Source Checklist](https://github.com/diggsweden/open-source-project-template/blob/main/docs/Open_Source_Checklist.md) and [Digg Sweden Open Source Project Template](https://github.com/diggsweden/open-source-project-template)

## ✅ Required

### Documentation
- [ ] **README.md** with:
  - [ ] Badges (version, openssf, license, reuse status)
  - [ ] Project description
  - [ ] Installation instructions  
  - [ ] Usage examples
  - [ ] Maintainer section
- [ ] **LICENSE** file
- [ ] **SECURITY.md** with vulnerability reporting process
- [ ] **CONTRIBUTING.md** with contribution guidelines
- [ ] **CODE_OF_CONDUCT.md**
- [ ] **docs/DEVELOPMENT.md** with development setup

### Legal & Compliance
- [ ] REUSE compliance (SPDX headers in all source files)
- [ ] License compatibility verified
- [ ] Name/trademark check performed
- [ ] Not dependent on SNAPSHOT dependencies

### Code Quality
- [ ] Test coverage ≥ 50%
- [ ] Public APIs have documentation
- [ ] Code has been reviewed

### Git Hosting and Security - CI/CD
- [ ] Pull request CI-workflow
- [ ] Test CI-workflow 
- [ ] Release CI-workflow 
- [ ] Branch protection enabled
- [ ] Branch ruleset enabled - only Authorized have write access
- [ ] Dependency vulnerability scanning in CI enabled
- [ ] Linter configured
- [ ] Secret scanning enabled
- [ ] GPG signing configured for releases
- [ ] SBOM generation configured
- [ ] OpenSSF Scorecard integration
- [ ] Signed commits practice documented
- [ ] CI lints and checks can be run locally without CI

### Release Requirements
- [ ] API stable (or implements stable spec)
- [ ] No planned breaking changes
- [ ] Version follows semantic versioning

### Java Library Maven/POM Configuration (if applicable)
- [ ] groupId, artifactId, version (SemVer)
- [ ] name, description, url
- [ ] licenses block
- [ ] scm block
- [ ] maven-enforcer-plugin configured
- [ ] central-release profile with:
  - [ ] maven-gpg-plugin
  - [ ] maven-source-plugin
  - [ ] maven-javadoc-plugin
- [ ] central-publishing-maven-plugin configured

### Java App Maven/POM Configuration (if applicable)
- [ ] groupId, artifactId, version (semantic versioning)
- [ ] name, description, url
- [ ] licenses block
- [ ] scm block
- [ ] maven-enforcer-plugin configured

### JS/TS Lib Configuration (if applicable)
- [ ] name, version (semantic versioning)
- [ ] description
- [ ] license
- [ ] repository block

## 🔵 Recommended

### Additional Quality
- [ ] Examples in docs
- [ ] **CHANGELOG.md** (Keep-a-Changelog format)
- [ ] Changelog flow in CI
- [ ] Security assessment completed and documented

### Development Practice
- [ ] Conventional commits format used
- [ ] Issue templates configured
- [ ] PR template configured

## Release Decision Criteria

✅ **Ready for 1.0.0**:

- All applicable items checked
- No SNAPSHOT dependencies
- Tests pass with good coverage
- API stable (no draft, no breaking changes planned)

⚠️ **Stay in 0.x when**:

- Implementing draft specifications
- API and usage is still evolving based on general feedback
- Breaking changes might be anticipated

