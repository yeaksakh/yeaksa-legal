# Playbook: take a Yeaksa Flutter app from "draft" to a closed-testing release on Google Play

Written after doing this for **Yeaksa Shop** (`com.yeaksashop.yeaksa`) on 2026-09-16. Give this file to an AI agent (Claude Code with the Claude-in-Chrome extension) together with the short prompt at the bottom, and it can repeat the whole process for another app.

Google's rule for personal developer accounts: **you cannot publish to production until a closed test has run for 14 days with at least 12 opted-in testers.** So the goal of this playbook is: finish every dashboard item → upload a signed `.aab` to a closed-testing track → submit for review → recruit testers → after 14 days, *Apply for production*.

---

## 0. Prerequisites (human does these once)

| Need | How |
|---|---|
| Play Console developer account, app already *created* (package name reserved) | play.google.com/console |
| Chrome profile logged into that Google account, **Claude in Chrome extension installed and signed in *in that profile*** | Each Chrome profile is a separate extension instance; the agent can only drive the profile where the extension is connected. |
| `gh` CLI logged into the GitHub account that owns `yeaksakh/yeaksa-legal` | `gh auth status` |
| A phone with the app installed, USB/wireless `adb`, **unlocked** while screenshots are taken | `adb devices` |
| A test account in the app for Google's reviewers (phone + password, no OTP at login) | The human types the password into Play Console themselves; the agent never handles passwords. |
| The **upload keystore** for this app (`android/app/upload-keystore.jks` + `android/key.properties`) | If Play already has an upload certificate registered for the package, the *same* key must be used. Check *Test and release → App integrity → Upload key certificate*. |

---

## 1. Gather the facts (agent, ~5 min, terminal)

Every declaration below must match what the app really does. Collect:

```bash
grep -n "uses-permission" android/app/src/main/AndroidManifest.xml
grep -n "applicationId\|targetSdk" android/app/build.gradle*
grep -n "^version" pubspec.yaml                      # versionName+versionCode
sed -n '/^dependencies:/,/^dev_dependencies:/p' pubspec.yaml   # look for ads/analytics/crashlytics/firebase/maps/payments
grep -rln "ImagePicker\|FilePicker\|Geolocator\|firebase_messaging\|delete.*account" lib
curl -s https://<backend>/api/v2/business-settings   # which social logins are enabled
```

Write down: package id, target SDK, login method (phone+password / email / OTP / social), which permissions are used and why, whether there are ads or analytics SDKs, payment providers, chat, user-generated content (reviews), file uploads, push notifications, in-app account deletion.

---

## 2. Privacy policy page (agent, terminal)

Repo: `https://github.com/yeaksakh/yeaksa-legal` → served by GitHub Pages at `https://yeaksakh.github.io/yeaksa-legal/`.

```bash
gh repo clone yeaksakh/yeaksa-legal && cd yeaksa-legal
cp -r _template <app-folder>              # e.g. yeaksa-driver
# edit <app-folder>/privacy-policy.html: replace {{APP_NAME}}, {{ANDROID_PACKAGE}}, {{EFFECTIVE_DATE}}
# then fix the "Data we collect", "App permissions" and "Sharing" sections to match step 1
# add a card for the app in index.html, and a row in README.md
git add -A && git commit -m "<App>: privacy policy" && git push
curl -sI https://yeaksakh.github.io/yeaksa-legal/<app-folder>/privacy-policy.html | head -1   # wait for HTTP 200 (~1 min)
```

URLs you will paste into Play Console:
- Privacy policy: `https://yeaksakh.github.io/yeaksa-legal/<app-folder>/privacy-policy.html`
- Account deletion: same URL + `#delete-account`
- Data deletion (optional): same URL + `#your-rights`

Keep the policy honest: contact email `myyeaksa@gmail.com`, address *Khan Russey Keo, Phnom Penh, Cambodia*, and a **Delete account** section (Play requires it for any app with accounts).

---

## 3. Play Console dashboard – "Finish setting up your app" (agent, browser)

Dashboard URL: `https://play.google.com/console/u/0/developers/<DEV_ID>/app/<APP_ID>/app-dashboard`. Do the items in this order (some are gated on earlier ones). After each *Save*, a dialog "Go to Publishing overview?" appears – click **Not now** and continue.

