# R* Calculator

A single-file web tool that walks an operator through an **R\*** dataset vulnerability assessment and produces a reproducible triage score with a recommended action. No build step, no dependencies, no server. Open the HTML file and score.

Live tool: **https://nyce-data.github.io/rstar-calculator/**

---

## What this is

The R\* Composite Priority Index is a rapid-assessment vulnerability index built for crisis conditions. It assumes institutional instability rather than steady-state operations, and it explicitly accounts for the human expertise dimension of data risk that most frameworks miss. It's one of three frameworks in an ensemble approach to climate data preservation:

- [**UReDI**](https://nyce-data.github.io/uredi-calculator/) answers *how much a dataset is worth and how hard it would be to recover.*
- **R\*** answers *how likely we are to lose it.*
- The Keeling Curve Foundation's **Long-Term Observations** inventory answers *which measurement series must keep running.*

Value, risk, and continuity are three different questions. Run together they catch what any one framework alone would miss.

This calculator is the companion to the [UReDI Calculator](https://nyce-data.github.io/uredi-calculator/) and follows the same single-file, zero-dependency design.

## What it does

The tool is built for operators working under time pressure with incomplete information. Someone who is not a climate data specialist should be able to run a defensible R\* assessment by working through four short checklists, each item a concrete condition that's either present or absent.

R\* is deliberately a triage tool, not an audit instrument. It's designed for the moment in early 2025 that created it: when reductions-in-force hit NOAA, NASA, and DOE in rapid succession and nobody could reliably say which datasets were about to break. The framework answers one question well: what needs to be mirrored right now?

The design goal is consistency:

1. **Repeat-run consistency.** The same operator scoring the same dataset twice should land in the same place.
2. **Cross-operator consistency.** Two operators scoring the same dataset should agree on the action tier, even if the exact scores differ by a few points.

Binary conditions (is there a documented succession plan? yes or no) produce more reproducible scores than subjective sliders. The judgment that checklists can't capture goes into a free-text justification box on every variable, so the reasoning travels with the score.

## The R\* framework

R\* is a weighted composite of four risk terms, each scored 0 to 100 and then weighted. The formula puts the heaviest weight on the most direct signal of imminent loss and the lightest on the hardest dimension to score precisely.

```
R* = 0.40·D + 0.30·P + 0.20·(1 - RRS) + 0.10·DIRS
```

**D (Data Risk), weight 0.40.** Probability of loss through technical failure, policy disruption, or infrastructure degradation. Five conditions: active program termination or defunding (30), storage system end-of-life (25), fragile or unfunded access infrastructure (20), format obsolescence (15), political sensitivity (10). This measures how likely the data is to be lost, not how valuable it is. Value lives on the UReDI axis.

**P (People Risk), weight 0.30.** The vulnerability created when critical institutional knowledge resides in a small number of individuals. Four conditions: no documented succession plan (35), key personnel retirement-eligible within two years (25), critical undocumented knowledge (25), single person as point of failure (15). This is the novel contribution of R\*. A dataset can technically exist while becoming operationally inaccessible the day its only expert walks out the door.

**RRS (Repository Resilience Score), weight 0.20, inverted.** The robustness of the hosting institution's data infrastructure. Because higher resilience means lower risk, it enters the formula inverted: a strong repository reduces R\*, a fragile one raises it. RRS is the one term with a two-tier structure. If the hosting repository has completed the formal Repository Resilience Scorecard from the Repository Crisis Scorecards project, that score is used directly. Otherwise a five-condition rapid checklist applies: geographic redundancy (25), regular backup verification (20), tested disaster recovery (20), stable institutional funding (20), track record through prior disruption (15).

**DIRS (Dependent Infrastructure Resilience Score), weight 0.10.** The invisible dependency layer: external infrastructure a dataset depends on but doesn't control. Five conditions: calibration facility dependency (25), observation network or sensor platform at risk (25), upstream data supply chain at risk (20), external processing infrastructure dependency (15), no alternative supplier (15). When a calibration lab loses funding, every dataset whose quality assurance depends on it becomes more fragile immediately, regardless of its own hosting situation.

### Scoring

Each variable is a binary additive checklist. Check the conditions that apply, sum the points, and the variable scores 0 to 100. The composite applies the weights, with RRS inverted:

```
D_contribution    = D × 0.40
P_contribution    = P × 0.30
RRS_contribution  = (1 - RRS/100) × 0.20 × 100
DIRS_contribution = DIRS × 0.10
R*                = sum of the four contributions
```

The RRS inversion is the one piece that surprises people. A repository scoring 80 on resilience contributes only (1 - 0.80) × 0.20 × 100 = 4 points to R\*. A repository scoring 0 contributes the full 20. An unknown repository is not a safe repository.

### Triage tiers

The thresholds map scores directly to action. They were calibrated against the framework's interview data and case applications, including the SEDAC program termination event, where R\* jumped from 49 to 89 when funding was discontinued and the framework correctly called for immediate mirroring.

| Tier | R\* | Action |
|------|-----|--------|
| Mirror now | 70 to 100 | Immediate multi-replica preservation; daily sync; 3+ geographic replicas |
| Stabilize & mirror | 40 to 69 | Harden repository governance; weekly sync; engage backup curatorial partners |
| Monitor | 0 to 39 | Metadata harvesting; monthly synchronization checks |

The threshold structure is intentionally asymmetric. The "Mirror now" tier casts a wide net so a high-risk dataset doesn't slip through, accepting that it will occasionally over-mobilize for a dataset that turns out to be less fragile than the initial read suggested. In a crisis, false negatives cost far more than false positives.

## Using the tool

1. Open the tool. The triage action matrix sits up top so the tiers are visible before you start. All four variable sections start collapsed.
2. **Dataset & curator.** Record the dataset name, its persistent identifier, and the primary data curator with email. The curator is the "P person," the human whose departure the People Risk term is built to detect.
3. **Score the four variables.** Open each section, read the definition, and check the conditions that apply. Use the justification box to record the reasoning behind your selections and any situation-specific factors the checklist doesn't cover.
4. For **RRS**, choose the tier first. If a formal Repository Resilience Scorecard exists, enter that score. Otherwise work the rapid checklist.
5. Read the **R\* composite, tier, and per-variable contributions** in the results panel as you go. The recommended action appears with the score.
6. **Export** the finished assessment as JSON, copy it to the clipboard, or print to PDF. Every export is timestamped and carries the per-variable justifications.

## Output

The JSON export is the canonical record of an assessment. It captures the dataset and curator metadata, the R\* composite and triage tier, every variable's raw score, weight, weighted contribution, scoring method, and the per-variable justification text, plus the full framework attribution. The structured format is what makes later cross-operator and cross-run analysis possible.

## Deployment

This is a single, self-contained HTML file. To deploy on GitHub Pages, place the app at the repository root as `index.html` and enable Pages for the `main` branch. No build pipeline is required.

```
rstar-calculator/
├── index.html      # the application (single file)
└── README.md
```

## Project status

Working build, under active road-testing and iteration. Known items for upcoming passes:

- **Blank-form score.** A fresh form shows R\* = 20 because an unscored repository contributes the full inverted RRS weight. The next build suppresses the score until the first condition is checked, so a blank form reads as "not yet assessed" rather than "scored 20."
- **Conditions as questions.** The checklist items are currently declarative conditions. They'll be reworded as direct yes/no questions to read more naturally in a field assessment.
- **Per-component evidence quality.** A score of 58 from solid evidence is operationally different from 58 from guesswork. An upcoming pass adds a verified / inferred / unknown selector to each variable so uncertainty is machine-readable.
- **Version string.** The footer version label is being synchronized with the companion Knowledge Graph template.

## Related

- [UReDI Calculator](https://nyce-data.github.io/uredi-calculator/): the value framework in the ensemble
- The Ensemble Framework for Climate Data Preservation, which sets out how R\*, UReDI, and the Keeling Curve Foundation's Long-Term Observations inventory work together

## Attribution

- **R\* Composite Priority Index:** Steve Diggs
- **UReDI framework (v3):** Thomas Karl
- **Repository Resilience Scorecard (RRS):** Repository Crisis Scorecards project, based on Schuster, Mayernik, Mullendore & Marquis (2023), *Bull. Amer. Meteor. Soc.*, 104, E2053-E2064 (doi:10.5281/zenodo.15208172)
- **Long-Term Observations inventory:** Keeling Curve Foundation
- **Tool design and implementation:** Steve Diggs

## License

TBD. The tool code will likely be released under MIT or a similar permissive license. The R\* framework itself is attributable to Steve Diggs, and the Repository Resilience Scorecard to the Repository Crisis Scorecards project.
