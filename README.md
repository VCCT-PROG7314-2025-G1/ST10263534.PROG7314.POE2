# Marley Maps – Dispensary Finder (PROG7314 POE)

Marley Maps is an Android app built for the PROG7314 Portfolio of Evidence.  
It helps users find nearby cannabis dispensaries and interact with them via reviews, directions, and calls, while also demonstrating advanced Android features such as SSO, offline support, notifications, and automated testing.

---

## 🎯 Project Goals

This app is designed to meet the PROG7314 POE Part 3 requirements:

- Single sign-on (SSO) login using **Google**.
- Biometric authentication preference (stored for future biometric flow).
- A **settings menu** that makes sense for the app.
- Connection to a **REST API** (Supabase) backed by a database.
- **Offline mode with sync** so some features still work without network.
- **Real-time notifications** using Firebase Cloud Messaging (FCM).
- **Multi-language support** with at least two South African languages.
- Automated testing via **unit tests** and **instrumentation tests**.
- A professional UI suitable for publication on the Play Store.

---

## 🧩 Features Overview

### 1. Authentication

- **Email/Password login & registration**  
  - Backed by a Supabase `users` table via the PostgREST API.  
  - `DataRepository.login(...)` and `DataRepository.registerUser(...)` handle Supabase calls.

- **SSO with Google**  
  - Implemented with Firebase Auth and Google Sign-In in `LoginActivity`.  
  - Users can sign in with their Google account and are taken to the dashboard.

- **Offline login fallback**  
  - After a successful online login, the app caches the last email/password in `UserSettings`.  
  - If Supabase login fails (e.g. offline) but the entered credentials match the cached ones, the user can still log in and access offline data.

### 2. Profile & Settings

`ProfileActivity` provides:

- Display of **Name** and **Email** of the currently signed-in user.
- **Enable Location** switch – when turned off, the dashboard disables distance-based sorting and defaults to rating sort.
- **Enable Biometric Login** switch – preference is stored in `UserSettings` to control future biometric prompts.
- **Language** spinner – in-app choice between English, Afrikaans, and isiXhosa.
- **Save Settings** button – persists the above preferences and returns to the Dashboard.
- **Logout** button – signs out of Firebase (if used) and returns to the Login screen.

### 3. Dispensary List & Details

- Dashboard displays a list of dispensaries (name, distance, rating).  
- Chips allow sorting by **distance** or **rating**, respecting the `Enable Location` setting.
- Tapping a dispensary opens `DispensaryDetailsActivity` which shows:
  - Dispensary name, address, rating, open/closed status.
  - Latest review (fetched via Google Places API).
  - Buttons to **Get Directions** (Google Maps intent) and **Call Dispensary**.
  - A photo/logo when available from the Places API.

### 4. Offline Mode & Sync

- `OfflineCache` uses `SharedPreferences` + `kotlinx.serialization` to cache the list of `Dispensary` objects.
- On Dashboard start:
  - If cached data exists, it’s loaded and a toast indicates *“Loaded offline dispensaries”*.
  - Otherwise, the in-memory list from `DataRepository` is used.

- The **Sync Offline** button on the dashboard:
  - Saves the current dispensary list into the offline cache.
  - Shows *“Offline data synced”*.
- Combined with offline login fallback, this gives users access to the app even without an active internet connection.

### 5. Real-time Notifications (FCM)

- Integrated **Firebase Cloud Messaging** with `MarleyFirebaseMessagingService`.
- The service:
  - Listens for incoming FCM messages.
  - Creates a notification channel (`marley_notifications`) on Android 8+.
  - Displays a system notification with the message title and body.
- Notifications can be tested using the Firebase console’s “Send test notification” feature.

### 6. Multi-language Support

- Default language: **English**.
- Additional languages:
  - **Afrikaans** – `res/values-af/strings.xml`
  - **isiXhosa** – `res/values-xh/strings.xml`
- `UserSettings` stores a `preferredLanguage` code (`en`, `af`, `xh`), selected in the Profile Settings via a `Spinner`.  
- After saving settings, the selected language is applied via `Configuration.setLocale`, and the user returns to the Dashboard.

