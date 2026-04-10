# Pricing Earthquake Prediction Market Contracts

## Overview

This project investigates the prices of contracts on [Polymarket](https://polymarket.com/event/how-many-6pt5-or-above-earthquakes-april-6-12) that ask: *"How many magnitude 6.5+ earthquakes will occur in a given week?"*

The key insight is that earthquake counts $N(t)$ for magnitudes 6.5+ roughly follow a **homogeneous Poisson process**, which lets us update probabilities in real time as the week progresses and earthquakes are observed.

---

## The Basics of Earthquakes

The distribution of earthquake timings and magnitudes can be thought of as a marked point process with the time $T$ being the location and the magnitude $M$ serving as the mark. A few stylized facts about this distribution are as follows:

1. **The Gutenberg-Richter Law.** The number of earthquakes drops roughly exponentially as magnitude increases. Typically, an increase in magnitude by 1 leads to a drop by a factor of 10.
2. **Omori's Law.** An earthquake is usually followed by smaller earthquakes known as *aftershocks*. Earthquakes are thus clustered in time and not evenly spread out. Larger earthquakes tend to trigger more aftershocks. The frequency of aftershocks after an earthquake decreases quickly with time at a roughly geometric rate.
3. **Bath's Law.** The largest aftershock after an earthquake is about 1.2 units smaller in magnitude.
4. **Background Seismicity.** Over long periods of time, earthquake timings are roughly a homogenous Poisson process.

---

## Pricing the contract

We are essentially trying to estimate the probablity that the number of earthquakes of magnitude greater than or equal to 6.5 when $t = 0, 1,, \dots, 6$ days are left is $k = 0, 1, 2.$ 

Empirical analysis reveals that earthquakes of magnitude ≥6.5 are relatively rare and the probabilty that there will be none in a given week is about 0.50. In the event there is an earthquake of ≥6.5, its aftershocks are unlikely to be ≥6.5 unless its magnitude is ≥7.7 according to Bath's law. But earthquakes of such magnitude are even rarer and the probability of one happening in a given week is less than 0.05. Therefore, homogenous Poisson process provides a good baseline for in more than 95% of cases.

In the event of a large enough earthquake (≥7.7) this dynamic changes, as Omori's law will kick in and the probability of another earthquake of ≥6.5 before the end of the week decays geometrically instead of exponentially as implied by the Poisson model. In such an event, the Poisson model would overestimate the probability of no earthquakes of ≥6.5 and underestimate the probability of 1 or more earthquakes of ≥6.5. To account for this, we can develop a more sophisticated prediction procedure by fitting Omori's law after a large earthquake (≥7.7). Alternatively, we can try to mitigate our exposure to large earthquakes by [buying another contract.](https://polymarket.com/event/how-many-7pt0-or-above-earthquakes-by-june-30) 

---

## Analysis and Fit

The following table depicts the empirical probability of an earthquake of magnitude ≥M happening in a given period of length $k=$1, 2,..., 7 days.

| Min Magnitude M | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 4.5 | 1.00 | 1.00 | 1.00 | 1.00 | 1.00 | 1.00 | 1.00 |
| 5.0 | 0.980 | 1.00 | 1.00 | 1.00 | 1.00 | 1.00 | 1.00 |
| 5.5 | 0.663 | 0.889 | 0.961 | 0.993 | 0.999 | 1.00 | 1.00 |
| 6.0 | 0.285 | 0.483 | 0.639 | 0.732 | 0.815 | 0.872 | 0.887 |
| 6.5 | 0.0967 | 0.182 | 0.264 | 0.331 | 0.389 | 0.444 | 0.495 |
| 7.0 | 0.0353 | 0.0690 | 0.101 | 0.133 | 0.163 | 0.199 | 0.223 |
| 7.5 | 0.0120 | 0.0241 | 0.0362 | 0.0482 | 0.0603 | 0.0724 | 0.0845 |
| 8.0 | 0.00190 | 0.00380 | 0.00580 | 0.00770 | 0.00960 | 0.0115 | 0.0134 |

![Table](empirical_prob_table.png)

We estimate the Poisson rate $\lambda$ to be 0.77. The price of the contract at time $t$ and total number of earthquakes $k$ after seeing $r$ significant earthquakes as
$$p(t, k, r) = \frac{1}{(k-r)!}e^{-\lambda(T-t)}[\lambda(T-t)]^{k-r}$$
where $T$ is the expiry. The market prices appears to be reasonably close to the empirical and Poisson estimates. As of now, the price descrepancy for $k=0$ is about 3.4% and the bid-ask spread is about 2%. 

![Prices and Probabilites](price_vs_model.png)

---

## Data Source

Historical earthquake data can be obtained from the [USGS Earthquake Catalog](https://earthquake.usgs.gov/earthquakes/search/).


## Acknowledgements

The author would like to thank Dr. Tomas Rubin for the idea and Claude code for the implementation.