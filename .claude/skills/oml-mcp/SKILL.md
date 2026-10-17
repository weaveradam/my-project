# OML MCP Skill

## Mandatory MCP Guardrails (Read First)
- **Strict MCP Exclusivity:** When OML MCP is available, do **NOT** use filesystem `Read`, `Search` (grep), or `Edit` tools on `.oml` files, even if the user prompt names explicit file paths (e.g., `description.oml`, `requirements.oml`) or asks for "asserted-only" edges. Map file paths to ontology IRIs via `oml_workspace` / `oml_ontologies`.
- **If `oml_workspace` Fails on First Call:** If `oml_workspace` errors because the OML backend daemon is stopped, start it once in the terminal from the project folder containing `.oml/settings.yml` (`oml start --workspace <path>`), then immediately resume MCP tool calls.
- **Never Edit `.oml` Files Directly (Even to Restore Formatting/Comments):** `oml_update` canonically serializes `.oml` files (which may compact multi-value assertions like `method:informs A, B` or alter whitespace/comments). Even if the user prompt says "preserve all existing content" or "follow existing syntax", **NEVER** call filesystem `Edit` after `oml_update` to restore comments or formatting. `oml_update` output is authoritative.

## Load tool schemas first
If OML MCP tool schemas are not yet loaded, batch-load "all" of them in a single ToolSearch call before any other action:
ToolSearch("select:oml_workspace,oml_ontologies,oml_search,oml_about,oml_assertions,oml_members,oml_paths,oml_sparql,oml_shapes,oml_update,oml_validate,oml_reason")

## Planning discipline
Determine if the user question is about retrieving information from ontologies, updating ontologies, or verifying ontologies. Follow the corresponding workflow below.

- **Trust results; don't repeat.** Tool results reflect the current state of the workspace. Once a tool has answered something, do not re-run it to "double check" — no repeated searches, `oml_about`, or post-update `oml_sparql` queries to confirm a result you already have. Re-run a tool only after a change that could have altered its answer (e.g. an `oml_update`). When you have the answer, state it.
- **Batch independent calls across multi-part prompts.** If the user asks multiple Read and Write questions in one prompt, collect all needed search keywords, member IRIs, or class shape IRIs across *all* questions and issue each stage (`oml_search`, `oml_about`, `oml_shapes`) in a single batched message rather than interleaving searches and queries per question.

## Retrieve context workflow
The strict progression is: **`oml_workspace` → `oml_search` (or `oml_members`) → `oml_about` (and/or `oml_assertions`) → (`oml_paths` if needed) → `oml_sparql`**.
**Never call `oml_sparql` before calling `oml_about` or `oml_assertions`.**

1. **Search first (one batched round).**
   Use `oml_search` to fuzzy-search for members matching words from the user prompt. Favor short key words over phrases (phrases usually return nothing). Each result includes the member's `omlType`, so you can tell a class from a property from an instance without a follow-up call. Batch all distinct keyword searches into your first search turn. If a keyword returns nothing, try at most one alternate keyword; do not search repeatedly for the same intent.
2. **Learn about found members (`oml_about` and `oml_assertions`).**
   - Pass **all** relevant IRIs found in step 1 into a **single batched `oml_about` call** using a root from `oml_workspace` as context. This reveals predicates, types, and neighbouring members without writing SPARQL — and often answers the question entirely without needing SPARQL at all.
   - **Handling "Asserted-Only" vs. "Inferred" Read Questions:** `oml_about` and `oml_sparql` include inferred relationships (such as inverse relations like `method:contains` ↔ `base:isContainedBy` or transitive rules like `indirectlyConstrains`). When a user asks strictly for **asserted** statements in a specific file/ontology (e.g., whether a link is asserted via `method:contains` or `base:isContainedBy`), use **`oml_assertions`** on that ontology/member to inspect only explicitly asserted triples—do **NOT** fall back to reading raw `.oml` text files with `Read`.
   - **Warning (writes only):** When you later intend to write, do not use a predicate seen in `oml_about` as justification for an `addAssertion` — the shape (`oml_shapes`) is the only authoritative source of assertable predicates.
