# Exercise improvements — WT 2026/27

## Structure: one folder per lecture, one notebook per exercise

| Lecture | Folder | Student notebook (`Assignment_<name>/`) | Solution (`Solution_<name>/`) | CPU runtime |
|---|---|---|---|---|
| 1 Data evolution | `materials_apis/` | `materials_apis_student.ipynb` | `materials_apis_solution.ipynb` | ~2–3 min (network) |
| 2 Fingerprints | `fingerprints/` | `fingerprints_student.ipynb` | `fingerprints_solution.ipynb` | ~2 min |
| 3 GNNs | `gnns/` | `gnn_student.ipynb` (+ `parity_plot.png`) | `gnn_solution.ipynb` | ~8 min |
| 4 Atomistic simulations | `atomistic_simulations/` | `ase_student.ipynb` (+ `PE_chains_POSCAR`) | `ase_solution.ipynb` | unchanged |

- Each student works on their own copy of the student notebook.
- **Every notebook starts with an AI usage statement** (no university template was available, so it is our own
  wording: no AI / AI used + table of tool, purpose, tasks, how checked + confirmation of understanding).
- **No notebook contains outputs** (students and solutions). Images that are part of the instructions (band-gap
  sketch, expected histogram) are markdown attachments and stay.
- Solution notebooks contain instructor answers to all discussion questions (cells marked **Answer (instructor)**).
  All solution notebooks were executed top to bottom without errors before the outputs were cleared; the numbers in
  the answers come from these runs.
- `materials_apis/data/` holds cached API results (offline fallback, no API key inside).
- The README describes the exercises by lecture and how students get their own Materials Project API key.
- The ASE notebooks were only moved and got the AI statement; their content is otherwise unchanged.

## 1 · Materials databases (`materials_apis/`)

NOMAD (Part A) and Materials Project (Part B) in one notebook; content as before, sections renamed A1–A4 / B0–B4.
Two Materials Project variables were renamed (`ti_o_entries`, `MP_FIELDS`) so that they do not overwrite Part A
variables of the same name.

- **Part A — NOMAD** (no key): count queries, aggregations, pagination, archive query for band gaps (stored in
  joule; spin channels). Uses the REST API via `requests`, because `nomad-lab`'s `ArchiveQuery` (1.4.3) crashes on
  current server data and is rate-limited (HTTP 429). Findings used in the questions: Gaussian and VASP dominate the
  O-containing entries ("contains O" ≠ "oxide"); 97/100 band-gap entries are spin-polarised and taking only the first
  channel is wrong for 4 of them.
- **Part B — Materials Project** (own key, or `USE_CACHED = True`): uses `mp_api.client.MPRester`. `mp-2657` comes
  back in a non-standard 12-atom cell → students convert to the conventional cell (a = 4.600, c = 2.959 Å). DFT ranks
  anatase below rutile (9.5 meV/atom) — opposite to experiment; computed gaps 1.77 / 2.06 eV vs exp. 3.0 / 3.2 eV;
  Ti–O hull mixes r2SCAN and GGA entries; Ti₄O₇ (a real Magnéli phase) is 8 meV/atom above the hull; 1 453 stable
  oxides with a computed gap of 2–4 eV.
- Technical: `pymatgen` cannot serialise the entries returned by `mp_api` (`TypeError: keys must be str … not
  Element`); the notebook strips `entry.data` before caching.

## 2 · Polymer data and fingerprints (`fingerprints/`) — new

The data handling of the original GNN exercise, extended with fingerprints. Contains the original data-loading task
(text, TODO list, expected-output image).

