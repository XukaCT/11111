# SmartFarm Dashboard

This project is a full-stack dashboard for monitoring agricultural sensor data against user-configured Crop Cards. It uses a React frontend and an Express/SQLite backend.

## I. Setup and Execution

**Prerequisites:** Node.js (v18+) and npm installed.

**1. Backend Installation & Run**
+ Navigate to the backend directory: `cd smartfarm` and `cd backend`
+ Install dependencies: `npm install`
+ Start the server: `node server.js`

**2. Frontend Installation & Run**
+ Open a new terminal and navigate to the frontend directory: `cd smartfarm` and `cd frontend`
+ Install dependencies: `npm install`
+ Start the Vite development server: `npm run dev`

## II. Architecture & Networking

**1. Base URLs**
*   **Frontend UI:** `http://localhost:5173`
*   **Backend API:** `http://localhost:3001`

**2. API Routes**

| Method | Route | Description |
| :--- | :--- | :--- |
| `GET` | `/api/crops` | Fetch all configured Crop Cards from SQLite |
| `POST` | `/api/crops` | Create a new Crop Card |
| `PUT` | `/api/crops/:id` | Update an existing Crop Card |
| `DELETE` | `/api/crops/:id` | Delete a Crop Card |
| `GET` | `/api/readings` | Fetch external sensor JSON data |

**3. Error Format**
All API errors return a standard JSON object containing a single `error` property and an appropriate HTTP status code (e.g., 400, 404, 500).
```json
{
  "error": "Detailed error message describing the failure."
}
```
## III. Data Management & Logic

**1. Database Creation and Seeding**

The backend uses SQLite (`database.sqlite`). Upon the first server startup, it automatically creates the crops table if it does not exist. A startup script automatically seeds it with initial default thresholds for standard `crops` ( Tomato, Lettuce, Wheat) to ensure a functional dashboard immediately upon launch.

**2. Data Ownerships**
+ SQLite Database: Owns and persists user-configured target thresholds, normal water recommendations, locations, and notes.
+ Sensor JSON: Acts as an external, read-only data feed providing current environmental conditions.
+ React State: Owns the combined analysis logic, merging database thresholds with external sensor data on the fly.

**3. Crop Name Matching**

Data from the sensor feed is mapped to Crop Cards using exact, case-sensitive string matching on the `crop_name` field. If a sensor reading's `crop_name` does not exactly match a database record, it is ignored by that specific card.

**4. Latest-Timestamp Selection**

Multiple readings for the same crop are filtered by first extracting all matches, then applying a string-based descending sort (`localeCompare`) on the ISO 8601 `timestamp` field. Index `0` of the sorted array is selected as the definitive latest reading.

**5. Dashboard Decision Priority**
The analysis engine evaluates the latest reading against the Crop Card using a strict top-down hierarchy. The first matching condition halts further checks:
+ Sensor Problem: `sensor_status` is 'Offline' or 'Faulty'.
+ Invalid Data: Numeric values fall outside physically possible bounds (moisture < 0 or > 100, etc.).
+ Dry: `soil_moisture` < `target_mi`n. (Triggers "Water crop").
+ Healthy: `soil_moisture` is between `target_min` and `target_max` inclusive.
+ Too Wet: `soil_moisture` > `target_max`. (Triggers "Stop watering").

! Note: High temperature (>35°C) and Rainfall (>=5mm) are evaluated independently as supplementary Alerts for valid, online readings, appending to the condition rather than overwriting it.


## IV. Development & AI Context

**2. Checks and Corrections to Sensor JSON**

+ Verified that timestamps conformed strictly to YYYY-MM-DDTHH:mm:ss for reliable string sorting.
+ Ensured the "out-of-range" numeric value (e.g., temperature 999) was applied to an older reading so it wouldn't disrupt the specific required state of the latest readings.
+ Confirmed the array length was exactly 20 objects, with an even distribution of 5 per crop type.

