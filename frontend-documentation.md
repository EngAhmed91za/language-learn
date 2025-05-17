# Frontend Documentation: CodePath AI

## 1. Overview

The application will consist of two primary frontend clients:

1.  **Web Application:** Built using React, targeting modern web browsers.
2.  **Mobile Application:** Built using React Native (managed via Expo), targeting iOS and Android platforms.

The overarching goal is to maximize code reusability for business logic, state management, and type definitions (primarily through a shared `app-core` package) while delivering a native look and feel and optimized user experience for each platform. TypeScript will be the primary language for both frontends to ensure type safety, improve developer experience, and enhance code maintainability.

## 2. Tech Stack

### Web Application

*   **Framework:** React (latest stable version)
*   **Language:** TypeScript
*   **Build Tool/Bundler:** Vite (recommended for its speed and modern development experience) or Create React App (CRA) if preferred.
*   **Routing:** React Router (`react-router-dom`) for declarative, client-side navigation.
*   **Styling:**
    *   **Primary:** Tailwind CSS for utility-first styling, enabling rapid UI development and consistency.
    *   **Optional/Complementary:** CSS Modules for component-scoped styles if needed, or a CSS-in-JS library like Emotion or Styled Components for dynamic styling requirements.
*   **UI Component Library (Optional):** Consider Material-UI (MUI), Chakra UI, or Ant Design for a set of pre-built, accessible components to accelerate development, if a specific design system is desired beyond Tailwind's capabilities.

### Mobile Application

*   **Framework:** React Native (managed with Expo for simplified setup, build, and deployment).
*   **Language:** TypeScript
*   **Routing:** Expo Router (file-system based routing, offering a streamlined approach similar to Next.js for web).
*   **UI Framework/Library:** React Native Paper (as suggested in the initial prompt, provides Material Design components out-of-the-box, supporting theming).
*   **Styling:**
    *   React Native's built-in `StyleSheet` API for component-specific styles.
    *   Potentially a utility library for styling (e.g., `twrnc` for Tailwind-like utilities in React Native if a consistent styling paradigm with web is highly desired) or `styled-components/native` for CSS-in-JS if preferred.

## 3. Project Structure (Monorepo Recommended)

A monorepo architecture (e.g., using Yarn Workspaces, PNPM Workspaces, Lerna, or Nx) is highly recommended to efficiently manage shared code, dependencies, and configurations between the web and mobile applications.

```plaintext
/LanguageLearnerApp
├── packages/
│   ├── app-core/           # Shared: business logic, TypeScript types/interfaces, API service layers, state management (stores, actions, selectors), utility functions, constants.
│   │   ├── src/
│   │   └── package.json
│   ├── web-app/            # React web application specific code
│   │   ├── public/
│   │   ├── src/
│   │   │   ├── components/     # Reusable UI components specific to web
│   │   │   ├── pages/          # Page-level components (routed)
│   │   │   ├── App.tsx
│   │   │   └── main.tsx
│   │   ├── vite.config.ts  # Or similar for CRA
│   │   └── package.json
│   └── mobile-app/         # React Native (Expo) mobile application specific code
│       ├── app/            # Screens/routes for Expo Router
│       ├── assets/
│       ├── components/     # Reusable UI components specific to mobile
│       ├── expo-config.js
│       ├── App.tsx
│       └── package.json
├── docs/                 # All markdown documentation files
├── .gitignore
├── package.json          # Root package.json for monorepo (manages workspaces)
└── tsconfig.base.json    # Base TypeScript configuration, extended by packages
```

## 4. Key UI Components (Examples - To Be Expanded)

This list is illustrative and will grow as the app is designed in more detail. Components should be designed with reusability in mind, though platform-specific implementations might be necessary.

*   **Shared Concept / Platform-Specific Implementation:**
    *   `Container/Layout`: Wrappers for screen/page structure.
    *   `Button`: Standard button component.
    *   `Card`: For displaying tutorial snippets or language choices.
    *   `List/ListItem`: For displaying lists of tutorials or lessons.
    *   `Modal`: For pop-up dialogs or information.
    *   `TextInput`: For any search or input fields.
