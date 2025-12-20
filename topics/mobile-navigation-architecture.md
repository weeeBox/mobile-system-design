# Mobile Navigation Architecture Deep Dive

Navigation is one of the most critical and complex aspects of mobile system design. As apps grow, tight coupling between screens ("Feature A imports Feature B") creates a monolithic spaghetti code structure that is hard to test, refactor, and modularize.

**The Challenge:** How do you design a navigation system that decouples feature modules, supports deep linking, handles complex flows (e.g., authentication), and maintains state across configuration changes?

## 1. Core Requirements

*   **Decoupling:** Module A should navigate to Module B without knowing Module B exists.
*   **Deep Linking:** The app must be able to open any specific screen from a URL (e.g., `app://profile/123`).
*   **Result Handling:** Screen B needs to return data to Screen A (e.g., selecting a contact).
*   **Back Stack Management:** Handling the hardware back button (Android) or swipe-to-back (iOS) correctly.
*   **Context/State Preservation:** Restoring the navigation stack after a process death.

## 2. Evolution of Navigation Patterns

### 2.1 Direct Intent/Push (The "Junior" Approach)
*   **Concept:** `startActivity(new Intent(context, DetailActivity.class))` or `navigationController.push(DetailViewController())`.
*   **Problem:** Tight coupling. The "Feed" module must compile the "Detail" module. You cannot split them into separate Gradle/Bazel modules easily. Circular dependencies arise quickly (Profile -> Feed -> Profile).

### 2.2 The Coordinator Pattern (iOS / Android)
A dedicated object handles the flow logic, removing it from the UI (Activity/ViewController).
*   **How it works:**
    1.  `FeedViewController` calls `coordinator.showDetail(id)`.
    2.  `FeedCoordinator` knows how to assemble the `DetailViewController`.
    3.  `FeedCoordinator` pushes it onto the stack.
*   **Pros:** Isolates navigation logic. Great for testing.
*   **Cons:** Coordinators can become "God Objects". Still requires some knowledge of destination classes unless combined with Dependency Injection.

### 2.3 The Router / URI-Based Navigation (The "Scalable" Approach)
Every screen is a URL.
*   **How it works:**
    1.  `FeedModule` asks: `Router.navigate("app://detail?id=123")`.
    2.  `Router` looks up the registration map.
    3.  `Router` instantiates the target screen and pushes it.
*   **Pros:** Zero compile-time coupling. Modules only know the Router interface. Identical logic for internal navigation and external Deep Links.
*   **Cons:** Loss of type safety (passing strings instead of objects).

## 3. Deep Dive: Modern Navigation Architecture

### 3.1 The Navigation Graph (Jetpack Navigation)
*   **Concept:** A centralized XML or Kotlin DSL file defines all possible destinations and paths ("Actions").
*   **Safe Args:** Uses compile-time code generation to enforce type safety for arguments, fixing the main downside of URIs.
*   **Nested Graphs:** Allows splitting a massive graph into sub-graphs (e.g., `LoginGraph`, `CheckoutGraph`) for modularization.

### 3.2 Feature-Based Navigation (Multi-Module)
In a modularized app (e.g., `:features:feed` and `:features:settings`), how do they talk?

**Option A: Interface Injection**
1.  Define an interface in a common module: `interface FeedNavigator { fun goToSettings() }`.
2.  Implement it in the main `app` module (which sees everything).
3.  Inject implementation into `FeedModule`.

**Option B: Implicit Intents / Deep Links**
1.  `FeedModule` fires an implicit Intent or Deep Link.
2.  The OS or a central Router resolves it.

### 3.3 Handling Global Flows (Authentication)
What if the user's token expires while they are 5 screens deep?
*   **The Interceptor:** The Navigation system should have an interceptor chain.
    *   `navigate(target)` -> `AuthInterceptor` checks token.
    *   If valid -> Proceed.
    *   If invalid -> Redirect to `LoginScreen` (and save `target` to redirect back after success).

## 4. Interview Checklist

1.  **Decoupling Strategy:** "I will use a Router pattern (or Coordinators) to ensure Feature A doesn't depend on Feature B."
2.  **Deep Linking:** "I will treat internal navigation and external deep links uniformly using a centralized URL resolver."
3.  **Type Safety:** "I will use code generation (like SafeArgs or a custom struct generator) to ensure parameter type safety between decoupled modules."
4.  **State Restoration:** "I will ensure the Router can serialize the back stack to handle process death."

## 5. Further Reading
*   **Square:** [The blind leading the blind (Coordinators)](https://khanlou.com/2015/01/the-coordinator/)
*   **Uber:** [RIBs Architecture (Router-Interactor-Builder)](https://github.com/uber/RIBs)
*   **Android:** [Navigation Component Guide](https://developer.android.com/guide/navigation)
*   **Airbnb:** [Deep Link Dispatch](https://github.com/airbnb/DeepLinkDispatch)
