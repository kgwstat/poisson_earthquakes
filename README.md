# Pricing Earthquake Prediction Market Contracts

## Overview

This project investigates the prices of contracts on [Polymarket](https://polymarket.com/event/how-many-6pt5-or-above-earthquakes-april-6-12) that ask: *"How many magnitude 6.5+ earthquakes will occur in a given week?"*

The key insight is that earthquake counts $N(t)$ roughly follow a **homogeneous Poisson process**, which lets us update probabilities in real time as the week progresses and earthquakes are observed.

---

## The Basics of Earthquakes

We are essentially concerned with modelling the distribution of the set of random points $\{(T_{j}, M_{j})\}_{j=1}^{n}$ where $T_{j}$ and $M_{j}$ denote the timing and magnitude of the earthquak $j$. In other words, the distribution of earthquake timings and magnitudes can be thought of as a marked point process with the time $T$ being the location and the magnitude $M$ serving as the mark. A few stylized facts about this distribution are as follows:

1. **The Gutenberg-Richter Law.** The number of earthquakes drops roughly exponentially as magnitude increases. 
$$\log_{10} N(M \geq m) \approx a - bm$$
2. **Omori's Law.** An earthquake is usually followed by smaller earthquakes known as *aftershocks*. Earthquakes are thus clustered in time and not evenly spread out. Larger earthquakes tend to trigger more aftershocks. The number of aftershocks after an earthquake at time $t = 0$ satisfies 
$$N(t \leq T \leq t + \Delta t) \propto \frac{\Delta t}{(c + t)^{p}}$$
The rate of aftershocks thus decreases quickly with time.

3. **Background Seismicity.** Over long periods of time, earthquake timings are roughly a homogenous Poisson process.
$$\mathbb{P}\{N(T \leq t) = n \} \approx \frac{(\lambda t)^{n}}{n!}e^{-\lambda t}$$

---

## The Contract

We are essentially trying to estimate
$$\mathbb{P}\{N(T \leq t, M \geq 6.5) = k\}$$
for $k = 1, 2, 3, \dots$

---

## A Baseline

If a long time has elapsed after a large enough earthquake has taken place then the homogeneous Poisson process seems like a good baseline forecast. However, as soon as there is a large earthquake, one needs to take into account the possibility of there being aftershocks of magnitude 6.5+. 

It so happens that empirically speaking earthquakes of magnitude 6.5+ are relatively rare and there is about 50% chance that there will be none in a given week. 

---

### Estimating r

r is estimated from historical USGS data — the average rate of magnitude 6.5+ earthquakes globally. For example, if history shows roughly 1.5 such events per week:

```
r = 1.5 / 7 ≈ 0.214  events/day
λ = r · T = 1.5       (for a full 7-day window)
```

---

## Dynamic Pricing as Time Evolves

The central question is: **how should the fair price of each contract change as we move through the week?**

At time **t** (days elapsed since the contract start), suppose we have observed **n = N(t)** earthquakes so far. The remaining time is **T - t**, during which additional earthquakes arrive as an independent Poisson process:

```
N(T) - N(t)  ~  Poisson(r · (T - t))
```

Let **μ(t) = r · (T - t)** be the expected remaining count. The fair price of each outcome bucket at time t, given N(t) = n, is:

### Exact bucket "exactly k earthquakes total"

```
P(N(T) = k | N(t) = n) = 0                                    if k < n
                        = e^(-μ) · μ^(k-n) / (k-n)!           if k ≥ n
```

In other words, conditional on n events already observed, we only need **k - n** more events from a `Poisson(μ(t))` distribution.

### "3 or more" bucket

```
P(N(T) ≥ 3 | N(t) = n) = 1                                    if n ≥ 3
                        = 1 - Σ_{k=n}^{2} e^(-μ) · μ^(k-n) / (k-n)!   if n < 3
```

### Fair Price Table at Time t (given n observed so far)

| Outcome | Fair Price |
|---------|------------|
| 0 total | 0 if n ≥ 1, else e^(-μ) |
| 1 total | 0 if n ≥ 2, else e^(-μ)·μ^(1-n) / (1-n)! |
| 2 total | 0 if n ≥ 3, else e^(-μ)·μ^(2-n) / (2-n)! |
| 3+ total | 1 if n ≥ 3, else 1 - P(0 total) - P(1 total) - P(2 total) |

where **μ = r · (T - t)** shrinks to zero as t → T.

---

## How Prices Evolve

The dynamics have intuitive structure:

- **As t → T with no new earthquakes**: μ → 0, so lower-count outcomes become near-certain and higher-count outcomes collapse to 0.
- **Each observed earthquake**: eliminates all outcomes below n, concentrating probability mass on outcomes ≥ n.
- **At t = 0**: prices are just the prior Poisson(λ) probabilities — no information yet.
- **At t = T**: prices are degenerate — the outcome is known with certainty.

This means prices can move sharply after each earthquake event and decay smoothly in between.

---

## Mispricing and Edge

At any point during the week, compare model-derived fair prices against current market prices. For example:

- If the "0" contract still trades at 10¢ after an earthquake has already occurred, it is mispriced — it is now worth exactly 0¢.
- If the "3+" contract trades at 20¢ but the model implies P(N(T) ≥ 3 | N(t) = 2, μ = 0.5) ≈ 39%, it is underpriced — buy it.

---

## Key Assumptions

1. **Stationarity** — the rate r is constant throughout the week (no time-of-day or aftershock effects).
2. **Independence** — future arrivals are independent of past arrivals given the rate (memoryless property of Poisson processes). This ignores aftershock clustering; an ETAS model would be more accurate but adds complexity.
3. **Historical rate is representative** — r estimated from past data applies to the current contract window.

---

## Data Source

Historical earthquake data can be obtained from the [USGS Earthquake Catalog](https://earthquake.usgs.gov/earthquakes/search/).
