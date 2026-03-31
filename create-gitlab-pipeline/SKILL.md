---
name: Create-GitLab-Pipeline
description: Create or improve a .gitlab-ci.yml for a .NET NuGet library with automated semver, preview/stable publishing, JUnit test reporting, git tagging, and GitLab releases with semantic commit changelogs
---

# Create GitLab Pipeline Skill

When invoked, this skill creates or rewrites a `.gitlab-ci.yml` for a .NET library project that publishes NuGet packages. It implements automated semantic versioning, duplicate pipeline prevention, preview/stable package publishing, JUnit test reporting, git tagging, and GitLab releases with auto-generated changelogs.

---

## Step 1: Analyse the Repository

Before writing any YAML, explore:

1. **Find the solution and project files:**
   ```bash
   find . -name "*.sln" -o -name "*.csproj" | head -20
   ```

2. **Read the library `.csproj`** — note the `<PackageId>`, existing `<Version>`, `<TargetFramework>`, and any `<PackageReadmeFile>`.

3. **Read the test `.csproj`** — check if `JunitXml.TestLogger` is already referenced.

4. **Check the existing `.gitlab-ci.yml`** if present.

5. **Check git tags** to understand the current version baseline:
   ```bash
   git tag -l "v*" --sort=-v:refname | head -5
   ```

---

## Step 2: Update Project Files

### Library `.csproj`

- **Remove** any static `<Version>X.Y.Z</Version>` — version is injected by CI via `-p:` flags.
- **Replace** it with a comment:
  ```xml
  <!-- Version is injected by CI: -p:PackageVersion, -p:AssemblyVersion, -p:FileVersion, -p:InformationalVersion -->
  ```
- **Enable README packaging** (uncomment or add):
  ```xml
  <PackageReadmeFile>README.md</PackageReadmeFile>
  ```
- **Add a `<None>` item** to include the repo-root README in the package (required by NuGet when the README lives outside the project directory):
  ```xml
  <ItemGroup>
      <!-- Include repo root README in the NuGet package (required by PackageReadmeFile) -->
      <None Include="../README.md" Pack="true" PackagePath="\" />
  </ItemGroup>
  ```

### Test `.csproj`

Add `JunitXml.TestLogger` for GitLab JUnit test report integration:
```xml
<PackageReference Include="JunitXml.TestLogger" Version="3.0.134" />
```

---

## Step 3: Write the `.gitlab-ci.yml`

Use the complete template below, substituting the correct values:

- `{PACKAGE_ID}` — from `<PackageId>` in the library csproj (e.g. `Aurora.MSGraph.Library`)
- `{LIBRARY_CSPROJ}` — relative path to the library csproj (e.g. `AuroraMsLibrary/AuroraMsLibrary.csproj`)
- `{TEST_CSPROJ}` — relative path to the test csproj (e.g. `AuroraMsTests/AuroraMsTests.csproj`)
- `{TEST_RESULTS_PATH}` — JUnit XML output path (e.g. `AuroraMsTests/TestResults/test-results.xml`)
- `{DOTNET_IMAGE}` — SDK Docker image (e.g. `mcr.microsoft.com/dotnet/sdk:10.0`)

