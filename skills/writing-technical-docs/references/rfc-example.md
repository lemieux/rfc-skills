<example_rfc>
<!--
IMPORTANT: This is a STYLE EXAMPLE only.
DO NOT use any content from this document in your RFC.
Study the STRUCTURE, TONE, and FORMAT, not the subject matter.
The topic (transit route optimization, MetroLink) is fictional and irrelevant to your task.
-->

# RFC: Dynamic Route Optimization for MetroLink

**Author:** Transit Technology Team
**Created:** February 2026

## Draft Status

**State:** Draft

**Items for review:**
- [ ] <!-- REVIEW: The 15-minute optimization interval is an initial estimate. Operations team should validate whether dispatchers can absorb route changes at this frequency during peak hours. -->
- [ ] <!-- REVIEW: ADA vehicle minimum counts per route tier need confirmation from the compliance department before we lock hard constraints in the solver. -->

## Abstract

This RFC proposes a dynamic route optimization system for MetroLink that adjusts bus routes in near-real-time based on passenger demand, vehicle positions, and traffic conditions. The design introduces a constraint satisfaction solver that produces candidate route adjustments, an adapter layer that integrates with the existing dispatch infrastructure, and a feedback loop that continuously improves demand predictions. All route changes require dispatcher approval before execution, preserving human oversight while capturing institutional knowledge that currently exists only in radio communications.

## Glossary

| Term | Definition |
|------|------------|
| **MetroLink** | The city's public transit authority operating bus service across the metropolitan area. Manages approximately 50 routes with 400 vehicles. |
| **CAD/AVL** | Computer-Aided Dispatch / Automatic Vehicle Location. MetroLink's existing system for tracking buses and communicating with drivers via radio. Contractually tied to the current radio vendor. |
| **Headway** | The time interval between consecutive buses on the same route. Lower headway means more frequent service. |
| **Deadhead** | Miles driven by a bus without passengers, typically when repositioning between routes or returning to the depot. |
| **Constraint Satisfaction** | A computational approach that finds solutions meeting a set of mandatory (hard) and preferred (soft) requirements. The solver explores candidate solutions and ranks them by how well they satisfy soft constraints while never violating hard ones. |

## Background

MetroLink operates approximately 50 bus routes serving 400 vehicles and 150,000 daily riders across the metropolitan area. The network spans 340 route-miles covering both the dense downtown grid and suburban radial corridors. Route planning happens on a quarterly cycle, driven primarily by spreadsheets that the scheduling team maintains. A team of four planners spends roughly six weeks each quarter analyzing ridership reports, adjusting timetables, and coordinating with the workforce scheduling group to match driver assignments to the updated routes. Dispatchers at the operations center monitor service in real time and make adjustments over radio when problems arise, such as bunching on a corridor or a driver calling in sick.

This process worked well when ridership followed predictable commute patterns, with morning peaks into downtown and evening peaks outbound. Over the past three years, ridership has shifted significantly. Event-driven travel (sports venues, concerts, weekend markets) now accounts for a growing share of trips, while traditional 9-to-5 commute ridership has declined. The quarterly planning cycle cannot keep pace with these changes, which means static routes sometimes run nearly empty on segments that used to be packed while overcrowding builds on corridors that were historically quiet.

Dispatchers compensate by making ad-hoc adjustments: pulling a bus off one route to reinforce another, short-turning a vehicle to cover a gap, or asking a driver to skip low-ridership stops during a surge. These decisions are often effective in the moment, but they happen over radio and are not recorded in any system. The scheduling team has no visibility into what dispatchers changed or why, so the same problems recur quarter after quarter. The institutional knowledge lives in the heads of experienced dispatchers, which creates risk as veteran staff retire.

