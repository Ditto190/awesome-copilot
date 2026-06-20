# Scoring Rubric

Use this rubric to rank agents, instructions, skills, and explicit collections with the same scoring language.

## Weighted Signals

Score each dimension from 0-5, then multiply by the weight.

| Dimension | Weight | What to look for |
| --- | --- | --- |
| Repository and stack match | 5 | Direct mention of the detected language, framework, cloud, database, or repo type |
| Task match | 4 | Direct support for the user's current task such as implementation, review, testing, docs, or deployment |
| Applicability metadata | 3 | `applyTo`, tools, assets, workflow wording, or examples that match the repo context |
| Bundle complementarity | 2 | Helps the chosen agent succeed without duplicating another selected artifact |
| Novelty / gap coverage | 2 | Fills a gap not already covered by local custom artifacts |
| Collection overlap | 2 | For explicit collections, overlap with the top-ranked individual artifacts |

## Tie-Breakers

When scores are tied, prefer:

1. artifacts with clearer frontmatter and descriptions
2. instructions with the strongest `applyTo` fit
3. skills with useful bundled assets
4. artifacts that reference the detected workflow more concretely
5. smaller, more coherent bundles over larger bundles

## Bundle Assembly Rules

- Choose the highest-scoring **agent** as the single best agent.
- Include every **instruction** whose weighted score is strong and whose `applyTo` or scope is relevant.
- Include **supporting skills** that either:
  - improve context gathering
  - improve execution for the selected task
  - reduce risk for the chosen agent
- Exclude artifacts that are redundant, off-stack, or weaker duplicates of a higher-ranked item.

## Virtual Collection Rule

If no `collections/*.collection.yml` files exist, treat the final scored bundle as a virtual collection and label it that way in the output.
