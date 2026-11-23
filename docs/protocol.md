# Designing a Synthetic Gene with Silent Internal Restriction Sites

A workflow from a donor vector through codon optimization to a synthesis-ready
insert carrying uniquely-cutting restriction sites for future variant cloning.

This protocol pairs the **Silent Sites** tool (placement finder) with **GenScript
GenSmart** (codon optimizer). The tool decides *which* enzymes are usable and
*where* their sites can go silently; GenScript produces the optimized DNA.

---

## Overview

1. Audit the donor vector to find which enzymes are usable (zero-cutters).
2. Filter that list to a working palette (NEB, rCutSmart, correct behaviour).
3. First codon optimization — a clean sequence with **no** sites.
4. Find and choose site placements with the Silent Sites tool.
5. Place the sites and re-optimize (round-trip) so the sequence stays optimal.
6. Validate the whole plasmid before ordering synthesis.

---

## Step 1 — Audit the donor vector (find the zero-cutters)

The goal is the set of enzymes that cut the **empty backbone zero times**. Only
these can become unique cutters once added to your insert — an enzyme that
already cuts the backbone once would become a double-cutter.

1. Open the donor vector in SnapGene with the gene-to-be-replaced already
   removed (the empty backbone).
2. Set the sequence to **circular** (essential — a linear reading misses sites
   spanning the origin and the deletion junction).
3. **Enzymes → Choose Enzymes…**, load a **NEB, 6+ bp** enzyme set.
4. Filter to enzymes that cut **0 times**. Export that list with recognition
   sites included. This is your candidate universe.

> Also export the **1+ cutters** if you want a cross-check: every NEB 6+ enzyme
> should appear on exactly one list. Keep the 1-cutters — they are candidates
> for linearizing the backbone for Gibson assembly.

---

## Step 2 — Build the working palette

Not every zero-cutter is usable. From the zero-cutter list, keep only enzymes
that are:

- **From NEB** (drop enzymes not sold by NEB if that is your supplier).
- **100% active in rCutSmart** (check NEB's activity chart; use HF versions
  where the wild-type needs a different buffer — e.g. BstEII-HF, BsiWI-HF).
- **Single-site cutters** (drop enzymes that need two sites to cleave, e.g.
  SacII, SgrAI, SfiI — a lone placed site will not cut).
- **Type IIP, cutting within their recognition sequence** (drop homing
  endonucleases and Type IIS enzymes; hold the Type IIS set aside to exclude).

The Silent Sites tool ships with an audited palette already meeting these
criteria (sticky, blunt, and degenerate tiers). If your vector's zero-cutter
list differs, add or remove enzymes with **+ Add custom enzyme** (enter the
recognition site and cut positions so the tool can compute the overhang).

---

## Step 3 — First codon optimization (a clean sequence)

Produce an optimized coding sequence containing **none** of your candidate
sites, so you can place them deliberately afterwards.

1. In GenScript GenSmart, paste the **protein sequence**.
2. Set the **expression host** (this sets the codon table).
3. In the **Excluded** box, enter your whole candidate palette **plus** the
   Type IIS set (so Golden Gate stays available later):

   ```
   [your sticky palette], BsaI, BsmBI, BbsI, SapI, PaqCI, AarI, BtgZI
   ```

4. Leave the **Kept** box empty. Do not let GenScript auto-add sites — you will
   place them yourself with position control.
5. Keep GC content in the 40–60% range. Submit.

The output is a clean, optimized CDS with zero copies of anything you care
about. Save it.

---

## Step 4 — Find and choose placements (Silent Sites tool)

1. Open the Silent Sites tool. Paste the **protein sequence** (same one).
2. Set the **host codon table** to match your GenScript run. (Human is verified
   against GenScript; for E. coli or yeast, confirm the table first.)
3. Select your enzyme palette — sticky is the default. Add custom enzymes if
   needed.
4. Click **Find placements**. Every enzyme's silent placements appear, in all
   three reading frames, each with its position and CAI (codon cost).

### Reading the results

- **CAI bar** — green (≥0.80) is cheap; amber has some cost; red forces rare
  codons. Higher is better.
