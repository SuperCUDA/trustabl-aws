# Trustabl on Amazon CodeCatalyst

1. **Vendor** this plugin into your repo: copy `scan/` to your repo.
2. Add `codecatalyst/workflows/trustabl.yaml` to `.codecatalyst/workflows/` in
   your repo (or paste it via the CodeCatalyst workflow editor).
3. **Configure** via the `Variables` block in the workflow (see the root README
   inputs table), e.g. `SEVERITY_THRESHOLD=high`.

A gate failure exits non-zero -> the action fails. Findings surface in the
**Reports** tab via the SARIF report (`trustabl.sarif`).

**Alternative (quick) path:** instead of the native script, run the existing
GitHub Action in a CodeCatalyst workflow via the GitHub Actions action
(`trustabl/trustabl-action@v0`). Verify GitHub-Actions-in-CodeCatalyst support
against current AWS docs.