```yaml
# ─────────────────────────────────────────────────────────────────────────────
# {PACKAGE_ID} — GitLab CI/CD
#
# Pipeline behaviour:
#   feat/* / fix/* (no open MR)  → build → test → publish preview NuGet
#   feat/* / fix/* (MR open)     → MR pipeline runs instead (no duplication)
#   MR pipeline                  → build → test → publish preview NuGet
#   push to main (post-merge)    → build → test → publish stable NuGet
#                                  → create git tag → create GitLab release
#   Manual trigger (Run pipeline)→ full pipeline with preview version
#
# Required CI variables (Settings → CI/CD → Variables):
#   GL_TOKEN  – project access token, Role: Maintainer, Scope: write_repository
#               Masked + Protected = OFF (must be available on all branches)
#
# NOTE: CI_JOB_TOKEN is read-only for git operations by GitLab design.
#       GL_TOKEN is required for git tag push regardless of user permissions.
# ─────────────────────────────────────────────────────────────────────────────

# Prevent duplicate pipelines: when an MR is open, skip the branch push
# pipeline and let the MR pipeline handle it instead.
workflow:
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
    - if: '$CI_COMMIT_BRANCH && $CI_OPEN_MERGE_REQUESTS'
      when: never
    - if: '$CI_PIPELINE_SOURCE == "web"'
    - if: '$CI_COMMIT_BRANCH == "main"'
    - if: '$CI_COMMIT_BRANCH =~ /^(feat|fix)\//'
    - when: never

image: {DOTNET_IMAGE}

# Full clone required so git can walk back to the previous tag
variables:
  GIT_DEPTH: 0

stages:
  - version
  - build
  - test
  - package
  - publish
  - tag
  - release

# ─────────────────────────────────────────────────────────────────────────────
# STAGE: version
#
# Computes the next semver by:
#   1. Finding the most recent vX.Y.Z git tag (defaults to v1.0.0)
#   2. Walking commits since that tag and applying conventional-commit rules:
#        feat!: / BREAKING CHANGE  → major bump
#        feat:                     → minor bump
#        fix: / docs: / chore: etc → patch bump
#
# Exports a dotenv artifact consumed by all downstream jobs:
#   NEXT_VERSION            – e.g. 1.3.0
#   PACKAGE_VERSION         – e.g. 1.3.0 (stable) | 1.3.0-preview.42 (preview)
#   ASSEMBLY_VERSION        – e.g. 1.3.0  (.NET CLR: no pre-release labels)
#   FILE_VERSION            – e.g. 1.3.0.42  (major.minor.patch.pipelineIid)
#   INFORMATIONAL_VERSION   – e.g. 1.3.0-preview.42
#   LAST_TAG                – e.g. v1.2.3
# ─────────────────────────────────────────────────────────────────────────────
compute_version:
  stage: version
  script:
    - |
      git fetch --tags --force
      LAST_TAG=$(git tag -l "v[0-9]*.[0-9]*.[0-9]*" --sort=-v:refname | head -1)
      if [ -z "$LAST_TAG" ]; then
        LAST_TAG="v1.0.0"
        echo "No previous tag found; starting from $LAST_TAG"
      else
        echo "Last tag: $LAST_TAG"
      fi

      VERSION_CORE="${LAST_TAG#v}"
      MAJOR=$(echo "$VERSION_CORE" | cut -d. -f1)
      MINOR=$(echo "$VERSION_CORE" | cut -d. -f2)
      PATCH=$(echo "$VERSION_CORE" | cut -d. -f3)

      COMMITS=$(git log "${LAST_TAG}..HEAD" --format="%s%n%b" 2>/dev/null || git log --format="%s%n%b")
      BUMP="patch"

      if echo "$COMMITS" | grep -qiE "^(feat|fix|refactor|perf|docs|chore|test|style|ci|build)\!:"; then
        BUMP="major"
      elif echo "$COMMITS" | grep -qiE "BREAKING.CHANGE"; then
        BUMP="major"
      elif echo "$COMMITS" | grep -qiE "^feat:"; then
        BUMP="minor"
      fi

      echo "Commit bump level: $BUMP"

      case "$BUMP" in
        major) MAJOR=$((MAJOR + 1)); MINOR=0; PATCH=0 ;;
        minor) MINOR=$((MINOR + 1)); PATCH=0 ;;
        patch) PATCH=$((PATCH + 1)) ;;
      esac

      NEXT_VERSION="${MAJOR}.${MINOR}.${PATCH}"
      echo "Next version: $NEXT_VERSION"

      if [ "$CI_COMMIT_BRANCH" = "main" ] && [ "$CI_PIPELINE_SOURCE" = "push" ]; then
        PACKAGE_VERSION="$NEXT_VERSION"
        INFORMATIONAL_VERSION="$NEXT_VERSION"
      else
        PACKAGE_VERSION="${NEXT_VERSION}-preview.${CI_PIPELINE_IID}"
        INFORMATIONAL_VERSION="${NEXT_VERSION}-preview.${CI_PIPELINE_IID}"
      fi

      ASSEMBLY_VERSION="${NEXT_VERSION}"
      FILE_VERSION="${NEXT_VERSION}.${CI_PIPELINE_IID}"

      cat > version.env <<EOF
      NEXT_VERSION=$NEXT_VERSION
      LAST_TAG=$LAST_TAG
      PACKAGE_VERSION=$PACKAGE_VERSION
      ASSEMBLY_VERSION=$ASSEMBLY_VERSION
      FILE_VERSION=$FILE_VERSION
      INFORMATIONAL_VERSION=$INFORMATIONAL_VERSION
      EOF
      cat version.env
  artifacts:
    reports:
      dotenv: version.env

# ─────────────────────────────────────────────────────────────────────────────
# STAGE: build
# ─────────────────────────────────────────────────────────────────────────────
build_library:
  stage: build
  needs:
    - job: compute_version
      artifacts: true
  script:
    - dotnet restore {LIBRARY_CSPROJ}
    - >
      dotnet build {LIBRARY_CSPROJ}
      --configuration Release
      --no-restore
      -p:GeneratePackageOnBuild=false
      -p:AssemblyVersion=$ASSEMBLY_VERSION
      -p:FileVersion=$FILE_VERSION
      -p:InformationalVersion=$INFORMATIONAL_VERSION
      -p:PackageVersion=$PACKAGE_VERSION
  artifacts:
    paths:
      - {LIBRARY_BIN_PATH}
      - {LIBRARY_OBJ_PATH}

# ─────────────────────────────────────────────────────────────────────────────
# STAGE: test
#
# Produces JUnit XML test results surfaced as:
#   - Tests tab on every MR (pass/fail per test)
#   - Test Cases view in GitLab (history, trends, flaky test detection)
# ─────────────────────────────────────────────────────────────────────────────
run_tests:
  stage: test
  needs:
    - job: build_library
      artifacts: true
    - job: compute_version
      artifacts: true
  script:
    - dotnet restore {TEST_CSPROJ}
    - dotnet build {TEST_CSPROJ} --configuration Release --no-restore
    - >
      dotnet test {TEST_CSPROJ}
      --configuration Release
      --no-build
      --verbosity normal
      --logger "junit;LogFilePath=TestResults/test-results.xml"
  artifacts:
    when: always
    reports:
      junit: {TEST_RESULTS_PATH}
    paths:
      - {TEST_RESULTS_DIR}

# ─────────────────────────────────────────────────────────────────────────────
# STAGE: package
# ─────────────────────────────────────────────────────────────────────────────
pack_library:
  stage: package
  needs:
    - job: build_library
      artifacts: true
    - job: run_tests
      artifacts: false
    - job: compute_version
      artifacts: true
  script:
    - rm -f {LIBRARY_NUPKG_GLOB}
    - >
      dotnet pack {LIBRARY_CSPROJ}
      --configuration Release
      --no-build
      -p:AssemblyVersion=$ASSEMBLY_VERSION
      -p:FileVersion=$FILE_VERSION
      -p:InformationalVersion=$INFORMATIONAL_VERSION
      -p:PackageVersion=$PACKAGE_VERSION
  artifacts:
    paths:
      - {LIBRARY_NUPKG_GLOB}

# ─────────────────────────────────────────────────────────────────────────────
# STAGE: publish
# ─────────────────────────────────────────────────────────────────────────────
publish_nuget:
  stage: publish
  needs:
    - job: pack_library
      artifacts: true
    - job: compute_version
      artifacts: true
  rules:
    - if: '$CI_COMMIT_BRANCH == "main" && $CI_PIPELINE_SOURCE == "push"'
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event" && $CI_MERGE_REQUEST_SOURCE_BRANCH_NAME =~ /^(feat|fix)\//'
    - if: '$CI_COMMIT_BRANCH =~ /^(feat|fix)\//'
    - if: '$CI_PIPELINE_SOURCE == "web"'
  script:
    - >
      dotnet nuget add source
      "${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/packages/nuget/index.json"
      --name gitlab
      --username gitlab-ci-token
      --password $CI_JOB_TOKEN
      --store-password-in-clear-text
    - dotnet nuget push "{LIBRARY_NUPKG_GLOB}" --source gitlab --skip-duplicate

# ─────────────────────────────────────────────────────────────────────────────
# STAGE: tag
#
# Creates a git tag vX.Y.Z using a project access token (GL_TOKEN).
# CI_JOB_TOKEN is read-only for git push by GitLab design.
#
# Setup: Settings → Access Tokens → Add new token
#   Role: Maintainer, Scope: write_repository
#   Then: Settings → CI/CD → Variables → GL_TOKEN (Masked, Protected = OFF)
# ─────────────────────────────────────────────────────────────────────────────
create_tag:
  stage: tag
  needs:
    - job: publish_nuget
      artifacts: false
    - job: compute_version
      artifacts: true
  rules:
    - if: '$CI_COMMIT_BRANCH == "main" && $CI_PIPELINE_SOURCE == "push"'
    - if: '$CI_PIPELINE_SOURCE == "web"'
  script:
    - |
      TAG="v${NEXT_VERSION}"
      echo "Creating tag ${TAG} at ${CI_COMMIT_SHA}"
      git config user.email "ci@${CI_SERVER_HOST}"
      git config user.name "GitLab CI"
      git remote set-url origin "https://oauth2:${GL_TOKEN}@${CI_SERVER_HOST}/${CI_PROJECT_PATH}.git"
      if git ls-remote --tags origin | grep -q "refs/tags/${TAG}$"; then
        echo "Tag ${TAG} already exists — skipping."
      else
        git tag -a "${TAG}" "${CI_COMMIT_SHA}" -m "Release ${TAG}"
        git push origin "${TAG}"
        echo "Tag ${TAG} created successfully."
      fi

# ─────────────────────────────────────────────────────────────────────────────
# STAGE: release
#
# Generates a changelog from conventional commits since the last tag and
# creates a GitLab Release with:
#   - Categorised changelog (Features, Bug Fixes, Docs, etc.)
#   - Author name in brackets after each entry
#   - Merge commits excluded
#   - NuGet installation instructions
# ─────────────────────────────────────────────────────────────────────────────
create_release:
  stage: release
  image: registry.gitlab.com/gitlab-org/release-cli:latest
  needs:
    - job: create_tag
      artifacts: false
    - job: compute_version
      artifacts: true
  rules:
    - if: '$CI_COMMIT_BRANCH == "main" && $CI_PIPELINE_SOURCE == "push"'
    - if: '$CI_PIPELINE_SOURCE == "web"'
  script:
    - apk add --no-cache git
    - git fetch --tags --force
    - |
      BREAKING=""
      FEATURES=""
      FIXES=""
      DOCS=""
      PERF=""
      REFACTORS=""
      TESTS=""
      CHORES=""
      OTHER=""

      while IFS= read -r line; do
        [ -z "$line" ] && continue
        SUBJECT="${line%|||*}"
        AUTHOR="${line##*|||}"
        # Skip merge commits
        case "$SUBJECT" in
          Merge*) continue ;;
        esac
        # Skip branch-name-style commit messages e.g. "Fix/username/description"
        # These are auto-generated MR titles from branch names — not useful in a changelog
        case "$SUBJECT" in
          Fix/*|fix/*|Feat/*|feat/*|Feature/*|feature/*|Chore/*|chore/*|Docs/*|docs/*|Refactor/*|refactor/*|Test/*|test/*|Perf/*|perf/*) continue ;;
        esac
        ENTRY="- ${SUBJECT} [${AUTHOR}]\n"
        case "$SUBJECT" in
          "feat!:"*)    BREAKING="${BREAKING}- ${SUBJECT#feat!: } [${AUTHOR}]\n" ;;
          "fix!:"*)     BREAKING="${BREAKING}- ${SUBJECT#fix!: } *(breaking)* [${AUTHOR}]\n" ;;
          "feat:"*)     FEATURES="${FEATURES}- ${SUBJECT#feat: } [${AUTHOR}]\n" ;;
          "fix:"*)      FIXES="${FIXES}- ${SUBJECT#fix: } [${AUTHOR}]\n" ;;
          "docs:"*)     DOCS="${DOCS}- ${SUBJECT#docs: } [${AUTHOR}]\n" ;;
          "perf:"*)     PERF="${PERF}- ${SUBJECT#perf: } [${AUTHOR}]\n" ;;
          "refactor:"*) REFACTORS="${REFACTORS}- ${SUBJECT#refactor: } [${AUTHOR}]\n" ;;
          "test:"*)     TESTS="${TESTS}- ${SUBJECT#test: } [${AUTHOR}]\n" ;;
          "chore:"*)    CHORES="${CHORES}- ${SUBJECT#chore: } [${AUTHOR}]\n" ;;
          *)            OTHER="${OTHER}${ENTRY}" ;;
        esac
      done <<EOF
      $(git log "${LAST_TAG}..HEAD" --no-merges --format="%s|||%an" 2>/dev/null || git log --no-merges --format="%s|||%an")
      EOF

      CHANGELOG=""
      [ -n "$BREAKING" ]   && CHANGELOG="${CHANGELOG}## 💥 Breaking Changes\n${BREAKING}\n"
      [ -n "$FEATURES" ]   && CHANGELOG="${CHANGELOG}## ✨ Features\n${FEATURES}\n"
      [ -n "$FIXES" ]      && CHANGELOG="${CHANGELOG}## 🐛 Bug Fixes\n${FIXES}\n"
      [ -n "$PERF" ]       && CHANGELOG="${CHANGELOG}## ⚡ Performance\n${PERF}\n"
      [ -n "$REFACTORS" ]  && CHANGELOG="${CHANGELOG}## ♻️ Refactoring\n${REFACTORS}\n"
      [ -n "$DOCS" ]       && CHANGELOG="${CHANGELOG}## 📚 Documentation\n${DOCS}\n"
      [ -n "$TESTS" ]      && CHANGELOG="${CHANGELOG}## 🧪 Tests\n${TESTS}\n"
      [ -n "$CHORES" ]     && CHANGELOG="${CHANGELOG}## 🔧 Chores\n${CHORES}\n"
      [ -n "$OTHER" ]      && CHANGELOG="${CHANGELOG}## 📝 Other Changes\n${OTHER}\n"
      [ -z "$CHANGELOG" ]  && CHANGELOG="No conventional commits found since ${LAST_TAG}.\n"

      INSTALL_INSTRUCTIONS="## 📦 Installation

      **dotnet CLI**
      \`\`\`
      dotnet add package {PACKAGE_ID} --version ${NEXT_VERSION}
      \`\`\`

      **Package Manager Console**
      \`\`\`
      Install-Package {PACKAGE_ID} -Version ${NEXT_VERSION}
      \`\`\`

      **\`.csproj\` reference**
      \`\`\`xml
      <PackageReference Include=\"{PACKAGE_ID}\" Version=\"${NEXT_VERSION}\" />
      \`\`\`
      "

      RELEASE_DESCRIPTION="${INSTALL_INSTRUCTIONS}
      ---
      ## 📋 What's Changed (since ${LAST_TAG})

      ${CHANGELOG}
      ---
      *Full diff: [${LAST_TAG}...v${NEXT_VERSION}](${CI_PROJECT_URL}/-/compare/${LAST_TAG}...v${NEXT_VERSION})*"

      printf '%b' "$RELEASE_DESCRIPTION" > release-notes.md
      cat release-notes.md
  release:
    tag_name: "v$NEXT_VERSION"
    name: "{PACKAGE_ID} v$NEXT_VERSION"
    description: "./release-notes.md"
```

