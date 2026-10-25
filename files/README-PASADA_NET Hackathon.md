PasadaNet USJR Hackathon 2026
Project Overview
PasadaNet is a dual-mode mobile web application designed to solve the most urgent problem facing Cebu's public utility vehicle (PUV) ecosystem: drivers earning less than ₱200 a day while diesel costs ₱150–170 per liter. The app gives jeepney, bus, and e-jeep drivers the information they need to decide — before departing — whether their next trip will make or lose them money. On the other hand, it empowers commuters to signal that they are waiting and see which vehicle is arriving first, eliminating unnecessary special rides.
PasadaNet requires zero new infrastructure. It runs entirely on smartphones already in drivers' and commuters' pockets, using existing LTFRB routes, real Cebu corridors, and simple math.

The Problem
Cebu PUV drivers operate on fixed routes with no ability to reroute. When traffic is heavy — at the Mactan Bridge, along Mandaue, in Minglanilla, or through Colon — their engines idle, burning 1.5 to 2.1 liters of diesel per shift for zero income. At ₱160 per liter, that idle burn alone can erase the day's profit. Most drivers have no tool to quantify this loss before it happens. They drive into traffic by default, not by choice.
Commuters face a parallel problem: without knowing when the next jeepney is coming, they pay ₱50–80 for a special tricycle ride that a regular PUV could have covered for a fraction of the cost.

The Solution
PasadaNet opens with a single question: Who are you today? The user selects either Driver or Commuter, and the app routes them into a completely separate dashboard built for their specific needs.
Driver Dashboard
Feature 1 — Route Profitability Heatmap
The driver selects their official LTFRB route number from a dropdown (e.g., 01K · Urgello–Parkmall, 06E · SRP–IT Park–Colon, 10M · Mactan Bridge–Colon). The app instantly calculates the fuel burn cost for one full loop based on the current traffic zone status — Red (heavy), Amber (moderate), or Green (clear) — and displays a plain-language verdict: Go, Caution, or Skip. No manual input is needed. The driver sees their route's idle fuel penalty in liters and pesos, the estimated gross fare per loop, and the net profit or loss per trip right now.
Feature 2 — Take-Home Predictor
The driver inputs their vehicle rent or boundary fee once at the start of the day. Each time they refuel, they tap a single button and enter the amount spent. The app tracks all fuel purchases and calculates a running take-home estimate: total fares collected minus total fuel minus rent. The result is color-coded — green for above ₱200, amber for borderline, red for a loss day. At the end of the shift, the driver sees exactly where their money went.
Feature 3 — City Flow Sync (Suggested Departure Timer)
When a route is currently in a Red or Amber traffic zone, the app shows a suggested waiting time before starting the next loop — for example, "Wait 25 minutes before departing on 04C Mandaue." The waiting time is expressed in liters and pesos saved, making it a concrete financial decision rather than a vague suggestion. This feature indirectly benefits the city: when drivers wait at terminals instead of entering congested corridors, the total number of idling vehicles in traffic drops, reducing city-wide congestion and emissions without any policy change or infrastructure.

Commuter Dashboard
Feature 4 — Waiting Beacon
A large, high-contrast button on the commuter screen activates a location signal indicating that the user is waiting for a PUV at their current stop. When the commuter boards a vehicle, they tap again to deactivate. Nearby drivers see the count of active waiting commuters along their route, which confirms that the next loop will have passengers — reducing deadhead fuel burns. The beacon button is designed for outdoor use: minimum 140px diameter, visible under direct Cebu sunlight, and operable with one thumb while standing.
Feature 5 — Incoming Vehicle Tracker
The commuter sees a live list of nearby PUVs organized by route number, with estimated arrival times and available seat count. ETAs are color-coded: green for under 5 minutes, amber for 5–15 minutes, red for over 15 minutes. This lets commuters decide whether to wait for the regular PUV or accept a special ride — with full information rather than guesswork. Knowing a jeepney arrives in 3 minutes saves ₱50–80 per trip.

Implementation Process
Hours 0–3 — Data and route architecture. Define the complete route database with LTFRB route codes, distances, average trips per day, fare per loop, and baseline fuel consumption per route. Assign traffic zone classifications to Cebu's key corridors.
Hours 3–8 — Role selector and driver dashboard. Build the landing screen with the two-role selector. Build the full driver flow: route picker, profitability heatmap, burn rate bar, stat cards, City Flow Sync timer, fuel logger, and take-home calculator. All logic runs client-side in JavaScript with no backend required.
Hours 8–16 — Commuter dashboard. Build the commuter beacon, waiting count display, and incoming vehicle list. Vehicle ETAs for the MVP are simulated using route traffic states and time-of-day logic — no live GPS feed is required to demonstrate the concept.
Hours 16–22 — Habal-Habal Reality Check. Test all UI elements for outdoor visibility and large-tap usability. Minimum tap targets of 44px enforced throughout. All text minimum 13px. Color contrast tested against bright light. The beacon button is verified at full size.
Hours 22–24 — Pitch preparation and Savings Math Card. Finalize the single-screen Savings Math summary. Rehearse the 3-minute pitch: problem statement, role selector demo, driver dashboard demo, commuter beacon demo, savings math close.

Judging Criteria Alignment
Local Utility — every route uses actual LTFRB codes and references specific Cebu locations: Mactan Bridge, SRP, Colon, Mandaue, Minglanilla, IT Park, Urgello.
Puyat Polish — the entire app is a single HTML file with embedded CSS and JavaScript. It loads instantly on any smartphone browser with no installation, no account, and no internet dependency for core calculations.
Diskarte Score — PasadaNet uses zero new infrastructure. It works with the BRT stops, jeepney terminals, V-Hire routes, and mobile phones that already exist in Cebu today.

Savings Math Card
1.5 to 2.1 liters of diesel saved per driver shift by avoiding Red Zone idle time. ₱255 or more recovered per driver per day — exceeding the typical daily take-home. ₱1,000 to ₱1,600 saved per commuter per month by eliminating unnecessary special rides. Zero pesos spent on new infrastructure.


