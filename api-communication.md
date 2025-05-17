# API Communication Documentation: CodePath AI

## 1. Overview

This document details how CodePath AI communicates with external APIs and services. The primary goals are to ensure secure, efficient, and reliable data exchange for fetching tutorial content and updates.

*   **Primary API:** Google Gemini API (for generating and retrieving tutorial content).
*   **Secondary Sources:** Predefined web URLs (e.g., GitHub repositories, trusted blogs) for fetching curated tutorial lists or supplementary content.
*   **Key Principles:** Secure API key management, robust error handling, efficient data caching (leveraging React Query), and clear feedback to the user regarding network operations.

## 2. Google Gemini API Communication

### 2.1. SDK and Method

*   The official `@google/generative-ai` JavaScript SDK will be utilized for all interactions with the Google Gemini API. This ensures access to the latest features and best practices for API communication.
*   Interactions will primarily involve sending carefully constructed prompts to appropriate Gemini models (e.g., `gemini-pro`) to request tutorial content, explanations, code examples, or answers to programming-related questions.

### 2.2. API Key Management (CRITICAL)

Given the sensitivity of the API key (`AIzaSyAOlgedurL4Uobpyagku_bqHWF7IFKoNAQ`), its management is paramount. **Under no circumstances should this key be hardcoded directly into version-controlled client-side code if the application were to be distributed.**