MetroLink has invested in Automatic Passenger Counter (APC) sensors on all vehicles and GPS tracking through the CAD/AVL system. These data sources exist but are only used for after-the-fact reporting. The APC data is aggregated into monthly ridership summaries for the board, and the GPS data feeds the real-time vehicle map that dispatchers monitor, but neither data stream influences the quarterly route planning process in any structured way.

The city also publishes a traffic conditions API as part of its open data initiative, providing average travel speeds on major corridors updated every five minutes. MetroLink has not previously consumed this data, but it represents a valuable signal for predicting service delays before they cascade across the network. The opportunity is to close the loop: use real-time data to inform route adjustments, capture those adjustments systematically, and feed outcomes back into planning.

## Problem Statement

Three problems prevent MetroLink from delivering efficient service as ridership patterns change:

1. **Static routes cannot adapt to real-time demand.** Routes are planned quarterly based on historical averages, so they lag behind actual ridership shifts by weeks or months. When a new event venue opens or a major employer changes its hybrid work schedule, routes continue running their old patterns until the next planning cycle. Even within a single day, demand shifts hour by hour in ways that a fixed timetable cannot reflect.

2. **No feedback loop between actual ridership and planning.** APC sensors collect boarding and alighting data on every trip, but this data flows into quarterly reports rather than influencing day-to-day operations. The scheduling team sees monthly averages aggregated at the route level, not the stop-level, time-of-day variance that dispatchers deal with every shift. A route that averages 800 riders per day might swing between 500 and 1,200 depending on whether a downtown event is scheduled, but the quarterly planning process sees only the 800.

3. **Dispatcher adjustments are invisible to the system.** When a dispatcher reroutes a bus or adjusts frequency, the decision happens over radio and is never recorded. These adjustments represent real operational intelligence about what the network needs, but they vanish after each shift. The planning process starts from scratch each quarter without learning from thousands of in-the-moment decisions. Internal estimates suggest dispatchers make 40 to 60 ad-hoc adjustments per weekday across the network, each one an implicit signal about where the published schedule fails to match reality.

## Proposal

### Architecture Overview

The system introduces three components that sit between MetroLink's existing data sources and the CAD/AVL dispatch system. The Route Solver ingests real-time data and produces candidate adjustments. The Dispatch Adapter translates those candidates into commands the CAD/AVL system can execute. The Feedback Collector compares predictions against outcomes and refines the solver's demand model over time.

```
┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│  APC Sensors  │  │  Vehicle GPS  │  │  Traffic API  │  │  Workforce   │
│  (ridership)  │  │  (positions)  │  │  (city data)  │  │  (schedules) │
└──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘
       │                 │                 │                 │
       ▼                 ▼                 ▼                 ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        ROUTE SOLVER                                │
│  Constraint satisfaction engine producing candidate adjustments    │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      DISPATCH ADAPTER                              │
│  Translates solver output into CAD/AVL commands                    │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    EXISTING CAD/AVL SYSTEM                         │
│  Radio-based dispatch (vendor contract through 2030)               │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     FEEDBACK COLLECTOR                             │
│  Predicted vs actual ridership comparison (hourly batch)           │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             └──────────▶ Back to Route Solver
```
*Architecture overview: data flows from sensors and external sources through the Route Solver, into the existing dispatch system via the Adapter, with the Feedback Collector closing the loop back to the solver.*

### Route Solver

The Route Solver is a constraint satisfaction engine that runs every 15 minutes during peak service hours (6:00 AM to 9:00 AM and 4:00 PM to 7:30 PM on weekdays). <!-- REVIEW: The 15-minute optimization interval is an initial estimate. Operations team should validate whether dispatchers can absorb route changes at this frequency during peak hours. --> During off-peak hours it runs every 30 minutes, since demand shifts more slowly and dispatchers have more bandwidth.

