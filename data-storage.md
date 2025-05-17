# Data Storage Documentation: CodePath AI

## 1. Overview

This document outlines the client-side data storage strategies for the CodePath AI application. Effective data storage is critical for enabling robust offline access, persisting user progress, saving application settings, and efficiently managing cached API responses. The approach aims for a balance between performance, capacity, and ease of use, with platform-specific considerations for web (React) and mobile (React Native/Expo).

All core storage logic and abstractions will be designed to reside within the `packages/app-core` shared directory where possible, promoting code reusability.

## 2. Types of Data to Store Locally

The following types of data will be stored on the client-side:

1.  **Tutorial Content:** Complete tutorial data, including lessons, explanations, code examples, and any associated metadata, downloaded from the Google Gemini API or other web sources. This is the largest chunk of data and essential for offline learning.
2.  **User Progress:** Information tracking the user's advancement through tutorials, such as completed lessons, current lesson, and overall tutorial status (e.g., 'not_started', 'in_progress', 'completed').
3.  **Application Settings:** User-configurable preferences, including the selected theme (light/dark), font size, current programming language focus, and other UI/UX related settings.
4.  **Cached API Responses (Managed by React Query):** Metadata and potentially small payloads from API calls (e.g., lists of available tutorials for a language) managed by React Query's persistence layer.
5.  **Global Application State (Managed by Zustand):** Selected parts of the global application state that need to persist across sessions, such as the `currentLanguage`, `activeTutorialId`, or `networkStatus`.

## 3. Web Application Storage (React with Vite/Create React App)

### 3.1. Primary Storage: IndexedDB

*   **Rationale:** IndexedDB is a low-level API for client-side storage of significant amounts of structured data, including files/blobs. It's a transactional database system available in all modern browsers, making it ideal for storing full tutorial content.
*   **Library:** `idb` (a small, promise-based wrapper around the IndexedDB API, making it much easier to use).
*   **Usage:**
    *   Storing complete tutorial objects (parsed from Gemini API responses or web sources).
    *   Potentially storing detailed user progress if it becomes complex.
*   **Schema (Conceptual for `tutorials` store in IndexedDB):**
    *   Object Store Name: `tutorials`
    *   Key Path: `id` (e.g., `python-basics-v1`)
    *   Value: An object containing `id`, `language`, `title`, `version`, `lessons: [{id, title, content, codeSnippets}]`, `fetchedAt: timestamp`.
    *   Indexes: `language`, `fetchedAt` (for querying or cleanup).

### 3.2. Secondary Storage: `localStorage`

*   **Rationale:** `localStorage` provides a simple synchronous key-value store, suitable for smaller amounts of data like user settings or flags.
*   **Usage:**
    *   **Zustand `persist` middleware:** Will use `localStorage` (via `createJSONStorage(() => localStorage)`) to save and rehydrate parts of the global application state (e.g., `appSettings`, `currentLanguage`).
    *   **React Query `persistQueryClient`:** The `@tanstack/query-sync-storage-persister` can use `localStorage` to persist the metadata and cached data of React Query.

### 3.3. Example `idb` Service (Conceptual)

