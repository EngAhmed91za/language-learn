# Offline Support Documentation: CodePath AI

## 1. Overview

A key design goal for CodePath AI is to provide a robust and seamless offline experience. Users should be able to access previously viewed or explicitly downloaded tutorial content, track their progress, and use most application features even when an internet connection is unavailable. This document details the strategies and mechanisms implemented to achieve this, leveraging local data storage, intelligent caching, and clear user interface feedback.

This builds upon the foundations laid in `data-storage.md` and `api-communication.md`.

## 2. Core Principles of Offline Support

1.  **Data Persistence:** Essential data (tutorial content, user progress, settings) is stored locally on the user's device.
2.  **Caching:** API responses and frequently accessed data are cached to minimize network dependency and speed up access.
3.  **Network Awareness:** The application actively monitors network connectivity status.
4.  **Graceful Degradation:** Online-only features are clearly indicated or disabled when offline, while core offline functionalities remain available.
5.  **User Feedback:** The UI transparently communicates the current network status and any implications for functionality.

## 3. Key Components Enabling Offline Support

*   **Local Databases:**
    *   **Web (React):** IndexedDB (via `idb`) for storing full tutorial content and detailed user progress.
    *   **Mobile (React Native/Expo):** SQLite (via `expo-sqlite`) for the same purpose.
*   **Key-Value Storage:**
    *   **Web:** `localStorage` for application settings, Zustand persisted state, and React Query cache metadata.
    *   **Mobile:** `AsyncStorage` (via `@react-native-async-storage/async-storage`) for the same.
*   **`@tanstack/react-query` with Persisters:**
    *   React Query's caching mechanism is central to offline support for API-fetched data.
    *   Cache persisters (`@tanstack/query-sync-storage-persister` for web, `@tanstack/query-async-storage-persister` for mobile) ensure that cached data survives application restarts and is available offline.
*   **Network Status Detection:**
    *   **Web:** `navigator.onLine` property and `online`/`offline` window events.
    *   **Mobile (Expo):** `@react-native-community/netinfo` library (or `expo-network` for basic status).
    *   The detected status will be managed in the global Zustand store (e.g., `useAppStore.getState().networkStatus`).

## 4. Scope of Offline Functionality

### 4.1. Available Offline:

*   **Accessing Downloaded/Cached Tutorials:** Users can open, read, and interact with any tutorial content that has been previously fetched and stored in the local database (IndexedDB/SQLite) or is present in the React Query persisted cache.
*   **Progress Tracking:** All progress made within tutorials (e.g., completing lessons, marking sections) will be saved locally and will function seamlessly offline.
*   **Application Settings:** User preferences (theme, font size, etc.) will be available and modifiable offline, as they are stored locally.
*   **Navigation:** Users can navigate between different sections of the app that rely on locally stored data.

### 4.2. Limited or Unavailable Offline:

*   **Fetching New/Uncached Tutorials:** Discovering or loading entirely new tutorials that have not been previously accessed or downloaded will not be possible.
*   **Live Content Updates:** Dynamic content updates from web sources (e.g., a feed of new programming articles, if such a feature existed) will not refresh.
*   **Direct Gemini API Interaction for New Content:** Features that require real-time generation of new content from the Gemini API (e.g., a hypothetical "Ask AI a new question" feature beyond tutorial content) will be disabled or clearly marked as unavailable.
*   **Checking for Application/Content Updates:** The ability to manually or automatically check for new versions of tutorials or app updates will be disabled.

## 5. Implementation Strategies

### 5.1. Proactive Content Caching and Downloading

*   **Automatic Caching (React Query):** When a user accesses a tutorial list or specific tutorial content online, React Query automatically caches the API response. The configured persister ensures this cache is written to `localStorage`/`AsyncStorage`.
*   **Saving to Local Database:** Upon successfully fetching tutorial content (e.g., from Gemini API), the application logic should explicitly save this structured content into the primary local database (IndexedDB/SQLite). This provides a more robust and queryable store for large content than relying solely on the React Query cache for everything.
    *   This can happen in the `onSuccess` callback of a `useMutation` (if fetching is an explicit action) or after a `useQuery` successfully fetches data.