### 7. Automated Testing

- **Unit test:** `DashboardSortTest` (`app/src/test/...`)  
  - Verifies:
    - distance parsing (`"1.2 km"` → `1.2`),  
    - correct sort order by distance and by rating.

- **Instrumentation test:** `DashboardActivityTest` (`app/src/androidTest/...`)  
  - Launches `DashboardActivity`.
  - Clicks the first list item.
  - Asserts that the details screen (`R.id.nameText`) is displayed.

This fulfills the “Automated testing in place” requirement.

---

## 🏗️ Architecture & Technology Stack

- **Language:** Kotlin (with one Java activity).
- **UI:** XML layouts, Material Components, ListView + ArrayAdapter.
- **Architecture:** Simple MV-ish structure with a `DataRepository` singleton and a `data` package.
- **Cloud services:**
  - **Supabase** for RESTful database access (users).  
  - **Firebase** for Auth (Google SSO), Analytics, and Cloud Messaging.
  - **Google Places API** for ratings, reviews, and photos (`DispensaryDetailsActivity`).
- **Offline storage:** SharedPreferences + JSON for cached dispensaries and user settings.
- **Testing:** JUnit4 + AndroidX Test + Espresso.

---

## 🔧 Setup & Running

### Prerequisites

- Android Studio (Arctic Fox or newer).
- Android SDK 26+.
- A device/emulator with Google Play Services (for Google Sign-In and FCM).

### Clone and open

```bash
git clone https://github.com/<ORG_OR_USER>/ST10263534.PROG7314.POE2.git
cd ST10263534.PROG7314.POE2

	•	Open the project in Android Studio.
	•	Let Gradle sync.

Configure Supabase & Firebase
	•	Supabase:
	•	SupabaseClient contains the supabaseUrl and supabaseKey.
	•	Ensure these match your Supabase project.
	•	Firebase:
	•	google-services.json is already included (linked to the MarleyMaps project).
	•	Ensure google-services Gradle plugin is applied (present in project-level Gradle).

Run
	•	Select a device/emulator and click Run ▶.
	•	Login via email/password (using existing Supabase test users) or Google Sign-In.

Running Tests

Unit tests
./gradlew test
or in Android Studio: right-click DashboardSortTest → Run.

Instrumentation tests
./gradlew connectedAndroidTest
or right-click DashboardActivityTest → Run (device/emulator required).

Release Notes

Prototype → Final POE version

Key changes from the prototype to this final submission:
	•	Added Google SSO via Firebase Auth.
	•	Added Profile Settings screen with name, email, and multiple settings (location, biometrics, language).
	•	Implemented offline cache for dispensaries and Sync Offline button.
	•	Added offline login fallback using last known credentials.
	•	Integrated Firebase Cloud Messaging for real-time notifications.
	•	Added multi-language support (English, Afrikaans, isiXhosa) and a language selector.
	•	Implemented unit tests for sorting logic and instrumentation tests for navigation.
	•	Polished UI layouts and wired settings to change Dashboard behaviour.

AI Use & Assistance
# AI Use Statement

During the development of Marley Maps, I used ChatGPT (OpenAI) as a coding assistant.  
The tool was used in the following ways:

- To help refactor and debug some Kotlin/Java code, especially around integration with Firebase Auth, Firebase Cloud Messaging, and SharedPreferences-based offline caching.
- To propose example implementations for unit tests and instrumentation tests, which I then adapted and integrated into `DashboardSortTest` and `DashboardActivityTest`.
- To assist with XML layout refinements (for example, structuring the Profile Settings screen and dashboard sort chips) and to ensure the UI remained consistent.
- To generate boilerplate documentation, such as this README structure and the outline of the AI-use statement itself.

All code generated or suggested by ChatGPT was reviewed, tested, and adapted by me to fit the requirements of the PROG7314 POE and the specific project context.  
Configuration values (API keys, Supabase URLs, etc.) were set up manually and not generated by AI.  
No external copyrighted code was pasted verbatim without attribution.

The responsibility for the final code, testing, and behaviour of the app remains mine.


