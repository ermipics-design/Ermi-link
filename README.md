# ERMI LINK — Google Play build package

This package contains the current ERMI LINK PWA wrapped as an Android WebView app and a GitHub Actions workflow that builds a release AAB against Android 16 / API 36.

## Important production note
The current contact-unlock payment flow is a prototype: payment references are stored in browser/local storage and the demo admin can approve them locally. Before public launch, move accounts, payments, contact unlocking, chat, and admin approval to a secure server/database and integrate an authorized BenefitPay merchant/payment API or verified merchant workflow.

## Build
1. Create a GitHub repository and upload this folder.
2. Open Actions → Build ERMI LINK Android AAB → Run workflow.
3. Download the generated `ERMI-LINK-release-AAB` artifact.
4. In Play Console create the app and upload the AAB to testing.

Package name: `com.ermilink.app`
Version: 1.0.0 / versionCode 1
Target SDK: 36

## Play launch checklist
- Full-distribution developer account and identity verification.
- Store listing, screenshots, icon and feature graphic.
- Privacy policy URL.
- Data Safety form and content rating.
- App access/test credentials if login is required.
- Closed test if Google requires it for the developer account.
- Play App Signing.

Never publish real payment credentials or secret API keys inside the Android app. Public merchant identifiers may be shown only when appropriate; secret keys must remain server-side.
