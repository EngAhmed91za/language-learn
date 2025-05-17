# Security Considerations Documentation: CodePath AI

## 1. Overview

While CodePath AI is designed for personal use, implementing sound security practices is essential, primarily to protect the Google Gemini API key and ensure the application's integrity. This document outlines the key security considerations, focusing on API key management, client-side data handling, secure communication, and best practices for dependencies and development.

The guiding principle is to minimize attack surfaces and protect sensitive assets, even in a personal-use context.

## 2. API Key Security (Google Gemini API)

**This is the most critical security aspect for CodePath AI.** The API key (`AIzaSyAOlgedurL4Uobpyagku_bqHWF7IFKoNAQ`) grants access to Google Cloud services, and its compromise could lead to unauthorized usage and associated costs.

### 2.1. Key Storage and Access (Personal Use Scope)

*   **NEVER Hardcode the API Key in Version Control:** The API key must never be directly embedded in JavaScript/TypeScript files or any other files committed to a Git repository.
*   **Web Application (React with Vite/CRA):**
    *   Store the API key in an environment variable: `VITE_GEMINI_API_KEY=AIzaSyAOlgedurL4Uobpyagku_bqHWF7IFKoNAQ`.
    *   This variable should be defined in a `.env` file (e.g., `apps/web/.env`).
    *   **Crucially, this `.env` file MUST be listed in the root `.gitignore` file.**
    *   Access in code: `import.meta.env.VITE_GEMINI_API_KEY`.
*   **Mobile Application (React Native with Expo):**
    *   Utilize Expo's mechanisms for managing secrets/environment variables.
        *   **Expo Application Services (EAS) Build Secrets:** For production builds, store the API key as a secret in EAS Build. It will be injected during the build process.
        *   **Development:** Use a local, non-committed `.env` file and a library like `react-native-dotenv` or configure it via `app.config.js` using `process.env` (ensuring the `.env` file is gitignored).
            ```javascript
            // Example for app.config.js (development, key from .env)
            // Ensure .env contains GEMINI_API_KEY=your_actual_key
            // And .env is in .gitignore
            export default {
              // ... other expo config
              extra: {
                geminiApiKey: process.env.GEMINI_API_KEY,
                eas: {
                  projectId: "your-eas-project-id"
                }
              },
            };
            ```
            Access in code: `Constants.expoConfig.extra.geminiApiKey` (after importing `expo-constants`).

### 2.2. Best Practice (If App Were Distributed or Shared)

*   **Backend Proxy:** The most secure approach would be to implement a backend proxy server. The client application (web/mobile) would make requests to this proxy. The proxy server, running in a secure environment, would securely store the API key (e.g., as a server-side environment variable) and make the actual calls to the Google Gemini API. This completely isolates the API key from the client.

### 2.3. Google Cloud IAM Recommendations for the API Key

*   **API Restrictions:** In the Google Cloud Console, restrict the API key to only allow access to the specific Gemini API services required by the application (e.g., Generative Language API).
*   **Application Restrictions (Limited Usefulness for Client-Side Heavy Apps):** While options exist to restrict keys by IP address or HTTP referrers, these are less effective for purely client-side applications or mobile apps where IPs are dynamic and referrers can be spoofed. They are more useful for keys used by a backend proxy.
*   **Monitoring:** Regularly monitor API usage in the Google Cloud Console for any unexpected or suspicious activity.
*   **Regeneration:** Consider regenerating the API key periodically, especially if there's any suspicion of compromise (though this can be disruptive for a personal app if not managed carefully).

## 3. Client-Side Data Security

### 3.1. Data Stored Locally

*   Tutorial content (text, code examples).
*   User progress (completed lessons, current status).
*   Application settings (theme, font size).

### 3.2. Sensitivity and Protection

*   For personal use, this data is generally not highly sensitive from a third-party privacy perspective. The main concern is data integrity and availability for the user.
*   **No Encryption at Rest (Current Scope):** Data stored in IndexedDB (web), SQLite (mobile), `localStorage` (web), and `AsyncStorage` (mobile) will not be encrypted by the application itself in the MVP. These storage mechanisms rely on the underlying operating system and browser security.
    *   **Consideration for Future/Shared Use:** If the application were to handle more sensitive information or be distributed, client-side encryption of data at rest would be a significant consideration. For mobile, `expo-secure-store` could be used for specific sensitive items. Web-based client-side encryption is more complex and often relies on user-provided passwords to derive encryption keys.

### 3.3. Cross-Site Scripting (XSS) - Web Specific

