# OR-Project

An operations-research prototype for crew pairing and assignment that demonstrates how data, modeling, and optimization can reduce cost, improve utilization, and increase schedule robustness.

**Executive Summary**
- **Context:** Crew pairing and rostering are primary drivers of airline operational cost and service reliability. Small improvements in pairing decisions compound across a network and across days.
- **Goal:** Provide a reproducible pipeline that reads minimal operational inputs, explores pairing opportunities, formulates a constrained optimization model, and produces improvement-ready pairing suggestions and diagnostics.
- **Impact:** The prototype quantifies trade-offs (cost vs. robustness) and identifies low-risk opportunities for operational pilots — e.g., reducing deadhead hours, smoothing duty periods, or reducing regulator-triggered reserve needs.

**Business Case & Metrics**
- **Primary KPIs:** Crew-hours, deadhead time, number of legal violations (should be zero in feasible solutions), overtime cost, and schedule recovery complexity.
- **Decision metrics:** Expected cost reduction, percent utilization increase, reserve reduction, and sensitivity under disruption scenarios.
- **Typical business questions this repo helps answer:** What is the expected annualized labor savings from revised pairings? Which flights are bottlenecks requiring additional reserve crew? How sensitive is the schedule to a 1-hour delay on major hubs?

**What's in this repository**
- `crew.csv`: sample crew records (IDs, base, qualifications, availability windows, contractual limits).
- `flight_pairs.csv`: candidate pairings / trip segments with flight times, origins/destinations, and minimum connection times.
- `OR_Project.ipynb`: the analysis notebook with EDA, model formulation, solver runs, and visualization cells.

**Data format (expected schemas)**
- `crew.csv` (columns): **crew_id**, **base**, **qual**, **max_duty_hours_per_day**, **available_from**, **available_to**.
- `flight_pairs.csv` (columns): **pair_id**, **flight_sequence**, **dep_time**, **arr_time**, **origin**, **destination**, **cost_proxy**.

**Technical design**
- Pipeline: data ingest → EDA & feature engineering → model generation → solver run → post-processing & diagnostics.
- Model (abstract): a set-partitioning / assignment MILP where each candidate pairing p is a binary decision variable $x_p\\in\\{0,1\\}$ and the objective is to minimize total pairing cost:

$$
\\min\\; \\sum_{p} c_p x_p
$$

subject to coverage and legality constraints; for each flight $f$:

$$
\\sum_{p:\\;f\\in p} x_p = 1
$$

and duty/rest / crew limits encoded as linear constraints (examples include sum of duty hours per day $\\le$ contractual limit, and rest-time separations enforced across consecutive paired trips). The notebook describes how to translate domain rules into linear constraints and penalty terms.

**Solvers & algorithms**
- Prototyping: pure-MILP using PuLP (GLPK/CBC), OR-Tools CP-SAT for hybrid encodings, or Pyomo with Gurobi/CPLEX for larger experiments.
- Heuristics: greedy construction, local search, or LNS (large neighborhood search) for larger instances; decomposition by base or time window to scale.

**Reproducibility & how to run**
1. Open `OR_Project.ipynb` and run cells top-to-bottom (preferable on colab, as the solver is optimized in a linux environment, see tutorials for windows OS).

2. If you prefer quick installs instead of `requirements.txt`:

```bash
pip install pandas numpy matplotlib pulp ortools jupyterlab
```

3. Replace `crew.csv` / `flight_pairs.csv` with your data (keep schema) and re-run.

**Experiments & expected outputs**
- The notebook includes baseline vs. optimized scenario runs, with tabular summaries and plots for utilization, cost components, and constraint shadow prices where available.
- Example plots: crew utilization histogram, heatmap of base-level under/over-supply, and a timeline view of selected pairings for a sample crew.

**Interpreting results**
- Look for improvements in the KPIs above and inspect constraint violations or tight constraints to surface operational risks.
- Use sensitivity scenarios (e.g., delay injections, reduced crew availability) to evaluate robustness.

**Extensions & next steps**
- Add realistic cost elements: overtime multipliers, hotel & transportation costs, fatigue penalties.
- Introduce stochastic modeling or robust optimization to capture delay risk.
- Build APIs to export optimized pairings to rostering systems or interactive dashboards for planners.

**Privacy & data considerations**
- This repository uses anonymized sample data. For production, follow your organization’s PII and security guidelines when handling crew identifiers and schedules.

**Contributing & contact**
- To propose changes or improvements, open an issue or a pull request. For larger integrations (e.g., solver licensing, production deployment), contact the repository owners through the issue tracker.

**License**
- See `LICENSE` for repository terms.

