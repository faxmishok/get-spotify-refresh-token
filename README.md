# How to Get a Spotify Refresh Token with Required Scopes

This guide explains how to obtain a new Spotify refresh token that includes the necessary scopes for accessing endpoints such as "currently playing" and "top tracks" using the Spotify API.

> **Note:** You must have a Spotify Developer account and a registered application on the [Spotify Developer Dashboard](https://developer.spotify.com/dashboard/applications) before proceeding.

---

## Prerequisites

- **Spotify Developer Account:** Create an account or log in at the [Spotify Developer Dashboard](https://developer.spotify.com/dashboard/applications).
- **Registered Application:** Ensure you have created an app and added a valid Redirect URI.
- **Client Credentials:** Have your Spotify Client ID and Client Secret handy.
- **Node.js Environment:** For running the provided Node.js script.  
  *Tip:* Node.js v18+ includes the `fetch` API by default.

---

## Step 1: Construct the Authorization URL

To begin, you need an authorization URL that requests the required scopes. For accessing the currently playing track and top tracks, include:

- `user-read-currently-playing`
- `user-top-read`

**Example Authorization URL:**

```bash
https://accounts.spotify.com/authorize?client_id=YOUR_CLIENT_ID&response_type=code&redirect_uri=YOUR_REDIRECT_URI&scope=user-read-currently-playing%20user-top-read&show_dialog=true
```
Replace:
`YOUR_CLIENT_ID` with your actual Spotify Client ID.
`YOUR_REDIRECT_URI` with the exact URI registered in your Spotify Developer Dashboard. 
*Ensure the URI matches exactly (protocol, domain, port, and trailing slash, if any).*

## Step 2: Authorize and Retrieve the Code
1. **Open the Authorization URL in your browser.**
2. **Log in to Spotify** (if needed) and grant the requested permissions.
3. After granting permissions, Spotify will redirect you to your specified Redirect URI with a query parameter named `code`. For example:

```bash
https://your-app.com/callback?code=AUTHORIZATION_CODE&state=some_state
```
4. **Copy the** `AUTHORIZATION_CODE` from the URL. This code is temporary and will be used to request your tokens.

## Step 3: Exchange the Authorization Code for Tokens
You now need to exchange the authorization code for an access token and a refresh token. This is done by sending a POST request to Spotify’s token endpoint.

**Install Dependencies (if needed):**
If you're using Node.js older than v18, install `node-fetch`:
```bash
npm install node-fetch
```
**Create a file named `getRefreshToken.js` *(you can also find it in the repo)* with the following content:**

```js
// If using Node.js v18+, you can remove the import statement below.
import fetch from 'node-fetch'; // For Node.js versions < 18; remove if not needed

const client_id = 'YOUR_CLIENT_ID';
const client_secret = 'YOUR_CLIENT_SECRET';
const redirect_uri = 'YOUR_REDIRECT_URI';
const code = 'YOUR_AUTHORIZATION_CODE'; // Replace with the code you copied from the URL

const basic = Buffer.from(`${client_id}:${client_secret}`).toString('base64');

async function getToken() {
  const response = await fetch('https://accounts.spotify.com/api/token', {
    method: 'POST',
    headers: {
      'Authorization': `Basic ${basic}`,
      'Content-Type': 'application/x-www-form-urlencoded',
    },
    body: new URLSearchParams({
      grant_type: 'authorization_code',
      code,
      redirect_uri,
    }),
  });

  const data = await response.json();
  console.log(data);
}

getToken().catch(console.error);
```

**Run the script:**

```bash
node getRefreshToken.js
```

**Response Example:**

```json
{
  "access_token": "BQD...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "AQD...",
  "scope": "user-read-currently-playing user-top-read"
}
```

### Troubleshooting
#### Invalid Redirect URI Error
* Double-check that the `redirect_uri` used in your script exactly matches the one registered in your Spotify Developer Dashboard.
#### Insufficient Scopes Error
* Make sure the authorization URL includes all necessary scopes **`(user-read-currently-playing and user-top-read)`.**
#### Node.js Module Type Warning
* If you receive warnings about module types, add `type:module` to your `package.json` if you wish to consistently use ES modules.

Hope it helped, peace out.
