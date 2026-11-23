# ONE1 Storm Roster: setup

A free website for picking ONE1's Team A and Team B for Desert Storm and Canyon Storm in Last War: Survival. R4s open one private link, with no login. Data is shared live through Firebase, and Gemini reads the screenshots.

You do this setup once, which takes about 15 minutes. After that, R4s only need the link.

---

## 1. Create the Firebase project (free)
1. Go to **console.firebase.google.com** and sign in with a Google account.
2. **Create a project**. Use any name, for example `storm-roster`. You can turn Google Analytics off.
3. On the project's home page, click the **Web** icon (`</>`) to add a web app. Give it any nickname. Leave "Firebase Hosting" unticked. Click **Register app**.
4. Firebase shows a code block that starts with `const firebaseConfig = {`. **Copy the whole block** and keep it for step 6.

## 2. Turn on anonymous sign-in
**Build → Authentication → Get started → Sign-in method → Anonymous → Enable → Save.**

This lets R4s connect without accounts. The private link is what keeps strangers out.

## 3. Create the database
**Build → Firestore Database → Create database.** If it asks for an edition, choose **Standard**. Pick a location close to you, choose **Start in production mode**, then click **Create**.

## 4. Paste the security rules
In Firestore, open the **Rules** tab, replace everything with the rules below, and click **Publish**:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    function signedIn() { return request.auth != null; }
    function isR4() { return signedIn() && exists(/databases/$(database)/documents/admins/$(request.auth.uid)); }
    function passwordSet() { return exists(/databases/$(database)/documents/auth/cfg); }

    // The R4 data. Open to anyone with the private code until an R4 password is set, then R4 devices only.
    match /alliances/{code}/{group}/{docId} {
      allow read, write: if signedIn() && code.size() >= 20 && group in ['data', 'log']
        && (isR4() || !passwordSet());
    }
    // The R4 password (stored only as a hash, nobody can read it back)
    match /auth/cfg {
      allow create: if signedIn() && !passwordSet();
      allow update: if isR4();
    }
    // A device becomes an R4 device by sending the matching hash
    match /admins/{uid} {
      allow read: if signedIn() && request.auth.uid == uid;
      allow create, update: if signedIn() && request.auth.uid == uid
        && request.resource.data.h == get(/databases/$(database)/documents/auth/cfg).data.h;
    }
    // The member view: anyone with the member link can read, only R4 devices can write
    match /views/{vc} {
      allow read: if signedIn() && vc.size() >= 12;
      allow write: if isR4();
    }
  }
}
```

The R4 data needs the private code (inside the R4 link) and, once you set an R4 password, a device that entered the password. Members only get the member link, which can read the published lineups and nothing else. Nobody can list or browse other data. **Set the R4 password right after you publish these rules** (see "R4 password and member link" below).

## 5. Put the site on GitHub Pages (free)
1. Go to **github.com**, sign in or create an account, and click **New repository**. Name it `storm-roster`, set it to **Public**, and click **Create repository**.
2. Click **uploading an existing file**, drag in `index.html` (this README is optional), then click **Commit changes**.
3. Go to **Settings → Pages**. Under "Build and deployment", set Source to **Deploy from a branch**, set Branch to **main** and **/ (root)**, then click **Save**.
4. Wait about a minute and refresh. Your site address appears at the top, for example `https://YOUR-NAME.github.io/storm-roster/`.
5. Back in Firebase, go to **Authentication → Settings → Authorized domains → Add domain** and add `YOUR-NAME.github.io`.

## 6. Create your alliance space
Open your site address. Under **Leader: first-time setup**, type your name in game, paste the Firebase config from step 1, and click **Create alliance space**.

## 7. Turn on screenshot reading (free Gemini key)
1. Go to **aistudio.google.com/apikey**, sign in, and click **Create API key**. Copy the key.
2. On the site, go to **06 Alliance settings**, paste the key, click **Save key**, then click **Test key**.

One key works for every R4. Anyone with the R4 link can use it, so use a free key with billing turned off. On the free tier, Google may use what you send to improve its products. Here, that's game screenshots.

## 8. Share with your R4s
Under **06 Alliance settings**, click **Copy** next to **R4 link** and send it to your R4s in a private chat. They open it, type their name under "Your name", and they're in. The site remembers them on that phone.

If the link ever leaks, do this:
1. Click **Download backup**.
2. Click **Disconnect this device**.
3. Create a new space.
4. Click **Restore backup**.
5. Send the new link to your R4s.

---

## Weekly routine
1. **Monday:** upload **Member list** screenshots to update 1st team power.
2. Upload the sign-ups, **Sign-ups · Team A** and **Sign-ups · Team B** separately. A player registers for one team only: if someone is in both lists, the newest upload wins. Only registered players are picked for a team, and a team with no list uploaded stays empty (so you can run Team A alone).
   Then check the roster and mark anyone away. Click **Copy for alliance mail**, register the teams in game, then click **Save roster**.
3. **Friday, after the battle:** upload **Battle results** screenshots and choose "The battle on Fri …". This marks no-shows and starts next week's roster.

## How the rotation works
1. Members who have waited the most events without a main slot go first.
2. When the wait is equal, last time's subs go first, then higher 1st team power.
3. The first 40 in line play main, the next 20 are subs, and the rest sit out. Power then decides Team A or Team B.

## Free limits
- **Firebase (Spark plan):** 50,000 reads and 20,000 writes a day, far more than one alliance uses.
- **Gemini free tier:** a limited number of requests per minute. If you hit it, wait a minute, or upload fewer screenshots at a time.


## R4 password and member link
In the site, open **Alliance settings** (step 06):
1. **R4 password:** press **Generate** (or type your own, 8+ characters), press **Set password**, and write it down. New devices that open the R4 link must enter it once. Devices that are already open stay open. If you forget it, set a new one from any open R4 device.
2. **Member link:** press **Turn on member link**, then **Copy**. Send that link to your members. They see everything but cannot change anything: the saved Team A and Team B lineups, the **History** of past rosters, and the **Members** list with power. They can search their own name and switch between English and Arabic. Battle scores stay R4-only. The page updates each time an R4 presses **Save roster** (or **Publish now**). **New link** stops the old one.