- **frame +0 / +1 / +2** — +1 and +2 straddle codons using wobble positions;
  these are placements a by-eye scan would miss.
- **Impossible** enzymes cannot go anywhere in this protein at any cost.

### Choosing the combination

Use the **Suggested combination** panel:

- **Strategy** — start with *Balanced*; it favours cleaner codons over raw
  count. Use *Max coverage* only if you need the most sites.
- **End margin** — leave at **30 aa**. This keeps sites clear of the 5′
  initiation region and clear of the restriction sites that flank the insert
  for cloning.
- **Overhang collisions** are enforced automatically — the optimizer will not
  pick two enzymes that leave compatible ends.
- **Veto** any enzyme you are reserving for the backbone by clicking its row;
  the tool re-solves around your choice.
- **Methylation flags** — a red **Dam/Dcm** badge means you must check the
  site's flanking bases (see Step 5). A yellow **CpG** badge only matters if you
  will digest DNA recovered from mammalian cells; E. coli-grown plasmid is
  unaffected.

**Match the chosen positions to your biology.** The optimizer places sites for
even spacing and low codon cost — it does not know your domain boundaries. Move
or veto sites so each brackets a region you actually intend to swap. Record the
final enzyme list and their amino-acid positions.

---

## Step 5 — Place the sites and re-optimize (round-trip)

Placing sites overwrites a few codons; re-optimizing absorbs that perturbation
back into a globally optimal sequence.

1. Edit your clean CDS (from Step 3) to introduce each chosen site at its
   position, using the exact codons the tool shows for that placement.
2. **Check the Dam/Dcm flanks now.** For any site with a Dam/Dcm flag, confirm
   the flanking bases do not form the blocking motif:
   - **ClaI** — avoid `ATCGATC` and `GATCGAT`.
   - **FseI / Dcm enzymes** — avoid a `CCWGG` context at the flank.
   Adjust a neighbouring codon (synonymously) if needed.
3. Paste the edited sequence back into GenScript. This time put your **placed
   enzymes in the Kept box** and keep the **Type IIS set in Excluded**. Submit.
4. GenScript holds your sites fixed and re-optimizes everything around them.

> **Verify the round-trip preserved your sites.** Before trusting the output:
> confirm each site is still at your position, appears exactly **once**, and
> that the sequence still translates to the **identical protein** at identical
> length. GenScript's "Kept" can sometimes satisfy a keep by relocating a site —
> test with one site first if unsure.
>
> **Re-check the Dam/Dcm flanks after re-optimization** — GenScript rewrites
> flanking codons, which can reintroduce a blocking context you had cleared.

---

## Step 6 — Validate before ordering

1. Assemble the final **whole plasmid** in silico (insert + backbone).
2. Run it through **NEBcutter V3** (or SnapGene's unique-cutter view) and
   confirm each placed enzyme cuts the whole construct **exactly once**.
3. Confirm no Type IIS site was introduced during optimization.
4. Confirm the insert still translates to the correct protein.
5. Check the final CAI against your host (GenScript reports it). A well-optimized
   gene lands around 0.8–0.95; individual off-peak codons are normal.

Order synthesis once all checks pass.

---

## Quick reference — the two GenScript runs

| Field | Run 1 (clean) | Run 2 (round-trip) |
|---|---|---|
| Protein / CDS input | protein sequence | your site-edited CDS |
| **Excluded** | full palette + Type IIS | Type IIS + unused palette enzymes |
| **Kept** | *(empty)* | the enzymes you placed |
| Result | clean, site-free CDS | optimized CDS with your sites locked |

An enzyme is in **exactly one** box per run — never both.

---

## Notes

- Because Gibson/HiFi assembly is typical for the first build, you do **not**
  need sites at the insert termini — the end margin in Step 4 reflects this.
- Compatible-overhang pairs (e.g. two enzymes both leaving 5′ GTAC) can
  cross-ligate. The tool prevents picking both automatically; if placing sites
  manually, keep them on different cassettes.
- CAI is a ranking heuristic. Trust the tool's relative ranking of placements,
  but confirm absolute CAI in GenScript before ordering.
- The Silent Sites tool runs entirely in your browser; nothing you paste is
  uploaded. Custom enzymes persist for the session only.
