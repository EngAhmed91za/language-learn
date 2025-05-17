# State Management Documentation: CodePath AI

## 1. Overview

Effective state management is fundamental to building a responsive, maintainable, and offline-capable application like CodePath AI. This document outlines the strategies and tools for managing different types of state within the React (web) and React Native (mobile) applications.

A layered approach will be employed, distinguishing between:

1.  **Local Component State:** State confined to individual UI components.
2.  **Global Application State:** State shared across multiple components or the entire application.
3.  **Server State / Asynchronous Data State:** Data fetched from external sources (like the Google Gemini API or web content URLs), along with its caching and synchronization logic.

The primary goals for state management are simplicity, performance, excellent developer experience, and robust offline support. Chosen tools like Zustand and `@tanstack/react-query` reflect these priorities. Most state management logic, especially global stores and data fetching hooks, will reside in the shared `packages/app-core` directory to ensure consistency and reusability between the web and mobile platforms.

## 2. Local Component State

*   **Purpose:** Manages UI-specific state that is not needed by other parts of the application. This includes things like form input values, the open/closed state of UI elements (e.g., modals, dropdowns), or conditional rendering logic within a single component.
*   **Tools:** React's built-in hooks:
    *   `useState`: For simple state variables (booleans, strings, numbers, small objects/arrays).
    *   `useReducer`: For more complex state logic within a component that might involve multiple sub-values or intricate update patterns.
*   **Example:** A `SearchBar` component might use `useState` to manage the current text input by the user.
    ```typescript
    // Example in a React component
    import React, { useState } from 'react';

    const SearchBar = () => {
      const [query, setQuery] = useState('');

      const handleChange = (event: React.ChangeEvent<HTMLInputElement>) => {
        setQuery(event.target.value);
      };

      // ... JSX for the search bar
      return <input type="text" value={query} onChange={handleChange} />;
    };
    ```

## 3. Global Application State

*   **Purpose:** Manages data that needs to be accessed, modified, or subscribed to by multiple components across different parts of the application. This also includes state that needs to persist across user sessions (e.g., user preferences, application settings).
*   **Chosen Tool: Zustand**
    *   **Rationale:** Zustand is a small, fast, and scalable state management solution with a simple, hook-based API that integrates naturally with React. It requires minimal boilerplate and is generally easier to pick up and use compared to Redux, especially for projects where extreme complexity or extensive middleware isn't an immediate requirement.
    *   **Alternative:** `@reduxjs/toolkit` (RTK) would be considered if the application's state logic grows significantly more complex, necessitating features like the Redux DevTools for advanced debugging, more sophisticated middleware, or a highly structured, opinionated global state pattern.
*   **Examples of Managed Global State:**
    *   `currentLanguage: string | null`: The programming language currently selected by the user (e.g., 'python', 'javascript').
    *   `activeTutorialId: string | null`: The unique identifier of the tutorial currently being viewed.
    *   `userProgress: Record<string, Record<string, { completedLessons: string[], status: 'not_started' | 'in_progress' | 'completed' }>>`: A nested object storing the user's progress for each language and tutorial (e.g., `userProgress['python']['basics'] = { completedLessons: ['intro', 'variables'], status: 'in_progress' }`).
    *   `appSettings: { theme: 'light' | 'dark', fontSize: 'medium' }`: User-configurable application settings.
    *   `networkStatus: 'online' | 'offline'`: The current network connectivity status of the application.
    *   `downloadedTutorials: Record<string, boolean>`: A map of tutorial IDs to a boolean indicating if they have been successfully downloaded for offline use.
*   **Structure (Zustand Store in `packages/app-core`):**
    ```typescript
    // packages/app-core/src/store/appStore.ts
    import create from 'zustand';
    import { persist, createJSONStorage } from 'zustand/middleware';
    import { zustandStorage } from './storage'; // Custom storage wrapper (see persistence section)

    interface AppState {
      currentLanguage: string | null;
      activeTutorialId: string | null;
      userProgress: Record<string, any>; // Define more specific type
      appSettings: { theme: 'light' | 'dark'; fontSize: 'medium' };
      // ... other state properties and actions
      setCurrentLanguage: (lang: string) => void;
      updateProgress: (language: string, tutorialId: string, lessonId: string) => void;
    }

    export const useAppStore = create<AppState>()(
      persist(
        (set, get) => ({
          currentLanguage: null,
          activeTutorialId: null,
          userProgress: {},
          appSettings: { theme: 'light', fontSize: 'medium' },
          setCurrentLanguage: (lang) => set({ currentLanguage: lang }),
          updateProgress: (language, tutorialId, lessonId) => {
            // Logic to update userProgress immutably
            const newProgress = { ...get().userProgress };
            // ... update logic ...
            set({ userProgress: newProgress });
          },
          // ... other initial values and actions
        }),
        {
          name: 'codepath-ai-app-storage', // Unique name for the persisted state
          storage: createJSONStorage(() => zustandStorage), // Use custom platform-agnostic storage
        }
      )
    );
    ```