The solver consumes four data inputs. Passenger demand comes from APC sensors as a heatmap of boardings and alightings per stop over the last hour, weighted toward the most recent 15 minutes. Vehicle positions come from GPS transponders reporting through the CAD/AVL system every 30 seconds. Traffic conditions come from the city's open data API, which publishes average corridor speeds every 5 minutes. Driver shift schedules come from the workforce management system, including break windows, shift end times, and route certifications.

Given these inputs, the solver produces a ranked list of candidate route adjustments. Each candidate specifies which vehicles to reassign, which route segments to increase or decrease in frequency, and the expected impact on passenger wait times and deadhead miles.

The solver enforces two categories of constraints. Hard constraints are inviolable: driver shift limits (no extension beyond scheduled end without union approval), ADA-accessible vehicle minimums per route tier, and maximum headway thresholds that differ by route classification. <!-- REVIEW: ADA vehicle minimum counts per route tier need confirmation from the compliance department before we lock hard constraints in the solver. --> Tier 1 routes (high frequency trunk lines) must maintain headways of 10 minutes or less, while Tier 3 routes (neighborhood connectors) allow headways up to 30 minutes. Soft constraints are goals the solver optimizes toward: minimizing average passenger wait time across the network and minimizing deadhead miles, with configurable weighting between the two.

```
                    ┌─────────────────────┐
                    │   COLLECT INPUTS    │
                    │  APC + GPS + Traffic │
                    │  + Shift Schedules   │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │  GENERATE CANDIDATES│
                    │  Constraint solving  │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │   RANK CANDIDATES   │
                    │  Score by soft       │
                    │  constraint fitness  │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
              No    │  DISPATCHER REVIEW  │  Yes
           ◄────────│  Accept top         ├────────►
           │        │  candidate?         │        │
           ▼        └─────────────────────┘        ▼
  ┌────────────────┐                    ┌────────────────┐
  │  LOG REJECTION  │                    │ SEND TO ADAPTER│
  │  (with reason)  │                    │ (execute)      │
  └────────────────┘                    └────────────────┘
```
*Optimization cycle: the solver generates and ranks candidates, then presents the top option to the dispatcher for approval before execution.*

The solver does not act autonomously. Every candidate adjustment goes to a dispatcher dashboard for review before execution. The dispatcher can accept, modify, or reject each suggestion, and rejection reasons are captured to improve future recommendations. This human-in-the-loop design was a deliberate choice for two reasons. First, MetroLink's board requires that a human authorize any change to published routes, since route changes affect public commitments to riders. Second, dispatchers have situational awareness that the solver cannot access: a parade blocking a street, a broken-down bus that has not yet been reported in the system, or a driver who radioed in about a medical issue.

The dashboard presents candidates with enough context for a quick decision. Each recommendation shows the affected routes on a map, the current versus proposed headways, the vehicles being reassigned and their current positions, and the projected impact on passenger wait times. Dispatchers in our early usability testing were able to evaluate and act on a recommendation in under 90 seconds.

### Dispatch Adapter

The Dispatch Adapter translates solver output into commands that the existing CAD/AVL system understands. This layer exists because MetroLink is contractually committed to the current radio vendor and CAD/AVL platform for four more years, which means we cannot replace the dispatch system or modify its internal protocols. The adapter sits alongside the CAD/AVL system and communicates through the vendor's published integration API, which supports a limited set of operations: reassign vehicle to route, update scheduled departure time, and send text message to driver's mobile data terminal.

The adapter maps solver-produced adjustments to sequences of these operations. A frequency increase on Route 12, for example, becomes a series of vehicle reassignments pulling buses from lower-demand routes, updated departure times for the reassigned vehicles, and text messages notifying affected drivers. The adapter also handles the reverse: when the solver recommends reducing frequency, it generates the commands to return borrowed vehicles to their original routes.

Because the CAD/AVL integration API processes commands sequentially and has a rate limit of 10 commands per minute, the adapter queues and batches operations. If the solver produces a complex adjustment requiring 25 commands, the adapter estimates completion time and reports it to the dispatcher dashboard so operators know when the change will take full effect.

