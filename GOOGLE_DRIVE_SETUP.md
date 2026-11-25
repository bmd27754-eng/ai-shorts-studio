# Google Drive integration setup

## OAuth settings

Enable Google Drive API and create an OAuth Client ID of type **Web application**. Add this exact Authorized redirect URI for local testing:

```text
http://localhost:3000/api/google-drive/callback
```

For deployment, also add the exact callback URL for the deployed app, for example `https://YOUR-VERCEL-DOMAIN/api/google-drive/callback`. Set `GOOGLE_DRIVE_REDIRECT_URI` to the same URL. The previously discussed `/api/auth/google/callback` path is not used by this integration.

Add the Google account used for testing under OAuth Audience → Test users.

## Environment variables

Set these on the web service and worker:

- `GOOGLE_DRIVE_CLIENT_ID`
- `GOOGLE_DRIVE_CLIENT_SECRET`
- `GOOGLE_TOKEN_ENCRYPTION_KEY` — exactly 64 hexadecimal characters (32 random bytes), identical on web and worker

Set this on the web service only:

- `GOOGLE_DRIVE_REDIRECT_URI` — exact callback URL above

Generate an encryption key locally with Node.js:

```powershell
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

Never commit real credentials or paste them into chat.

## Upload behavior

The browser sends 8 MiB chunks directly to a Google Drive resumable-upload session. The server does not receive the 1 GB file body. The worker downloads the private source from Drive, then stores rendered clips, thumbnails and SRT files in the same user's Drive. The web app streams private Drive files through an owner-checked API route.

The project requests the `drive.file` scope, limiting access to files created/opened by this app rather than broad access to the user's entire Drive. Drive storage is subject to the user's available quota (commonly 15 GB shared with Gmail and Google Photos on a standard free Google Account), API quotas, and Google policies.

## Important limitations before public deployment

- This repository's demo sign-in stores an email-based user ID in browser local storage; it is not production authentication. Do not expose this build publicly with multiple users until real server-side authentication and authorization are implemented.
- OAuth Audience in **Testing** has a 7-day authorization/refresh-token lifetime for scopes beyond basic profile scopes. You may need to reconnect Google Drive every 7 days while the app remains in Testing. See Google's OAuth documentation: https://developers.google.com/identity/protocols/oauth2
- Instagram auto-publishing is not available with private Drive output files because Instagram requires a publicly fetchable media URL. YouTube publishing can download the private clip via the worker.
- A 1 GB upload is not the same as a successful 1 GB video-processing job. Render Free memory, execution time, disk, and FFmpeg limits may prevent processing a video of that size. Test upload and worker processing separately.
- The code has not been tested against a live Google OAuth account in this environment. Complete the deployment environment variables and run an actual small-file upload before attempting a 1 GB test.
