# User Personas

During Sprint 1, we defined three core user personas to guide our requirements elicitation and feature prioritization. These personas represent our primary target audiences and their specific needs regarding the Dublin Bikes system.

---

## 1. The Daily Student Commuter

* **Name:** Eoin
* **Role:** Computer Science Student at UCD
* **Age:** 20
* **Tech Literacy:** High
* **Bio:** Eoin lives in student accommodation near the city center and commutes to UCD daily. He is on a tight budget and prefers cycling over paying for the bus. He is often running late for his 9:00 AM lectures.

### Goals:
* **Speed & Reliability:** Needs to know immediately if a bike is available at his nearest station before he leaves his apartment.
* **Parking Assurance:** Needs to know if there is a free stand near the campus engineering building so he isn't late trying to dock the bike.
* **Weather Check:** Wants to know if it will rain during his 20-minute ride so he can decide whether to take the bus instead.

### Frustrations:
* Walking to a station only to find it empty.
* Arriving at university and finding the station full, forcing him to cycle to a further station and walk back.
* Getting caught in unforeseen rain showers.

### Relevant Project Features:
* **Prediction Model:** Eoin relies on the machine learning feature to predict if bikes will still be available in 10 minutes when he arrives at the station.
* **Weather Integration:** Uses the integrated weather data to make a "go/no-go" decision.

---

## 2. The Punctual Office Worker

* **Name:** Sarah
* **Role:** Marketing Manager
* **Age:** 29
* **Tech Literacy:** Medium
* **Bio:** Sarah works in the Dublin Docklands (Silicon Docks). She takes the train into the city and uses a city bike for the "last mile" to her office. She has important meetings in the morning and cannot afford to be late or look messy.

### Goals:
* **Planning Ahead:** She likes to plan her route the night before or early in the morning.
* **Efficiency:** She wants the most direct route from the train station to her office to minimize travel time.
* **Routine Analysis:** She wants to see historical trends to understand which days of the week are busiest so she can adjust her schedule.

### Frustrations:
* Uncertainty about how long the ride will take.
* Not knowing the best route to take to avoid heavy traffic or roadworks.

### Relevant Project Features:
* **Journey Planner (Optional Feature):** Sarah is the primary user for the "Journey Planning" feature to find the optimal route from Station A to Station B.
* **Historical Data:** She checks the historical occupancy trends to see if Monday mornings are typically bad for bike availability at the train station.

---

## 3. The Weekend Tourist

* **Name:** Matteo
* **Role:** Visitor from Italy
* **Age:** 35
* **Tech Literacy:** Low-Medium
* **Bio:** Matteo is visiting Dublin for a weekend city break. He doesn't know the city layout well. He wants to see the main sights (Trinity College, Guinness Storehouse) and thinks cycling is a fun way to explore. He is currently standing on a street corner looking at his phone.

### Goals:
* **Exploration:** Wants to find stations near major landmarks easily on a visual map.
* **Ease of Use:** Needs a simple interface to find the closest station to his current GPS location.
* **Information:** He has questions about how the system works (e.g., "Do I need a card?", "Is it free for 30 minutes?").

### Frustrations:
* Getting lost or not knowing where the stations are relative to the tourist sites.
* Confusing user interfaces or needing to navigate complex menus.
* Language barriers or lack of local knowledge.

### Relevant Project Features:
* **Map Visualization:** Matteo relies heavily on the Google Maps interface to see colored markers and his live GPS location.
* **GenAI Chatbot (Optional Feature):** Matteo uses the embedded assistant to ask questions like, "Where is the nearest station to Temple Bar?" or "Is it going to rain this afternoon?".