The adapter also handles error recovery when the CAD/AVL system rejects commands. A command might be rejected because a vehicle has already been reassigned by a dispatcher acting independently, or because a driver has gone off-duty since the solver last checked shift data. When a rejection occurs, the adapter marks that command as failed, notifies the dashboard with the reason, and continues with the remaining commands in the batch. It does not attempt to recompute the optimization, since the solver will incorporate the new vehicle state in its next cycle. This fail-forward approach prevents a single rejected command from blocking an entire adjustment sequence.

The adapter maintains a command audit log that records every command sent to the CAD/AVL system, its outcome, and the associated optimization ID. This log serves two purposes: it gives the operations team a clear record of what the system changed and when, and it provides training data for improving the solver's awareness of which types of adjustments the CAD/AVL system is likely to reject.

### Feedback Collector

The Feedback Collector closes the loop between predictions and reality. It runs as an hourly batch job that compares the Route Solver's predicted ridership per route segment against actual boardings and alightings recorded by APC sensors during the same period. The output is a set of correction factors that adjust the solver's demand model, making future predictions more accurate for each corridor and time window.

The collector also tracks dispatcher rejection patterns. If dispatchers consistently reject adjustments for a particular route or time of day, the collector flags this so the team can investigate whether the solver's model is missing a constraint or whether the dispatchers need additional training on the system's capabilities.

Corrections feed into the solver's demand model as exponentially weighted moving averages, with recent observations carrying more influence than older ones. This means the system adapts to gradual shifts (a new office building filling up over months) without overreacting to one-time anomalies (a transit strike in a neighboring city causing a single-day spike).

The collector maintains separate correction profiles for weekdays, Saturdays, and Sundays, since demand patterns differ substantially across these categories. It also tracks event days separately, building a library of event profiles that the solver can reference when a similar event appears on the calendar. Over time, the system should predict demand for a downtown basketball game with reasonable accuracy based on historical data from prior games at the same venue, adjusted for day of week and season.

### API Contracts

#### Request Route Optimization

When the optimization interval triggers (or a dispatcher requests an ad-hoc optimization), the system invokes the solver.

**POST /routes/optimize**

Request:
```json
{
  "region": "downtown-core",
  "timestamp": "2026-02-10T17:15:00Z",
  "demand_window_minutes": 60,
  "constraints": {
    "max_headway_tier1_minutes": 10,
    "max_headway_tier2_minutes": 20,
    "max_headway_tier3_minutes": 30,
    "ada_vehicle_minimum_per_route": 1,
    "driver_shift_extension_allowed": false
  },
  "optimization_weights": {
    "passenger_wait_time": 0.7,
    "deadhead_miles": 0.3
  }
}
```

Response (200):
```json
{
  "optimization_id": "opt_2026021017150042",
  "candidates": [
    {
      "rank": 1,
      "score": 0.87,
      "adjustments": [
        {
          "type": "frequency_increase",
          "route_id": "route_12",
          "segment": "Main St to Convention Center",
          "current_headway_minutes": 15,
          "proposed_headway_minutes": 8,
          "vehicles_needed": 2,
          "source_routes": ["route_34", "route_41"]
        }
      ],
      "projected_impact": {
        "avg_wait_reduction_seconds": 210,
        "deadhead_increase_miles": 4.2,
        "passengers_affected": 1850
      }
    }
  ],
  "hard_constraints_satisfied": true,
  "solver_runtime_ms": 2340
}
```

The `candidates` array is ordered by score, and the dispatcher dashboard presents the top candidate by default. Each candidate includes projected impact so the dispatcher can make an informed decision. If no candidate achieves a score above 0.4 (meaning the improvement over the current state is marginal), the solver returns an empty candidates array with a `recommendation` field set to `"maintain_current_service"`, and the dashboard shows a status message rather than prompting for action.

