# App Store & Play Store Compliance Review

Review the provided code or feature implementation to ensure it strictly complies with both the **Apple App Store Review Guidelines** and the **Google Play Store Policies**. If any violations are found, point them out explicitly and suggest refactors.

## Core Reference Documentation
- **Apple App Store Review Guidelines**: [developer.apple.com/app-store/review/guidelines/](https://developer.apple.com/app-store/review/guidelines/)
- **Apple Human Interface Guidelines (HIG)**: [developer.apple.com/design/human-interface-guidelines](https://developer.apple.com/design/human-interface-guidelines/)
- **Google Play Developer Policy Center**: [play.google.com/about/developer-content-policy](https://play.google.com/about/developer-content-policy/)
- **Material Design Guidelines**: [m3.material.io](https://m3.material.io/)

---

## 1. Permissions & Privacy
*Ref: Apple Guidelines 5.1.1 (Data Collection and Storage) | Google Play User Data Policy*

- **Just-In-Time Requesting**: Verify that permissions (Camera, Location, Contacts, Photo Library, Microphone, etc.) are NOT requested at app launch. They must only be requested when the user explicitly triggers a feature that requires them.
- **Rationale Strings**: Ensure that `Info.plist` and `AndroidManifest.xml` have clear, user-friendly strings explaining exactly *why* the permission is needed (e.g., `NSCameraUsageDescription`, `NSPhotoLibraryUsageDescription`). 
- **Account Deletion**: If the code scaffolded involves user profiles or account settings, verify that a clear, native option to **Delete Account** is included. *Ref: Apple Guideline 5.1.1(v) | Google Play Data Deletion requirement.*

## 2. UI / UX & Platform Behaviors
*Ref: Apple Guidelines 4.0 (Design) | Google Play Core App Quality*

- **Safe Areas**: Verify that top-level screens or floating elements are wrapped in a `SafeArea` widget to prevent overlapping with the iOS notch, dynamic island, or Android system bars.
- **Hardware Back Navigation**: Check that Android hardware back button behavior is respected (using `WillPopScope` or the newer `PopScope` API) so users do not get trapped.
- **Accessibility (a11y)**: Check that custom controls use `Semantics` widgets appropriately. Verify text scaling is supported without overflowing (e.g., using flexible layouts, wrapping, or `TextOverflow.ellipsis`).
- **Dark Mode Support**: Ensure colors are not hardcoded in ways that break when the system swaps to Dark Mode. Colors should come from `Theme.of(context).colorScheme` or the predefined `AppColors` palette.

## 3. Monetization & Authentication
*Ref: Apple Guidelines 3.1.1 (In-App Purchase) & 4.8 (Sign in with Apple) | Google Play Payments Policy*

- **Sign in with Apple**: If the code adds third-party single sign-on (SSO) like Google or Facebook, ensure that **Sign in with Apple** is also implemented. Apple will reject the app otherwise.
- **Native Payments (IAP)**: If the feature unlocks premium digital content, subscriptions, or features, ensure that native In-App Purchases (IAP) are used rather than third-party gateways (like Stripe or PayPal) which violate both platform's core policies for digital goods.

## 4. Content & Data Handling
*Ref: Apple Guidelines 1.2 (User-Generated Content) | Google Play Spam and Minimum Functionality*

- **User-Generated Content (UGC)**: If the app displays UGC, verify there is functionality to report abusive content and block users.
- **External Links**: If the app links to external websites, ensure these links do not bypass native purchasing or hide prohibited content.
- **Secure Storage**: Ensure sensitive data (passwords, tokens) is never logged in plaintext and is only saved using `SecureStorageHelper` rather than basic `SharedPreferences`.

---

**Instructions to AI:**
1. Read through the candidate code.
2. Cross-reference the implementation against the 4 categories and the provided reference links above.
3. List any potential Compliance Risks.
4. Output the corrected, store-compliant code.
