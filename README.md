# Charger Utilisation & Efficiency Analysis

Measuring how much the EV charging network is used, how long chargers sit idle, where load peaks, and which stations or chargers are bottlenecks, to optimise infrastructure and reduce congestion.

## The Question

Is the charging network the right size? This project measures utilisation and idle time for every charger and station, finds peak-load periods and bottleneck stations, ranks chargers, and estimates how much capacity is actually needed.

## Repo Structure

```
├── notebooks/        → full analysis notebook
├── output/           → CSV outputs 
├── dashboard/        → Power BI .pbix file + page screenshots
├── slides/           → presentation deck (PPTX)
└── documentation/    → project documentation and executive summary (PDF)
```

## Data

The source is the EV charging dataset: `stations` (33 stations), `chargers` (199 chargers: 95 rated 50 kW, 69 rated 150 kW, 35 rated 300 kW) and `sessions` (294,024 sessions, January 2022 – December 2024). The raw file is not included in this repo. It is hosted as a Kaggle Dataset: https://www.kaggle.com/datasets/sanyazuberi/ev-charging-raw-dataset.

Output files (in `output/`):
- `charger_metrics.csv`: one row per charger: utilisation, sessions, energy and revenue per day, power ratio, capacity factor, utilisation rank
- `station_summary.csv`: one row per station: utilisation, peak-window utilisation, share of time near-full, and chargers needed vs installed
- `utilisation_station_by_hour.csv`: station × hour-of-day utilisation (the station heatmap data)
- `utilisation_network_weekday_hour.csv`: network utilisation by weekday and hour
- `network_load_by_hour_2024.csv`: average network load (kW) by hour of day, weekday vs weekend

## How Utilisation Was Measured

- **Utilisation** is the share of a charger's available time that it is in use. **Idle time** is the remainder.
- **Available time** runs from 1 January of the station's activation year to the end of 2024. The `installation_date` field was not used, because it conflicts with session activity (30% of sessions occur before their charger's recorded installation date). Every station's first session falls in its activation year, which supports this choice.
- **Overlapping sessions were merged.** 24,449 sessions (8.3%) start before an earlier session on the same charger has finished, and 3,808 start at the exact same moment as another on that charger. Adding up session durations would overstate busy time by 5.2%, so overlapping sessions were merged into continuous busy periods before calculating utilisation.
- In 99.8% of the overlapping cases another charger at the same station was free, so the overlaps are an allocation or logging problem, not a capacity shortage.

## Key Findings

- **The network has far more capacity than demand.** Utilisation is 5.4%, so the average charger is idle for 22.7 of 24 hours. Even the busiest hour of the week (Monday 11:00) is only 11.5%.
- **Use is concentrated at midday.** Weekday midday (10:00–14:59) averages 11.2% utilisation, weekend midday 8.4%, and overnight 1.2%. That window delivers 40.7% of daily energy.
- **Station size and charger type drive utilisation.** Every station receives about 13.9 sessions a day whatever its size, so utilisation falls from 8.5% at 4-charger stations to 4.2% at 8-charger stations. 50 kW chargers average 7.4% against 3.2% for 300 kW chargers, because slower chargers are occupied longer per session.
- **Congestion is almost non-existent.** Only 14 of 33 stations ever have every charger busy at once, and the highest share of time for any station is 0.061% (about 5 hours a year). The nine 4-charger stations come closest, led by STN_030 and STN_022 in London.
- **Peak load is a small share of installed power.** The network averaged 882 kW in 2024 (3.4% of 25,600 kW installed). The weekday peak, at 13:00, averages 1,882 kW (7.4%), and the highest single hour reached 2,796 kW (10.9%).
- **Rankings are real for groups, not individual chargers.** Charger type and station size explain 99.7% of the differences in utilisation. Among chargers of the same type at the same station size, rankings do not repeat between odd and even months (correlation −0.09).

## Ranking

| Ranked by | Highest group | Lowest group |
|---|---|---|
| Utilisation | 50 kW at 4-charger stations: 11.0% | 300 kW at 8-charger stations: 2.3% |
| Revenue per charger per day | 300 kW at 4-charger stations: £153 | 50 kW at 8-charger stations: £34 |

Utilisation is not the same as earnings: the top 10 chargers by utilisation earn £67–73 a day, while the bottom 10 earn £72–80, because each session on a 300 kW charger delivers more energy. The notebook gives the full top and bottom 10 chargers and the ranking of all 15 groups.

## Capacity

Using how many chargers are busy at once, the smallest number that would cover each station's demand was compared with what is installed:

| Demand covered | Chargers needed (network) | Installed |
|---|---|---|
| 99% of the time | 70 | 199 |
| 99.9% of the time | 101 | 199 |
| 99.99% of the time | 127 | 199 |

At the 99.9% standard almost every station needs only 3 chargers, and only two stations (STN_009 and STN_023, both 8-charger stations) need 4.

## Recommendations

1. **Do not add chargers now.** Utilisation, congestion and peak load all show spare capacity.
2. **Watch the nine 4-charger stations first.** They are the first candidates for expansion if demand per station grows.
3. **Right-size the 8-charger stations** and plan 4–5 chargers for new sites. The surplus (about 98 chargers at the 99.9% standard, range 72–129) is indicative only.
4. **Review the need for 300 kW units before buying more.** They are 17.6% of chargers and 41% of installed power but deliver 33% of the energy. A decision needs capital-cost data this dataset does not contain.
5. **Pilot a change to the peak-price window.** Only 23.6% of energy falls in the 16:00–20:59 premium window, against 40.7% at the midday peak.
6. **Fix charger allocation** so each session goes to a free charger.
7. **Use group tiers, not individual charger rankings.**

## Dashboard

Built in Power BI, four pages:
- **Utilisation Heatmaps**: network (weekday × hour) and station (station × hour) heatmaps
- **Idle Time & Peak Load**: idle share by hour and average network load by hour
- **Bottlenecks & Capacity**: time near-full by station, installed vs needed chargers, and a bottleneck ranking table
- **Charger Ranking**: top and bottom 10 chargers, utilisation by charger type and station size, and revenue per charger by type

See `dashboard/` for the `.pbix` file and page screenshots.

## Reproducing This Project

1. Download the raw dataset from the Kaggle link above.
2. Run the notebook in `notebooks/` from top to bottom in a fresh session. It writes the CSV files and the `figures/` folder.
3. Open the `.pbix` in `dashboard/` and refresh the data sources if the CSV paths have changed.

## Tools Used

Python (pandas, NumPy, SciPy, Matplotlib, Seaborn) · Kaggle Notebooks · Power BI

## Limitations

- The dataset appears simulated. Each station receives about 13.9 sessions a day regardless of its size, so station size largely determines utilisation by construction.
- The "chargers needed" estimate assumes demand stays at 2022–2024 levels, and leaves no buffer for maintenance or faults.
- There is no fault or downtime data, so long idle spells cannot be classed as outages.
- Available time assumes each station operates from 1 January of its activation year, because `installation_date` is unreliable.
- Load spreads each session's energy evenly over its duration and is measured at hourly resolution.