```typescript
// In packages/app-core/src/storage/webDbService.ts
import { openDB, DBSchema, IDBPDatabase } from 'idb';

interface Lesson {
  id: string;
  title: string;
  content: string; // Markdown or structured content
  codeSnippets?: Array<{ language: string; code: string; }>;
}

interface Tutorial {
  id: string; // Composite key, e.g., "python_introduction_v1"
  language: string;
  title: string;
  version: number;
  description?: string;
  lessons: Lesson[];
  fetchedAt: number;
  lastAccessedAt?: number;
}

interface UserProgress {
  tutorialId: string; // Foreign key to Tutorial.id
  completedLessons: string[]; // Array of lesson IDs
  currentLessonId?: string;
  status: 'not_started' | 'in_progress' | 'completed';
  updatedAt: number;
}

interface AppDBSchema extends DBSchema {
  tutorials: {
    key: string;
    value: Tutorial;
    indexes: { 'by_language': string; 'by_lastAccessedAt': number };
  };
  userProgress: {
    key: string; // tutorialId
    value: UserProgress;
    indexes: { 'by_status': string };
  };
}

let dbPromise: Promise<IDBPDatabase<AppDBSchema>> | null = null;

const DB_NAME = 'CodePathAIDB';
const DB_VERSION = 1;

function getDbInstance(): Promise<IDBPDatabase<AppDBSchema>> {
  if (!dbPromise) {
    dbPromise = openDB<AppDBSchema>(DB_NAME, DB_VERSION, {
      upgrade(db, oldVersion, newVersion, transaction) {
        console.log(`Upgrading DB from version ${oldVersion} to ${newVersion}`);
        if (oldVersion < 1) {
          const tutorialStore = db.createObjectStore('tutorials', { keyPath: 'id' });
          tutorialStore.createIndex('by_language', 'language');
          tutorialStore.createIndex('by_lastAccessedAt', 'lastAccessedAt');

          const progressStore = db.createObjectStore('userProgress', { keyPath: 'tutorialId' });
          progressStore.createIndex('by_status', 'status');
        }
        // Add further upgrade paths as schema evolves
      },
    });
  }
  return dbPromise;
}

// --- Tutorial Operations ---
export async function saveTutorialWeb(tutorial: Tutorial): Promise<string> {
  const db = await getDbInstance();
  return db.put('tutorials', { ...tutorial, fetchedAt: Date.now(), lastAccessedAt: Date.now() });
}

export async function getTutorialWeb(id: string): Promise<Tutorial | undefined> {
  const db = await getDbInstance();
  const tutorial = await db.get('tutorials', id);
  if (tutorial) {
    // Update last accessed time (fire and forget)
    db.put('tutorials', { ...tutorial, lastAccessedAt: Date.now() }).catch(console.error);
  }
  return tutorial;
}

export async function getTutorialsByLanguageWeb(language: string): Promise<Tutorial[]> {
  const db = await getDbInstance();
  return db.getAllFromIndex('tutorials', 'by_language', language);
}

// --- User Progress Operations ---
// Similar functions for saveUserProgressWeb, getUserProgressWeb, etc.
```

## 4. Mobile Application Storage (React Native with Expo)

### 4.1. Primary Storage: SQLite

