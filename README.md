# Agrolyst Authentication Console

Private-facing authentication console and OAuth authorization UI for **https://authentication.agrolystfarms.com/**.

## Deployment

Deploy the repository root as the static site's publish directory so the root `index.html` serves at the subdomain root. Add `authentication.agrolystfarms.com` as a custom domain in the hosting provider, configure the DNS record the provider specifies, and enable HTTPS.

Pages:
- Admin console: `/dashboard/`
- OAuth seller sign-in and consent UI: `/oauth/consent/`

## OAuth 2.1 setup — required before real connections work

The consent page is implemented against Supabase Auth's OAuth server methods, but a real authorization flow will not work until OAuth Server is enabled and a client is registered in the Supabase project.

1. In the Supabase dashboard for project `rcrgooivgssuerrgnuqw`, open **Authentication → OAuth Server** and enable OAuth 2.1 server capabilities.
2. Set the **Authorization Path** to `/oauth/consent`.
3. Set the Site URL to `https://authentication.agrolystfarms.com` so the configured authorization UI resolves to `https://authentication.agrolystfarms.com/oauth/consent`. Before changing the Site URL, review existing email-confirmation, password-reset and other auth redirects; add each required exact callback to **Authentication → URL Configuration → Redirect URLs** so the main website continues to work.
4. Register each requesting app under **Authentication → OAuth Apps**. Use exact HTTPS redirect URLs; choose a public client for browser/mobile apps and a confidential client only for a server-side app. Keep any confidential client secret on its server.
5. Test with a non-admin seller account and a registered test client. Confirm the consent page shows the client name, redirect URI and requested scopes, and that both approve and deny return through Supabase Auth's validated redirect.
6. Use asymmetric JWT signing keys if you need OpenID Connect ID tokens; follow Supabase's current signing-key migration guidance first.

Supabase's current setup guide: https://supabase.com/docs/guides/auth/oauth-server/getting-started

## Security notes

- The browser uses only the Supabase publishable key.
- The page is marked `noindex,nofollow,noarchive`; this is not access control.
- Ordinary seller sign-in is separate from the admin allowlist; sellers do not need to be administrators to authorize an OAuth client.
- The consent UI reads the current OAuth request using `authorization_id` and uses Supabase Auth's `getAuthorizationDetails`, `approveAuthorization` and `denyAuthorization` methods. It does not accept a hand-entered redirect URL.
- OAuth client management and secret operations must be performed through Supabase's protected OAuth app management or a properly authenticated server-side API.
- Never add a Supabase service-role/secret key or confidential OAuth client secret to this repository or browser code.

## Local preview

Open the pages with a local static server. Without a real `authorization_id` created by a registered OAuth request, the consent page intentionally asks the user to start from the connecting application instead of inventing a login flow.
