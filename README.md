# Agrolyst Authentication Console

Private-facing authentication console for **https://authentication.agrolystfarms.com/**.

## Deployment

Deploy the repository root as the static site's publish directory so the root `index.html` serves at the subdomain root. Add `authentication.agrolystfarms.com` as a custom domain in the hosting provider, configure the DNS record the provider specifies, and enable HTTPS.

## Security status

- The browser uses only the Supabase publishable key.
- The page is marked `noindex,nofollow,noarchive`; this is not access control.
- Sign-in uses the existing Agrolyst Supabase project.
- The UI checks the verified user's `app_metadata.oauth_admin` claim. A normal signed-in user is not an administrator.
- OAuth client registration, client-secret operations, and redirect management are intentionally disabled until a protected server-side management API and administrator assignment procedure are configured.
- Never add a Supabase service-role/secret key or OAuth client secret to this repository or browser code.

## Supabase configuration

Use the project's Authentication URL configuration to allow the required callback URLs for the new subdomain only after deciding the exact callback paths. Keep the existing Agrolyst login and callback routes working for the main website.

## Local preview

Open `index.html` with a local static server. This is a browser-based shell; privileged OAuth management still requires the protected backend described above.
