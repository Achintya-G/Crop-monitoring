# Project Plan: Crop Type, Moisture Stress & Irrigation Advisory (Learning Edition)

**Team size:** 4 | **Experience:** Mostly beginners | **Timeline:** 3 weekends (+ 1 short prep week) | **Goal:** Learn the tools and learn to work as a team — a working (if rough) pipeline is the bonus, not the bar.

---

## 1. Why This Scope Is Different From the Original Proposal

The source proposal assumes a 30-hour hackathon with a pre-defined pilot area, pre-identified crop labels, and a team that already knows GEE, SAR processing, and ML. None of that applies to you yet. Trying to build all three objectives (crop type + moisture stress + irrigation advisory) at full depth in 3 weekends, from zero, will most likely produce four tired people and no working pipeline.

So this plan **deliberately descopes** the project into a smaller vertical slice that still touches every stage of the original architecture diagram — Data → Crop Type → Phenology/Moisture Stress → Irrigation Advisory — just shallower at each stage. You can go deeper later once the tools feel familiar.

| Original hackathon scope | This 3-weekend version |
| --- | --- |
| National/operational relevance, multiple crops, multiple seasons | One small pilot area, 2–3 crop classes (e.g., "paddy / other crop / bare-fallow") |
| Deep learning (LSTM, U-Net) for classification | Random Forest only (fast, interpretable, beginner-friendly) |
| Rigorous accuracy validation (>85%, Kappa) | Simple train/test split + confusion matrix, just to learn the workflow |
| Full crop-water-balance irrigation model | A simplified rule-based advisory (e.g., NDVI/NDWI anomaly + rainfall shortfall → "stressed / okay") |
| Dashboard-ready output package | A handful of static/interactive maps + one summary notebook or simple web page |
| Ground-truth field data | Self-labeled samples from visual inspection + published crop calendars (see Section 3) |

---

## 2. Recommended Pilot Area & Ground Truth Approach

You don't have a study area or ground truth yet, so here's a concrete way to get one this weekend instead of losing days to indecision.

**Pick a small, well-documented command area, not a huge or obscure one.** Two ways teams commonly do this in the literature you'd be extending:

- Academic GEE-based studies have used small, well-known pilot areas like the **Ramappa command area** (Telangana) for Sentinel-1/2 cropping-pattern change detection, and the **IARI experimental farm** (New Delhi) for crop classification with ~100 known fields.
- You don't need something that famous — any **small (5–20 km²) irrigated command area or cluster of fields near a canal**, ideally somewhere a team member has local knowledge of what's typically grown (helps with labeling sanity-checks), works fine.

**For ground truth, since real field surveys aren't realistic in 3 weekends, use a "beginner-honest" self-labeling method:**

1. Open the area in Google Earth / Bhuvan high-resolution basemap imagery.
2. Cross-reference with the regional **Kharif/Rabi crop calendar** (published by state agriculture departments or ICAR) to know what's plausible for the season you're imaging.
3. Manually mark 40–80 sample points/polygons per class by visual field texture and seasonal greening pattern (e.g., flooded/puddled fields in June–July = likely paddy).
4. Label clearly in your data as **"visually-labeled, not field-verified"** — this is a legitimate limitation to state, not something to hide. It's completely fine for a learning project.

**Data sources to use (confirmed current as of your project date):**

- **Sentinel-1 (SAR)** and **Sentinel-2 (optical)** — both available directly inside Google Earth Engine's public data catalog (`COPERNICUS/S2_SR_HARMONIZED`, `COPERNICUS/S1_GRD`), no separate download needed.
- **ISRO optical data (LISS-III/AWiFS/LISS-IV)** — available via **Bhoonidhi** (bhoonidhi.nrsc.gov.in), ISRO/NRSC's open data portal; sign-up required, some products are instant-download, others delayed. Use this only if you want the "Indian satellite" angle — Sentinel-2 alone is enough to learn the pipeline.
- **Rainfall / reference ET** — CHIRPS or ERA5 datasets, also available inside GEE, no separate download.

**Decision point for your team:** confirm your pilot area and season (which Kharif/Rabi season, which state/district) in your first working session — don't let this drag past day 1.

---

## 3. Tools & Environment Setup

Since a goal here is getting comfortable with the tools and with working as a team, set these up together, not solo:

| Category | Tool | Notes |
| --- | --- | --- |
| Satellite processing | **Google Earth Engine** (Code Editor + Python API) | Free account, sign up at code.earthengine.google.com |
| Notebooks / compute | **Google Colab** | Shared, free GPU/CPU, no local setup pain |
| Version control | **GitHub** (one shared repo) | Use branches per person, PRs to merge — this alone is a great team-workflow exercise |
| Task tracking | **GitHub Projects** or **Trello** | Simple kanban: To Do / In Progress / Done |
| Communication | **Discord or WhatsApp group + a weekly 15-min call** | Async updates + one sync check-in per weekend |
| GIS/visual QA | **QGIS** (free) | For quickly eyeballing outputs, sanity-checking labels |
| Ground truth data source | **Bhoonidhi**, Google Earth high-res imagery, state crop calendars | See Section 2 |

