---
name: repository-context-bundle-recommender
description: 'Collect repository context, retrieve relevant awesome-copilot artifacts, score candidate agents/instructions/skills, and return a ready-to-use bundle that acts like a retrievable collection for the current repository.'
argument-hint: 'Optional: target repo area, stack, or task such as "React API", "docs and testing", or "security review bundle"'
---

# Repository Context Bundle Recommender

Use this skill to turn the awesome-copilot repository into a retrievable artifact collection for the current workspace. It scans the repository being worked on, retrieves matching artifacts from the awesome-copilot corpus, scores them, and returns one best agent plus the best supporting instructions and skills as a ready-to-use bundle.

This skill is intentionally a **bundle selector**, not a duplicate library. Reuse the existing `suggest-awesome-github-copilot-agents`, `suggest-awesome-github-copilot-instructions`, and `suggest-awesome-github-copilot-skills` skills as candidate sources when they help, then combine their results into one cross-artifact recommendation.

## Retrievability Contract

The bundle is discoverable because its frontmatter and instructions are written for the repository indexers already used in this repo:

- `name` matches the folder name, so `parseSkillMetadata` can load it reliably.
- `description` contains the retrieval terms the corpus needs: **repository context**, **artifacts**, **agents**, **instructions**, **skills**, **bundle**, and **collection**.
- The repository generators use this frontmatter in `eng/update-readme.mjs` and `eng/generate-website-data.mjs`, which makes the skill show up in generated docs and search data such as:
  - `docs/README.skills.md`
  - website `search-index.json` / `llms.txt` outputs when generated
- The workflow below tells agents exactly how to scan, retrieve, score, and assemble the bundle, so the skill is not just discoverable but also operational once loaded.

## When to Use This Skill

Use this skill when you need to:

1. infer a repository's domain, stack, and workflow needs from its files
2. retrieve matching artifacts from the awesome-copilot corpus
3. turn related artifacts into a single recommended collection-like bundle
4. explain how an agent should apply the selected bundle and why each item was chosen

Do **not** use this skill when the user already named the exact agent, instruction, or skill they want. In that case, fetch the requested artifact directly.

## Inputs

Gather evidence from:

- workspace manifests and lockfiles (`package.json`, `pyproject.toml`, `pom.xml`, `Cargo.toml`, etc.)
- language and framework indicators
- CI, deployment, infra, and container files
- local custom artifacts already present in the target repository
- the awesome-copilot corpus:
  - `agents/`
  - `instructions/`
  - `skills/`
  - `collections/` if present
  - generated docs such as `docs/README.agents.md`, `docs/README.instructions.md`, and `docs/README.skills.md`
  - generated website data such as `search-index.json` or `llms.txt` if available

## Workflow

### Phase 1: Workspace Scan

Build a short context profile before looking for artifacts:

1. Detect primary languages from manifests and file extensions.
2. Detect frameworks, runtime, cloud, database, CI, test, and deployment signals.
3. Detect repository intent from README, docs, and recent user request context.
4. Detect existing local agents, instructions, and skills so you do not recommend duplicates.
5. Summarize the result as:

```text
domain:
primary stack:
secondary stack:
repo type:
delivery/test signals:
current task:
existing local artifacts:
```

### Phase 2: Artifact Corpus Retrieval

Retrieve candidate artifacts in the least expensive order that still preserves relevance:

1. **Use generated indexes first when available**
   - Search `docs/README.agents.md`, `docs/README.instructions.md`, and `docs/README.skills.md`.
   - If generated website data exists, search `search-index.json` or `llms.txt` for the same terms.
2. **Fall back to source artifact metadata**
   - Read frontmatter from `agents/*.agent.md`, `instructions/*.instructions.md`, and `skills/*/SKILL.md`.
   - Prefer artifacts whose metadata explicitly mentions the detected stack, task, or workflow.
3. **Reuse existing discovery skills instead of recreating their logic**
   - `suggest-awesome-github-copilot-agents`
   - `suggest-awesome-github-copilot-instructions`
   - `suggest-awesome-github-copilot-skills`
4. **Check for collections if they exist**
   - Search `collections/*.collection.yml`.
   - If no collection files exist, continue and assemble a virtual collection from the top-ranked artifacts.

### Phase 3: Scoring and Matching

Load [`references/scoring-rubric.md`](references/scoring-rubric.md) and score each candidate artifact. Use the same signal families across agents, instructions, skills, and collections:

- repository and stack match
- task match
- applicability metadata match (`applyTo`, tools, bundled assets, or explicit workflow language)
- complementarity with the rest of the bundle
- novelty vs. already-installed local artifacts

Hard rules:

- return **exactly one** best agent
- return **one or more** instructions if any clear match exists
- return **one or more** supporting skills if they materially help the chosen agent succeed
- do not recommend duplicates already installed locally unless the point is to update or replace them
- do not recommend a bundle item without a one-sentence rationale tied to workspace evidence

### Phase 4: Collection Matching

If `collections/*.collection.yml` exists:

1. score each collection by overlap with the detected context and top-ranked artifacts
2. prefer collections whose members reinforce each other instead of overlapping redundantly
3. use the highest-scoring collection as the base bundle, then prune low-fit members

If no explicit collections exist:

1. treat the top-ranked cross-artifact set as a **virtual collection**
2. define that virtual collection as:
   - one best agent
   - the best matching instructions
   - the best supporting skills
3. state clearly that the collection was synthesized from retrievable artifacts rather than loaded from a manifest

### Phase 5: Produce the Ready-to-Use Bundle

Return the result in this format:

```markdown
## Repository Context
- Domain:
- Primary stack:
- Current task:
- Key evidence:

## Matched Collection
- Collection: <explicit collection path or "virtual collection">
- Why it matches:

## Recommended Bundle
### Best agent
- `agents/...`
- Why chosen:
- How to use:

### Best instructions
- `instructions/...`
- Why chosen:
- How they apply:

### Supporting skills
- `skills/...`
- Why chosen:
- When to invoke:

## Agent Execution Guidance
1. Load the best agent first.
2. Apply the selected instructions for the relevant files or stack.
3. Invoke the supporting skills for specialized subtasks.
4. Mention any skipped artifacts and why they ranked lower.
```

## Selection Guidance

- Prefer artifacts with explicit stack vocabulary in frontmatter or descriptions over filename-only matches.
- Prefer instructions with `applyTo` patterns that align with the files actually present in the workspace.
- Prefer skills with bundled assets when those assets materially improve execution.
- Prefer a narrower, coherent bundle over a broad but noisy one.
- Only include the three `suggest-awesome-github-copilot-*` skills in the final bundle when the user's goal is ongoing artifact discovery or synchronization. Otherwise, use them as internal helpers and recommend domain skills instead.

## Why This Becomes a Retrievable Collection

This skill turns a flat repository of artifacts into a usable collection mechanism:

1. frontmatter makes the skill itself indexable and searchable
2. generated docs and search outputs make artifact metadata retrievable
3. the workflow converts retrieved candidates into a scored set
4. collection matching resolves that scored set into one bundle an agent can immediately use

That makes the collection both **discoverable** at search time and **actionable** at run time.