When the dispatcher accepts a candidate, the dashboard sends a `POST /routes/optimize/{optimization_id}/accept` request with the selected candidate's rank and any modifications the dispatcher made. This acceptance record feeds into the Feedback Collector's analysis of which types of adjustments dispatchers trust and which they tend to modify before approving.

#### Retrieve Feedback for a Route

The feedback endpoint returns prediction accuracy and correction factors for a given route.

**GET /routes/{route_id}/feedback?period=7d**

Response (200):
```json
{
  "route_id": "route_12",
  "period_start": "2026-02-03T00:00:00Z",
  "period_end": "2026-02-10T00:00:00Z",
  "segments": [
    {
      "segment": "Main St to Convention Center",
      "predicted_daily_riders": 2400,
      "actual_daily_riders": 2815,
      "correction_factor": 1.17,
      "trend": "increasing",
      "dispatcher_overrides": 3,
      "override_reasons": ["event_surge", "event_surge", "construction_detour"]
    },
    {
      "segment": "Convention Center to Riverside",
      "predicted_daily_riders": 1100,
      "actual_daily_riders": 980,
      "correction_factor": 0.89,
      "trend": "stable",
      "dispatcher_overrides": 0,
      "override_reasons": []
    }
  ],
  "model_accuracy_percent": 84.2
}
```

The `correction_factor` feeds directly into the solver's demand model. A factor of 1.17 means the solver has been underestimating ridership on that segment by 17%, so future predictions for that corridor and time window are adjusted upward. The `trend` field indicates whether the correction factor is moving in a consistent direction over the requested period, which helps the operations team distinguish between a segment that is genuinely growing in ridership and one that fluctuates around its baseline.

The `dispatcher_overrides` count and associated reasons provide visibility into how often human judgment diverges from the solver's recommendations for each segment. A high override count with consistent reasons (such as repeated `"construction_detour"` entries) suggests that the solver's model is missing a real-world factor that should be incorporated.

### Data Freshness and Staleness

Each data source has a different update frequency, which creates timing considerations for the solver. APC data arrives as boarding and alighting events in near-real-time (within seconds of the door opening), but the demand heatmap is computed over a rolling window, so it inherently lags. GPS positions update every 30 seconds, which is sufficient for route-level decisions but too coarse for precise stop-arrival predictions. Traffic data from the city API updates every 5 minutes, which means the solver may work with corridor speeds that have already changed.

The solver timestamps each input and discards data older than a configurable staleness threshold (currently set to 5 minutes for GPS, 10 minutes for traffic, and 60 minutes for demand heatmaps). If a data source goes completely silent (no updates within twice the normal interval), the solver logs an alert and proceeds with the remaining sources rather than blocking. The rationale is that a partially informed optimization is better than no optimization, and the dispatcher reviewing the candidate will notice if a recommendation seems inconsistent with conditions they observe.

### Handling Edge Cases

**GPS accuracy in urban canyons.** Downtown corridors with tall buildings cause GPS positions to drift by 50 to 200 meters, which can make the solver believe a bus is on an adjacent street or further along a route than it actually is. We handle this by snapping GPS positions to the known route geometry using a map-matching algorithm that considers the vehicle's previous known position, heading, and speed to select the most plausible route assignment. If the raw GPS position is more than 300 meters from any route, we discard it and use the last valid position plus estimated progress based on scheduled speed.

The Feedback Collector tracks GPS discard rates per corridor, broken down by time of day (since atmospheric conditions affect satellite signal quality). Corridors with discard rates above 15% are flagged for review, since high discard rates mean the solver is operating with stale position data for those vehicles. These corridors are the priority candidates for Bluetooth beacon installation during Phase 2.

