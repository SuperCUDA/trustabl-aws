# Trustabl on AWS CodePipeline (via CodeBuild)

1. **Vendor** this plugin into your repo: copy `scan/` and
   `codepipeline/buildspec.yml` to your repo.
2. **Create a CodeBuild project** that uses `buildspec.yml`. Image
   `aws/codebuild/standard:7.0` (has bash, curl, jq, git, tar).
3. **Add a CodeBuild action** to a Build/Test **stage** of your pipeline, with
   the source artifact as input.
4. **Configure** via environment variables on the project/action (see the root
   README inputs table), e.g. `SEVERITY_THRESHOLD=high`.

A gate failure exits non-zero -> the CodeBuild action fails -> the pipeline
stage fails. Artifacts (`trustabl.json`, `trustabl.sarif`,
`trustabl-summary.md`) are emitted to the artifact bucket.

**Optional — Security Hub:** convert findings to ASFF and
`aws securityhub batch-import-findings` to surface them in Security Hub (needs
Security Hub enabled + IAM `securityhub:BatchImportFindings`). Not in v0.1.0.
