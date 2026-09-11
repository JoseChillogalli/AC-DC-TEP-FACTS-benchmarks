# AC/DC–FACTS Transmission Expansion Planning Benchmarks

Test-system data accompanying the paper *A MISOCP Formulation for Integrated Expansion
Planning of Hybrid AC/DC Transmission Networks with VSC-HVDC, FACTS Devices and
Reactive Power Compensation*.

Three transmission expansion planning benchmarks in which candidate AC circuits,
VSC-HVDC links, static synchronous series compensators (SSSCs), static synchronous
compensators (STATCOMs) and modular capacitor banks compete within a single
optimization problem. The three systems share candidate-asset conventions, a single
primary cost source and the same operating assumptions, so results are comparable
across them.

This deposit contains the test-system data only. The optimization model is described in
the accompanying paper.

## Files

`garver6_acdc_facts.dat` is the Garver 6-bus system with remote generation at bus 6,
`ieee24_acdc_facts.dat` is the IEEE 24-bus RTS-96 with a bulk transfer requirement over
the 700-km corridor between buses 22 and 8, and `ieee118_acdc_facts.dat` is the IEEE
118-bus system with geographically dispersed candidates and a bulk corridor between
buses 25 and 70. Each file is a self-contained AMPL data file for the all-technologies
case; the restricted configurations reported in the paper are obtained by fixing the
corresponding investment binaries to zero.

## What these files assume

The power base is 100 MVA. The objective is pure investment: `CRF = 1` and
`OPF_SCALE = 0` are declared inside each file, so operating cost does not enter the
objective. Every candidate corridor admits at most three new circuits, and the
operational angle-difference limit is 40 degrees.

Nodal demands are multiplied by `load_scale`, declared inside each file and applied in
the nodal balances. This matters: summing the raw `Pd` and `Qd` entries does not give
the operating condition reported in the paper. The factors are 1.40 for Garver, 0.75
for the IEEE 24-bus and 1.00 for the IEEE 118-bus system, which yield 1064 MW,
7378 MW and 6242 MW of active demand respectively.

Every candidate DC link is described as a single equivalent DC branch with `dc_p = 1`.
Monopolar and bipolar alternatives are distinguished through their power rating, DC
resistance and investment cost, not through the pole count, so a bipolar link does not
appear as two poles in the data.

Device costs come from a single primary source, the MISO MTEP24 Transmission Cost
Estimation Guide in 2024 dollars: 11,873 USD/MVAr for capacitor banks, 226,013
USD/MVAr for STATCOMs and 159,750 USD/MVAr-series for SSSCs. AC and DC line costs are
corridor dependent and VSC-terminal costs follow the MTEP24 station curve by rating
class. Every cost entry in the files already includes the bus-bay position of the
connection point, which is why a device costs slightly more than its rate times its
rating.

## Using the files

Load one data file, fix to zero the binaries of the technologies excluded by the
configuration under study, and solve. The AC-only case fixes `xBC`, `xSC`, `xST`, `xDC`
and `xCV`; the single-technology cases release only the corresponding one; the
all-technologies case fixes nothing.

Solve each configuration in a fresh model session. Chaining several configurations in
one session degrades numerical conditioning and can return solutions that violate
variable bounds while still being reported as optimal.

On the IEEE 118-bus system the capacitor-bank layout is degenerate. At 11,873 USD/MVAr
the banks are nearly free relative to the AC and HVDC investments, so several bank
distributions are co-optimal within the MIP gap. The AC plan and the HVDC link are
run-invariant, but the number and placement of banks may differ between runs and
solver versions without changing the conclusions.

Two of the three systems behave differently under an exact AC/DC feasibility check. In
the Garver and IEEE 118-bus systems every reported configuration admits an operating
point. In the IEEE 24-bus system the AC-only, AC+SSSC and AC+STATCOM configurations do
not, within the candidate catalog provided here.

## Citation

If you use these benchmarks, please cite the accompanying paper.