**Solver cannot satisfy all hard constraints.** During severe disruptions (multiple vehicle breakdowns, a major road closure), it may be impossible to satisfy every hard constraint simultaneously. When this happens, the solver returns a partial solution that satisfies as many hard constraints as possible, ranked by a fixed priority order. ADA requirements sit at the top of this priority hierarchy and are never relaxed under any circumstances. Maximum headway constraints can be temporarily exceeded, but the solver flags which routes are affected and by how much. The dispatcher must explicitly acknowledge operating outside normal service parameters by confirming a dialog in the dashboard, and the system logs the exception for regulatory reporting. These exception records are included in the monthly operations report to the transit authority board.

**Special events.** Concerts, sports games, and festivals create demand spikes that differ from normal patterns in both magnitude and timing. The system supports pre-loaded event profiles that override the demand model for specific stops and time windows. MetroLink's events coordinator enters known events into a calendar with estimated attendance and start/end times, and the solver treats the event profile as a hard demand floor for the affected area. For large events at the downtown arena (capacity 18,000), the event profile pre-positions additional vehicles on feeder routes 45 minutes before the scheduled end time, based on historical dispersal patterns.

For unanticipated surges, the solver detects anomalies when ridership exceeds predictions by more than 40% at a cluster of adjacent stops over two consecutive measurement periods. This triggers an ad-hoc optimization cycle regardless of the normal interval. The anomaly detection uses a spatial clustering algorithm so that a single overcrowded stop does not trigger a network-wide re-optimization, but three adjacent stops all showing unexpected demand does.

### Component Responsibilities

| Component | Responsibility |
|-----------|---------------|
| **Route Solver** | Ingest data from all four sources, generate candidate adjustments, enforce hard and soft constraints, rank candidates by score |
| **Dispatch Adapter** | Translate solver output into CAD/AVL commands, batch and queue operations within rate limits, notify drivers via mobile data terminal |
| **Feedback Collector** | Compare predicted vs actual ridership, compute correction factors, track dispatcher rejection patterns, detect demand anomalies |
| **Dispatcher Dashboard** | Present candidate adjustments for human review, capture accept/reject decisions with reasons, display system health metrics |
| **CAD/AVL System** | Execute vehicle reassignments and schedule updates, relay driver communications (existing, not modified) |
| **APC Sensors** | Collect boarding and alighting counts per stop (existing, not modified) |

## Abandoned Ideas

### Full Dispatch System Replacement

The most direct path to dynamic routing would be replacing the CAD/AVL system entirely with a modern platform that supports programmatic route management. Several vendors offer systems where route changes flow through an API and reach drivers on tablet devices in the cab, eliminating the radio-command bottleneck and the batch queuing our adapter requires. We explored proposals from two vendors and found that both could support the solver's output format natively, which would remove an entire architectural layer. However, MetroLink signed a 7-year contract with the current radio and CAD/AVL vendor in 2023 that includes exclusivity provisions for dispatch-related communications. Breaking the contract would cost approximately $4 million in early termination fees, and the legal team confirmed there is no renegotiation window until 2027. Building the adapter is additional complexity that we would prefer to avoid, but it lets us deliver value within the existing contractual boundaries and proves the concept before a potential vendor transition. We revisit full system replacement when the contract comes up for renewal in 2030.

### ML-Based Demand Prediction

Rather than the correction-factor approach in the Feedback Collector, we considered training a machine learning model (likely gradient-boosted trees or a recurrent neural network) on historical ridership data to predict demand at the stop and time-of-day level. The accuracy gains would be meaningful, since ML models can capture complex interactions between weather, day of week, school calendars, and special events that simple weighted averages miss. We built a proof-of-concept that improved prediction accuracy from 84% to 91% on historical data. The blocker was governance, not technology. MetroLink's board must justify route changes to the public at monthly meetings, and board members were uncomfortable approving changes based on a model they could not inspect or explain. The constraint satisfaction approach is not as accurate, but it is auditable: every adjustment traces back to specific inputs and constraint weights that a non-technical board member can follow. If the board's comfort with ML tools evolves over time, we can revisit this by replacing the demand model inside the solver without changing the rest of the architecture, since the solver's interface to the demand model is deliberately abstracted for exactly this reason.

