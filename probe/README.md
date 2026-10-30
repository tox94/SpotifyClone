# Titan OS Playback Probe

A single page (`index.html`, no build step) that checks whether a Titan OS TV can play Spotify inside a web app. It reports:

- **DRM (EME):** Widevine, PlayReady and ClearKey support, which audio codecs each one handles, and whether its decryption module (CDM) actually starts. Spotify's Web Playback SDK needs Widevine.
- **Audio codecs:** what the `<audio>` element and Media Source Extensions can play.
- **Autoplay:** whether sound can start before you press a key.
- **Device:** Chromium version, screen size, HTTPS, localStorage.
- **Remote keys:** the key code of every remote button you press, for later.
- **Spotify live test (optional):** logs in to your account, starts a real Spotify player on the TV and checks that a track actually plays.

The full results are also printed to the console as `TITAN_PROBE_RESULTS {...}` and are available as `window.titanProbeResults`.

## 1. Host it over HTTPS

DRM only works on HTTPS pages, so the TV must load the page from an HTTPS URL. Any static host works, for example:

- **GitHub Pages:** push this `probe` folder to a repo and turn on Pages.
- **Netlify Drop:** drag the `probe` folder onto https://app.netlify.com/drop.
- **Cloudflare Pages:** upload the folder as a direct upload project.

Note the final URL, for example `https://yourname.github.io/titan-probe/`. Keep the trailing slash.

## 2. (Optional) Create a Spotify app for the live test

Skip this if you only want the DRM checks.

1. You need Spotify Premium. Development-mode apps require it.
2. Open https://developer.spotify.com/dashboard and create an app.
   - **Redirect URI:** your exact hosted URL from step 1, for example `https://yourname.github.io/titan-probe/`.
   - **APIs used:** tick *Web API* and *Web Playback SDK*.
3. Copy the app's **Client ID**. No client secret is needed: the page uses the PKCE login flow.
4. Under *User Management*, add the email of the Spotify account you will log in with. Development mode allows at most 5 users.

Your probe URL is then:

```
https://yourname.github.io/titan-probe/?client_id=YOUR_CLIENT_ID
```

## 3. Open it on the TV with DevView

1. Log in to the Titan OS Partner Portal (https://partners.titanos.tv). Open **DevView** and request access if you don't have it yet. Titan OS reviews these requests.
2. In DevView's **Sandbox**, add an app and use the probe URL from step 2 (or step 1 if you skip the live test).
3. On the TV, install the **DevView** app. On 2020–2022 Philips models it's in the App Gallery; on newer models look under Apps in the Lifestyle carousel.
4. In the portal's **Device Management**, choose *Add new device* and enter the code shown on the TV.
5. Launch the probe from DevView.

## 4. Use it

- **Left / Right** moves between buttons, **OK** presses one, **Up / Down** scrolls.
- The DRM checks run on their own. The coloured box at the top gives the verdict.
- For the live test: press **Spotify: log in** and sign in on the TV (you only need to do this once), then **Spotify: start player**, then **Play on this TV**. It plays your most recently played track. The test passes when the track's position keeps advancing. Check that you can actually hear it.

## What the result means

| Verdict | Meaning for the Spotify TV app |
| --- | --- |
| Green after the live test | Spotify plays inside the web app. Build the full app with in-app playback. |
| Green before the live test | Widevine and AAC work, so playback is likely. Run the live test to be sure. |
| Red "No Widevine" or a failed live test | In-app playback won't work. The app would act as a remote control and play on another device (phone, speaker). |

Send back a photo of the verdict box and the DRM table, or the `TITAN_PROBE_RESULTS` line from the DevView console.

## Trying it on a computer first

`localhost` counts as a secure context, so DRM checks work locally:

```bash
python3 -m http.server 8080 --directory probe
```

Then open http://127.0.0.1:8080/. For the Spotify live test locally, also add `http://127.0.0.1:8080/` as a redirect URI in the Spotify dashboard. Spotify doesn't accept `localhost` there, only the `127.0.0.1` address.