*   **Rationale:** SQLite is a full-featured, serverless, transactional SQL database engine that is commonly used in mobile applications for robust local data storage.
*   **Library:** `expo-sqlite` (Expo's module for interacting with SQLite databases).
*   **Usage:** Storing tutorial content (lessons, examples as stringified JSON or in normalized tables), user progress, and other structured application data.

### 4.2. Secondary Storage: `AsyncStorage`

*   **Rationale:** `AsyncStorage` (provided by `@react-native-async-storage/async-storage`) is a simple, unencrypted, asynchronous, persistent key-value storage system. It's suitable for smaller pieces of data.
*   **Usage:**
    *   **Zustand `persist` middleware:** Will use `AsyncStorage` (via a custom adapter `createJSONStorage(() => asyncStorageAdapter)` that wraps `@react-native-async-storage/async-storage`) for global state persistence.
    *   **React Query `persistQueryClient`:** The `@tanstack/query-async-storage-persister` can use `AsyncStorage`.

### 4.3. Example `expo-sqlite` Service (Conceptual)

```typescript
// In packages/app-core/src/storage/mobileDbService.ts
import * as SQLite from 'expo-sqlite';

// Define Tutorial and UserProgress interfaces similar to webDbService.ts
interface Tutorial { /* ... as above ... */ }
interface UserProgress { /* ... as above ... */ }

const db = SQLite.openDatabase('codepathai.db');

export const initMobileDB = (): Promise<void> => {
  return new Promise((resolve, reject) => {
    db.transaction(tx => {
      tx.executeSql(
        'CREATE TABLE IF NOT EXISTS tutorials (id TEXT PRIMARY KEY NOT NULL, language TEXT, title TEXT, version INTEGER, description TEXT, lessons_json TEXT, fetchedAt INTEGER, lastAccessedAt INTEGER);',
        [],
        () => {},
        (_, error) => { console.error('Error creating tutorials table:', error); reject(error); return true; }
      );
      tx.executeSql(
        'CREATE TABLE IF NOT EXISTS userProgress (tutorialId TEXT PRIMARY KEY NOT NULL, completedLessons_json TEXT, currentLessonId TEXT, status TEXT, updatedAt INTEGER);',
        [],
        () => { resolve(); },
        (_, error) => { console.error('Error creating userProgress table:', error); reject(error); return true; }
      );
    });
  });
};

// --- Tutorial Operations ---
export const saveTutorialMobile = (tutorial: Tutorial): Promise<void> => {
  return new Promise((resolve, reject) => {
    db.transaction(tx => {
      tx.executeSql(
        'INSERT OR REPLACE INTO tutorials (id, language, title, version, description, lessons_json, fetchedAt, lastAccessedAt) VALUES (?, ?, ?, ?, ?, ?, ?, ?);',
        [
          tutorial.id,
          tutorial.language,
          tutorial.title,
          tutorial.version,
          tutorial.description || null,
          JSON.stringify(tutorial.lessons),
          Date.now(),
          Date.now(),
        ],
        () => { resolve(); },
        (_, error) => { console.error('Error saving tutorial:', error); reject(error); return true; }
      );
    });
  });
};

export const getTutorialMobile = (id: string): Promise<Tutorial | null> => {
  return new Promise((resolve, reject) => {
    db.transaction(tx => {
      tx.executeSql('SELECT * FROM tutorials WHERE id = ?;', [id],
        async (_, { rows }) => {
          if (rows.length > 0) {
            const item = rows.item(0);
            const tutorial: Tutorial = {
              ...item,
              lessons: JSON.parse(item.lessons_json),
              lastAccessedAt: Date.now(), // Update last accessed time
            };
            // Update lastAccessedAt in DB (fire and forget)
            db.transaction(txUpdate => {
                txUpdate.executeSql('UPDATE tutorials SET lastAccessedAt = ? WHERE id = ?;', [Date.now(), id]);
            });
            resolve(tutorial);
          } else {
            resolve(null);
          }
        },
        (_, error) => { console.error('Error getting tutorial:', error); reject(error); return true; }
      );
    });
  });
};

// --- User Progress Operations ---
// Similar functions for saveUserProgressMobile, getUserProgressMobile, etc.
```

## 5. Shared Storage Abstraction (`packages/app-core`)

To maximize code reusability and maintain a clean architecture, a platform-agnostic storage service interface will be defined in `packages/app-core/src/storage/storageService.ts`. Platform-specific implementations (web using `idb`, mobile using `expo-sqlite`) will then implement this interface.

```typescript
// packages/app-core/src/storage/storageService.ts (Interface)
export interface IStorageService {
  init(): Promise<void>;
  saveTutorial(tutorial: Tutorial): Promise<string | void>; // string for web (key), void for mobile
  getTutorial(id: string): Promise<Tutorial | null | undefined>;
  getTutorialsByLanguage(language: string): Promise<Tutorial[]>;
  // ... methods for user progress
}

// Platform-specific files would then export an instance that implements IStorageService.
// A factory function or conditional imports would provide the correct service instance to the app.
```

This allows higher-level application logic (e.g., in custom React Query query functions or Zustand actions) to interact with storage in a platform-independent way.

## 6. Data Synchronization, Updates, and Cleanup

*   **Initial Seeding/Fetching:** On first load or when new content is requested, data is fetched from APIs and saved to local storage.
*   **Updates:** A mechanism to check for updated tutorial versions or new tutorials will be needed. If an updated tutorial is found, the local version can be replaced.
*   **Cleanup:** To manage device storage, a strategy for cleaning up old or unused tutorials might be implemented (e.g., least recently accessed, or user-initiated cleanup).

## 7. Security and Privacy

*   For this personal-use application, data stored locally (tutorial content, progress) is not considered highly sensitive from a privacy perspective beyond the user's own learning activities.
*   No client-side encryption of the main database (IndexedDB/SQLite) is planned for the MVP. If sensitive data were ever to be stored, `expo-secure-store` on mobile could be used for specific items, or more complex encryption strategies explored.
*   The primary focus is on data integrity and availability for offline use.