*   **For Personal, Local-Only Use (Current Scope):**
    *   **Web (Vite/Create React App):**
        *   Store the API key in an environment variable within a `.env` file at the root of the web project (e.g., `apps/web/.env`).
        *   Content: `VITE_GEMINI_API_KEY=AIzaSyAOlgedurL4Uobpyagku_bqHWF7IFKoNAQ`
        *   Access in code: `import.meta.env.VITE_GEMINI_API_KEY`.
        *   **Crucially, `apps/web/.env` MUST be added to the project's root `.gitignore` file.**
    *   **Mobile (Expo):**
        *   Utilize Expo's system for managing environment variables or build secrets.
        *   One common approach is to use a configuration file (e.g., `app.config.js` or `app.json`) with a placeholder, and inject the actual key during the build process or via a non-committed local configuration file that `expo-constants` can read.
        *   Example using `extra` in `app.config.js` (ensure the actual key isn't committed):
            ```javascript
            // apps/mobile/app.config.js
            export default {
              // ... other configurations
              extra: {
                geminiApiKey: process.env.GEMINI_API_KEY || 'YOUR_FALLBACK_OR_PLACEHOLDER_KEY_FOR_DEV',
                eas: {
                  projectId: "your-eas-project-id"
                }
              },
            };
            ```
            The `GEMINI_API_KEY` environment variable would be set in the build environment (e.g., EAS Build secrets) or a local `.env` file not committed to git.
*   **Security Best Practice (If App Were Distributed Publicly or to Others):**
    *   A backend proxy server (e.g., a simple Node.js/Express app or a serverless function) would be implemented.
    *   The client application (web/mobile) would make requests to this proxy.
    *   The proxy server would securely store the Gemini API key (e.g., as an environment variable on the server) and make the actual calls to the Gemini API on behalf of the client.
    *   This approach completely shields the API key from the client-side.

### 2.3. Request Structure (Conceptual Example)

API calls will be encapsulated within service functions, typically residing in `packages/app-core/src/services/geminiService.ts`.

```typescript
// packages/app-core/src/services/geminiService.ts
import { GoogleGenerativeAI, HarmCategory, HarmBlockThreshold } from '@google/generative-ai';

// This function should ideally determine the API key based on the platform (web/mobile)
// For simplicity in this example, we'll assume it's available.
const getApiKey = (): string => {
  // Web (Vite)
  if (import.meta.env && import.meta.env.VITE_GEMINI_API_KEY) {
    return import.meta.env.VITE_GEMINI_API_KEY;
  }
  // Mobile (Expo - assuming it's loaded into Constants)
  // import Constants from 'expo-constants';
  // if (Constants.expoConfig?.extra?.geminiApiKey) {
  //   return Constants.expoConfig.extra.geminiApiKey;
  // }
  // Fallback or error if not found - for robust implementation
  console.warn('Gemini API Key not found. Please ensure it is configured correctly.');
  return 'MISSING_API_KEY'; // Or throw an error
};

const API_KEY = getApiKey();
const genAI = new GoogleGenerativeAI(API_KEY);

const model = genAI.getGenerativeModel({
  model: 'gemini-1.5-flash-latest', // Or 'gemini-pro' or other suitable models
});

const generationConfig = {
  temperature: 0.7, // Controls randomness. Lower for more factual, higher for more creative.
  topK: 0,
  topP: 0.95,
  maxOutputTokens: 8192,
};

const safetySettings = [
  { category: HarmCategory.HARM_CATEGORY_HARASSMENT, threshold: HarmBlockThreshold.BLOCK_MEDIUM_AND_ABOVE },
  { category: HarmCategory.HARM_CATEGORY_HATE_SPEECH, threshold: HarmBlockThreshold.BLOCK_MEDIUM_AND_ABOVE },
  { category: HarmCategory.HARM_CATEGORY_SEXUALLY_EXPLICIT, threshold: HarmBlockThreshold.BLOCK_MEDIUM_AND_ABOVE },
  { category: HarmCategory.HARM_CATEGORY_DANGEROUS_CONTENT, threshold: HarmBlockThreshold.BLOCK_MEDIUM_AND_ABOVE },
];

export async function fetchContentFromGemini(promptText: string): Promise<string> {
  if (API_KEY === 'MISSING_API_KEY') {
    throw new Error('Gemini API Key is not configured. Cannot fetch content.');
  }

  try {
    const parts = [{ text: promptText }];
    const result = await model.generateContent({
      contents: [{ role: 'user', parts }],
      generationConfig,
      safetySettings,
    });

    const response = result.response;
    if (response.promptFeedback?.blockReason) {
        throw new Error(`Content generation blocked: ${response.promptFeedback.blockReason}`);
    }
    return response.text();
  } catch (error) {
    console.error('Error fetching content from Gemini API:', error);
    // Enhance error message or classify error type for better UI feedback
    if (error instanceof Error && error.message.includes('API key not valid')) {
        throw new Error('Invalid Gemini API Key. Please check your configuration.');
    }
    throw new Error('Failed to fetch content from Gemini. Please try again later.');
  }
}
```

### 2.4. Response Handling and Parsing

*   Gemini API responses (typically Markdown or structured text) will need to be parsed and transformed into a consistent data structure that the application can render (e.g., an array of lesson objects, each with title, content, code examples).
*   This parsing logic will reside in the service layer or dedicated utility functions.
*   Error responses from the API (e.g., rate limits, invalid requests, content blocking) will be caught and handled gracefully. React Query's error state will be used to inform the UI.

### 2.5. Caching with React Query

*   `@tanstack/react-query` will be used to manage API calls to Gemini.
*   Successful responses will be cached, reducing redundant API calls for the same content and enabling offline access to previously fetched tutorials.
*   Cache keys will be structured carefully (e.g., `['geminiTutorial', language, tutorialId]`) to ensure correct data retrieval.

## 3. Web Content Update Communication

### 3.1. Purpose

To fetch lists of available tutorials, updates to existing tutorial metadata, or potentially supplementary content from predefined, trusted web sources (e.g., a JSON file in a GitHub repository, a specific API endpoint on a trusted blog).

### 3.2. Method

*   Standard HTTP GET requests will be made using a library like `axios` (or the built-in `fetch` API), wrapped by React Query hooks.
*   The URLs for these web sources will be configurable, likely stored in `packages/app-core/src/config/constants.ts`.
*   Example: Fetching a list of available Python tutorials from a JSON file hosted on GitHub.

```typescript
// packages/app-core/src/services/webContentService.ts
import axios from 'axios';

const TUTORIAL_INDEX_URL = 'https://example.com/path/to/tutorial-index.json'; // Configurable URL

interface TutorialIndexEntry {
  id: string;
  title: string;
  language: string;
  // ... other metadata
}

export async function fetchTutorialIndex(): Promise<TutorialIndexEntry[]> {
  try {
    const response = await axios.get<TutorialIndexEntry[]>(TUTORIAL_INDEX_URL);
    return response.data;
  } catch (error) {
    console.error('Error fetching tutorial index from web source:', error);
    throw new Error('Failed to fetch latest tutorial list.');
  }
}
```

### 3.3. Data Format

*   The application will expect structured data from these web sources, primarily JSON for lists and metadata, or Markdown for actual content if fetched directly.

### 3.4. Caching

*   React Query will also manage the fetching and caching of data from these web sources, similar to how Gemini API responses are handled.

## 4. General API Communication Principles

*   **Centralized Logic:** All API interaction functions will be centralized within service modules in `packages/app-core/src/services/` for better organization and reusability.
*   **Loading States:** UI components will use React Query's `isLoading`, `isFetching` states to display appropriate loading indicators (spinners, skeleton screens).
*   **Error States:** User-friendly error messages will be displayed based on React Query's `isError` and `error` properties. Errors will be logged for debugging.
*   **Timeouts and Retries:** `axios` and React Query provide configurations for request timeouts and automatic retries for transient network issues.
*   **Offline First:** The primary mechanism for offline support is robust caching via React Query, potentially augmented with a persistence layer (e.g., `@tanstack/query-sync-storage-persister`). The UI will clearly indicate when content is being served from cache or when it's unavailable due to being offline and not previously downloaded/cached.