**Repo structure suggestion:**

```text
crop-monitoring/
  data/            # AOI boundary, ground-truth points, raw exports
  notebooks/        # one folder per person/role
  src/              # shared reusable functions (indices, plotting, etc.)
  outputs/          # maps, figures, final deliverables
  docs/             # this plan, meeting notes, README
```

---

## 4. Team Roles (4 people)

Roles map directly onto the four stages of the original architecture diagram, so everyone owns one clean vertical slice and can see how it plugs into the others.

### Role 1 — Data & Preprocessing Lead

**Owns:** Data Sources → Analysis Ready Data

- Set up GEE access for the whole team; define and share the AOI (area of interest) geometry
- Pull Sentinel-2 (optical) and Sentinel-1 (SAR) collections for the pilot area/season
- Cloud-masking (optical), basic speckle handling (SAR), temporal compositing (weekly/fortnightly)
- Produce a clean "Analysis Ready Data" export/asset that the rest of the team builds on
- **Learning focus:** GEE fundamentals, ImageCollections, filtering/compositing, optical vs. SAR data quirks

### Role 2 — Crop Classification / ML Lead

**Owns:** Feature Extraction → Crop Type Classification Model

- Compute vegetation indices (NDVI, NDWI, EVI) and basic SAR features (VV, VH, VH/VV ratio)
- Assemble the self-labeled ground truth samples (working with Role 1's AOI) into a training table
- Train a Random Forest classifier (GEE's built-in classifier or scikit-learn), generate the crop-type map
- Validate: simple train/test split, confusion matrix, overall accuracy
- **Learning focus:** feature engineering for remote sensing, supervised classification, basic model evaluation

### Role 3 — Phenology & Moisture Stress Lead

**Owns:** AI-Crop Phenology → Moisture Stress Detection Model

- Build simple phenology markers from the NDVI time series (start of season, peak greenness) for the pilot fields
- Compute a stress index (e.g., a simplified Vegetation Condition Index from NDVI anomalies vs. a baseline) and, if time allows, a basic SAR-backscatter-based moisture signal
- Classify stress into simple bins (e.g., low/moderate/high) per growth stage
- **Learning focus:** time-series thinking, anomaly detection, translating an index into an interpretable category

### Role 4 — Irrigation Advisory, Integration & Team Lead

**Owns:** Meteorological/Ancillary Data → Irrigation Advisory Maps, plus overall integration

- Pull rainfall/ET data for the AOI; build a simplified water-deficit rule (e.g., stress index + rainfall shortfall → "needs irrigation" flag)
- Combine Role 1–3's outputs into a final map/notebook/simple dashboard (even a well-organized Colab notebook with maps and a short write-up counts)
- Runs the lightweight team process: keeps the task board updated, runs the weekly 15-min sync, tracks what's blocking whom
- **Learning focus:** combining heterogeneous data sources, basic decision-rule design, project coordination

**Cross-training note:** because everyone's a beginner, do the first onboarding session (Section 6, Prep Week) *together* so nobody is stuck alone learning GEE syntax while others move ahead. After that, roles specialize but pair up for at least one hour each weekend on someone else's piece — that's where a lot of the team-workflow learning happens.

---

## 5. Timeline: Prep Week + 3 Weekends

### Prep Week (before Weekend 1) — ~2–3 hrs, individually or one shared session

- [ ] Everyone: create GEE account, verify access
- [ ] Everyone: run through one short official GEE "Get Started" tutorial (Code Editor + one Python API example)
- [ ] Team: 30-min call — confirm pilot area candidate, target season, and repo setup
- [ ] Role 4: create GitHub repo + Trello/Projects board with the task list from Section 7

### Weekend 1 — Foundation: Data In, Area Confirmed

| Day | Role 1 | Role 2 | Role 3 | Role 4 |
| --- | --- | --- | --- | --- |
| Sat | Finalize AOI polygon; pull Sentinel-2 & Sentinel-1 collections; first cloud-mask pass | Research/shortlist crop classes for the area; draft labeling approach | Explore NDVI time series API calls on a test point | Set up rainfall/ET data pull for AOI; finalize repo structure |
| Sun | Produce first Analysis-Ready composite (weekly/fortnightly); share as GEE asset | Start manually labeling 40–80 ground-truth points via Google Earth imagery + crop calendar | Sketch what "moisture stress index" will look like conceptually; review literature | Team sync (15–30 min): confirm AOI locked, review blockers, adjust board |

**End of Weekend 1 checkpoint:** AOI locked, ARD composite exists, ground-truth labeling underway.

### Weekend 2 — Core Build: Classify & Detect Stress

| Day | Role 1 | Role 2 | Role 3 | Role 4 |
| --- | --- | --- | --- | --- |
| Sat | Support Role 2/3 with data access issues; add SAR features (VV, VH, ratio) to the stack | Finish ground-truth labels; compute NDVI/NDWI/SAR features per sample; train first Random Forest | Compute NDVI-based phenology markers (SOS, peak) for sample fields | Pull historical rainfall baseline; start drafting the simple water-deficit rule |
| Sun | QA the feature stack in QGIS (sanity-check composites visually) | Generate first crop-type map; run train/test split + confusion matrix | Build the stress index; classify stress into low/moderate/high per growth stage | Team sync: review crop map + stress index together, flag issues |

**End of Weekend 2 checkpoint:** working crop-type map (even if rough accuracy), working stress index — the two hardest technical pieces exist.

### Weekend 3 — Integration, Advisory Logic, Wrap-Up

| Day | Role 1 | Role 2 | Role 3 | Role 4 |
| --- | --- | --- | --- | --- |
| Sat | Help wherever needed; clean up/document the data pipeline notebook | Refine classifier if time allows; write up accuracy results | Refine stress classification; help translate into advisory logic | Combine stress index + rainfall shortfall into the irrigation advisory rule; generate advisory map |
| Sun | Full team: assemble final notebook/README with maps, findings, and honest limitations (ground truth caveat, small AOI, simplified model). Do a 20-min internal "demo" to each other. | | | |

**End of Weekend 3 checkpoint:** one coherent notebook/repo that walks Data → Crop Map → Stress Map → Advisory Map, with a short written summary of what worked, what didn't, and what you'd do with more time.

---

## 6. Task Checklist by Phase

### Phase 1 — Foundation

- [ ] GEE + Colab + GitHub set up for all 4 members
- [ ] AOI polygon defined and shared
- [ ] Sentinel-2 & Sentinel-1 collections pulled for target season
- [ ] Cloud-masking / basic SAR handling applied
- [ ] Ground-truth labeling method agreed and started

### Phase 2 — Core Features

- [ ] NDVI, NDWI, EVI computed
- [ ] SAR features (VV, VH, ratio) computed
- [ ] Ground-truth sample table finalized
- [ ] Random Forest crop classifier trained
- [ ] Confusion matrix / accuracy computed
- [ ] NDVI time-series phenology markers computed
- [ ] Moisture stress index computed and binned

### Phase 3 — Integration

- [ ] Rainfall/ET data pulled for AOI
- [ ] Simplified irrigation advisory rule defined
- [ ] Advisory map generated
- [ ] All outputs combined into one notebook/repo

### Phase 4 — Polish

- [ ] README written (what it does, how to run it, limitations)
- [ ] Maps cleaned up for readability (legend, title, AOI boundary shown)
- [ ] Internal demo / walkthrough between team members
- [ ] Retro: what to learn next / what to build next iteration

---

## 7. Definition of Done for This Sprint

You're done, successfully, if by the end of Weekend 3 you have:

1. A shared GitHub repo with clean, runnable notebooks (even if outputs are rough).
2. A crop-type map for the pilot area, with a stated (even if modest) accuracy figure.
3. A moisture stress map/index for at least one growth stage.
4. A simple irrigation advisory map or table (rule-based is fine).
5. A short written summary of limitations and what you learned.

Polish, accuracy targets, and dashboard aesthetics are explicitly **not** the bar — tool familiarity and a working team process are.

---

## 8. Risks & Mitigations

| Risk | Mitigation |
| --- | --- |
| GEE learning curve eats the whole first weekend | Do the prep-week tutorial before Weekend 1 starts; Role 1 pre-tests the exact API calls needed |
| Ground truth labeling takes longer than expected | Cap it — 40–60 points is enough to learn the workflow; don't chase completeness |
| SAR processing (speckle filtering, backscatter interpretation) is genuinely harder for beginners | Treat SAR as a stretch feature for Weekend 2; the classifier can run on optical-only features first, add SAR if time allows |
| One role blocks another (e.g., Role 2 needs Role 1's data) | Keep a shared "dummy"/test AOI export ready by end of Prep Week so no one is idle |
| Scope creep toward the full hackathon vision | Re-read Section 1 before adding anything new; park extra ideas in Section 9 |
| Team coordination drifts (common in first team projects) | One 15–30 min sync each weekend is non-negotiable, even if short |

---

## 9. Stretch Goals (only if Weekend 3 finishes early)

- Add a second season for phenology comparison (Kharif vs. Rabi)
- Try an LSTM on the NDVI time series instead of static features
- Build a very simple Streamlit or Colab-widget dashboard instead of static maps
- Expand the AOI or add a second pilot area
- Properly validate a handful of ground-truth points via satellite basemap cross-checks by more than one team member (informal inter-rater check)

---

*This plan intentionally trades completeness for learnability. Once your team is comfortable with GEE, feature engineering, and the git/task-board workflow, a second iteration can push toward the fuller hackathon-scale version — larger AOI, deep learning models, real crop-water-balance irrigation logic, and a proper dashboard.*
