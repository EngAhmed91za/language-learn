# Context: AI-Powered Language Learning App

## App Idea

A cross-platform (mobile and web) React application designed to help users learn any programming language through tutorials and step-by-step instructions. The app will leverage the Google Gemini API for content delivery and will be able to retrieve information from the web to keep its content updated dynamically. A key feature is offline data access. The app is intended for personal use, so no login/authentication is required.

## Purpose

To provide a flexible, accessible, and up-to-date platform for individuals to learn programming languages at their own pace, with AI-powered assistance and offline capabilities.

## Target Audience

The primary target audience includes:

*   **Beginners:** Individuals new to programming looking for a structured and supportive learning environment.
*   **Intermediate Learners:** Developers looking to pick up new languages or refresh their knowledge on existing ones.
*   **Self-Learners:** Anyone who prefers a self-paced, tutorial-based learning approach.

## Key Features

*   **Multi-Language Support:** Ability to learn a wide range of programming languages.
*   **Tutorial-Based Learning:** Step-by-step tutorials and instructions.
*   **Google Gemini API Integration:** AI-powered content delivery and potentially interactive learning experiences.
*   **Dynamic Web Updates:** Automatic retrieval and integration of new content/tutorials from the web.
*   **Cross-Platform:** Accessible via a unified link on both mobile (React Native with Expo) and web (React).
*   **Offline Access:** All data, including tutorials and user progress, can be stored and accessed offline.
*   **No Login Required:** Unrestricted access for personal use.
*   **Interactive Coding Practice (Potential):** Inline coding exercises to reinforce learning.
*   **Progress Tracking (Potential):** Mechanisms to track user progress through tutorials and languages.

## Tech Stack Considerations

*   **Frontend (Mobile):** React Native with TypeScript, Expo, Expo Router.
*   **Frontend (Web):** React with TypeScript.
*   **UI Framework (Mobile):** React Native Paper (suggestion, can be adapted).
*   **AI Processing:** Google Gemini API (API Key: `AIzaSyAOlgedurL4Uobpyagku_bqHWF7IFKoNAQ`)
    *   **Security Note:** This API key must be handled securely. It is strongly recommended to use environment variables and a backend proxy for API calls in a production environment, rather than hardcoding the key in client-side application code. For personal use, ensure the key is not publicly exposed in repositories.
*   **Data Storage (for offline):** A local database solution such as SQLite (e.g., via `expo-sqlite` for React Native) or IndexedDB for web applications. For data synchronization across platforms, if ever needed, a backend solution like Supabase (mentioned in the user's template) with its offline capabilities could be explored.