---

## Step 4: Key Design Principles

### Versioning (.NET specifics)

| Variable | Format | Example | Reason |
|---|---|---|---|
| `AssemblyVersion` | `major.minor.patch` | `1.3.0` | CLR constraint — no alpha characters |
| `FileVersion` | `major.minor.patch.iid` | `1.3.0.42` | Adds pipeline IID for traceability |
| `InformationalVersion` | Full SemVer | `1.3.0-preview.42` | Human-readable, visible in assembly metadata |
| `PackageVersion` | Full SemVer | `1.3.0-preview.42` | Used by NuGet package resolution |

### Stable vs Preview

| Condition | Package version |
|---|---|
| Push to `main` (post-merge) | `1.3.0` (stable) |
| Everything else | `1.3.0-preview.{pipelineIid}` |

### No Duplicate Pipelines

`workflow: rules` using `$CI_OPEN_MERGE_REQUESTS` ensures when an MR is open for a branch, only the MR pipeline runs — not both the branch push pipeline and the MR pipeline simultaneously.

### GL_TOKEN Warning

`CI_JOB_TOKEN` is **read-only for git push** by GitLab design regardless of user permissions. A project access token stored as `GL_TOKEN` is always required for tag creation. Crucially, the variable must **not** have the Protected flag enabled unless the branch is also protected — otherwise GitLab silently omits the variable and auth will fail with HTTP 401/403.

