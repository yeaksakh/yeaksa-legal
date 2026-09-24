# Yeaksa – Legal pages

Public privacy policies and account-deletion instructions for every app published by Yeaksa, served with GitHub Pages at **https://yeaksakh.github.io/yeaksa-legal/**.

One folder per app:

| App | Package | Privacy policy URL |
|---|---|---|
| Yeaksa Shop | `com.yeaksashop.yeaksa` | https://yeaksakh.github.io/yeaksa-legal/yeaksa-shop/privacy-policy.html |
| YeaksaBoy (ដឹកជញ្ជូនយក្សា) – delivery rider app | `com.yeaksa.delivery_boy` | https://yeaksakh.github.io/yeaksa-legal/yeaksa-boy/privacy-policy.html |
| Yeaksale – staff app for businesses on Yeaksa | `com.yeaksale.yeaksa` | https://yeaksakh.github.io/yeaksa-legal/yeaksale/privacy-policy.html |
| Yeaksaundry – staff app for laundry shops on Yeaksa | `com.yeaksaundry.yeaksa` | https://yeaksakh.github.io/yeaksa-legal/yeaksaundry/privacy-policy.html |
| Yeaksarehouse (ឃ្លាំងយក្សា) – warehouse staff app for shops on Yeaksa | `com.yeaksa.warehouse` | https://yeaksakh.github.io/yeaksa-legal/yeaksarehouse/privacy-policy.html |
| Yeaksacturing (ផលិតកម្មយក្សា) – factory-floor app for businesses on Yeaksa | `com.yeaksa.manufacturing` | https://yeaksakh.github.io/yeaksa-legal/yeaksacturing/privacy-policy.html |
| Yeaksa Point (ពិន្ទុយក្សា) – Yeaksa loyalty points | `com.yeaksa.yeaksapoint` | https://yeaksakh.github.io/yeaksa-legal/yeaksa-point/privacy-policy.html |
| YeaksaReport (របាយការណ៍យក្សា) – reports app for businesses on Yeaksa | `com.yeaksa.yeaksareport` | https://yeaksakh.github.io/yeaksa-legal/yeaksa-report/privacy-policy.html |
| SurMekhea (សួរមេឃា) – assistant app for businesses on Yeaksa | `com.yeaksa.surmekhea` | https://yeaksakh.github.io/yeaksa-legal/surmekhea/privacy-policy.html |
| Mekhea (មេឃា) – business app for owners and staff on Mekhea | `com.mekhea.mekhea` | https://yeaksakh.github.io/yeaksa-legal/mekhea/privacy-policy.html |
| Franchiser – staff attendance app for businesses on Mekhea | `com.yeaksa.franchiser` | https://yeaksakh.github.io/yeaksa-legal/franchiser/privacy-policy.html |

Account-deletion URL (for Play Console → Data safety) is the same page with `#delete-account`, e.g. `https://yeaksakh.github.io/yeaksa-legal/yeaksa-shop/privacy-policy.html#delete-account`.

## Adding a new app

1. Copy `_template/` to a new folder named after the app, e.g. `yeaksa-driver/`.
2. In the new `privacy-policy.html`, replace `{{APP_NAME}}`, `{{ANDROID_PACKAGE}}` and `{{EFFECTIVE_DATE}}`, then edit the **Data we collect**, **App permissions** and **Sharing** sections so they match what that app really does (the template describes a shopping app).
3. Add a card for the app in `index.html`.
4. Commit and push to `main` — Pages redeploys in about a minute.
5. Paste `https://yeaksakh.github.io/yeaksa-legal/<folder>/privacy-policy.html` into that app's Play Console / App Store Connect.

`privacy-policy.html` at the root only redirects to the Yeaksa Shop policy (kept so the original URL keeps working).
