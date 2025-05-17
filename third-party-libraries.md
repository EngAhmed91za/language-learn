# Third-Party Libraries Documentation: CodePath AI

This document outlines the key third-party libraries anticipated for use in the CodePath AI project. The selection prioritizes functionality, maintainability, performance, and compatibility with the React/React Native (Expo) ecosystem. This list will evolve as development progresses and specific needs are refined.

## 1. Core / Shared Libraries (intended for `packages/app-core`)

These libraries provide functionalities common to both web and mobile applications.

*   **State Management:**
    *   **`zustand`**: A small, fast, and scalable bearbones state-management solution. Chosen for its simplicity and minimal boilerplate compared to Redux, making it suitable for this project's scale.
        *   *Alternatives:* `@reduxjs/toolkit` with `react-redux` if a more structured, opinionated global state solution with extensive middleware capabilities is preferred.
*   **Data Fetching & Server State Caching:**
    *   **`@tanstack/react-query` (v4 or v5)**: Powerful asynchronous state management for fetching, caching, synchronizing, and updating server state. Essential for managing data from the Gemini API and web content updates, and for offline support.
*   **HTTP Client (for API Calls):**
    *   **`axios`**: A promise-based HTTP client for the browser and Node.js. Offers features like request/response interception, timeout configuration, and better error handling than the native `fetch` API.
        *   *Alternatives:* Native `fetch` API (requires more manual setup for features like timeouts and interceptors).
*   **Utility Functions:**
    *   **`lodash`** (or individual modules like `lodash.debounce`, `lodash.throttle`, `lodash.get`): Provides a comprehensive suite of utility functions. Consider importing specific functions to minimize bundle size (e.g., `import get from 'lodash/get'`).
    *   **`date-fns`** or **`dayjs`**: Lightweight and modern libraries for date and time manipulation. Preferred over Moment.js due to smaller bundle sizes and immutability.
*   **Google Gemini API SDK:**
    *   **`@google/generative-ai`**: The official Google AI JavaScript SDK for interacting with the Gemini API. This will be used for all communications with the Gemini service.
*   **Type Definitions:**
    *   Various `@types/*` packages from DefinitelyTyped (e.g., `@types/lodash`) for libraries that do not bundle their own TypeScript definitions.

## 2. Web Application Libraries (`packages/web-app`)

*   **Framework & Core:**
    *   `react`
    *   `react-dom`
*   **Routing:**
    *   `react-router-dom` (v6+): For declarative client-side routing.
*   **Styling:**
    *   `tailwindcss`: A utility-first CSS framework for rapid UI development.
    *   `postcss`, `autoprefixer`: Peer dependencies for Tailwind CSS.
    *   *(Optional, if CSS-in-JS is needed for specific components):* `@emotion/react`, `@emotion/styled` or `styled-components`.
*   **UI Component Library (Optional - if a pre-built set is desired):**
    *   `@mui/material` (Material-UI) along with `@emotion/react`, `@emotion/styled`.
    *   `@chakra-ui/react` along with `@emotion/react`, `@emotion/styled`, `framer-motion`.
*   **Syntax Highlighting (for code blocks):**
    *   `react-syntax-highlighter`: To render code snippets with language-specific syntax highlighting.
*   **Local Storage (Offline - IndexedDB Wrapper):**
    *   `idb`: A small, promise-based wrapper around the IndexedDB API, making it easier to use.

## 3. Mobile Application Libraries (`packages/mobile-app` - Managed by Expo)

Expo SDK manages versions for many core React Native libraries and provides its own suite of APIs.

*   **Framework & Core (Expo SDK):**
    *   `react`, `react-native`
    *   `expo`: The core Expo package.
    *   `expo-router`: For file-system based navigation.
    *   `expo-sqlite`: For local SQLite database access.
    *   `expo-file-system`: For direct file system access (storing/reading tutorial content files).
    *   `expo-asset`: For managing assets like images and fonts.
    *   `expo-font`: For loading custom fonts.
    *   `expo-splash-screen`, `expo-status-bar`: For controlling the splash screen and status bar.
*   **UI Framework:**
    *   `react-native-paper`: Provides Material Design components and theming for React Native.
*   **Persistent Storage (Key-Value):**
    *   `@react-native-async-storage/async-storage`: For simple, asynchronous, persistent, key-value storage.
*   **Syntax Highlighting (for code blocks):**
    *   `react-native-syntax-highlighter` (or a similar native-focused library). If performance with large code blocks becomes an issue, a custom component leveraging a WebView with a web-based highlighter might be a fallback, but native solutions are preferred.
*   **SVG Support (if needed for icons/illustrations not covered by fonts/Paper):**
    *   `react-native-svg`: For rendering SVG images.

## 4. Development & Tooling (Project Root or relevant packages)

*   **Language:**
    *   `typescript`
*   **Linters & Formatters:**
    *   `eslint`: With plugins such as `eslint-plugin-react`, `eslint-plugin-react-hooks`, `@typescript-eslint/eslint-plugin`, `eslint-plugin-prettier`, `eslint-plugin-react-native`.
    *   `prettier`: For consistent code formatting.
    *   `husky`: For managing Git hooks (e.g., pre-commit).
    *   `lint-staged`: To run linters/formatters on staged files before committing.
*   **Monorepo Management (if adopted):**
    *   Native Yarn/PNPM workspace features, or tools like `Lerna` or `Nx` for more advanced monorepo capabilities.
*   **Build Tools (Web):**
    *   `vite`: As the primary build tool for the web application.
    *   *(If CRA is used instead):* `react-scripts`.

## Considerations for Library Selection:

*   **Bundle Size & Performance:** Especially critical for the web application. Prioritize libraries that are optimized for size and performance. Utilize bundle analysis tools.
*   **Maintenance & Community Support:** Favor libraries that are actively maintained, have good documentation, and a strong community backing.
*   **Compatibility:** Ensure chosen libraries are compatible with the target versions of React, React Native, Expo SDK, and TypeScript.
*   **Licensing:** Verify that the licenses of all third-party libraries are compatible with the project's intended use (most open-source libraries use permissive licenses like MIT or Apache 2.0).
*   **Security:** Regularly audit dependencies for known vulnerabilities using tools like `npm audit`, Snyk, or GitHub Dependabot. Keep libraries updated to patched versions.