*   **App-Specific Components:**
    *   `LanguageSelectorCard`: Displays a language and allows selection.
    *   `TutorialListItem`: Represents a single tutorial in a list, showing title, brief description, progress.
    *   `TutorialDetailView`: Renders the full content of a selected tutorial, including text, code blocks, and interactive elements.
    *   `LessonNavigationView`: Buttons or tabs for navigating between lessons/sections within a tutorial.
    *   `CodeBlockDisplay`: Formats and displays code snippets, ideally with syntax highlighting (e.g., using `react-syntax-highlighter` for web, a similar library or custom solution for mobile).
    *   `InteractiveExerciseWrapper`: A component to host different types of interactive coding exercises.
    *   `GlobalSearchBar`: For searching across all available content.
    *   `OfflineModeIndicator`: A visual cue (e.g., a small banner or icon) indicating if the app is currently operating in offline mode.
    *   `ProgressBar`: To show user progress within a tutorial or language.

## 5. User Flows

This section outlines the key user journeys through the application. Each flow should describe the steps a user takes to complete a specific task.

### Learning a New Language

1.  **Select Language:** User lands on the home screen and selects a language from the available options.
2.  **Browse Tutorials:** User views a list of tutorials for the selected language.
3.  **Start Tutorial:** User selects a tutorial to begin.
4.  **Engage with Content:** User reads explanations, interacts with code examples, and completes exercises.
5.  **Track Progress:** User's progress within the tutorial is saved.
6.  **Complete Tutorial:** User finishes the tutorial and potentially moves to the next one.

### Signing Up / Logging In

1.  **Access Authentication:** User clicks on a 'Sign Up' or 'Log In' button/link.
2.  **Enter Credentials:** User provides email/username and password.
3.  **Submit Form:** User submits the form.
4.  **Authentication:** System verifies credentials.
5.  **Access App:** Upon successful authentication, user is directed to the home screen or a relevant dashboard.

### Searching for Tutorials

1.  **Access Search:** User clicks on the global search bar.
2.  **Enter Query:** User types keywords related to the desired tutorial.
3.  **View Results:** Application displays a list of matching tutorials.
4.  **Select Tutorial:** User clicks on a search result to view the tutorial details.

### Using Offline Mode

1.  **Enable Offline Mode:** User navigates to settings and toggles the offline mode option (or the app automatically detects offline status).
2.  **Access Downloaded Content:** User browses and accesses tutorials that were previously downloaded.
3.  **View Offline Indicator:** An indicator is visible to show the user is in offline mode.
4.  **Attempt Online Action (Optional):** If the user attempts an action requiring network (e.g., downloading a new tutorial), the app provides feedback that it's unavailable offline.

## 6. Screen Layouts / Wireframes

This section provides high-level visual representations of the main screens and their key elements. Wireframes help illustrate the structure and layout without focusing on detailed styling.

### Home Screen

*   **Layout:** Grid or list of `LanguageSelectorCard` components.
*   **Elements:** App header (with potential user profile/settings access), Global Search Bar, list of available languages.

### Tutorial List Screen

*   **Layout:** List view.
*   **Elements:** Header (with back button to Home, language name), Global Search Bar, list of `TutorialListItem` components.

### Tutorial Detail Screen

*   **Layout:** Scrollable content view.
*   **Elements:** Header (with back button to Tutorial List, tutorial title), `ProgressBar`, main content area displaying tutorial sections (`CodeBlockDisplay`, text, `InteractiveExerciseWrapper`), `LessonNavigationView` (for multi-part tutorials).

### Profile / Settings Screen

*   **Layout:** Form/list view.
*   **Elements:** Header, user information display, options for settings (e.g., theme, offline mode toggle), potentially sign-out button.

*(Include wireframes or descriptions of key screens here, e.g., Home Screen, Tutorial List Screen, Tutorial Detail Screen, Profile Screen, etc.)*

## 7. Detailed Component Descriptions

This section provides high-level visual representations of the main screens and their key elements. Wireframes help illustrate the structure and layout without focusing on detailed styling.

*(Include wireframes or descriptions of key screens here, e.g., Home Screen, Tutorial List Screen, Tutorial Detail Screen, Profile Screen, etc.)*

## 7. Detailed Component Descriptions

Building upon the list in Section 4, this section provides more detailed documentation for individual UI components, including their purpose, props, state, and any specific behaviors.

### Example: `LanguageSelectorCard`

*   **Purpose:** Displays a single language option on the home screen, allowing the user to select it.
*   **Props:**
    *   `languageName` (string): The name of the language.
    *   `icon` (ReactNode): An icon representing the language.
    *   `onSelect` (function): Callback function triggered when the card is clicked.
