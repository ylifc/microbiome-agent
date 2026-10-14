---
name: microbiome-visualization
description: Design, render and review microbial figures for community ecology, phylogeny, MAGs, gene abundance, KEGG/NR/COG/arCOG annotation, pathways/modules, contig gene maps and WGCNA. Provides figure-specific guidance, R/Python tools and scientific data checks.
---

Produce figures from recorded data, with exact identity joins and visible limitations. Cover the common figure families across amplicon, shotgun metagenomics, isolates, MAGs, microbial functional omics and culture experiments. This is a broad maintained atlas, not a claim that every specialist plot or third-party package is installed or verified. Use numerical plotting software for data figures; never manufacture trees, statistics or missing values with image generation.

For substantial new figures, first show the intended question, data source, scientific tradeoffs, and a small layout preview. Discuss meaningful choices (overview versus detail, quality/coverage inclusion, support statistic, grouping and color encoding). Routine reversible styling within an agreed design does not require repeated approval.

## Choose the relevant guide

Read only the section needed for the user's question. Every atlas entry gives the question, data contract, concrete R/other-software route, interpretation limits and acceptance checks.

| Figure request | Reference |
| --- | --- |
| Phylograms, cladograms, circular/unrooted/time trees, clade summaries, annotation rings, branch support, tree comparison and discordance | [Phylogeny atlas](references/phylogeny-atlas.md) |
| Abundance/composition, diversity, rarefaction, ordination, differential results, ecological networks, overlap, longitudinal and multi-omics views | [Community atlas](references/community-atlas.md) |
| Read/assembly QC, MAG quality and lineage, ANI/AAI, pangenomes, synteny/genome maps, pathways, AMR/virulence, virus/plasmid and laboratory data | [Genomics and experimental atlas](references/genomics-atlas.md) |
| R/ggplot2/phyloseq/vegan recipes and their exact schemas | [R community recipes](references/r-community-recipes.md) |
| R, Python, iTOL, FigTree, Cytoscape, IGV and other software workflows | [Software recipes](references/software-recipes.md) |
| Reusable data/statistical/visual acceptance rules and selecting a defensible encoding | [Figure standards](references/figure-standards.md) |
| Run the dependency-light R table plotting CLI | [Table CLI](references/table-cli.md) |
| Gene TPM/counts, KEGG/NR/COG/arCOG integration, gene-family versus locus identity, module/pathway mapping | [Functional data contract](references/functional-data-contract.md) |
| Functional abundance, pathway detail, module step coverage, contig gene neighborhoods and layered associations | [Functional figure atlas](references/functional-atlas.md) |
| R functional heatmaps, gene arrows, contribution diagrams and pathway overlays | [R functional recipes](references/functional-r-recipes.md) |
| Local gene-to-function aggregation and contig plotting tools | [Functional CLI](references/functional-cli.md) |
| Weighted gene co-abundance/co-expression networks, eigengenes, trait associations and stability | [WGCNA analysis and figures](references/wgcna.md) |

For functional figures, distinguish measured quantities from annotation membership. DNA-derived TPM describes relative gene abundance, not transcription or flux. Preserve multi-database evidence without adding the same gene once per hit. KO→module and KO→pathway are versioned many-to-many relationships; NR descriptions are not KO assignments. Show module step completeness separately from member abundance, and distinguish KEGG modules from WGCNA modules. A representative gene family's TPM cannot be copied to every member locus and summed as contig-specific abundance. Read the functional data contract before adapting these joins.

For WGCNA, inspect the independent sample count and assay before choosing a network. Use the WGCNA guide's preflight, transformation, categorical trait and stability provisions. In the current DBDPE project there are 12 independent cage pools; a 12-sample network is an exploratory sensitivity analysis with substantial limits, not a validated regulatory mechanism. Skill availability does not mean the WGCNA R package or additional annotation databases are installed, nor that a new analysis has been executed.

## Implementations and their boundaries

1. **R tables** — `scripts/plot_table_base.R` renders nine already-analysed table types with base R: composition, heatmap, raw alpha-diversity values with median, ordination coordinates, volcano, forest, MAG quality, rarefaction curves and growth/time series. It writes PDF/SVG/PNG, exact data and hashes. It does not compute diversity, fit models, correct p-values, infer networks or normalize abundances. See its table contract for explicit effect units, q thresholds and interval reference.
2. **R trees** — `scripts/plot_phylogeny.R` provides selected `ape` layouts with native branch/identity preservation and a dependency preflight. Package absence is a capability gap, not permission to install into a production environment. Advanced `ggtree`, `ggtreeExtra`, `treeio` and `phytools` routes are described in the phylogeny atlas; check actual versions before adapting them.
3. **Python large trees** — retain the separately reviewed `scripts/visualize_microbiome.py` for full identity annotation and readable detail pages. Its stricter contract is below. It is not a general table renderer.
4. **Advanced recipes/GUI** — use the atlas and software guide for tool-specific adapters, keeping source files, transformations, code/session or project files and a local export. A documented recipe is not a runtime certification. Recheck installed packages; do not upload unpublished trees or sample tables to hosted services without authorization.
5. **Functional aggregation and contig views** — `scripts/aggregate_gene_functions.py` and `scripts/plot_contig_functions.R` provide narrowly scoped local adapters; use their exact contracts and preserve the chosen membership/allocation policy. They do not annotate sequences, reconstruct metabolic activity or resolve ambiguous locus abundance.
6. **WGCNA** — `scripts/wgcna_analysis.R` separates preflight, power diagnostics and model fitting. Read the analysis guide first; missing packages or insufficient independent samples must be reported explicitly. Actual network fitting is a statistical analysis, not merely a plotting option.

Use [references/figure-contract.md](references/figure-contract.md) for the input schema, CLI, and validation states. The bundled `scripts/visualize_microbiome.py` provides a deterministic renderer:

- Finite overview for 1,000+ tips, all native branches retained; readable leaf windows in PDF/SVG/PNG, exact genome index and branch-support table.
- Group/sample/phylum colors, complete wrapped identity labels, explicit missing values, and validated membership declarations for identical-alignment clusters. Only declared groups with more than one member are marked unresolved; singleton and unknown-size groups are separate. It does not establish sequence or biological identity.
- Completeness/contamination scatter and group genome counts. Counts are genomes, never inferred biological replicates; no significance testing is performed.
- Full source and output hashes, software/script versions, HTML navigation, and separate visual QA status.

Never replace or alter a running/frozen scientific pipeline to improve a figure. Render into a new isolated output directory. Preserve native topology, leaf identities, branch lengths and numeric supports. Read support order from actual source evidence: SH-aLRT, UFBoot and sCF are different measures. This renderer accepts ML support labels only; keep independently audited sCF in a separately labeled branch table or use a specifically reviewed sCF view. Missing support and zero support are different.

Render as `preview`. Open the overview, legend/quality panel, first/middle/last detail pages and the densest support page. Inspect at readable size, checking label overflow, leaders, low/zero/missing support, missing metadata, color key consistency, and exact identity coverage. Record actual inspected files and limitations. Only then may `accept-qa` mark the visualization `validated`; that status does not certify source biology or authorize production. Synthetic previews remain prominently labeled synthetic after QA.

For a new data type or plotting requirement outside these implementations, adapt the relevant recipe and the same source/identity/QA contract in an isolated script. Keep plotting-only work distinct from statistical analysis, distance calculation, topology inference, network estimation or new scientific filtering. Those operations need the corresponding analysis design and provenance; a beautiful figure does not authorize or validate them.