| Section | Content | Result in the executed solution |
|---|---|---|
| 1 | Load the dataset, PSMILES, histogram; draw lowest/highest band-gap polymers | 3 380 `Egc` entries |
| 2 | Data quality: elements outside the one-hot list; entries with net charge; duplicates via `psmiles` canonicalisation | I (8) and H (54) not in the list; 12 charged `[N+]` entries (mean gap 0.61 vs 4.54 eV) removed → 3 368; one polymer listed twice with 2.33 / 2.46 eV |
| 3 | Morgan fingerprints, Tanimoto similarity | `[*]C[*]` vs `[*]CC[*]`: 0.50 (1.00 after canonicalisation); polystyrene's neighbours span 2.0–5.2 eV |
| 4 | 70/15/15 split, seed 42 (same split as the GNN notebook) | 2 357 / 505 / 506; both duplicate copies in training |
| 5 | Random forest on fingerprints; radius chosen on the validation set | test MAE 0.344 eV (mean predictor 1.304); radius 1 best on validation → test 0.331 eV |
| 6.1 | Multi-component fingerprints: MOFs (concatenate node + linker Morgan fingerprints, boss-requested) | MOF-5-like node+linker vector, length 2048 (3 nonzero bits node / 12 linker); swapping the linker (BDC → BPDC) leaves the node half identical and only changes the linker half |
| 6.2 | Atomistic (3D) fingerprints for crystals: ACSF $G_2$ on FCC Al/Cu and BCC Fe via `ase.build.bulk` + `ase.neighborlist` (boss-requested) | rotation-invariant (exact); unweighted $G_2$ cannot separate Cu (fcc) from Fe (bcc) — distance 0.014 vs Al-Cu/Al-Fe ≈ 2.8 — because their nearest-neighbour distances coincide (2.556 vs 2.486 Å); atomic-number weighting ($G_{2,w}$) separates them (distance 24.7) |
| 6.3 | Discussion: the lecture's own closing prompt (concrete, wood, carbon fibre, glass, aluminium), reused as a wrap-up | — |

**polyBERT — removed per boss feedback (30 Oct 2026 PR comment): "remove the optional polyBERT part, they have no
access to it."** The Section 6 polyBERT task (and its mentions in the GNN notebook's final discussion) were deleted;
`kuelumbus/polyBERT` still returns HTTP 401 on Hugging Face, so this closes that open question from the previous
round. The freed Section 6 slot now holds the MOF and atomistic-fingerprint material the boss asked to add instead
(see lecture 2 slides on multi-component and 3D fingerprints).

## 3 · Graph neural networks (`gnns/`)

One notebook. The data is loaded with the same cleaning and split as in the fingerprints notebook (given code).

**Part A — basics** (original notebook, Sections 1–7): graphs from PSMILES, receptive field, DataLoaders (the split
itself is given), GCN, training, evaluation, mean-predictor baseline, predictions for well-known polymers.
Result: test MAE 0.322 eV (R² 0.92; mean predictor 1.304 eV). `[*]C[*]` and `[*]CC[*]` (both polyethylene) get
6.84 vs 8.00 eV → motivates Part B.

**Part B — advanced** (B0–B5): functions that Part B redefines are renamed (`atom_features_b`, `smiles_to_graph_b`,
`predict_graphs`, `mae_of`, …) so that Part A cells keep working when re-run.

| Section | Concept | Result |
|---|---|---|
| B1 | "Other" element slot; bond features as `edge_attr` | 26 atom / 6 bond features |
| B2 | GCN vs GIN vs GINE | 0.342 / 0.364 / 0.349 eV — within seed noise (≈ 0.01 eV) |
| B3 | Periodic polymer graphs (repeat-unit invariance) | exactly invariant for PE/PS/PTFE (star-atom model up to 0.8 eV off) |
| B4 | Sulfur hold-out (out-of-distribution) | 0.40 → 0.88 eV; mean predictor 1.32 → 1.67 eV |
| B5 | Deep ensemble | σ 2.6× larger out of distribution but far below the error |

The fingerprint random forest (0.33–0.34 eV) is as accurate as the GNNs (0.32–0.36 eV) on the same test set; the
final questions discuss when a GNN is worth it.

## Suggestions for the ASE exercise (not changed)

- `ExpCellFilter` is deprecated in ASE 3.29 ("Use FrechetCellFilter for better convergence").
- Student notebook: tasks are unnumbered ("### Task") but referred to by number ("… from Task 3").
- Runtime of Section 7 is given as "≈ 5 min" (student) and "20–40 min" (solution).

## Ideas not yet implemented

- Link the exercises: relax rutile and anatase with MACE in the ASE exercise and compare with the Materials Project
  ranking from exercise 1; point out that MACE is itself a message-passing GNN.
- A common header for all notebooks (objectives, prerequisites, time) and a "what's next" box.