*   **Risk:** If tutorial content fetched from the Gemini API or other external web sources contains malicious scripts, and this content is rendered directly as HTML without proper sanitization, XSS vulnerabilities could arise.
*   **Mitigation:**
    *   **Treat External Content as Untrusted:** Always assume content from external APIs/URLs could be malicious.
    *   **Sanitization:** When rendering HTML content that originates externally (e.g., converting Markdown to HTML), use a reputable HTML sanitization library like `DOMPurify` on the web. This library will strip out dangerous tags (e.g., `<script>`, `<iframe>`) and attributes (e.g., `onerror`).
        ```javascript
        // Example with DOMPurify
        import DOMPurify from 'dompurify';
        const unsafeHtmlFromApi = '<p>Hello <script>alert("XSS")</script></p>';
        const cleanHtml = DOMPurify.sanitize(unsafeHtmlFromApi);
        // Render `cleanHtml` using dangerouslySetInnerHTML
        ```
    *   **React's Default Escaping:** React automatically escapes string content rendered within JSX (e.g., `{variable}`), which helps prevent many XSS attacks. However, using `dangerouslySetInnerHTML={{ __html: content }}` bypasses this, making sanitization essential for such cases.
    *   **Code Snippet Rendering:** Use syntax highlighting libraries that are designed to display code safely and do not execute it.

## 4. Secure Communication (HTTPS)

*   All API calls to the Google Gemini API and any other external web services (for content updates, etc.) **MUST** be made over HTTPS.
*   HTTPS encrypts data in transit, protecting it from eavesdropping and man-in-the-middle attacks.
*   The `@google/generative-ai` SDK and modern fetching libraries (`axios`, `fetch`) default to HTTPS for secure URLs.

## 5. Third-Party Library Security (NPM Packages)

*   **Risk:** Vulnerabilities in third-party dependencies can introduce security flaws into the application.
*   **Mitigation:**
    *   **Regular Updates:** Keep dependencies updated to their latest stable and secure versions. Use commands like `npm outdated` or `yarn outdated` to check.
    *   **Vulnerability Scanning:** Regularly use `npm audit` or `yarn audit` to identify known vulnerabilities in the project's dependencies and apply fixes or updates as recommended.
    *   **Source Scrutiny:** Prefer well-maintained and reputable libraries. Be cautious when adding new, less-known dependencies.
    *   **Minimize Dependencies:** Only include libraries that are genuinely needed to reduce the potential attack surface.

## 6. Input Validation

*   **Risk:** While CodePath AI is primarily for content consumption, if any user input is used to construct prompts for the Gemini API (e.g., a search feature that feeds into a prompt), there's a minor risk of prompt injection if not handled carefully.
*   **Mitigation:** Sanitize or validate any user input that is used in API requests or rendered in the UI. For prompts, ensure user input is clearly delineated from the main prompt structure.

## 7. Error Handling

*   **Avoid Leaking Sensitive Information:** Error messages displayed to the user or logged in client-side consoles (especially in production builds) should not reveal sensitive information (e.g., parts of API keys, detailed system paths, or verbose stack traces that could aid an attacker).
*   Provide generic, user-friendly error messages in the UI.
*   Log detailed, technical error information securely (e.g., only in development builds or to a secure logging service if one were used).

## 8. Development and Build Practices

*   **`.gitignore`:** Ensure that sensitive files (like `.env` files containing API keys), local build artifacts, and editor/OS-specific files are correctly listed in `.gitignore`.
*   **Principle of Least Privilege:** If creating service accounts or configuring cloud resources, grant only the minimum necessary permissions required for the application to function.

## 9. Platform-Specific Security

*   **Mobile App Permissions (Expo):** The application should request only the minimum necessary permissions. For CodePath AI, this primarily involves network access. `expo-sqlite` and `AsyncStorage` generally do not require explicit user-facing storage permissions beyond standard app installation.

## 10. No Authentication/Authorization (Current Scope)

*   The application is designed for personal use and explicitly excludes user login, authentication, or authorization features. This significantly simplifies the security model as there are no user accounts, sessions, or user-specific access controls to manage and protect from common web vulnerabilities like CSRF, session hijacking, etc.
*   If the scope were to change to include multiple users or shared data, implementing robust authentication (e.g., OAuth 2.0, OpenID Connect) and authorization mechanisms would become a top priority.

By adhering to these considerations, CodePath AI can maintain a security posture appropriate for its intended use, with the primary focus on safeguarding the Google Gemini API key.