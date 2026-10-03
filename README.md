# Spotify Random Saved Episode — iPhone Shortcut

This package gives you a genuinely uniform random draw over the podcast episodes currently saved in **Your Episodes** on Spotify.

The final everyday workflow is:

**Tap “Random Podcast” → Spotify opens the selected saved episode.**

The only persistent credentials in the Shortcut are:
- your Spotify **Client ID** (not secret), and
- a Spotify **refresh token** with `user-library-read` scope.

No Spotify Client Secret is used.

## Part 1 — Host the authorization helper

The included `index.html` is a static page. GitHub Pages is an easy place to host it.

1. On GitHub, create a repository such as `spotify-random-episode-auth`.
2. Put `index.html` and `.nojekyll` from this package in the repository root.
3. In the repository, go to **Settings → Pages**.
4. Under **Build and deployment**, choose **Deploy from a branch**.
5. Choose your default branch (normally `main`) and `/(root)`, then save.
6. Open the resulting Pages site. It will usually look like:

   `https://YOUR-GITHUB-USERNAME.github.io/spotify-random-episode-auth/`

The helper page contains no Client ID or token in its source code. Those are entered at runtime and stored only in that browser's local storage.

## Part 2 — Create the Spotify Developer app

1. Open: https://developer.spotify.com/dashboard
2. Create an app.
3. Copy its **Client ID**. You do not need the Client Secret.
4. In the app settings, add the **exact redirect URI** shown on your hosted helper page.
   - The trailing slash matters.
   - Example:
     `https://YOUR-GITHUB-USERNAME.github.io/spotify-random-episode-auth/`
5. Save the Spotify app settings.

Spotify's Development Mode currently requires the app owner to have Spotify Premium.

## Part 3 — Get the refresh token

1. Visit your hosted helper page on your iPhone.
2. Paste the Spotify Client ID.
3. Tap **Save Client ID & connect Spotify**.
4. Approve the requested permission. The helper asks only for:
   - `user-library-read`
5. Spotify redirects back to the helper.
6. Copy the displayed:
   - Client ID
   - Refresh token

Treat the refresh token like a password.

Spotify currently gives developer-app refresh tokens a six-month lifetime. When it expires, revisit this helper, authorize again, and replace the refresh token in the Shortcut.

## Part 4 — Build the iPhone Shortcut

Create a new shortcut named **Random Podcast**.

Add these actions in order.

### A. Configuration

**1. Text**
Paste your Spotify Client ID.

Rename its magic variable to `ClientID` if you want.

**2. Text**
Paste your Spotify refresh token.

Rename its magic variable to `RefreshToken`.

### B. Get a fresh access token

**3. URL**

`https://accounts.spotify.com/api/token`

**4. Get Contents of URL**

Configure:
- Method: `POST`
- Request Body: `Form`

Add these form fields:
- `grant_type` = `refresh_token`
- `refresh_token` = the output of the RefreshToken Text action
- `client_id` = the output of the ClientID Text action

**5. Get Dictionary Value**
- Key: `access_token`
- Dictionary: output of **Get Contents of URL**

Rename this magic variable `AccessToken`.

### C. Ask Spotify how many saved episodes you have

**6. URL**

`https://api.spotify.com/v1/me/episodes?limit=1&offset=0`

**7. Get Contents of URL**

Configure:
- Method: `GET`
- Header:
  - `Authorization` = `Bearer ` followed immediately by the `AccessToken` magic variable

**8. Get Dictionary Value**
- Key: `total`

Rename it `Total`.

**9. Calculate**
- `Total - 1`

Rename it `MaxOffset`.

**10. Random Number**
- Minimum: `0`
- Maximum: `MaxOffset`

Rename it `RandomOffset`.

### D. Fetch exactly that random episode

**11. Text**

Enter:

`https://api.spotify.com/v1/me/episodes?limit=1&offset=`

Then append the `RandomOffset` magic variable directly after the equals sign.

**12. Get Contents of URL**

Configure:
- Method: `GET`
- URL: output of the previous Text action
- Header:
  - `Authorization` = `Bearer ` followed immediately by the `AccessToken` magic variable

### E. Extract the Spotify episode URL

**13. Get Dictionary Value**
- Key: `items`

**14. Get Item from List**
- Choose: `First Item`

**15. Get Dictionary Value**
- Key: `episode`

**16. Get Dictionary Value**
- Key: `external_urls`

**17. Get Dictionary Value**
- Key: `spotify`

### F. Open it

**18. Open URLs**

Input: output of the previous action.

That is the finished Shortcut.

## Optional polish

- Add **Random Podcast** to the Home Screen.
- Assign it a custom icon.
- Use Siri: “Random Podcast.”
- If you want to see what was chosen before Spotify opens, insert **Show Result** between steps 17 and 18 and also extract `episode.name`.

## What the randomization is doing

Spotify's saved-episodes endpoint returns a `total` count and supports an arbitrary `offset`.

The Shortcut:
1. retrieves `total`,
2. samples an integer uniformly from `0` through `total - 1`,
3. asks Spotify for the single episode at that offset.

So each saved episode has the same probability of being selected on a run, subject only to your library changing between the two API calls.

## Six-month reauthorization

Spotify currently expires refresh tokens issued to developer apps after six months. Refreshing the one-hour access token does **not** extend that six-month clock.

When the Shortcut eventually starts failing at the token request:
1. revisit the helper page,
2. authorize Spotify again,
3. copy the new refresh token,
4. replace the refresh-token Text action in the Shortcut.