*   **State:** None (stateless component).
*   **Behavior:** When clicked, it should call the `onSelect` function with the selected language's identifier.

*(Add detailed descriptions for other key components here.)*

## 8. Future Enhancements

This section lists potential features or improvements planned for future iterations of the application.

*(List future ideas here, e.g., Offline mode, Gamification features, More language options, etc.)*

## 5. State Management

A layered approach to state management will be adopted:

*   **Local Component State:** Managed using React's built-in hooks (`useState`, `useReducer`) for UI state that is specific to a single component (e.g., form inputs, toggle states).
*   **Global Application State:**
    *   **Solution:** Zustand or Redux Toolkit (RTK). Zustand is favored for its simplicity and minimal boilerplate, making it a good fit for this project unless complex middleware or a very large state structure becomes necessary.
    *   **Managed State:** Current selected language, active tutorial, user progress across tutorials/languages, application settings (e.g., theme preference), offline status, recently accessed items.
    *   **Location:** Global state logic (stores, actions, hooks) will reside in the `packages/app-core` to be shared between web and mobile.
*   **Server State / Data Fetching & Caching:**
    *   **Solution:** React Query (`@tanstack/react-query`) or SWR. React Query is robust and provides excellent tools for managing asynchronous data, caching, background updates, and optimistic updates.
    *   **Usage:** For fetching tutorial content (initially from local/bundled JSON, later from the Google Gemini API or dynamic web updates). It will handle caching strategies crucial for offline support and minimizing API calls.
    *   **Location:** API interaction logic and React Query hooks/configurations will be part of `packages/app-core`.
*   **Persistence for Offline Support:**
    *   **Strategy:** Persist essential global state (e.g., user progress, downloaded tutorial content identifiers, settings) and cached data from React Query.
    *   **Web:** IndexedDB (via a lightweight wrapper like `idb` from Jake Archibald) for structured data and larger content. `localStorage` for very simple key-value pairs (like user preferences).
    *   **Mobile (Expo):**
        *   `expo-sqlite` for structured relational data (e.g., user progress, metadata of downloaded tutorials).
        *   `expo-file-system` for storing larger tutorial content files (e.g., JSON or markdown files).
        *   `AsyncStorage` (via `@react-native-async-storage/async-storage`) for simple key-value data.
    *   **Integration:** State management solutions like Zustand (with `zustand/middleware/persist`) and React Query (with custom persisters or `persistQueryClient`) can be configured to automatically save and rehydrate their state to/from the chosen storage mechanisms.

## 6. Navigation

*   **Web:** `React Router` (`react-router-dom`) will be used for its declarative routing capabilities, enabling standard web navigation patterns (URL-based routing, browser history integration).
*   **Mobile:** `Expo Router` will be used. It offers file-system-based routing, simplifying the setup of native navigation patterns like stack navigation, tab navigation, and deep linking.

## 7. Styling Approach

*   **Web:** Primarily **Tailwind CSS** for its utility-first approach, allowing for rapid development and consistent styling. CSS Modules can be used for components requiring more complex, scoped styling not easily achieved with Tailwind alone.
*   **Mobile:** **React Native `StyleSheet.create`** for performant, component-specific styles. **React Native Paper** will provide a base set of Material Design themed components. For more advanced theming or utility styles, consider libraries like Restyle (Shopify) or `twrnc` if a Tailwind-like experience is desired (though this adds a dependency and potential learning curve).

## 8. Accessibility (A11y)

Ensuring the application is accessible to all users is a priority.

*   **Web:** Adhere to Web Content Accessibility Guidelines (WCAG) 2.1 AA standards. This includes using semantic HTML, providing text alternatives for non-text content, ensuring keyboard navigability, sufficient color contrast, and proper ARIA attribute usage where necessary.
*   **Mobile:** Follow platform-specific accessibility guidelines for iOS and Android. Utilize React Native's accessibility props (`accessible`, `accessibilityLabel`, `accessibilityHint`, `accessibilityRole`, `accessibilityState`) to make components understandable and operable via assistive technologies.
*   **Testing:** Regularly test with screen readers (e.g., NVDA/JAWS for web, VoiceOver for iOS, TalkBack for Android) and keyboard-only navigation.

## 9. Internationalization (i18n) - Future Consideration

While not an MVP requirement, the architecture should be planned to accommodate future internationalization (e.g., using a library like `i18next` with shared translation files in `app-core`).