### Driver-Initiated Rerouting via Mobile App

We prototyped a mobile app that would let drivers suggest route adjustments based on what they observe from the cab: a bus stop with an unusually long queue, a street blocked by construction, a park-and-ride lot that has filled up earlier than expected. The idea was compelling because drivers have ground-level awareness that no sensor can replicate. During a four-week pilot with 20 volunteer drivers, adoption was uneven. Some drivers submitted several observations per shift while others never opened the app. The union raised concerns that the app could be used to track driver behavior or penalize those who did not participate, and negotiations over data usage stalled. Beyond the labor relations issues, the inconsistent adoption meant the system could not rely on driver input as a data source. We would be optimizing based on observations from whichever drivers happened to participate on a given day, which introduces bias toward routes staffed by engaged drivers. The current design captures dispatcher adjustments instead, since dispatchers already make these decisions and the system records them without requiring additional voluntary participation. We may revisit a lighter version of driver input in the future, perhaps limited to structured reports about road closures or stop conditions, but only after the core system has proven its value and the union relationship around technology adoption has stabilized.

## Risks

### Union Concerns About Automated Dispatch Decisions

The transit workers' union has historically resisted technology that they perceive as replacing dispatcher judgment or increasing surveillance of drivers. During the 2021 introduction of automatic schedule adherence monitoring, the union filed a grievance that took six months to resolve, so we know this is sensitive territory. The solver's recommendations could be seen as automating decisions that dispatchers currently own, particularly if adoption pressure makes dispatchers feel they must accept every suggestion.

We mitigate this by keeping dispatchers firmly in the approval loop: the system recommends, but a human always decides. We have scheduled briefings with union leadership to walk through the design and demonstrate that the system augments rather than replaces dispatchers. The rejection-logging feature was specifically designed to avoid capturing individual dispatcher performance metrics, recording only aggregate patterns across all dispatchers. If the union negotiations surface additional concerns, we may need to adjust the dashboard to give dispatchers more control over which recommendations they see and how frequently the system prompts them.

### GPS Accuracy Degradation in Downtown Corridors

MetroLink's downtown core has several streets with 20-plus-story buildings on both sides, creating urban canyons where GPS accuracy drops significantly. In testing, we observed position errors of up to 200 meters on three downtown corridors, which is enough to confuse the solver about which route a vehicle is actually serving. The map-matching mitigation reduces but does not eliminate this problem, because parallel routes on adjacent streets can both be plausible matches. We plan to install Bluetooth Low Energy beacons at the 15 highest-traffic downtown stops during Phase 2, which provide sub-5-meter accuracy when a bus passes within range. If beacon deployment is delayed, the solver falls back to schedule-based position estimation for the affected corridors, which is less accurate but prevents incorrect reassignments.

### Passenger Confusion With Changing Routes

Dynamic route adjustments mean that the posted schedule at a bus stop may not match the actual service. Passengers who planned their trip based on a fixed timetable could find themselves waiting longer than expected or seeing an unfamiliar route number. This risk is most acute for infrequent riders who rely on printed schedules rather than the MetroLink app. We depend on the digital signage system at major stops to display real-time arrival information, but only 60% of stops currently have digital signs. For stops without digital signage, we will publish a simplified "service window" (for example, "a bus every 8 to 15 minutes") rather than exact departure times. The rider communications team is developing messaging for the app and website that explains how dynamic service works and why it leads to shorter overall wait times.

There is also a regulatory dimension to this risk. The transit authority's operating charter requires that published schedules be "reasonably accurate," which was written with static timetables in mind. Legal counsel has reviewed our approach and believes that real-time arrival displays satisfy this requirement, but we should confirm this interpretation with the transit oversight board before expanding beyond the pilot phase. If the board disagrees, we may need to constrain the solver to produce adjustments within a narrower band of the published schedule, which would reduce the system's ability to respond to large demand shifts but would keep us within the charter's requirements.

