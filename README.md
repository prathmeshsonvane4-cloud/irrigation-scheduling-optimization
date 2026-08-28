# Irrigation Scheduling Optimisation under Variable Rainfall

A linear programming model that allocates irrigation water between reservoir storage and groundwater across a rice growing season, minimising groundwater draw while meeting crop demand.

**M.Tech project, Systems Techniques in Water Resources Engineering, Indian Institute of Science (IISc) Bengaluru.**

---

## The problem

India is the world's largest user of groundwater, and a farmer with a pond, a borewell and an unpredictable monsoon has to decide every day which source to draw from. Draw too much groundwater and the aquifer depletes. Rely on stored surface water alone and the crop fails in a dry spell.

That is an allocation problem with hard constraints, which makes it a natural fit for linear programming.

---

## Setup

| | |
|---|---|
| Field | 1 acre, rice |
| Location | Bhandara district, Maharashtra (21.20 N, 80.19 E) |
| Season | 1 June to 31 October 2020, 153 days |
| Rainfall | NASA POWER daily corrected precipitation (PRECTOTCORR) |
| Crop demand | 10 mm/day, from FAO guidance of 1200 to 1400 mm per rice season |
| Solver | PuLP |

Daily irrigation demand is crop water requirement minus rainfall, floored at zero on days when rain exceeds demand.

## Model

**Decision variables**, per day: reservoir use, groundwater use, deficit, reservoir storage.

**Objective**

```
minimise  sum( groundwater_use + penalty * deficit )
```

The penalty on deficit is set high, so meeting crop demand takes priority. Subject to that, the model avoids groundwater wherever rainfall and stored surface water can cover the requirement.

**Constraints**

```
reservoir_use + groundwater_use + deficit  >=  irrigation_demand     demand satisfaction
groundwater_use                            <=  max_pumping_limit     abstraction cap
storage(t)  =  storage(t-1) + rainfall - reservoir_use               storage balance
0  <=  storage  <=  max_capacity                                     reservoir capacity
reservoir_use + groundwater_use            <=  irrigation_demand     no over-irrigation
all variables >= 0
```

---

## Results

| | |
|---|---|
| Total crop water demand | 103.51 mm |
| Irrigation supplied | 103.51 mm |
| Total deficit | 0 mm |
| Average reservoir storage | 130.48 mm |
| Peak reservoir storage | 226.96 mm |

Demand was met in full with no deficit. The more useful result is *how*: the optimiser leans on rainfall first, draws stored surface water second, and reaches for groundwater only when neither covers the day's requirement. Storage builds through wet spells and draws down through dry ones, which is the behaviour you would want from a real operating rule.

**Worth being honest about:** zero deficit means the constraints were not binding for this site and season. There was enough water. The model would be more interesting under tighter groundwater caps, a smaller pond, or a drought year, and that is the obvious next test.

---

## Repository

```
notebook/   irrigation_lp_model.ipynb   PuLP formulation, solve, plots
data/       nasa_power_bhandara_2020.csv   daily precipitation, NASA POWER
            S1_Bhandara_2020.csv
report/     full project report
```

Built with Python, PuLP, pandas and matplotlib.

---

## Data sources

| Source | Where |
|---|---|
| NASA POWER daily meteorology | [power.larc.nasa.gov](https://power.larc.nasa.gov/) |
| Crop water requirement guidance | [FAO](https://www.fao.org/) |

Both are open and small enough to commit, so this repository reproduces end to end.

---

## Credits

Author: **Prathmesh Sonvane**, M.Tech Water Resources and Environmental Engineering, IISc Bengaluru
Course: Systems Techniques in Water Resources Engineering, under Prof. D. Nagesh Kumar
