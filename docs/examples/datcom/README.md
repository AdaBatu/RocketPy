# DATCOM comparisons

**Work in progress:** DATCOM currently produces poor flight predictions for several comparison rockets. The discrepancies are unresolved and are being investigated; these results should not be treated as validated predictions.

Only two example notebooks are included:

- [Recorded rockets](rocket_comparison.ipynb): DATCOM, ordinary RocketPy and
  recorded altitude, upward vertical velocity, CD and stability margin where
  available, with signed percentage errors. Apogee-fitted constant CD is
  explicitly inferred calibration, not a measurement or native correction.
  Apogee bars cover all 16 rockets: green is the closer paired prediction and
  red is the farther one, with recorded apogee as the dashed reference.
- [AST-HER / Asteria](asteria_comparison.ipynb): the same configuration, tanks
  and atmosphere using ordinary RocketPy's supplied Asteria drag curve versus
  native DATCOM. **AST-HER has not flown.** Percentages are model differences.

Missing measurements are shown as unavailable. Scalar CD comparisons use Mach
0.6 and zero incidence; static margin uses loaded ignition and Mach zero
(DATCOM uses its lowest positive Mach). NDRT's measured stability conditions
are unspecified. Velocity uses recorded vertical/altimeter channels, never a
numerical derivative presented as a measurement. Fitted CD uses one constant
for powered and coast drag in the ordinary source model; it is not unique.

Source models and raw flight records stay in their original repository folders.
`comparison_data/` contains reproduction scripts, source hashes, reference
notes, QA masks, fitted residuals, values and model histories. Unspecified
construction assumes carbon fibre with vinyl, without inventing a measured
roughness or fin section. The recorded-rocket DATCOM models retain their
explicit smooth 2% HEX / 5% bevel / sharp-edge assumptions. AST-HER uses its
saved Asteria section inputs. All native runs include DAMP and use adaptive
20-point Mach grids and signed alpha coverage from rail through apogee.

Reproduce from the repository environment:

```sh
python docs/examples/datcom/comparison_data/reproduce_rockets.py
python docs/examples/datcom/comparison_data/prepare_comparisons.py
```

Set `DATCOM_EXECUTABLE` to a native executable when the sibling Asteria copy is
unavailable. The latter command fits effective CD against the recorded apogees;
those are calibration results, not independent validation. Notebook execution
reads the saved comparison outputs and requires no new native runs.

The [apogee bar chart](comparison_data/apogee_comparison.png) is also exported
as [SVG](comparison_data/apogee_comparison.svg), with values in
`comparison_data/apogee_comparison.csv`. Rebuild only these exports with
`python docs/examples/datcom/comparison_data/plot_apogee_comparison.py`.
Labels show signed percentage error. P50/P75/P90/P95 in
`comparison_data/apogee_error_percentiles.csv` are empirical absolute-error
percentiles across the same paired rockets, not per-flight uncertainty.
Missing DATCOM results remain unavailable, and archived RocketPy values are
marked. Colour ranking is withheld when the competing prediction is absent.

The supplied CD group includes RASAero, constants, curves of unspecified
provenance and Juno's explicitly CFD-scaled curve. It is not universally CFD.
RocketPy has no separate built-in geometry-to-CD predictor. The example loader
now preserves legacy post-construction `power_on_drag`/`power_off_drag` edits
in the seven-input solver coefficient; this corrects stale curve use for
Juno and Bella Lui without changing any RocketPy equations. The two affected
ordinary/DATCOM comparisons and the associated metrics have been regenerated.

The [manual audit](comparison_data/datcom_usage_audit.json) records an
independent reproduction of the guide's worked example, `PART` invariance for
all 11 native comparison models, source nozzle dimensions and base-drag
sensitivity flights. Reproduce it with:

```sh
python docs/examples/datcom/comparison_data/audit_datcom_usage.py
```