We plan to present the pilot results to the oversight board after Phase 2, accompanied by data showing how dynamic adjustments compared to the published schedule and how rider satisfaction was affected. A favorable board response would clear the regulatory path for network-wide expansion.

## Rollout

**Phase 1: Infrastructure**
Deploy the Route Solver, Dispatch Adapter, and Feedback Collector in the staging environment connected to production data feeds. The solver consumes live APC, GPS, traffic API, and workforce schedule data in read-only mode, producing candidate adjustments that are displayed on the dispatcher dashboard but never sent to the CAD/AVL system.

This phase validates three things: data pipeline reliability (can we sustain continuous ingestion from all four sources without gaps), solver performance under real data volumes (does it produce candidates within the 15-minute cycle), and the dispatcher dashboard user experience (can dispatchers understand and evaluate recommendations quickly). Dispatchers review solver recommendations as a training exercise during this phase, providing structured feedback on whether the suggestions align with their operational judgment. Their feedback is recorded and used to tune solver parameters before enabling live operation.

**Phase 2: Pilot on Three Downtown Routes**
Enable full loop operation on Routes 12, 15, and 22, which are high-frequency trunk lines through the downtown core. These routes were chosen because they have the highest ridership variance, the most frequent dispatcher interventions under the current system, and full digital signage coverage at every stop. The Dispatch Adapter sends commands to the CAD/AVL system for these three routes only.

Phase 2 depends on completing the dispatcher dashboard training from Phase 1 and receiving union sign-off on the operational procedures for the pilot routes. We will assign two dedicated dispatchers to monitor solver-recommended adjustments during this phase, so they can provide detailed feedback on recommendation quality without the pressure of managing the full network simultaneously. The pilot runs for a minimum duration sufficient to observe weekday, weekend, and at least two special event scenarios before we evaluate Phase 3 readiness.

**Phase 3: Corridor Expansion**
Extend optimization to all routes in the downtown and midtown corridors, covering approximately 18 routes and 120 vehicles. This phase tests the solver's ability to optimize across interconnected routes where reassigning a vehicle from one route affects headways on another. Cross-route optimization is substantially more complex than single-route adjustments because changes cascade: pulling a bus from Route 34 to reinforce Route 12 degrades Route 34's headway, which may trigger a further reassignment from Route 41 to compensate.

Phase 3 depends on successful Phase 2 metrics (prediction accuracy above 80%, dispatcher acceptance rate above 60%, no service disruptions attributable to the system) and deployment of Bluetooth beacons at the 15 critical downtown stops to address GPS accuracy concerns. The Feedback Collector must also demonstrate that its correction factors are converging rather than oscillating, which would indicate that the demand model is stabilizing for the pilot routes.

**Phase 4: Full Network**
Enable optimization across all 50 routes. At this scale, the solver must handle the full complexity of the network, including suburban routes with long headways, express routes with limited stops, and cross-town routes that share corridors with trunk lines. Phase 4 depends on Phase 3 stability and a decision about how to communicate dynamic service to riders at stops without digital signage. The printed schedule replacement strategy must be finalized before expanding to routes that serve primarily non-app riders. This phase also requires that the Feedback Collector's demand models have accumulated enough data across all route types to produce reliable predictions, since suburban and express routes have different demand characteristics than the downtown trunk lines covered in earlier phases.

At full network scale, the solver handles approximately 400 vehicles and 50 routes simultaneously. Performance testing must confirm that the solver can produce candidate adjustments within the cycle interval even at this scale. If solver runtime exceeds the interval, we partition the network into independent optimization zones (downtown, midtown, suburban east, suburban west) that the solver processes sequentially within each cycle.

</example_rfc>
