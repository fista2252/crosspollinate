# Crosspollinate: Serverless cold-start latency

**Problem (abstracted):** Keep a dormant resource ready to serve instantly when demand arrives in unpredictable bursts, without paying to keep it running.
**Core contradiction:** Keeping capacity warm makes first responses fast but costs money while idle. Scaling to zero is cheap, but the first request waits 2-4 s.
**Fields searched:** hibernation physiology, microbial and plant bet hedging, power-grid unit commitment, cold-climate automotive engineering, EMS ambulance deployment, inventory management

## Top ideas

### 1. Off-time-dependent start states — from power-grid unit commitment  (fit 5 · novelty 4 · credibility 5)
Unit-commitment models give each plant a temperature, so start-up cost depends on how long it has been off: a still-hot plant restarts cheaper and faster than a cold one. GE goes further and blows warming air through turbines after shutdown to keep a hot restart possible. For your functions, replace the binary provisioned/evicted choice with a decaying "temperature": warm instance → memory snapshot → disk snapshot → cold. Each step costs less to hold and more to restart.
Source: https://arxiv.org/abs/1408.2644 · https://patents.google.com/patent/US8210801B2/en

### 2. Learned-schedule pre-activation — from cold-climate automotive engineering  (fit 5 · novelty 3 · credibility 5)
Honda's ECU records every engine start, spots patterns like "9:00 on weekdays", and heats the air-fuel sensor a set time before the predicted start. Block heaters run on timers for the same reason: a few hours before the start is enough. Learn each function's recurring call times and pre-warm one cold-start duration ahead of the predicted call instead of holding capacity all day.
Source: https://patents.google.com/patent/US8055438B2/en · https://en.wikipedia.org/wiki/Block_heater

### 3. Bet hedging with a dormant fraction — from microbial and plant biology  (fit 4 · novelty 5 · credibility 5)
Plants keep part of their seed bank dormant as insurance against drought, and yeast keep a slow-growing subpopulation that pre-expresses the stress protectant Tsl1 and survives sudden heat. Neither population goes all-or-nothing. Pre-warm a fixed fraction of instances across your functions, sized by what a cold burst costs you, rather than provisioning everything or nothing.
Source: https://en.wikipedia.org/wiki/Bet_hedging_(biology) · https://doi.org/10.1371/journal.pbio.1001325

### 4. Tiered spinning and non-spinning reserves — from power-grid unit commitment  (fit 5 · novelty 2 · credibility 5)
Grids hold spinning reserve (online, instant) and non-spinning reserve (offline, online after a short delay), and they weigh having enough against overpaying for rarely used capacity. Keep a minimal hot pool for instant traffic and let a fast-start tier take the overflow. Open solvers such as UnitCommitment.jl show how to co-optimise the two tiers.
Source: https://en.wikipedia.org/wiki/Operating_reserve · https://github.com/ANL-CEEESA/UnitCommitment.jl · https://news.ycombinator.com/item?id=28889748

### 5. Demand-posted standby — from EMS ambulance deployment  (fit 4 · novelty 4 · credibility 5)
EMS services split the week into 168 hourly slots, forecast calls per slot, and post ambulances to match. Newer work redeploys idle units in real time as calls come in. Set provisioned concurrency per function per hour-of-week from your logs, and move warm capacity between functions as load shifts.
Source: https://patents.google.com/patent/US6058370A/en · https://doi.org/10.2139/ssrn.3009043 · https://github.com/krishayyy/ems-relocation

### 6. A dedicated burst organ for waking up — from hibernation physiology  (fit 3 · novelty 5 · credibility 4)
Hibernating hamsters rewarm with brown fat, whose heat output rises in the cold, and stimulating it shortened arousal. Give the init phase its own short burst of CPU and I/O, separate from steady-state limits, and spend it on loading your 400 MB bundle.
Source: https://doi.org/10.1152/ajpregu.00053.2011

### 7. Safety-stock sizing — from inventory management  (fit 4 · novelty 2 · credibility 4)
Planners hold safety stock of z·σ·√L above forecast demand to cover the restock lead time. Size the warm pool the same way, with the lead time set to your cold-start duration. Watch out: bursty traffic is heavier-tailed than the normal demand the formula assumes.
Source: https://en.wikipedia.org/wiki/Safety_stock · https://github.com/durak2005/dynamic-safety-stock-simulator

## Try first
Replay a week of invocation logs through a small simulator that compares today's provisioned concurrency with a warm → snapshot → cold decay (idea 1), and plot p99 start latency against GB-seconds held. If the decay wins, add learned pre-activation (idea 2) on top.
