# Agrolyst Authentication Console

Private-facing authentication console and OAuth authorization UI for **https://authentication.agrolystfarms.com/**.

## Deployment

Deploy the repository root as the static site's publish directory so the root `index.html` serves at the subdomain root. Add `authentication.agrolystfarms.com` as a custom domain in the hosting provider, configure the DNS record the provider specifies, and enable HTTPS.

Pages:
- Admin console and OAuth client management: `/dashboard/`
- OAuth seller sign-in and consent UI: `/oauth/consent/`
- Temporary OAuth test client: `/oauth/test-client/`

## Dashboard OAuth client management

The admin dashboard calls the `oauth-admin` Supabase Edge Function. The function requires a valid signed-in user and checks `public.admin_users` for an active `admin` or `super_admin` record before each operation. It supports listing registered clients, creating public or confidential clients, and deleting clients. Redirect URIs are validated server-side (HTTPS, with localhost HTTP permitted for development); wildcard redirect URIs are not accepted. The function runs with JWT verification enabled. No Supabase service-role/secret key is included in browser code.

Confidential client secrets are returned only at creation time so the authorized administrator can store them in a server-side secret manager. Do not put them into browser applications. Public client IDs may be used in browser/mobile apps.

## OAuth 2.1 setup — required before real connections work

The consent page is implemented against Supabase Auth's OAuth server methods, but a real authorization flow will not work until OAuth Server is enabled and a client is registered in the Supabase project.

1. In the Supabase dashboard for project `rcrgooivgssuerrgnuqw`, open **Authentication → OAuth Server** and enable OAuth 2.1 server capabilities.
2. Set the **Authorization Path** to `/oauth/consent`.
3. Set the Site URL to `https://authentication.agrolystfarms.com` so the configured authorization UI resolves to `https://authentication.agrolystfarms.com/oauth/consent`. Before changing the Site URL, review existing email-confirmation, password-reset and other auth redirects; add each required exact callback to **Authentication → URL Configuration → Redirect URLs** so the main website continues to work.
4. For a first test, open `https://authentication.agrolystfarms.com/oauth/test-client/` and copy the exact redirect URI shown on that page. In **Authentication → OAuth Apps**, create a **public** test client and register that exact URI. Do not create a confidential client for this static browser test.
5. Copy the generated public **Client ID** into the test page and click **Start OAuth test**. Use a test seller account. The page initiates Authorization Code + PKCE, verifies the returned `state`, and confirms that approval or denial redirects back. This first-stage test does **not** exchange the authorization code for tokens.
6. Confirm the consent page shows the requesting app, redirect URI and requested scopes. Test both **Approve** and **Deny**.
7. Use asymmetric JWT signing keys if you need OpenID Connect ID tokens; follow Supabase's current signing-key migration guidance first.

Supabase guides:
- https://supabase.com/docs/guides/auth/oauth-server/getting-started
- https://supabase.com/docs/guides/auth/oauth-server/oauth-flows

## Security notes

- The browser uses only the Supabase publishable key.
- The pages are marked `noindex,nofollow,noarchive`; this is not access control.
- Ordinary seller sign-in is separate from the admin allowlist; sellers do not need to be administrators to authorize an OAuth client.
- The consent UI reads the current OAuth request using `authorization_id` and uses Supabase Auth's `getAuthorizationDetails`, `approveAuthorization` and `denyAuthorization` methods. It does not accept a hand-entered redirect URL.
- The test client is a temporary verification harness, not a production OAuth integration. It checks the returned state and code/error but intentionally does not exchange the code for tokens.
- OAuth client management and secret operations must be performed through Supabase's protected OAuth app management or a properly authenticated server-side API.
- Never add a Supabase service-role/secret key or confidential OAuth client secret to this repository or browser code.

## Local preview

Open the pages with a local static server. Without a real `authorization_id` created by a registered OAuth request, the consent page intentionally asks the user to start from the connecting application instead of inventing a login flow.