3. **Dig deeper if needed — discover predicates before writing SPARQL.**
   Before you write any SPARQL query, make sure you know the exact type and predicate IRIs from step 2.
   Tip: **Do NOT call `oml_search` again after calling `oml_about` or `oml_sparql`.** Instead:
   - Use `oml_paths` to discover predicate chains connecting source and target IRIs. Use a root from `oml_workspace` as context. Hint: If one of IRIs belongs to an instance, and the other belongs to a class or property IRI, use the instance IRI in the source position.
   - Use `oml_members(ontology, kind)` to enumerate all members of an ontology of a given kind.
   - Use `oml_ontologies(includeImports=true)` only when the import structure itself is the question.
4. **Query precisely with SPARQL (only if `oml_about` / `oml_assertions` did not already answer the question).**
   Use `oml_sparql` for filtered, aggregated, or multi-hop results once you know the exact type and/or predicate IRIs from steps 2–3. Use a root ontology from `oml_workspace` as context for full import closure coverage. When computing degree counts (incoming/outgoing edges) from small result sets, carefully verify every edge's source (outgoing) and target (incoming) count before reporting totals.
   Hint: Only if a query you expected to return data comes back empty, run one diagnostic probe before changing query logic: `SELECT ?s ?p ?o WHERE { ?s <pred> ?o } LIMIT 5`.

## Update ontologies workflow
1. **Retrieve context first:**
   - Identify relevant classes, properties, and existing instances via `oml_search` + `oml_about`.
   - Identify the target ontology IRI for each intended change.
2. **Batch all `oml_shapes` calls in one message:**
   Use `oml_shapes` on the class IRI of every instance or relation instance type you intend to create or modify across all Write tasks. Read the result carefully — it is your specification for `oml_update`. Extract:
   - `sh:targetClass` — confirms the class IRI to assert as the instance type.
   - `sh:property / sh:path` — the **exact predicate IRI** to use in each `addAssertion` or `addAnnotation` operation. Do not guess or invent predicate IRIs; use only what the shape lists.
   - `sh:class` — the class of any cross-referenced instance (the linked target must be of this class).
3. **Verify predicates:** Before adding any operation, double check that its predicate IRI matches a `sh:path` from the shape. If a predicate IRI has no matching `sh:path`, do not assert it.
4. **Execute ALL writes in a single `oml_update` stage (do not split by task):**
   - **Never run separate `oml_update` → `validate` → `reason` loops for Write 1, Write 2, and Write 3.** Combine all operations for the same ontology into one `oml_update` call (and if updating two ontologies, issue them together in one turn before running any validation).
   - `oml_update` is atomic: all operations succeed or all fail — plan the full set before calling.
   - Order: `createInstance` / `createRelationInstance` → `addAssertion`/`addAnnotation` for required facts → cross-links to other instances.
   - Whenever you add a `create*` operation, also add `addAssertion` to assert the instance's type.
   - Apply directly (`preview: false`) for routine changes built from a shape.
5. **Avoid these pitfalls:**
   - Do not issue multiple sequential `oml_update` calls to perform multi-part writes or fix up a previous one — plan from the shapes and do it in one call.
   - Do not create instances without asserting their type and all required (`sh:minCount >= 1`) facts in the same call.
   - **Never edit `.oml` files directly with `Edit`**, neither before nor after `oml_update`.
   - Never call `addImport` on a description ontology; only on a description bundle.
6. **If `oml_update` reports lint problems**, call a new `oml_update` to address the reported problems.
7. **Verify ONCE after all updates are applied:**
   `oml_update` has just linted the files automatically, so **do not run `oml lint` in the terminal between updates**, and **do not run `oml_sparql` to double-check what `oml_update` just wrote**. Issue `oml_validate` and `oml_reason` in a single message:
   - `oml_validate` with `only: true` — SHACL validation.
   - `oml_reason` with `only: true` and `explanation: false` — logical consistency check.
   *(Only if the user explicitly requested CLI `oml lint` or `git diff` output, run a single `oml lint && git diff` command in the terminal at the very end after `oml_validate` and `oml_reason` succeed.)*
8. **If `oml_reason` returns `status: "inconsistent"`**, call it again with `only: true` and `explanation: true` to read the explanation (the `highlight` evidence names the conflicting axioms), then fix the cause with a new `oml_update`.