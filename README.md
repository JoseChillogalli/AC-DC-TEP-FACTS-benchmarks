# AC/DC–FACTS Transmission Expansion Planning Benchmarks

Test-system data accompanying the paper *A MISOCP Formulation for Integrated AC/DC
Transmission Expansion Planning with STATCOMs, SSSCs, and Reactive Power Compensation*.

Three transmission expansion planning benchmarks in which candidate AC circuits,
VSC-HVDC links, static synchronous series compensators (SSSCs), static synchronous
compensators (STATCOMs) and modular capacitor banks compete within a single
optimization problem. The three systems share candidate-asset conventions, a single
primary cost source, and the same operating assumptions, so results are comparable
across them.

## Files

| File | System | Paper table |
|---|---|---|
| `garver6_acdc_facts.dat` | Garver 6-bus, remote-generation scenario | Table 4 |
| `ieee24_acdc_facts.dat` | IEEE 24-bus RTS-96, bulk-corridor scenario | Table 5 |
| `ieee118_acdc_facts.dat` | IEEE 118-bus, dispersed candidate set | Table 6 |

Each file is a self-contained AMPL data file for the all-technologies case, in which
every candidate technology is simultaneously available. The restricted configurations
reported in the paper are obtained by fixing the corresponding binaries to zero
(see *Reproducing the configurations* below).

## Common conventions

| Setting | Value |
|---|---|
| Power base | `Sbase = 100` MVA |
| Objective | Pure investment (`CRF = 1`, `OPF_SCALE = 0`) |
| New circuits per candidate corridor | `Nmax = 3` (uniform in all three systems) |
| Angle-difference limit | 40° (`tan_amax = 0.8391`, `theta_max_rad = 0.6981`) |
| HVDC representation | Single equivalent DC branch (`dc_p = 1`); monopole/bipole distinguished by rating, DC resistance and cost, not by pole count |
| Cost source | MISO MTEP24 Transmission Cost Estimation Guide (USD 2024) |

**Demand scaling.** Nodal demands in each file are multiplied by `load_scale`, which is
declared inside the file and applied in the nodal balances. Summing the raw `Pd`/`Qd`
entries without this factor does not give the operating condition reported in the paper:

| System | `load_scale` | Nominal demand | Operating demand |
|---|---:|---|---|
| Garver 6-bus | 1.40 | 760 MW / 228 MVAr | 1064 MW / 319 MVAr |
| IEEE 24-bus | 0.75 | 9837 MW / 2951 MVAr | 7378 MW / 2213 MVAr |
| IEEE 118-bus | 1.00 | 6242 MW / 2941 MVAr | 6242 MW / 2941 MVAr |

## Unit costs

Device costs come from a single primary source (MISO MTEP24, Table 2.3-8), so the
price hierarchy is consistent across systems. Per-unit costs differ only through the
bus-bay position, which scales with the voltage class of the connection point.

| Technology | Rate | Per unit |
|---|---|---|
| Capacitor bank | 11,873 USD/MVAr | 1.19 MUSD per 100-MVAr module (Garver); 0.59 MUSD per 50-MVAr module (IEEE 24/118) |
| STATCOM | 226,013 USD/MVAr | 150 MVA + bay: 36.1 MUSD at 230 kV, 35.6 MUSD at 138 kV |
| SSSC | 159,750 USD/MVAr-series | 30 MVA series source + bay: 7.0 MUSD at 230 kV, 6.5 MUSD at 138 kV |
| VSC terminal | MISO station curve by rating class | 159 MUSD (500 MW), 239 MUSD (750 MW), 346 MUSD (1500 MW), 230.5 MUSD (1000 MW) |
| AC line | 1.30 / 2.25 / 3.00 MUSD/km at 138 / 345 / 500 kV | corridor dependent, plus bays |
| DC line | 1.50 / 1.75 / 1.95 MUSD/km by rating class | corridor dependent |

## System descriptions

### Garver 6-bus (remote generation)

Six buses, six existing AC circuits. Generation at buses 1, 3 and 6 totals 1140 MW
(reactive capability −30 to 332 MVAr). Bus 6 hosts 610 MW and is initially
disconnected, so connecting it is the central expansion requirement; with the
operating demand of 1064 MW against 1140 MW of installed capacity, the remaining
generation alone cannot serve the load.