| # | Item | Answer used for Yeaksa Shop (adjust to step 1) |
|---|---|---|
| 1 | **Privacy policy** | paste the URL, Save |
| 2 | **Ads** | *No, my app does not contain ads* (no ad SDK in pubspec) |
| 3 | **Government apps** | No |
| 4 | **Health** | *My app does not have any health features* → Next → Save |
| 5 | **Financial features** | *My app doesn't provide any financial features* → Next → Save (a store wallet / club points is not a regulated financial product) |
| 6 | **Sign in details** (App access) | *Yes, restricted* → Add details: name, username = test phone, **password typed by the human**, instructions (see box below), tick *full access* → Add → Save |
| 7 | **Target audience** | *18 and over* only → Next ×4 → Save |
| 8 | **Content rating** | email `myyeaksa@gmail.com`, category *All Other App Types*, tick IARC terms (**ask the human first – it is an agreement**), questionnaire: everything **No** except: *users interact / exchange content* = **Yes** if the app has chat or reviews (then No to all sub-questions: not primary content, no nudity, no block/report/moderation unless it exists), *online content* = **Yes** (products from server) with No to violent/sexual/language/drugs, *purchase digital goods* = **Yes** (No loot boxes). Save on step 2, then Next → Save. Expected result: ESRB *Everyone* + "Users Interact", "In-App Purchases". |
| 9 | **Data safety** | see §3.1 |
| 10 | **App category & contact details** (Store settings) | App, category **Shopping** (or the right one), contact email `myyeaksa@gmail.com`, website `https://yeaksa.com` → *Save and publish* |
| 11 | **Store listing** | see §4 |
| + | **Advertising ID** declaration | It is *not* on the dashboard – it appears as "1 issue" on the Publishing overview. Answer **No** (manifest has `AD_ID` removed, no ad SDK). |

Reviewer instructions text (≤500 chars) used in item 6:

> Login: open the app, tap Profile/Login, enter the phone number above in the phone box (no country code needed) and the password, then tap Login. No OTP or 2-step code is needed for this account. Browsing products works without login; login is needed for cart, checkout, orders, wallet and chat. For checkout choose Cash on Delivery so no real payment is made. Account deletion is under Profile > Settings > Delete account.

### 3.1 Data safety answers (Yeaksa Shop)

Step 2: collects data = Yes; encrypted in transit = Yes; account creation = *Username, password, and other authentication*; delete-account URL = `…#delete-account`; partial deletion = Yes with URL `…#your-rights`.

Step 3/4 – data types. For each: **Collected**, not ephemeral, purposes as listed; **Shared** = transferred to sellers / delivery partners.

| Data type | Shared? | Required? | Purposes |
|---|---|---|---|
| Name | yes | required | App functionality, Developer communications, Account management |
| Email address | no | optional | Developer communications, Account management |
| User IDs | no | required | App functionality, Fraud prevention, Account management |
| Address | yes | optional | App functionality |
| Phone number | yes | required | App functionality, Developer communications, Fraud prevention, Account management |
| Purchase history | yes | optional | App functionality, Account management |
| Approximate + Precise location | yes | optional | App functionality |
| Other in-app messages (chat) | yes | optional | App functionality |
| Photos | yes | optional | App functionality |
| Files and docs | no | optional | App functionality |
| In-app search history | no | optional | App functionality, Personalization |
| Other user-generated content (reviews) | yes | optional | App functionality |
| Device or other IDs (push token) | no | optional | App functionality, Developer communications |

Not collected: contacts, calendar, health, audio, web browsing, crash logs (no Crashlytics), advertising ID.

---

## 4. Store listing assets (agent, terminal + browser)

Requirements: icon **512×512 PNG**, feature graphic **1024×500**, phone screenshots **2–8, 9:16 or 16:9**, 7-inch and 10-inch tablet screenshots (same files are accepted).

Text: app name ≤30, short description ≤80, full description ≤4000 (see the Yeaksa Shop listing for a template: what you can buy / why shop / easy to use / contact / privacy URL).

### 4.1 Generate graphics (Python + Pillow, DejaVu fonts)

