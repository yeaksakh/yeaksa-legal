# Yeaksa – Legal pages

Public privacy policies and account-deletion instructions for every app published by Yeaksa, served with GitHub Pages at **https://yeaksakh.github.io/yeaksa-legal/**.

One folder per app:

| App | Package | Privacy policy URL |
|---|---|---|
| Yeaksa Shop | `com.yeaksashop.yeaksa` | https://yeaksakh.github.io/yeaksa-legal/yeaksa-shop/privacy-policy.html |
| YeaksaBoy (ដឹកជញ្ជូនយក្សា) – delivery rider app | `com.yeaksa.delivery_boy` | https://yeaksakh.github.io/yeaksa-legal/yeaksa-boy/privacy-policy.html |
| Yeaksale – staff app for businesses on Yeaksa | `com.yeaksale.yeaksa` | https://yeaksakh.github.io/yeaksa-legal/yeaksale/privacy-policy.html |
| Yeaksaundry – staff app for laundry shops on Yeaksa | `com.yeaksaundry.yeaksa` | https://yeaksakh.github.io/yeaksa-legal/yeaksaundry/privacy-policy.html |

Account-deletion URL (for Play Console → Data safety) is the same page with `#delete-account`, e.g. `https://yeaksakh.github.io/yeaksa-legal/yeaksa-shop/privacy-policy.html#delete-account`.

## Adding a new app

1. Copy `_template/` to a new folder named after the app, e.g. `yeaksa-driver/`.
2. In the new `privacy-policy.html`, replace `{{APP_NAME}}`, `{{ANDROID_PACKAGE}}` and `{{EFFECTIVE_DATE}}`, then edit the **Data we collect**, **App permissions** and **Sharing** sections so they match what that app really does (the template describes a shopping app).
3. Add a card for the app in `index.html`.
4. Commit and push to `main` — Pages redeploys in about a minute.
5. Paste `https://yeaksakh.github.io/yeaksa-legal/<folder>/privacy-policy.html` into that app's Play Console / App Store Connect.

`privacy-policy.html` at the root only redirects to the Yeaksa Shop policy (kept so the original URL keeps working).
