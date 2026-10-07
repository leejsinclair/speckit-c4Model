# Changelog

All notable changes to this project are documented here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and the project uses
[Semantic Versioning](https://semver.org/). The extension and the preset share
one version number.

## [0.1.0] - 2026-10-08

### Added

- Extension `c4`: commands `speckit.c4.system`, `speckit.c4.feature` and
  `speckit.c4.validate`; optional `after_plan` and `after_implement` hooks;
  templates for the C4 context view, container view, ER view and per-feature
  architecture document; shared diagram conventions.
- Preset `c4-model`: wraps `speckit.plan` to generate `architecture.md` in
  Phase 1, and appends a System Context section to the spec template and an
  Architecture section to the plan template.