### Conventional Commits

The pipeline expects commit messages following [Conventional Commits](https://www.conventionalcommits.org/):

| Prefix | Semver bump | Release section |
|---|---|---|
| `feat!:` / `BREAKING CHANGE` | Major | 💥 Breaking Changes |
| `feat:` | Minor | ✨ Features |
| `fix:` | Patch | 🐛 Bug Fixes |
| `docs:` | Patch | 📚 Documentation |
| `perf:` | Patch | ⚡ Performance |
| `refactor:` | Patch | ♻️ Refactoring |
| `test:` | Patch | 🧪 Tests |
| `chore:` | Patch | 🔧 Chores |

Merge commits are excluded from the changelog via `--no-merges`.

---

## Step 5: Update the README

Add or replace the CI/CD Pipeline section in `README.md` with:

1. **Pipeline Triggers table** — showing version format, NuGet publish, tag, and release per scenario
2. **Pipeline Stages table** — mapping each stage/job to its purpose
3. **Automated Versioning** — conventional commit bump rules
4. **Version Variables table** — all 4 .NET version properties
5. **Test Results** — JUnit XML GitLab integration explanation
6. **GitLab Release Contents** — what each release includes
7. **Required CI/CD Variables** — GL_TOKEN setup steps
8. **Manual Release trigger** — step-by-step Run pipeline instructions

Also update the **Package Registry** section with the new `1.x.x` version format and all install methods.

---

## Step 6: Verify

After writing all files, confirm:

- [ ] Library csproj has no static `<Version>` and includes README `<None>` item
- [ ] Test csproj has `JunitXml.TestLogger` reference
- [ ] `.gitlab-ci.yml` has `GIT_DEPTH: 0`
- [ ] `workflow: rules` includes `web` trigger and MR deduplication
- [ ] `create_tag` uses `oauth2:${GL_TOKEN}` remote URL
- [ ] `create_release` uses `--no-merges` and `%s|||%an` format
- [ ] README documents `GL_TOKEN` setup with Protected = OFF warning