*   **Persistence:** Zustand's `persist` middleware is used to automatically save designated parts of the global store to local storage (`localStorage` on the web, `AsyncStorage` on mobile via a custom storage adapter). This allows the application state (like `currentLanguage`, `userProgress`, `appSettings`) to be rehydrated when the user reopens the app.
    *   A custom `zustandStorage` adapter (shown conceptually above) would abstract `localStorage` and `@react-native-async-storage/async-storage` to provide a unified interface for the `persist` middleware.

## 4. Server State / Asynchronous Data Management

*   **Purpose:** Manages data fetched from external asynchronous sources, primarily the Google Gemini API and web URLs for content updates. This includes handling loading states, errors, caching, background synchronization, and optimistic updates.
*   **Chosen Tool: `@tanstack/react-query` (formerly React Query)**
    *   **Rationale:** React Query is a powerful library specifically designed for managing server state in React applications. It significantly reduces boilerplate for data fetching logic, provides robust caching mechanisms (essential for offline support), handles background updates, and simplifies managing loading and error states.
*   **Examples of Managed Server State Data:**
    *   Tutorial content (lessons, code examples, explanations) fetched from the Gemini API for a specific tutorial ID.
    *   Lists of available tutorials for a selected programming language.
    *   Updated content or new tutorial metadata fetched from web sources.
*   **Usage Example (Conceptual - in `packages/app-core` or feature-specific modules):**
    ```typescript
    // packages/app-core/src/features/tutorials/hooks/useTutorialContentQuery.ts
    import { useQuery } from '@tanstack/react-query';
    import { fetchTutorialContentFromApi } from '../api/tutorialApi'; // Your API fetching function

    // Type for tutorial content
    interface TutorialContent { id: string; title: string; lessons: Array<{ id: string; title: string; content: string; }>; }

    export const useTutorialContentQuery = (tutorialId: string | null) => {
      return useQuery<TutorialContent, Error>(
        ['tutorialContent', tutorialId], // Query key: uniquely identifies this query
        () => fetchTutorialContentFromApi(tutorialId!), // Fetch function
        {
          enabled: !!tutorialId, // Query will only run if tutorialId is truthy
          staleTime: 5 * 60 * 1000, // Data is considered fresh for 5 minutes
          cacheTime: 24 * 60 * 60 * 1000, // Data remains in cache for 24 hours (even if stale)
          // For offline support, React Query can be configured with a persister
          // (e.g., @tanstack/query-sync-storage-persister or a custom one using idb/expo-sqlite)
        }
      );
    };
    ```
*   **Caching and Offline Support with React Query:**
    *   React Query's built-in caching is the first line of defense. Data fetched is cached in memory.
    *   For persistent offline caching, React Query can be integrated with storage persisters. Libraries like `@tanstack/query-sync-storage-persister` (for simple `localStorage`/`AsyncStorage`) or `@tanstack/query-async-storage-persister` can be used. For more complex storage needs (like storing large tutorial content in IndexedDB or SQLite), a custom persister might be developed.
    *   This allows cached API responses to be available even when the application is offline, significantly enhancing usability.

## 5. Data Flow Summary

1.  **User Interaction:** An event occurs (e.g., user clicks a button to select a language).
2.  **Component Handler:** The event handler in the React component is triggered.
3.  **Global State Update (if applicable):** If the interaction affects global state, an action from the Zustand store is called (e.g., `useAppStore.getState().setCurrentLanguage('python')`).
4.  **State Change & Re-render:** The Zustand store updates its state. Components subscribed to this part of the store automatically re-render with the new data.
5.  **Asynchronous Data Fetch (if applicable):** If the interaction requires new data from an external source (e.g., loading tutorials for the newly selected language), a component will use a React Query hook (e.g., `useTutorialsListQuery('python')`).
6.  **React Query Lifecycle:**
    *   Checks its cache for existing (fresh or stale) data matching the query key.
    *   If data is cached and fresh, returns it immediately.
    *   If data is stale or not cached, initiates a fetch using the provided fetch function.
    *   Manages `isLoading`, `isError`, `isSuccess`, `data`, and `error` states, which components can use to render appropriate UI.
    *   Updates its cache with the fetched data.
7.  **Component Renders Data:** The component uses the data (and loading/error states) provided by Zustand and/or React Query to render the UI.
8.  **Persistence:** Throughout this process, Zustand's `persist` middleware and React Query's configured persister automatically save relevant state and cached data to the device's local storage, ensuring it's available for future sessions and offline use.