```python
from PIL import Image, ImageDraw, ImageFont, ImageFilter
src = Image.open('assets/app_icon_1024.png').convert('RGBA')
src.resize((512,512), Image.LANCZOS).save('out/icon_512.png')

W,H = 1024,500
bg = Image.new('RGB',(W,H),(4,6,40)); px = bg.load()
for y in range(H):
    for x in range(W):
        dx,dy = (x-300)/700,(y-250)/300; v = max(0,1-(dx*dx+dy*dy)**0.5)
        px[x,y] = (int(4+30*v), int(6+40*v), int(40+140*v))
bg = bg.convert('RGBA'); logo = src.resize((380,380))
m = Image.new('L',(380,380),0); ImageDraw.Draw(m).rounded_rectangle((0,0,379,379),radius=64,fill=255)
bg.paste(logo,(70,60),m); d = ImageDraw.Draw(bg)
B='/usr/share/fonts/truetype/dejavu/DejaVuSans-Bold.ttf'; R='/usr/share/fonts/truetype/dejavu/DejaVuSans.ttf'
d.text((500,110),'App Name',font=ImageFont.truetype(B,66),fill=(255,230,0))
d.text((502,195),'One-line tagline',font=ImageFont.truetype(R,34),fill=(255,255,255))
for i,l in enumerate(['•  Benefit one','•  Benefit two','•  Benefit three']):
    d.text((502,250+i*36),l,font=ImageFont.truetype(R,24),fill=(205,215,255))
d.text((502,385),'yeaksa.com',font=ImageFont.truetype(B,28),fill=(255,230,0))
bg.convert('RGB').save('out/feature_graphic_1024x500.png')
```

### 4.2 Screenshots from the phone

```bash
D="adb -s <serial>"
$D shell svc power stayon true            # phone must be UNLOCKED, else screencap is black
$D shell monkey -p <package> -c android.intent.category.LAUNCHER 1; sleep 10
$D exec-out screencap -p > out/shot_home.png
$D shell input tap X Y; sleep 4; $D exec-out screencap -p > out/shot_2.png   # bottom-nav tabs etc.
$D shell svc power stayon false
```
Skip any screen that shows a real user's name/phone (Profile) or private documents in product photos. Then frame each shot into 1080×1920 with a caption:

```python
from PIL import Image, ImageDraw, ImageFont, ImageFilter
W,H=1080,1920; B='/usr/share/fonts/truetype/dejavu/DejaVuSans-Bold.ttf'; R='/usr/share/fonts/truetype/dejavu/DejaVuSans.ttf'
for i,(fn,title,sub) in enumerate([('shot_home.png','Headline','Sub line'), ...],1):
    bg=Image.new('RGB',(W,H),(6,10,60)); px=bg.load()
    for y in range(H):
        t=y/H; row=(int(6+20*(1-t)),int(10+30*(1-t)),int(60+110*(1-t)))
        for x in range(W): px[x,y]=row
    d=ImageDraw.Draw(bg); f1=ImageFont.truetype(B,60); f2=ImageFont.truetype(R,34)
    d.text(((W-d.textlength(title,font=f1))/2,90),title,font=f1,fill=(255,230,0))
    d.text(((W-d.textlength(sub,font=f2))/2,175),sub,font=f2,fill=(220,228,255))
    sh=Image.open(fn).convert('RGB'); sh=sh.crop((0,90,sh.width,sh.height))      # drop status bar
    th=1600; tw=int(sh.width*th/sh.height); sh=sh.resize((tw,th),Image.LANCZOS)
    m=Image.new('L',sh.size,0); ImageDraw.Draw(m).rounded_rectangle((0,0,tw-1,th-1),radius=44,fill=255)
    bg.paste(sh,((W-tw)//2,260),m); bg.save(f'out/screenshot_{i}.png')
```

### 4.3 Upload in the browser (Play Console quirks)

- The "Add assets" button opens a native file picker the agent cannot see. Work-around that works: click *Add assets* with JS (`button.click()`), the page then creates a hidden `<input type=file>`; give it an `aria-label`, locate it with `find`, and use the extension's **file_upload** tool on that ref. The file lands in the *asset library* side panel.
- In the side panel, select the uploaded assets (hover a row → click the circle at its left edge) and press **Add** (bottom-right of the panel) to attach them to the slot. Uploaded files stay in the library, so tablet slots can reuse the phone screenshots.
- Text fields: typing with the mouse sometimes misses the field. Setting the value via the native setter + `input` event is reliable:
  `Object.getOwnPropertyDescriptor(HTMLTextAreaElement.prototype,'value').set.call(el,text); el.dispatchEvent(new Event('input',{bubbles:true}))`
