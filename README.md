# Mesengenic AI

**A target-agnostic generative agent for epistatic discovery in protein fitness landscapes.**

Navigating the evolutionary latent space to identify the next generation of therapeutic agents.

Built for the Advance Health Hackathon, 2026.

---

## The problem

**Finding, not founding.** Drug discovery today uses magnifying glasses rather than pens. Big pharma spends around $2.6 billion per drug searching for biosimilar molecules, accumulating mutational debt along the way. Intent is decoupled from execution: synthesis hardware sits idle because human intuition can only discover at the speed of a lab bench.

**A ceiling rather than a floor.** Nature is exhaustive, and the solutions to functions new to nature lie in the latent space. A better shovel is still a shovel.

**Friction and R&D waste.** High-throughput screening is slow, and the pipeline is fragmented across identification, design, structural prediction, physics validation and synthesis.

Underneath all of this sits a modelling problem. Most AI models for protein engineering assume a smooth, additive fitness landscape. Biology is epistatic, which means the effect of one mutation depends on the others around it. Two failure modes follow:

- **Hallucination.** Neural networks generate protein variants that are stoichiometrically impossible, acting as structure engines rather than synthesists.
- **Catastrophic forgetting.** As networks scale, they lose previously learned patterns and forget what makes a protein viable.

The protein fitness landscape contains over a billion possible variants, but only thousands can be tested via high-throughput lab screening. That sampling gap is a major reason why 90% of drug candidates fail.

## The approach

We treat millions of years of evolution as a mathematical prior. A variational autoencoder learns the hidden grammar of a protein family and maps its fitness landscape into a low-dimensional latent space. Because every protein in the prior has been proven to exist, the space is stoichiometrically valid, and the prior prevents hallucination.

Three features of the landscape matter:

| Feature | What it means |
|---|---|
| **High density** | Regions evolution has heavily explored. Exploitation zones. |
| **Epistatic peaks** | High-fitness variants between explored regions. Biologically unsampled but mathematically probable. |
| **Frustrated gaps** | Low density with high indicated fitness. The exploration frontier. |

## The platform

Not a tool, but an intelligence layer for biology. Four components:

| Component | What it does |
|---|---|
| **Mesengenic OS** | The intelligence layer for biology |
| **Mesengenic Suite** (Terminal + Lambda) | A natural language interface for labs |
| **Connect API**, with Radar and Sisyphus validation | High-confidence designs for partners, and printability for users |
| **Mesengenic Treasury** | Latent IP notarisation and sovereign royalty |

### Integrated intelligence pipeline

Requests arrive from Terminal-V (Streamlit UI on Vercel Edge) or Terminal-C (local CLI) and are routed by the Mesengenic API Gateway. Inference runs on Mesengenic Lambda (RunPod serverless A100s); core VAE training runs in The Forge on AWS EC2 P6 instances (H200s), orchestrated with AWS SageMaker HyperPod.

Generated candidates pass through a verification loop before they go anywhere:

1. **Radar**, a forensic auditor scoring structural confidence (pLDDT), applies the hubris filter.
2. **Sisyphus**, a molecular dynamics stress simulator, tests stability. Failures return latent coordinates to the generative engine.
3. Surviving candidates become sovereign candidates and cross **Atlas Connect**, the API bridge, as a SHA-256 hash to the Sovereign Treasury (PostgreSQL on secure AWS RDS) and a JSON payload to the physical synthesis partner.
4. Experimental yield data flows back into The Forge as a reality feedback loop, driving gradient weight updates to **Sophocles**, the generative engine behind VAE v2 and v3.

## Prototype: TEM-1 β-lactamase

TEM-1 is a surface protein found in antibiotic-resistant bacteria and a gold-standard protein ML benchmark, with well-characterised experimental fitness data from real lab assays. Antimicrobial resistance costs an estimated $66 billion globally each year.

**Input and processing**

- 10,129 evolutionary sequences from the beta-lactamase family (Pfam PF13354)
- 254 core aligned positions retained after occupancy filtering
- 40 training epochs, 2-dimensional latent space, β = 4.0

**Results**

- Model error reduced from 689 to 377.6
- 17 epistatic peak candidate regions identified in the fitness landscape
- 38 high-variability positions mapped onto the 3D crystal structure (PDB 1BTL)

## The dashboard

`mesengenic_vae_dashboard.html` is a self-contained interactive dashboard with no build step. Open it in any modern browser.

It contains five linked views:

1. **Evolutionary latent manifold.** A density heatmap of the learned fitness landscape with all 10,129 sequences plotted by latent coordinate and coloured by normalised fitness. Epistatic peak candidates are marked as diamonds.
2. **3D structure (PDB 1BTL).** An interactive, rotatable Cα backbone of TEM-1, colouring the active site region and the high-entropy positions against the conserved backbone. Drag to rotate, scroll to zoom.
3. **Training dynamics.** Total ELBO loss alongside its categorical cross-entropy reconstruction term and the β-weighted KL divergence term, across all 40 epochs.
4. **Positional entropy.** Per-position entropy across the alignment, highlighting the 38 variable positions.
5. **Epistatic peak candidates.** A sortable table of peak coordinates with score and density.

## Model architecture

```
Protein ──encode──> Latent manifold ──decode──> De novo protein
                   (fitness landscape)
                            ^
                            └── loss function, weight adjustment
```

**Loss function.** Categorical cross-entropy regulates reconstruction accuracy, since each decoder output is a probability distribution over 21 amino acid categories via one-hot encoding.

**β · KL divergence** controls how exploratory versus exploitative the model is. A higher β pushes the model further into probable but unsampled regions.

## Tech stack

**Model:** PyTorch, NumPy, SciPy, PHATE, Torch Geometric
**Visualisation:** Plotly, Three.js, Streamlit
**Infrastructure:** AWS EC2 (H200), SageMaker HyperPod, RunPod serverless (A100), PostgreSQL on AWS RDS, Vercel Edge
**Data:** Pfam PF13354, PDB 1BTL, MaveDB, UniProtKB, InterPro

## Running the dashboard

The dashboard is a single HTML file with all data embedded and both libraries loaded from CDN. No installation is needed.

```bash
# clone the repository, then simply open the file
open mesengenic_vae_dashboard.html      # macOS
start mesengenic_vae_dashboard.html     # Windows
```

An internet connection is required on first load so the browser can fetch Plotly and Three.js.

## Impact

**Responsible by design**

- Public data only: MaveDB, UniProtKB, Pfam and InterPro. No patient data.
- Decision support, not autopilot. The model prioritises; the scientist decides.
- Confidence before action. Candidates are ranked and filtered before any wet-lab spend, and every sequence carries a confidence score. When the model is wrong, the failure retrains the model and strengthens its hyperparameterisation; it never reaches a patient.

**Research and industry**

- $4,499.86 saved per candidate: $0.14 of cloud validation against a $4,500 wet-lab assay
- Search space compressed from 10⁹ to roughly 1,000 high-confidence candidates
- 3 minutes 47 seconds of triage in place of months of iteration

**Clinical and research**

- Stronger candidate selection in high-need areas: antimicrobial resistance, cancer pathways, gene therapy and autoimmune targets
- More time on viable biology, less on experimental dead ends
- A clearer bridge from model output to testable wet-lab next steps

## Competitive landscape

| | AlphaFold 3 / Chai-1 | Cradle-1 / Xaira | Mesengenic AI |
|---|---|---|---|
| **Capability** | Structural prediction: how does this sequence fold? | Property optimisation: make this 10% more stable | Generative discovery of viable variants in unsampled regions |
| **Primary logic** | Static, relying on known PDB structures | Wet-lab iterative, requiring 96-well plate data to start | Evolutionary, using manifold density priors |
| **Data basis** | A research tool | Traditional, with slow manual filings | Native, with an integrated sovereign treasury |
| **IP handling** | A digital coordinate map | An optimised lab candidate | A notarised, time-stamped asset |

## Roadmap

- **Validation.** Spearman ρ correlation against experimental MaveDB fitness scores, closing the loop between computational prediction and laboratory results, and establishing the software as a benchmarking tool.
- **VAE v2 and v3 (Sophocles).** Explore stable abiological manifolds, meaning physical conformations that are permissible but unsampled, and add causal inference to expand the model's synthetic capacity.
- **Partnerships.** Work with R&D leads and wet-lab collaborators to validate predicted peaks in vitro, which would be the first experimental confirmation of computationally discovered epistatic variants.

## Market context

The AI drug discovery market stood at $4.6 billion in 2025 and is projected to reach $49.5 billion by 2034 at 30% CAGR (Global Market Insights, 2025). Four customer groups sit closest to the work: clinical trial labs needing library reduction, synthetic biomanufacturers designing optimised enzymes, pharma R&D teams derisking lead optimisation, and biotech startups and CROs seeking platform scalability.

## Vision

Shift the industry from searching for drugs to designing solutions: compress the optimisation phase that kills 90% of candidates, and turn evolutionary white space into a pipeline of novel, patentable therapeutics.

Feed the hardware. Fortify the pipeline. Fund the future.

---

© 2026 Mesengenic AI
