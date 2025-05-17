# Testing Strategy Documentation: CodePath AI

## 1. Overview

To ensure the reliability, functionality, and quality of the CodePath AI application, a comprehensive testing strategy is essential. This document outlines the various levels of testing, tools and frameworks to be employed, key areas of focus, and general best practices. The strategy aims to catch bugs early, facilitate refactoring, and provide confidence in the application's behavior, especially concerning its core features like API interaction, data persistence, and offline support.

Testing efforts will cover shared logic in `packages/app-core` as well as platform-specific components and behaviors in the web (React) and mobile (React Native/Expo) applications.

## 2. Testing Pyramid

Our testing approach will follow the principles of the testing pyramid:

1.  **Unit Tests (Foundation - Most Numerous):**
    *   **Focus:** Testing the smallest, isolated units of code, such as individual functions, React components in isolation (presentational aspects), utility functions, and specific logic within state management (e.g., Zustand store actions/selectors, individual reducers if `useReducer` is heavily used).
    *   **Goal:** Verify that each unit behaves as expected given various inputs.
    *   **Tools:** Jest, React Testing Library (for component rendering and basic interaction).

2.  **Integration Tests (Middle Layer - Moderate Number):**
    *   **Focus:** Testing the interaction between several units or modules. Examples include:
        *   A component interacting with a Zustand store.
        *   A service function making a (mocked) API call and processing the response.
        *   Components that fetch data using React Query hooks (testing the hook's integration with mocked API services).
        *   Interaction with local database services (IndexedDB/SQLite mocks).
    *   **Goal:** Ensure that different parts of the application work together correctly.
    *   **Tools:** Jest, React Testing Library, Mock Service Worker (MSW) for API mocking, mocked storage implementations.

3.  **End-to-End (E2E) Tests (Peak - Fewest, Most Comprehensive):**
    *   **Focus:** Simulating real user scenarios from the user's perspective, interacting with the full application stack (UI, state management, mocked APIs, and local storage interactions).
    *   **Goal:** Validate that complete user flows function as intended and meet user requirements.
    *   **Tools (Considerations for a personal project):**
        *   **Web:** Cypress or Playwright.
        *   **Mobile (Expo/React Native):** Detox or Appium (can have a steeper setup curve).
        *   **Initial Approach:** For this personal project, manual E2E testing will be the primary method for validating full user flows. Automated E2E tests might be introduced selectively for the most critical paths if the project complexity grows significantly.

## 3. Tools and Frameworks

*   **Test Runner & Assertion Library:** **Jest**
    *   Industry standard for JavaScript testing, well-integrated with React/React Native ecosystems.
    *   Provides test execution, a rich assertion library (`expect`), and powerful mocking capabilities.
*   **Component Testing:**
    *   **`@testing-library/react`** (for web)
    *   **`@testing-library/react-native`** (for mobile)
    *   Promotes testing components based on user interaction and accessibility, leading to more resilient tests.
*   **API Mocking:**
    *   **Mock Service Worker (MSW):** Intercepts actual network requests at the network level, allowing for realistic API mocking without modifying application code. Ideal for integration tests involving API calls.
    *   **Jest Mocks (`jest.fn()`, `jest.spyOn()`, `jest.mock()`):** Useful for mocking specific functions or modules within unit tests or simpler integration tests.
*   **Static Analysis & Code Quality:**
    *   **TypeScript:** Catches type-related errors at compile time.
    *   **ESLint:** Enforces code style and identifies problematic patterns.
    *   **Prettier:** Ensures consistent code formatting.
*   **Accessibility Testing (Web):**
    *   **`jest-axe`:** Integrates with Jest to automatically test rendered component output for basic WCAG violations.

## 4. Key Areas and Strategies for Testing

### 4.1. Core Logic (`packages/app-core`)

*   **State Management (Zustand):**
    *   Unit test individual actions: verify state changes correctly.
    *   Unit test selectors: ensure they derive the correct data from the state.
    *   Test persistence/rehydration logic by mocking the underlying storage (`localStorage`/`AsyncStorage`).
*   **API Service Functions (e.g., `geminiService.ts`, `webContentService.ts`):**
    *   Integration test with MSW to mock various API responses (successful data, error responses like 4xx/5xx, network errors).
    *   Unit test any complex data transformation or parsing logic within these services.
*   **Data Storage Services (e.g., `webDbService.ts`, `mobileDbService.ts`):**
    *   Integration test CRUD (Create, Read, Update, Delete) operations. This might involve mocking IndexedDB/SQLite APIs or using in-memory versions if feasible for the testing environment.
    *   Test data migration logic if schema versions are introduced.
*   **Utility Functions:** Thoroughly unit test all utility functions with a variety of inputs, including edge cases and invalid inputs.

### 4.2. UI Components (Web & Mobile)

*   **Rendering:** Verify components render correctly based on different props and states.
*   **User Interactions:** Simulate user events (clicks, input changes, gestures) using React Testing Library and assert the expected outcomes (e.g., state changes, callback invocations, UI updates).
*   **Conditional Rendering:** Test that UI elements appear, disappear, or change based on application state (e.g., loading indicators, error messages, data-driven content, offline status).
*   **Accessibility (Web):** Use `jest-axe` for automated checks on rendered component output.

### 4.3. React Query Hooks & Data Fetching

*   For custom hooks that encapsulate `useQuery` or `useMutation`:
    *   Mock the API service layer (using MSW or Jest mocks).
    *   Verify correct handling of `isLoading`, `isSuccess`, `isError`, `data`, and `error` states.
    *   Test caching behavior (e.g., data is served from cache on subsequent identical requests, stale-while-revalidate logic if configured).
    *   Test `enabled` option behavior and query retries.

### 4.4. Offline Support

*   **Network Status Simulation:** Mock the network status detection mechanism (`navigator.onLine`, `NetInfo`) to force the application into 'online' or 'offline' states within tests.
*   **Data Access Offline:** Verify that previously downloaded/cached tutorial content is accessible and rendered correctly when the application is in an offline state.
*   **Progress Saving Offline:** Ensure user progress updates are saved to local storage when offline.
*   **UI Feedback:** Test that UI elements correctly reflect the offline status (e.g., offline indicators, disabled online-only buttons).
*   **Synchronization on Reconnect:** Test that React Query's `refetchOnReconnect` behavior works as expected for relevant queries.

### 4.5. User Flows (Primarily Manual E2E for MVP)

Key user flows to be manually tested thoroughly:
1.  **Application Launch:** Initial state, loading of persisted settings/progress.
2.  **Language and Tutorial Selection:** Navigating to a language, viewing its tutorials, selecting one.
3.  **Tutorial Interaction:** Navigating through lessons, viewing content, interacting with code snippets (if interactive).
4.  **Offline Scenario:** Launching app offline, accessing downloaded content, making progress, then going online to see if anything needs syncing (though for a personal app, local state is primary).
5.  **Settings Management:** Changing theme or other settings and verifying they persist.
6.  **Error Handling:** Simulating API errors (e.g., via browser dev tools or temporary MSW overrides) and verifying user-friendly error messages and graceful degradation.

## 5. Test Execution and Environment

*   **Local Development:** Run tests frequently using `jest --watch` to get immediate feedback.
*   **Pre-commit Hooks:** Implement pre-commit hooks (e.g., using Husky and `lint-staged`) to automatically run linters, Prettier, and potentially a subset of fast unit tests before code is committed.
*   **Continuous Integration (CI) - Future Consideration:** If the project were to be hosted on a platform like GitHub, setting up a CI pipeline (e.g., GitHub Actions) to run all automated tests on every push or pull request would be beneficial for catching regressions early.

## 6. Test Data Management

*   Utilize mock data fixtures (e.g., JSON files or inline objects in test files) to represent API responses, initial store states, and data for seeding mocked databases.
*   Keep mock data realistic enough to cover test cases but concise and easy to manage.

## 7. Iterative Testing Approach

*   Begin by writing unit tests for critical core logic (utilities, state actions) and complex UI components.
*   Incrementally add integration tests for key service interactions (API calls, database operations) and component compositions.
*   Focus manual E2E testing on the most critical and frequently used user flows.
*   Testing is an ongoing activity: write new tests for new features and write tests to cover any bugs found and fixed (regression testing).

This testing strategy provides a framework for building a high-quality, reliable CodePath AI application.