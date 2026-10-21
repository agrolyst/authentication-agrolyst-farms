# Agrolyst Authentication Console

Private-facing authentication console and OAuth authorization UI for **https://authentication.agrolystfarms.com/**.

## Deployment

Deploy the repository root as the static site's publish directory so the root `index.html` serves at the subdomain root. Add `authentication.agrolystfarms.com` as a custom domain in the hosting provider, configure the DNS record the provider specifies, and enable HTTPS.

Pages:
- Admin console and OAuth client management: `/dashboard/`
- OAuth seller sign-in and consent UI: `/oauth/consent/`
- Temporary OAuth test client: `/oauth/test-client/`

## Dedicated authentication Supabase project

The authentication console now targets the dedicated Supabase project **Agrolyst Authentication**:

- Project ref: `hbfhrilouvbvcpfqwmta`
- Project URL: `https://hbfhrilouvbvcpfqwmta.supabase.co`
- Region: `eu-central-1`
- Frontend uses only the project's publishable key; privileged keys must remain server-side.

The original Agrolyst website project `rcrgooivgssuerrgnuqw` is intentionally unchanged. OAuth client registrations and user accounts are project-specific and do not automatically transfer to the new project. Recreate required OAuth clients in the new project and re-enter their exact redirect URIs; re-upload logos or save branding for the newly created client IDs. Existing logo files in the old project remain there until you choose to migrate them.

### Configure Auth before production use

In the new project's Supabase Dashboard:

1. Open **Authentication → URL Configuration**.
2. Set **Site URL** to `https://authentication.agrolystfarms.com`.
3. Add the exact redirect URLs needed for the authentication console, including `https://authentication.agrolystfarms.com/`. Add any other callback URLs only when they are actually used.
4. Open **Authentication → OAuth Server**, enable the OAuth 2.1 server, and set the **Authorization Path** to `/oauth/consent`.
5. Under the OAuth app/client management section, recreate the required clients with their exact registered redirect URIs.
6. Check email confirmation and password-reset flows before inviting users. The main farm website remains on its original Supabase project and should keep its own Auth URL configuration.

### First administrator account

The sign-in page now allows an account to be created in the new authentication project. Creating an account does **not** grant administrator permissions.

1. Open `https://authentication.agrolystfarms.com/` after the updated site is deployed, create an account, and confirm the email if required.
2. In the new project's SQL Editor, grant administrator access to that exact account using the email address you registered:

```sql
insert into public.admin_users (user_id, display_name, role, is_active)
select id, 'Agrolyst Admin', 'super_admin', true
from auth.users
where lower(email) = lower('REPLACE_WITH_YOUR_REGISTERED_EMAIL')
on conflict (user_id) do update
set display_name = excluded.display_name,
    role = excluded.role,
    is_active = true;
```

3. Verify exactly one row was inserted/updated, then sign in again at `/dashboard/`.

Never put a service-role or secret key in HTML or share it in chat. The `oauth-admin` Edge Function is deployed to the new project with JWT verification enabled.

## Dashboard OAuth client management

The admin dashboard calls the `oauth-admin` Supabase Edge Function. The function requires a valid signed-in user and checks `public.admin_users` for an active `admin` or `super_admin` record before each operation. It supports listing registered clients, creating public or confidential clients, and deleting clients. Redirect URIs are validated server-side (HTTPS, with localhost HTTP permitted for development); wildcard redirect URIs are not accepted. The function runs with JWT verification enabled. No Supabase service-role/secret key is included in browser code.

Confidential client secrets are returned only at creation time so the authorized administrator can store them in a server-side secret manager. Do not put them into browser applications. Public client IDs may be used in browser/mobile apps.

## OAuth 2.1 setup — required before real connections work

The consent page is implemented against Supabase Auth's OAuth server methods, but a real authorization flow will not work until OAuth Server is enabled and a client is registered in the Supabase project.

1. In the Supabase dashboard for the dedicated authentication project `hbfhrilouvbvcpfqwmta`, open **Authentication → OAuth Server** and enable OAuth 2.1 server capabilities.
2. Set the **Authorization Path** to `/oauth/consent`.
3. Set the Site URL to `https://authentication.agrolystfarms.com` so the configured authorization UI resolves to `https://authentication.agrolystfarms.com/oauth/consent`. Add the exact authentication-console URLs and any actual app callback URLs to **Authentication → URL Configuration → Redirect URLs**. Keep the main farm website on its original Supabase project and do not change that project's Auth URL settings for this setup.
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