Candidates: 15 AC corridors (every bus pair), 15 VSC-HVDC alternatives (ten rated
500 MW, five rated 750 MW as monopoles with metallic return, serving every corridor
that terminates at bus 6), modular 100-MVAr capacitor banks with up to three modules
per bus and 150-MVA STATCOMs at buses 2, 4 and 5, and SSSCs on existing branches
1–5 and 2–4.

### IEEE 24-bus RTS-96 (bulk corridor)

Twenty-four buses, 38 existing AC lines, 12 generating units. A 2000 MW generation
hub is introduced at bus 22 with a matching demand increase at bus 8, so the transfer
must cross a single 700 km corridor. That corridor offers a 500 kV AC option
(up to three parallel circuits) and a ±500 kV, 1500 MW bipolar VSC-HVDC link.

Candidates: 28 AC corridors, nine VSC-HVDC alternatives (eight rated 500 MW plus the
bulk bipole), six 150-MVA STATCOMs, four SSSCs, four switchable 50-MVAr capacitor
banks (one module per bus).

Corridor B1 (8–22) parameters are derived from its physical length: `r = 0.028 Ω/km`,
`x = 0.28 Ω/km` and `bch = 4.66 µS/km` over 700 km on a 2500 Ω base, with 60 % shunt
reactor compensation.

### IEEE 118-bus (dispersed candidates)

One hundred eighteen buses, 54 generators totalling 12,432 MW, 186 existing AC lines,
voltage limits 0.94–1.06 pu. Reactive demands are scaled by 1.6 to create reactive
scarcity. The bulk corridor 25–70 spans 800 km at 500 kV.

Candidates: 30 AC corridors distributed across the network, 18 VSC-HVDC links
(seventeen rated 500 MW plus one 1000 MW monopole on corridor 25–70) requiring 36
converter terminals, twelve 150-MVA STATCOMs, six SSSCs, and capacitor-bank
alternatives at 48 buses with one 50-MVAr module each.

The candidate corridors are deliberately dispersed rather than concentrated near the
deficit bus: concentrated candidate sets weaken the second-order cone lower bound and
prevent convergence, whereas the dispersed set closes to a negligible gap.

## Reproducing the configurations

Each file yields the all-technologies case directly. The restricted configurations are
obtained by fixing the corresponding investment binaries to zero before solving:

| Configuration | Fixed to zero |
|---|---|
| AC only | `xBC`, `xSC`, `xST`, `xDC`, `xCV` |
| AC + banks | `xSC`, `xST`, `xDC`, `xCV` |
| AC + SSSC | `xBC`, `xST`, `xDC`, `xCV` |
| AC + STATCOM | `xBC`, `xSC`, `xDC`, `xCV` |
| AC + HVDC | `xBC`, `xSC`, `xST` |
| All technologies | none |

Example (AMPL):

```
reset;
model integrated_misocp.mod;
data ieee118_acdc_facts.dat;
fix {(i,h) in BCMOD} xBC[i,h] := 0;   # AC + HVDC case
fix {l in LSC} xSC[l] := 0;
fix {i in NST} xST[i] := 0;
option solver gurobi;
option gurobi_options 'mipgap=0.0001 threads=12 mipfocus=2';
solve;
```

Solve each configuration in a fresh model. Chaining several configurations in one
model session degrades numerical conditioning and can return solutions that violate
variable bounds while still being reported as optimal.

### Reproducibility of the capacitor-bank layout

On the IEEE 118-bus system the capacitor-bank layout is degenerate: at
11,873 USD/MVAr the banks are nearly free relative to the AC and HVDC investments,
so several bank distributions are co-optimal within the MIP gap. The AC plan, the
HVDC link and the SSSC are run-invariant; the total investment reproduces to within
about 1 MUSD (roughly 0.06 %) across solver runs and versions. Tightening the gap
typically returns a plan with six banks instead of the eight reported in the paper,
at an investment of 2027.6 MUSD instead of 2028.8 MUSD. This is expected behaviour
of the benchmark, not a failure to reproduce.

## Verification

Investment plans obtained from the second-order cone model are verified by fixing all
investment binaries and solving the resulting continuous AC/DC optimal power flow in
the exact polar formulation, which restores the voltage-product identities on every
energized bus pair. Initial angles are recovered from the conic solution through a
breadth-first traversal of the energized network using `atan2(T, R)`; a flat start
fails on heavily loaded plans.

For the IEEE 24-bus system, the AC-only, AC+banks and AC+STATCOM plans admit no exact
AC/DC operating point, so active power-flow control is technically required rather
than merely cheaper.

## Citation

If you use these benchmarks, please cite the accompanying paper.
