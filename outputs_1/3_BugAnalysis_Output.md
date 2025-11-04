# 🐞 Bug Analysis and Solution: Delayed Profile Loading

## 1. Root Cause Analysis (RCA)

The delayed rendering of the user's profile icon and name (5 seconds) after successful login, specifically on Firefox, points to a **synchronization failure** between the frontend rendering process and a slow backend API call.

| Layer of Failure | Probable Cause | Justification |
| :--- | :--- | :--- |
| **Backend/API** | **Slow API Call for User Profile Data** | The dynamic component (profile details) relies on a **separate, asynchronous API request** (e.g., `/api/user/profile`). This request is taking **~5 seconds** to resolve. |
| **Frontend/Browser** | **Browser-Specific Rendering Issue (Firefox)** | The explicit reporting on Firefox suggests a timing dependency issue, likely related to how Firefox handles **asynchronous component initialization** or **CSS race conditions** upon component mount. |
| **Synchronization** | **Lack of Content Synchronization** | The presence of a **blank space** confirms the component lacks an initial, immediate placeholder state (a **skeleton screen**) to fill the 5-second gap caused by the slow API response. |

**Conclusion:** The primary cause is the **5-second latency** of the dedicated user profile API call, which the frontend failed to handle gracefully, exposing the delay directly to Firefox users.

---

## 2. Test Prevention Strategy: New Automation Test Cases

To enforce synchronization and prevent this regression, the following **Automation Test Cases** must be added to the CI/CD pipeline, focusing on explicit wait conditions and browser matrix coverage.

| Test ID | Focus Area | Automation Test Case (Gherkin Format) | Rationale & Tooling |
| :--- | :--- | :--- | :--- |
| **AUT-PRF-001** | **Explicit Wait for User Component** | **Given** a user successfully completes the MFA login process.<br>**When** the Inbox page finishes its initial DOM load.<br>**Then** the automation framework must **explicitly wait** for the profile element (`id="user-profile-icon"`) to be **Visible and Stable** within **1.5 seconds**. | Ensures the critical element appears quickly. Uses **Selenium/Cypress Explicit Waits**. |
| **AUT-PRF-002** | **Network Performance Simulation (API)** | **Given** a successful login is executed, **with network simulation configured for 500ms latency** on all API calls.<br>**When** the Inbox page attempts to call the `/api/user/profile` endpoint.<br>**Then** the profile component placeholder (**skeleton screen**) must render immediately, and the final user data must appear within **1.0 second** of the API response being received. | Tests the mandatory skeleton screen implementation under simulated slowness. Uses **Cypress/Playwright Network Interception**. |
| **AUT-PRF-003** | **Browser Matrix Timing Test (Firefox)** | **Given** Test Case AUT-PRF-001 is executed specifically within the **Firefox browser environment**.<br>**When** the successful login completes.<br>**Then** the total time from the final login redirect to the **profile icon being fully loaded (with image/name)** must be less than **2.0 seconds**. | Directly addresses the browser-specific bug. Requires CI/CD pipeline to include **Firefox** in the critical path test matrix. |

---

## 3. Documentation Update

The following documentation sections must be immediately updated to enforce new performance and synchronization standards.

### 3.1. ⚙️ Automation Readme / Test Standards Document

* **New Synchronization Standard: Critical Viewports and Components (CVCC) Standard.**
    * **Update:** All elements above the fold, including the user profile header, must use **Explicit Waits** in automation. Maximum permitted wait for a post-initial-load critical component is **2.0 seconds**.
* **Browser Matrix Requirement:**
    * **Update:** The **Full Regression Suite** must be executed against **Chrome (Latest)** and **Firefox (Latest)**. Performance-sensitive tests are designated as **Critical Path Tests (CPT)** and must pass on both browsers.

### 3.2. 📈 Performance Budget Documentation

* **API Latency Thresholds:**
    * **New Threshold:** Introduce a maximum permitted **P95 Latency** for the `/api/user/profile` endpoint: **500 milliseconds (ms)**.
    * **Action:** Any API categorized as **Crucial User Detail (CUD)** must now utilize a **Caching Layer** to adhere to the 500ms threshold.
* **Frontend Rendering Budget:**
    * **New Requirement:** **Critical Viewport Content (CVC) Time** must be **< 3.0 seconds**. Any component with dynamic loading exceeding 1.0 second must display a **Skeleton Screen/Placeholder**.
