# Symmetry News App — Project Report

## 1. Introduction

This project implements a **News App with Article Upload functionality** for the Symmetry technical assessment. The app allows journalists to browse daily news from NewsAPI and publish their own articles with thumbnails to Firebase, following **Clean Architecture** principles with **BLoC** state management.

The app is built entirely in **Dart/Flutter**, targeting all platforms (Android, iOS, Web, macOS, Desktop), with a Firebase backend (Firestore + Cloud Storage).

## 2. Learning Journey

### Technologies Learned & Applied
- **Flutter & Dart** — Cross-platform UI framework, widget tree, state management
- **BLoC Pattern** (`flutter_bloc ^9.1.1`) — Event-driven state management separating business logic from UI
- **Clean Architecture** — Three-layer separation: Domain (pure Dart) → Data (Firebase, APIs) → Presentation (UI, BLoC)
- **Firebase Firestore** — NoSQL cloud database for storing published articles
- **Firebase Cloud Storage** — Media storage for article thumbnails (`media/articles/{id}/`)
- **Retrofit + Dio** — HTTP client for consuming NewsAPI REST endpoints
- **Floor (sqflite)** — Local SQLite database for saving articles offline
- **GetIt** — Service locator pattern for dependency injection
- **image_picker** — Cross-platform image selection with `Uint8List` bytes for web compatibility

### Resources Used
- Flutter official documentation
- Firebase for Flutter documentation
- BLoC library documentation and examples
- Clean Architecture principles by Robert C. Martin

## 3. Challenges Faced

### 3.1 Cross-Platform Compatibility (Web)
**Problem:** `dart:io File` is not available on Flutter Web, breaking the image upload flow.
**Solution:** Refactored the entire thumbnail pipeline to use `Uint8List` bytes instead of `File` objects. Changed Firebase Storage from `putFile` to `putData` which works on all platforms.

### 3.2 Firebase Storage CORS
**Problem:** Images uploaded to Firebase Storage couldn't load in the browser due to CORS restrictions.
**Solution:** Configured CORS policy on the Firebase Storage bucket using `gsutil cors set` to allow GET/HEAD requests from all origins.

### 3.3 NewsAPI CORS on Web
**Problem:** NewsAPI blocks direct browser requests (CORS policy).
**Solution:** Implemented a Dio interceptor that routes requests through a CORS proxy only when running on web (`kIsWeb`), keeping native platforms unaffected.

### 3.4 Local Database on Web
**Problem:** sqflite/Floor is not supported on Flutter Web, causing app crash on startup.
**Solution:** Created `_NoOpAppDatabase` and `_NoOpArticleDao` fallback classes that return empty results on web, allowing the app to degrade gracefully while still functioning.

### 3.5 Xcode 26 Code Signing (macOS/iOS)
**Problem:** Xcode 26 beta adds an irremovable `com.apple.provenance` extended attribute to all files, breaking code signing.
**Solution:** This is an Apple beta bug with no workaround. The app runs on web and Android while awaiting a fix from Apple.

## 4. Architecture Overview

```
lib/
├── config/          — Routes, theme
├── core/            — DataState, UseCase interfaces, constants
├── features/
│   ├── daily_news/
│   │   ├── domain/  — ArticleEntity, repository interface, use cases
│   │   ├── data/    — ArticleModel, NewsApiService, Floor DB, repository impl
│   │   └── presentation/ — DailyNews screen, ArticleDetail, SavedArticles, BLoCs
│   └── article_publisher/
│       ├── domain/  — JournalistArticleEntity, repository interface, use cases
│       ├── data/    — JournalistArticleModel, Firestore data source, repository impl
│       └── presentation/ — CreateArticle screen, BLoC, widgets
└── injection_container.dart — GetIt dependency registration
```

### Key Design Decisions
- **DataState<T> wrapper** — Encapsulates success/error responses cleanly across layers
- **One UseCase per operation** — `GetArticleUseCase`, `PublishArticleUseCase`, etc.
- **Domain layer is pure Dart** — No Firebase, Flutter, or third-party imports
- **Models extend Entities** — `ArticleModel extends ArticleEntity` with JSON/DB serialization
- **Combined feed** — `RemoteArticlesBloc` fetches from both NewsAPI and Firestore, merging published articles at the top of the feed

## 5. Functionality Implemented

### ✅ Home Screen (Daily News)
- Displays articles from NewsAPI (top headlines)
- Shows journalist-published articles from Firestore at the top of the feed
- Bookmark icon to access saved articles
- FAB (+) button to create new articles
- Tap article to view full details

### ✅ Create Article Screen
- Title input field
- Image picker (Attach Image button with camera icon)
- Article content textarea
- Publish Article button with validation (all fields required)
- Uploads thumbnail to Firebase Storage, saves metadata to Firestore
- Success/Error feedback via SnackBars

### ✅ Article Detail Screen
- Full article view with title, date, image, and content
- Save to local bookmarks (FAB)

### ✅ Saved Articles Screen
- List of locally bookmarked articles
- Remove articles from bookmarks

### ✅ Firebase Backend
- Firestore security rules with field validation
- Storage security rules (5MB limit, images only)
- DB schema documented in `backend/docs/DB_SCHEMA.md`

## 6. Proof of Project

The app can be tested by:
1. Running `flutter run -d chrome` in the `frontend/` directory
2. The home screen loads articles from NewsAPI + Firestore
3. Tapping the (+) FAB navigates to the Create Article form
4. Publishing an article uploads the image and saves to Firestore
5. The published article appears at the top of the home feed
6. Articles can be bookmarked locally

## 7. Reflection & Future Directions

### What I Learned
- Clean Architecture enforces excellent separation of concerns but requires discipline in layer boundaries
- BLoC provides predictable, testable state management through its event/state pattern
- Cross-platform Flutter development requires careful handling of platform-specific APIs (File vs Uint8List)
- Firebase provides a rapid backend solution with built-in security rules

### Future Improvements
- Add unit and widget tests for BLoCs and use cases
- Implement article editing and deletion
- Add user authentication (Firebase Auth) for author identity
- Implement pagination for the article feed
- Add offline-first support with Firestore local persistence
- Dark mode theme support