*   **(Optional) Explicit Download Feature:** Consider adding a "Download for Offline Access" button for individual tutorials or entire language packs. This would trigger a more aggressive fetching and saving of all associated content to the local database.

### 5.2. Network Status Monitoring

A service in `packages/app-core` will monitor network status and update the Zustand store:

```typescript
// packages/app-core/src/services/networkManager.ts (Conceptual)
import { useAppStore } from '../store/appStore'; // Your Zustand store
// For Web:
if (typeof window !== 'undefined') {
  const updateStatus = () => useAppStore.getState().setNetworkStatus(navigator.onLine ? 'online' : 'offline');
  window.addEventListener('online', updateStatus);
  window.addEventListener('offline', updateStatus);
  updateStatus(); // Initial check
}

// For Mobile (using @react-native-community/netinfo - to be initialized in app setup):
// import NetInfo from '@react-native-community/netinfo';
// export const initializeNetworkEvents = () => {
//   NetInfo.addEventListener(state => {
//     const isOnline = !!state.isConnected && !!state.isInternetReachable;
//     useAppStore.getState().setNetworkStatus(isOnline ? 'online' : 'offline');
//   });
// };
```

### 5.3. Conditional Data Fetching and UI Behavior

*   **Data Fetching Flow:**
    1.  UI component requests data (e.g., tutorial content) via a React Query hook.
    2.  React Query hook first checks its in-memory and persisted cache.
    3.  If not found or stale, the custom `queryFn` is invoked.
    4.  Inside `queryFn`:
        a.  Attempt to retrieve data from the primary local database (IndexedDB/SQLite).
        b.  If found in DB, return it. React Query will cache this.
        c.  If not in DB AND `networkStatus` is 'online', attempt to fetch from the network API.
        d.  If fetched successfully, save to local DB AND let React Query cache it.
        e.  If `networkStatus` is 'offline' and not found in DB or cache, the query will result in an error or a specific 'unavailable offline' state.
*   **UI Adaptation:**
    *   Components subscribe to `networkStatus` from the Zustand store.
    *   Buttons or features requiring online access are disabled (e.g., `disabled={networkStatus === 'offline'}`).
    *   Informative messages or visual cues (e.g., a global banner, icons on list items) indicate offline status or that content is served from cache.

### 5.4. UI/UX for Offline States

*   **Global Offline Indicator:** A subtle, persistent banner or status icon (e.g., in the header or footer) indicating "Offline Mode" or "No Internet Connection."
*   **Content Availability:**
    *   Tutorials available offline should be clearly distinguishable (e.g., a small download icon, no visual change if seamless).
    *   Tutorials not downloaded and requiring an internet connection could be greyed out or have an icon indicating they are online-only.
*   **Error Messages:** If an online action is attempted while offline, provide a user-friendly message like, "This feature requires an internet connection. Please connect and try again."

### 5.5. Data Synchronization on Reconnection

*   **React Query `refetchOnReconnect`:** Configure React Query to automatically refetch relevant queries when the application comes back online. This helps ensure data freshness.
*   **Background Sync (Future Enhancement):** For a more advanced system (especially if a backend were involved), a queue for pending operations made offline could be implemented, to be processed upon reconnection. For this personal app, local progress is king, so this is less critical.
*   The app might trigger an automatic check for new tutorial lists or metadata from web sources upon returning online.

## 6. Testing Offline Capabilities

*   **Web Browsers:** Use developer tools (e.g., Chrome DevTools -> Network tab -> "Offline" checkbox; Firefox -> Network Monitor -> Throttle (Offline)).
*   **Mobile Emulators/Simulators:** Most provide options to simulate offline or poor network conditions.
*   **Physical Devices:** Toggle Wi-Fi and mobile data off.
*   **Test Cases:**
    1.  Launch app offline: Verify access to cached/downloaded content and settings.
    2.  Go offline while using the app: Verify continued functionality and UI updates.
    3.  Attempt online-only actions while offline: Verify graceful failure and user messages.
    4.  Return online: Verify data refetching (if applicable) and restoration of online features.
    5.  Verify progress saving and retrieval entirely offline.

By implementing these strategies, CodePath AI will provide a reliable and valuable learning experience, irrespective of the user's internet connectivity.