# Backend Documentation: CodePath AI

## 1. Overview

For its primary intended use as a personal, offline-first language learning application, **CodePath AI does not require a traditional, complex backend server for its core MVP functionality.** The architecture prioritizes client-side operations and local data storage.

Key interactions that might typically involve a backend are handled as follows:

1.  **Google Gemini API Interaction:** Calls to the Google Gemini API are planned to be made directly from the client-side (web/mobile). This approach is suitable for personal use where the API key can be managed securely by the user/developer. For any broader distribution, a backend proxy would be essential for security.
2.  **Web Content Updates:** Fetching updated tutorial content from designated web sources can be initiated by the client or managed by a separate, simple script/service run by the user.
3.  **Data Storage:** All user progress and tutorial content are stored locally on the client device to ensure offline access.

If the application's scope evolves to include features like centralized user accounts, cross-device synchronization beyond simple file sharing, or collaborative features, then a dedicated backend (e.g., Node.js with Express/NestJS, Python with Django/FastAPI, or a Backend-as-a-Service like Supabase or Firebase) would become necessary.

## 2. Google Gemini API Interaction

*   **Method:** Direct client-side HTTP requests from the React (web) or React Native (mobile) application to the Google Gemini API endpoints.
*   **API Key Management (Critical for Personal Use):**
    *   The Google Gemini API key (`AIzaSyAOlgedurL4Uobpyagku_bqHWF7IFKoNAQ`) must **NEVER** be hardcoded directly into version-controlled source code if the repository is public or might be shared.
    *   **Web Application (e.g., using Vite):** Store the API key in an environment variable (e.g., in an `.env` file: `VITE_GEMINI_API_KEY=YOUR_API_KEY`). This file should be added to `.gitignore`.
    *   **Mobile Application (Expo):** Utilize Expo's system for managing secrets or environment variables (e.g., through EAS Build secrets or a configuration file like `app.config.js` that reads from an uncommitted `.env` file during development).
    *   **Security Advisory:** For any application intended for distribution beyond strictly personal use, exposing an API key on the client-side poses a significant security risk. The standard secure practice is to implement a **backend proxy server**. This server would securely store the API key and make requests to the Gemini API on behalf of the client, preventing key exposure.
*   **Functionality:** The API will be used for fetching tutorial content, generating explanations, providing code examples, and potentially powering interactive Q&A features within the app.

## 3. Web Content Update Mechanism

This mechanism allows the app's tutorial content to be updated from external web sources.

*   **MVP Approach (Client-Initiated or Manual Script):**
    1.  **Curated Source List:** Maintain a list of trusted URLs pointing to raw learning content (e.g., Markdown files in GitHub repositories, specific articles on technical blogs that permit reuse).
    2.  **Client-Side Fetching:** The application itself could have a feature allowing the user to trigger a fetch operation from these URLs. The fetched content would then be processed and stored locally.
    3.  **Manual Script:** Alternatively, a separate script (e.g., Node.js or Python) could be developed. The user would run this script periodically. It would download content from the predefined sources, transform it into a consistent JSON or Markdown format, and place these files in a directory accessible to the web and mobile apps (e.g., within their assets or a designated local data folder).
*   **Future Enhancement (Automated via Lightweight Service):**
    *   A serverless function (e.g., using AWS Lambda, Google Cloud Functions, or Vercel/Netlify Functions) could be deployed.
    *   This function would be triggered on a schedule (e.g., daily) or via a webhook.
    *   It would perform the tasks of fetching, processing, and structuring the content from web sources.
    *   The processed content could then be stored in a simple cloud storage solution (e.g., AWS S3, Google Cloud Storage) or a lightweight database, from which the client applications can pull updates.
    *   This approach centralizes the update logic and reduces the burden on the client applications.

## 4. Data Storage (Client-Side Focus)

As detailed in the <mcfile name="frontend-documentation.md" path="c:\Users\hzr90\OneDrive\Desktop\Test\LanguageLearnerApp\docs\frontend-documentation.md"></mcfile>, all primary application data, including tutorial content and user progress, is stored locally on the user's device. This is fundamental to the offline-first requirement.

*   **Web:** IndexedDB, potentially via a wrapper like `idb`.
*   **Mobile:** `expo-sqlite` for structured data and `expo-file-system` for storing content files.
*   No backend database is planned for the personal use case.

## 5. API Design (Hypothetical - If a Backend Proxy/BFF is Introduced)

Should a minimal backend (e.g., a Backend-For-Frontend or API proxy) be introduced later, for instance, to securely manage the Gemini API key:

*   **Technology Stack (Suggestion):** Node.js with a lightweight framework like Express.js or NestJS (if more structure is desired), aligning with the existing JavaScript/TypeScript ecosystem.
*   **Communication Protocol:** RESTful APIs are likely sufficient. GraphQL might be considered if data fetching requirements become significantly more complex.
*   **Example Endpoints (for a Gemini API Proxy):**
    *   `POST /api/v1/gemini/generate-content`
        *   *Request Body:* `{ prompt: string, context: any }`
        *   *Response Body:* `{ content: string }` or error object.
        *   *Action:* Securely forwards the request to the Google Gemini API using the server-stored API key.
    *   `GET /api/v1/content/updates-check` (If backend manages web content updates)
        *   *Response Body:* `{ lastUpdated: string, updateAvailable: boolean, updateUrl?: string }`

## 6. Authentication & Authorization

*   **Not Required:** As explicitly stated in the user requirements, the application is for personal use and mandates unrestricted access. Therefore, no user authentication (login/signup) or authorization mechanisms are to be implemented in the client applications or any potential minimal backend components.

## 7. Hosting (for Potential Minimal Backend Components)

If lightweight backend components (like serverless functions for content updates or a simple API proxy) are developed:

*   **Serverless Functions:** AWS Lambda, Google Cloud Functions, Azure Functions, Vercel Functions, Netlify Functions.
*   **Lightweight Proxy Server/BFF:** Platform-as-a-Service (PaaS) options like Heroku (free/hobby tiers), Render, Fly.io, or container-based hosting on services like Google Cloud Run or AWS Fargate.