The audit found and corrected component damping being selected instead of
total-vehicle damping when parsing `PART` output. These comparison decks do
not request `PART`, so their existing predictions are unaffected. GENESIS and
Bella Lui have source nozzle exit diameters larger than their thrust files'
listed motor case diameters; those inputs need hardware verification.
GENESIS remains 10.5% above recorded apogee even with full base drag during
the burn (3,223 m versus 2,916 m). This bounds the nozzle-area contribution;
it does not establish a measured nozzle correction. Fin section, bevel,
edge radius and surface finish remain assumptions. The guide also describes
approximate airfoil properties after its crest-critical-Mach warning, which
all 11 native grids produce. Parsed native diagnostics are now retained.
Some warnings concern unflown grid points. The guide also assumes sharp
trailing edges in its airfoil section solver and approximates blunt sections
as ideal airfoils. Neither a carbon-fibre label nor USER coordinates establish
that these approximations describe the real fin edges.

Explicit trailing-edge sweep and running Mach 0.6 alone versus in a larger
grid produce identical CA/CN in all 11 controlled checks. Equivalent metre
and inch decks show a small native transonic unit sensitivity: Andromeda's
CA at Mach 0.9 is 0.396 versus 0.404, localized to body pressure/wave drag in
the partial output. Its Mach 0.3/0.6 values agree. The audit records this
remaining numerical discrepancy; it does not silently select the result
closer to the recorded flight.
Running the affected full-flight models with equivalent inch decks changes
apogee by -12.06 m for Andromeda, -0.78 m for Astra, -0.01 m for GENESIS and
+0.09 m for Valetudo. These changes are much smaller than their flight errors.

The [fin-parameter audit](comparison_data/fin_parameter_audit.json) tests
additional section assumptions without replacing the baseline comparisons.
Reproduce its 33 coefficient probes and 15 flight runs with:

```sh
python docs/examples/datcom/comparison_data/audit_fin_parameters.py
```

Trailing-edge thickness is a substantial missing hardware input. In the
controlled GENESIS run at Mach 0.6, changing a sharp USER section to a section
with a full-thickness trailing edge raises zero-incidence CD from 0.472 to
0.586. Native partial output attributes the increase to trailing-edge drag:
0.0285 per fin, or 0.114 for four fins. Doubling the HEX total thickness/chord
from 2% to 4% gives only 0.474 at that speed; 0.5/1 mm leading-edge-radius
probes give 0.472. Thickness and bevels have larger transonic effects.

| Rocket | Recorded apogee AGL (m) | Baseline HEX (m) | Sharp USER control (m) | Blunt USER hypothesis (m) |
| --- | ---: | ---: | ---: | ---: |
| GENESIS | 2916 | 3297 (+13.1%) | 3325 (+14.0%) | 3025 (+3.7%) |
| Astra | 3250 | 3696 (+13.7%) | 3773 (+16.1%) | 3378 (+3.9%) |
| Andromeda | 3443 | 4171 (+21.1%) | 4293 (+24.7%) | 3974 (+15.4%) |

The USER controls distinguish edge changes from differences between native
HEX and USER methods. The blunt hypothesis retains 2% thickness/chord,
which tapers in physical thickness: for GENESIS it is 4 mm at the root and
2.2 mm at the tip. It is not a measured constant-thickness plate. A separate
4 mm plate / 5 mm bevel HEX probe gives GENESIS 3284 m, so that change alone
does not close the gap. All flight variants stay within the sampled Mach,
alpha and beta grids at dynamic pressure above 100 Pa.

Ordinary RocketPy's fin inputs describe planform and lift; whole-vehicle drag
comes from the separately supplied drag curve. Its optional `airfoil` input
is a lift curve, not cross-section coordinates. DATCOM additionally needs
thickness, bevels and edge geometry. `ZUPPER`/`ZLOWER` describe each half of
the section, while our `thickness_to_chord` is their sum. `LMAX`/`LFLAT` are
chord fractions, `LER` is a dimensional radius, and `SSPAN` is normally from
the centreline; our `SSPAN(1)=0` convention correctly requests exposed-span
automatic body attachment. These conversions were checked. The hardware
section values remain unknown, and no fit or default change is applied.
