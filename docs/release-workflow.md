# Release Workflow

This repository follows a small, repeatable release flow:

1. Keep the next release note in `CHANGELOG.md` under a dated version heading.
2. Build and validate the two shipped EXEs with the GitHub Actions release workflow.
3. Tag the release as `v<version>` so `.github/workflows/release.yml` publishes the assets.
4. The release assets are the single-file Windows EXEs for `ConsoleReader` and `ReplayWatcher`.

Release style used here:

- Changelog format: Keep a Changelog.
- Versioning: Semantic Versioning.
- Release artifacts: Windows single-file EXEs published from `ConsoleReader` and `ReplayWatcher`.