- Click **Save** (bottom right), confirm the "Change saved" toast.

---

## 5. Signed App Bundle (agent, terminal)

```bash
# only if the app has NO upload key registered in Play yet (check App integrity page first!)
keytool -genkeypair -v -keystore android/app/upload-keystore.jks -storetype JKS -keyalg RSA -keysize 2048 -validity 10000 \
  -alias upload -storepass "$PW" -keypass "$PW" -dname "CN=<App>, OU=Mobile, O=Yeaksa, L=Phnom Penh, ST=Phnom Penh, C=KH"
printf 'storePassword=%s\nkeyPassword=%s\nkeyAlias=upload\nstoreFile=upload-keystore.jks\n' "$PW" "$PW" > android/key.properties
git check-ignore android/key.properties android/app/upload-keystore.jks   # both must be ignored
flutter build appbundle --release      # → build/app/outputs/bundle/release/app-release.aab
```
Bump `version:` in `pubspec.yaml` first – Play rejects a versionCode that was already uploaded. Back up the keystore + `key.properties` somewhere safe; if Play already has a different upload certificate, either use that key or request an *upload key reset* on the App integrity page.

---

## 6. Closed testing track (agent, browser)

Test and release → **Closed testing** → *Create track* (or use the existing "Alpha").

1. **Countries / regions** tab: add the countries (Yeaksa used all 177). Save.
2. **Testers** tab: *Create email list*, name it, paste ≥12 Gmail addresses (or add a Google Group). Optionally a feedback email. Save.
3. **Releases** tab → *Create new release* → upload the `.aab` (same hidden-input trick as §4.3) → release name (defaults to "13 (5.8.3)") → release notes inside `<en-US> … </en-US>` → **Next**. If an old duplicate upload shows "Version code already used", remove it with its *clear* (✕) button.
4. **Preview and confirm** → must say *Ready to release* → **Save**.
5. **Publishing overview** → wait for "quick checks" (≈15 min) → fix anything under *View N issues* (this is where the **Advertising ID** declaration appears) → **Submit N changes for review** → confirm *Send changes for review*. Status becomes "Changes in review".

---

## 7. After Google approves (human)

1. Share the opt-in link with the testers: `https://play.google.com/apps/testing/<package>` – each must click **Become a tester**, then install from Play.
2. Keep ≥12 opted-in for **14 consecutive days**.
3. Dashboard → **Apply for production access** → answer the questionnaire about the test → wait for approval → create a *Production* release with the same `.aab` (or a newer versionCode) → submit.

---

## 8. Browser gotchas the agent should know (Claude in Chrome + Play Console)

- Check `list_connected_browsers` first; the Play Console profile must be the *selected* browser.
- Play Console is heavy. Screenshot coordinates were sometimes off by a constant factor and the renderer occasionally froze for 30–45 s. Prefer **`find` → click by ref**, JS `button.click()`, or focus + `Enter`, and re-verify state by reading `document.body.innerText`.
- Radios/checkboxes in the Data safety dialogs are native `<input>`s inside the dialog; clicking them via JS works, but the dialog's **Save** needs a real click (ref or coordinates at the bottom-right of the dialog).
- The category dropdown only opens with a real click or *focus + Enter*; then click the `[role=option]` with JS.
- "Save and publish" on Store settings → contact details publishes immediately (harmless for a draft app).
- Every save shows "Go to Publishing overview?" – *Not now*.
- Never type the reviewer account password; let the human do it.

---

## 9. Prompt to give an agent for the next app

> Read `https://raw.githubusercontent.com/yeaksakh/yeaksa-legal/main/PLAYBOOK-closed-testing.md` and follow it for the app **<APP NAME>** (package `<PACKAGE>`, Flutter project at `<PATH>`). The Play Console app already exists under developer `<DEV_ID>`, app `<APP_ID>`. Use my Chrome profile that has the Claude extension connected. Create the privacy policy in `yeaksakh/yeaksa-legal/<folder>/`, complete every dashboard item, build the store listing (take screenshots from my phone `<adb serial>` – I will unlock it), create/finish the closed-testing release and submit it for review. I will type the reviewer test-account password myself when you reach *Sign in details*, and I will confirm before you accept the IARC terms. Tell me at the end what I still have to do (testers, upload key, production access).