**3. AI Use Statement**
+ Which tool: Google Gemini.
+ Final AI Prompt for Sensor Data:
"Generate a valid JSON array containing exactly 20 simulated SmartFarm sensor readings. Use these crop_name values exactly and create exactly 5 readings for each: Tomato, Lettuce, Wheat, Maize. Every object must contain exactly these fields: crop_name, timestamp, soil_moisture, temperature, rainfall, sensor_status, notes. Use timestamps in YYYY-MM-DDTHH:mm:ss format. Timestamps must be distinct within each crop. The same timestamp may be used by different crops. Mix the array order so the latest reading is not always the last object. Use sensor_status only as Online, Offline or Faulty. Most numeric values must be realistic: soil_moisture 0-100, temperature 0-50, rainfall 0-50. Include exactly one structurally valid older reading with one deliberately out-of-range numeric value. That invalid reading must not be the latest reading for its crop. Make the latest readings produce these cases with the default Crop Card settings: - latest Tomato: Online, Dry, temperature above 35 C; - latest Lettuce: Online and Healthy; - latest Wheat: Online, Too Wet, rainfall at least 5 mm; - latest Maize: sensor_status Faulty. Return only the JSON array. Do not use Markdown or explanation."
+ Problem found and corrected in AI-generated data:  
When the AI generated the 20 sensor readings, it struggled to align the chronological timestamps with the assignment's dashboard requirements. For example, it correctly created a Tomato reading with a temperature of 38°C (to trigger the "High temperature" alert), but it assigned that reading a timestamp of `10:00:00`. It then generated a normal, healthy Tomato reading with a timestamp of `14:00:00`. Because my application correctly selects the reading with the *greatest* timestamp, the dashboard displayed the healthy reading instead of the required hot/dry scenario. I corrected this by manually editing the JSON timestamps to guarantee that the specific objects containing the required assignment states (Tomato hot/dry, Maize faulty, Wheat rain) mathematically held the latest chronological timestamp for their respective crops.
+ How crop_name uniqueness and exact matching were verified:
  *   **Uniqueness:** I verified that the JSON data contained no typos or duplicate crop entries by passing the extracted names through a JavaScript `Set` (`[...new Set(readings.map(r => r.crop_name))]`). Because a `Set` only stores unique values, this confirmed the data cleanly contained exactly 4 unique crops (Tomato, Lettuce, Wheat, Maize).
  *   **Exact Matching:** I verified exact matching in the frontend filtering logic (`getAvailableCropNames` and `analyseCrop`). I used the strict equality operator when comparing the sensor feed to the database (`r.crop_name === cropName`). This guarantees a case-sensitive, perfect match, ensuring data is never accidentally applied to the wrong Crop Card.
+ How the greatest timestamp and required latest cases were verified
  *   **Greatest Timestamp:** I verified this programmatically. Because the JSON timestamps are in strict `YYYY-MM-DDTHH:mm:ss` format, I used a descending string sort (`b.timestamp.localeCompare(a.timestamp)`). This mathematically guarantees that the chronologically newest reading will always end up at index `[0]`.
  *   **Required Latest Cases:** To verify that the specific required scenarios (e.g., Tomato = Dry/High Temp) actually worked, I performed a visual UI test. I ran the backend with the default Crop Card thresholds seeded in SQLite, loaded the JSON sensor feed, and visually confirmed on the React dashboard that the correct alerts and conditions were rendered for the latest reading of each respective crop.
+ Implementation Decision Explained:
  *   **Decision:** I chose to derive the final dashboard analysis (the condition, recommended water, and alerts) dynamically on the fly during the React render cycle, rather than storing those calculated results in a separate React state variable or saving them to the SQLite database.
  *   **Explanation:** The dashboard relies on two completely separate, changing data sources: user-editable Crop Cards from SQLite and an external JSON sensor feed. If I stored the calculated condition (e.g., "Dry") in the database or its own state variable, it would instantly become stale the second a user edited their moisture thresholds or the sensor feed refreshed. By computing the dashboard results dynamically (`const dashboardResults = crops.map(...)`) immediately before rendering, I guarantee the UI always displays the exact, real-time intersection of the user's latest rules and the newest sensor data, entirely avoiding state-mismatch bugs.
      
**4. Project Limitation**

The application relies on HTTP polling (manual "Refresh" button) to fetch updated readings. It does not use WebSockets, meaning critical alerts (like a sudden temperature spike) are not pushed to the client in real time.
