# Jenkins to GitHub Actions Migration Report

## Summary

Migrated the repository Jenkins configurations to GitHub Actions in `.github/workflows/migrated-jenkins.yml` and archived the original Jenkins files under `.github/ci-archive/`.

## Source Jenkins files

| Original file | Archived file | Pipeline type | Notes |
| --- | --- | --- | --- |
| `Jenkinsfile` | `.github/ci-archive/Jenkinsfile` | Declarative pipeline | Conditional stages using Jenkins `when`, environment variables, and branch checks. |
| `msbuilddotnet/Jenkinsfile` | `.github/ci-archive/msbuilddotnet/Jenkinsfile` | Scripted pipeline | Windows .NET restore, MSBuild, VSTest, artifacts, HTML report, and master-branch deployment. |

## Created GitHub Actions workflow

- `.github/workflows/migrated-jenkins.yml`

### Job mappings

| Jenkins stage | GitHub Actions mapping |
| --- | --- |
| Root pipeline `agent any` | `conditional-stages` job on `ubuntu-latest` |
| Jenkins environment variables | Workflow-level `env` values |
| `when { expression { false } }` | Step-level `if: ${{ false }}` |
| `when { branch 'master' }` | Step-level source-branch check using `github.head_ref || github.ref_name` |
| Jenkins AND/OR/environment conditions | Equivalent GitHub Actions expressions |
| Scripted `node` checkout | `actions/checkout` pinned to commit SHA |
| `nuget restore` | `nuget restore SolutionName.sln` on `windows-latest` |
| Jenkins `tool 'MSBuild'` | `microsoft/setup-msbuild` pinned to commit SHA |
| Jenkins `tool 'VSTest'` | `vstest.console.exe` on `windows-latest` |
| `publishTestResults` | TRX artifact upload with `actions/upload-artifact` |
| `archiveArtifacts` and `publishHTML` | Build artifact and report uploads |
| Master-only deploy | Release artifact is named `deployment-package` on `master` |

## Shared libraries

No Jenkins shared library references were present, so no shared library expansion was required.

## Credentials and secrets

No Jenkins credentials or secret bindings were present. No GitHub repository secrets are required by the migrated workflow.

## Actions and pinning

All third-party workflow dependencies are official or verified publisher actions and are pinned to commit SHAs:

- `actions/checkout` v4: `11d5960a326750d5838078e36cf38b85af677262`
- `actions/upload-artifact` v4: `ea165f8d65b6e75b540449e92b4886f43607fa02`
- `NuGet/setup-nuget` v2: `d105a947828025cd7a980103c35ba2bfae586d0f`
- `microsoft/setup-msbuild` v2: `6fb02220983dee41ce7ae257b6f4d8f9bf5ed4ce`

The GitHub Advisory Database runtime check was attempted for these Actions dependencies, but the tool could not complete because it reported that a GitHub token is required.

## Validation

- Passed: `go run github.com/rhysd/actionlint/cmd/actionlint@v1.7.7 .github/workflows/migrated-jenkins.yml`.
- Passed: secret scan of migrated workflow, migration report, and archived Jenkinsfiles reported no secrets.

## Behavioral notes

The migrated MSBuild job preserves the Jenkins commands and paths (`SolutionName.sln`, `ProjectName.Tests`, and `ProjectName/bin/Release`). If those files are placeholders rather than repository files, the workflow will require the corresponding .NET solution and project artifacts before the job can complete successfully.

The Jenkins deploy stage copied build output to `C:\Deploy\ProjectName\` on the Jenkins node. Because GitHub-hosted runners are ephemeral and no remote deployment target or credentials were defined in the Jenkins source, the migrated workflow uses mutually exclusive release upload steps: `deployment-package` on `master` for downstream deployment and `project-release` on other branches.
