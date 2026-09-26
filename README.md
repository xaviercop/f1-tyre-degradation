# F1 Tyre Degradation Model — 2026 Season

Measuring how much lap time Formula 1 cars lose to tyre wear, using real timing data
from the 2026 season.

The headline result is not what I expected. Across the window in which each compound
is actually used, soft, medium and hard tyres degrade at broadly similar measured
rates — around 0.05 s/lap. Where they differ sharply is in **usable life**: the median
soft stint lasts 12 laps, medium 16, hard 22. Teams pit when a tyre reaches a
performance threshold, so each compound is only ever observed while it is still
working. The compound difference shows up as how long the tyre lasts, not as how
fast it falls away.

![Degradation curves by compound](results/degradation_curves.png)

---

## Data

14 completed races of the 2026 Formula 1 season, retrieved through the
[FastF1](https://github.com/theOehrly/Fast-F1) API. **16,255 raw laps** across 22
drivers.

Only 2026 is used. The 2026 regulations changed the power units, the aerodynamics and
the Pirelli constructions, so a 2026 car degrades its tyres differently from a 2025
car on different rubber. Pooling seasons across that boundary would mix two different
sets of physics into one model and make every result harder to interpret. One
regulation era keeps the question clean.

---

## Cleaning

The effect being measured is small — on the order of 0.05 seconds per lap of tyre age.
Lap times move by far more than that for reasons which have nothing to do with tyres:
a pit stop costs twenty seconds, a safety car thirty, traffic one to three. Left in,
that noise is orders of magnitude larger than the signal, and a model will spend its
capacity explaining safety cars instead of tyre wear.

Nine filters, each removing laps whose slowness has a known cause other than the tyre.

| Filter | Removed | Remaining |
|---|---:|---:|
| Missing lap time | 291 | 15,964 |
| Wet / intermediate compounds | 39 | 15,925 |
| Out-laps | 499 | 15,426 |
| In-laps | 511 | 14,915 |
| Not green-flag throughout | 1,488 | 13,427 |
| Lap 1 | 165 | 13,262 |
| Stints under 5 laps | 101 | 13,161 |
| Missing tyre age | 19 | 13,142 |
| Warm-up laps (tyre age ≤ 2) | 466 | **12,676** |

**12,676 laps retained — 22% of the raw data removed.**

![Raw stint before cleaning](results/raw_stint_uncleaned.png)

The chart above is a single uncleaned stint, and it contains the argument for the whole
pipeline. The lap at 113 seconds is the out-lap — the first lap on a new set begins in
the pit lane under the speed limit, with the stop itself included. The lap at 105
seconds is a safety car lap. Everything else sits in a tight band around 94 seconds
with a gentle upward drift. Two of twenty-two laps carry twenty seconds of noise that
has nothing to do with rubber.

Two filters deserve a note.

**Track status.** FastF1 records every status that applied at any point during a lap,
concatenated into one string. A value of `'167'` does not mean status 167 — it means
the lap ran green, then a virtual safety car was deployed, then it ended, all within
those ninety seconds. Only laps marked exactly `'1'` were green from start to finish.
Filtering for laps that merely *contain* a `1` would have kept every compromised lap
in the dataset. This was the largest single filter, removing 1,488 laps.

**Warm-up.** A new set is not at working temperature for the first couple of laps, and
the driver has just rejoined into traffic. Those laps are slow for reasons unrelated to
wear, and leaving them in dragged the start of every degradation curve upward.

---

## Fuel correction

This is the hardest part of the problem and the part most easily got wrong.

Within a stint, tyre age and race lap number increase together, one for one. Two
opposite effects run simultaneously: the tyre ages and slows the car, while fuel burns
off and speeds it up. A 2026 car burns roughly 1.6 kg per lap, and lap time responds to
weight at roughly 0.03 s/kg — about **0.05 seconds per lap** of pure fuel-burn gain.

Every raw lap time is therefore the *net* of degradation and fuel burn, and
degradation measured off uncorrected times is always too small.

No statistical method can separate the two from within-stint data alone, because they
never vary independently — they move in lockstep, every lap, in every stint. The way
out is not cleverer statistics but physics: the fuel effect can be calculated
independently and removed, leaving what is attributable to the tyre.

Each lap time is normalised to a full-fuel equivalent:

```
corrected = raw + 0.05 × (lap number − 1)
```

### Sensitivity to the assumption

That 0.05 is an estimate, not a measurement. So the whole analysis was rerun across a
range of plausible values:

| Assumed fuel effect | SOFT | MEDIUM | HARD |
|---|---:|---:|---:|
| 0.03 s/lap | 0.031 | 0.040 | 0.031 |
| 0.04 s/lap | 0.041 | 0.050 | 0.041 |
| 0.05 s/lap | 0.051 | 0.060 | 0.051 |
| 0.06 s/lap | 0.061 | 0.070 | 0.061 |
| 0.07 s/lap | 0.071 | 0.080 | 0.071 |

Every measured degradation rate shifts **exactly one-for-one** with the assumption.

This is an identity, not a coincidence. Within a stint, `LapNumber = TyreLife +
constant`, so adding `fuel × (LapNumber − 1)` is equivalent to adding
`fuel × TyreLife` plus a constant — and a term proportional to TyreLife goes directly
into the fitted slope. Therefore:

```
measured degradation = raw slope + assumed fuel coefficient
```

Two consequences, and they matter:

- **Absolute degradation rates are conditional on the fuel assumption.** The data
  cannot establish whether the true rate is 0.03 or 0.07 s/lap. Any figure quoted here
  should be read as "given a 0.05 s/lap fuel effect".
- **Comparisons between compounds are unaffected.** Medium sits 0.009 s/lap above soft
  and hard at every fuel value tested. The fuel term cancels, so that difference is
  robust.

Separating the two would require data in which tyre age and fuel load vary
independently — comparing equal tyre ages at different fuel loads across stints, which
this within-stint method cannot do.

---

## Results

### Degradation rate

Median per-stint rate, fitted individually to each of 659 stints, at the 0.05 s/lap
fuel assumption:

| Compound | Stints | Median rate | Std dev |
|---|---:|---:|---:|
| SOFT | 121 | 0.051 s/lap | 0.222 |
| MEDIUM | 253 | 0.060 s/lap | 0.107 |
| HARD | 285 | 0.051 s/lap | 0.080 |

Rates were fitted per stint rather than pooled across all laps of a compound.
Degradation is a within-stint phenomenon — what happens to *one* set of tyres as *that*
set ages. Pooling mixes it with differences between stints that have nothing to do with
tyre age: a driver on a 40-lap hard stint is managing the tyre, a driver on a 10-lap
soft stint is attacking. Fitting each stint independently isolates the effect.

### Usable life

![Stint length by compound](results/stint_length.png)

| Compound | Median stint | Mean stint |
|---|---:|---:|
| SOFT | 12 laps | 14.4 |
| MEDIUM | 16 laps | 17.2 |
| HARD | 22 laps | 22.9 |

This is where the compounds separate, and it explains why the degradation curves sit on
top of each other. Teams pit when a tyre reaches a certain drop-off, so each compound is
only ever observed inside the window where it still works. A soft tyre is never seen at
22 laps old because nobody runs one that long — the steep part of its curve is censored
out of the data by the strategy that generated it.

The soft standard deviation of 0.222 against hard's 0.080 follows from the same fact:
soft stints are short, so each slope is fitted to fewer points and is correspondingly
noisier.

---

## Model evaluation

A `HistGradientBoostingRegressor` predicting lap time delta from tyre age and compound,
evaluated two ways. In both cases whole groups are held out, so no lap from a test
stint ever appears in training.

| Split | Baseline MAE | Model MAE | Result |
|---|---:|---:|---|
| Held-out **stints** (circuits seen in training) | 0.655 s | **0.615 s** | model 6% better |
| Held-out **races** (circuits never seen) | 0.629 s | 0.716 s | model 14% worse |

The baseline predicts the training mean for every lap — it knows nothing about tyres.
Without it, an MAE figure has no scale and means nothing.

**The contrast between those two rows is the main modelling result.** Tyre degradation
is genuinely learnable: given circuits it has seen, the model beats a constant. But it
does not transfer to a circuit it has never seen. Bahrain eats tyres; Monaco barely
touches them. With only tyre age and compound as inputs, the model has no way of knowing
which kind of circuit it is being asked about, so it predicts an average that is wrong in
circuit-specific ways.

A feature ablation confirms the mechanism:

| Features | MAE (race split) |
|---|---:|
| Baseline (constant) | 0.629 s |
| Tyre age + compound | 0.716 s |
| + stint number | 0.719 s |
| + lap number | 0.756 s |
| Tyre age + compound, constrained model | 0.707 s |

Adding lap number makes things worse, and constraining the model makes things better —
both signatures of overfitting. Lap number is the clearest offender: Monaco runs 78 laps
and Spa runs 44, so "lap 50" means late-race at one circuit and mid-race at another. The
model learns a rule from the training circuits that is simply false at the test ones.

**On splitting.** Both evaluations hold out whole groups rather than random rows. A
random split would place lap 12 and lap 13 of the same stint on opposite sides — same
driver, same car, same tyre, sixty seconds apart — and the model would score beautifully
while having learned nothing transferable. The race-level split is what exposed the
generalisation failure at all; a random split would have hidden it completely.

To predict degradation at an unseen circuit, the model would need features describing
the circuit: surface abrasiveness, track and air temperature, layout characteristics,
average speed. This is the reason teams run Friday practice.

---

## Limitations

- **Traffic is not modelled.** A car held up behind another runs slower for reasons
  unrelated to its tyres. Laps more than a couple of seconds off a stint's own median
  are excluded, but this is a heuristic, not a solution.
- **Track evolution is not corrected.** Circuits get faster through a race as rubber is
  laid down. This biases every measured slope downward, in the same direction as the
  fuel effect.
- **No circuit-level features.** The direct cause of the generalisation failure above.
- **Absolute rates depend on the fuel assumption**, as the sensitivity analysis shows.
  Only compound comparisons are assumption-free.
- **Soft stints are short**, so soft degradation rates are the noisiest of the three by
  a wide margin.
- **Some stint data is reconstructed.** FastF1 reported fixing incorrect tyre stint
  information for several drivers, so a small number of tyre-age values are the
  library's inference rather than raw timing.
- **One season, 14 races.** The 2026 calendar was incomplete at the time of analysis.

---

## Running it

```bash
git clone https://github.com/xaviercop/f1-tyre-degradation.git
cd f1-tyre-degradation
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
mkdir -p cache results
jupyter notebook notebooks/01-exploration.ipynb
```

The first run downloads each race session from the FastF1 API and caches it in
`cache/`, which takes several minutes and grows to a few hundred megabytes. Subsequent
runs load from cache in seconds. The cache directory is gitignored.
