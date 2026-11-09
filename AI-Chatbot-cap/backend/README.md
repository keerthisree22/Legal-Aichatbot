# Run locally

1. In Firebase Console, enable Email/Password under Authentication sign-in providers.
2. Ensure `firebase-config.js` uses the same Firebase project as `FIREBASE_PROJECT_ID`.
3. Keep `GEMINI_API_KEY` in `backend/.env`; never put it in frontend files.
4. From this directory, run `npm install` and `npm start`.
5. Open `http://localhost:3000`, create an account, and send a question with up to 3 files.

The Firebase web configuration is used by the browser and is not a server secret. The backend verifies Firebase ID tokens before forwarding chat messages and attachments to Gemini. Chat history remains in this browser's local storage; uploaded chat attachments are sent for the current request and are not stored by the backend.

## Chat file uploads

Supported attachments are PDF, TXT, Markdown, PNG, JPEG, and WebP. The server accepts at most 3 files per chat request, up to 4 MB each. Attachments are sent to Gemini with the user's question and are not persisted by the server.

## Legal reference library

`knowledge-base.json` contains starter answer safeguards and official Indian source links, not a collection of substantive statutes. Upload current official legal text as plain `.txt` or `.md` files so the assistant can cite and use it. Admin uploads are limited to 5 files per request, 100 KB each, and 150,000 characters total.

Add the Firebase Authentication UID of each trusted administrator to `FIREBASE_ADMIN_UIDS` in `backend/.env`, separated by commas, then restart the backend. Admin UIDs can be found for a signed-in user in Firebase Console under Authentication > Users. The server checks this UID on every admin request.

Send a multipart `POST /api/admin/knowledge` request with a Firebase ID token, a repeated `documents` field for each `.txt`/`.md` file, and an optional `source` field containing the official source URL or citation. For example, from PowerShell after obtaining an ID token:

```powershell
curl.exe -H "Authorization: Bearer $idToken" `
	-F "source=https://www.indiacode.nic.in/" `
	-F "documents=@official-law.txt;type=text/plain" `
	http://localhost:3000/api/admin/knowledge
```

The admin list endpoint is `GET /api/admin/knowledge`. Do not upload personal case files or confidential documents to the shared legal reference library.

## Deploy to Render

1. Push this project to a GitHub repository. The root `.gitignore` excludes `backend/.env` and `node_modules`; do not commit API keys.
2. In Render, choose **New + > Blueprint**, connect the repository, and apply the `render.yaml` blueprint.
3. Enter the value of `GEMINI_API_KEY` from your local `backend/.env` directly in Render's environment-variable prompt. Never put it in `firebase-config.js` or source control. `FIREBASE_PROJECT_ID` is set by the blueprint.
4. After deployment, copy the `https://<service-name>.onrender.com` URL from Render. In Firebase Console, add its hostname (without `https://`) under Authentication > Settings > Authorized domains.
5. Open the Render URL, sign in, and send a test message. The `/api/health` endpoint is used for Render health checks.

The free Render service may sleep when idle and uses an ephemeral filesystem. The checked-in starter `knowledge-base.json` is deployed with the app, but documents added later through the admin upload endpoint can be lost when the service restarts or redeploys. Use persistent storage (such as Firestore or a paid persistent disk) before relying on admin uploads in production.