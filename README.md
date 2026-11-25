# Entra MFA and SSO Lab

A hands-on lab on a free Microsoft Entra ID tenant (Default Directory). It covers MFA registration, sign-in log analysis, and two SSO sign-ins, one OIDC and one SAML. Nothing here uses a paid license.

## What I set up

- **Tenant:** Microsoft Entra ID Free, with one test user (`testuser`) alongside my admin account.
- **MFA:** Security defaults enabled. The test user registered Microsoft Authenticator, and sign-in prompted for MFA.
- **OIDC app (`lab-oidc-app`):** single-tenant app registration with a web redirect URI at `http://localhost:8080/callback`. I ran the authorization request in a browser, accepted the consent prompt, and reached the redirect with a sign-in code.
- **SAML app (`lab-saml-app`):** non-gallery enterprise application with SAML single sign-on configured. Entity ID and reply URL were placeholder values, since the lab has no service behind them.

## What the logs showed

| Test | Result in the sign-in log | What it tells me |
|---|---|---|
| Wrong password for `testuser` | Failure, error 50126 (invalid username or password) | Failed at the password step, before MFA |
| MFA request denied on phone | Did not appear in the log during my attempts | Denials don't always log as a new sign-in, so I didn't rely on this |
| Portal sign-in after MFA registration | Success | Sign-in completed after the MFA step |
| OIDC app sign-in | Interrupted (consent), then Success | The app sign-in completed |
| SAML test sign-in (signed in as my admin account) | Success | Test sign-in uses the signed-in user, so it wasn't tested as `testuser` |

## Finding: app sign-ins showed single-factor

Several OIDC app sign-ins were logged as single-factor, and I didn't see an MFA prompt for them, even after revoking the test user's sessions.

The Authentication details tab explained it. The sign-in was marked **Previously satisfied**, meaning Entra accepted an MFA claim from an earlier sign-in and didn't ask again. Microsoft's sign-in log guidance describes this as expected behaviour when a previous MFA claim is still valid.

Practical takeaway: a user may not see an MFA prompt on every sign-in, and the log's Authentication requirement and Authentication details fields are where to check why.

## What I did not test

- **Conditional Access.** A Conditional Access policy needs an Entra ID P1 or P2 license. This tenant is on the free tier, and I did not start a paid trial. The policy is documented from Microsoft's docs but not tested here.
- **iOS.** Only Windows and Android clients were tested.
- **Entra MFA enforced per app.** Because of the single-factor finding above, I did not show MFA being required for the OIDC or SAML app.

## Screenshots

Stored in `screenshots/`, with identifiers blurred:

- `01-failed-signin-50126.png`: failed password sign-in, error 50126
- `02-security-defaults-enabled.png`: security defaults enabled on the tenant
- `03-app-registration-overview.png`: `lab-oidc-app` overview
- `04-oidc-signin-previously-satisfied-mfa.png`: OIDC sign-in, MFA claim reused
- `05-saml-app-overview.png`: `lab-saml-app` overview
- `06-saml-sso-configuration.png`: SAML single sign-on configuration
- `07-saml-signin-success.png`: SAML test sign-in in the log

## Open items

- Add a SAML vs OIDC comparison section.
- Test Conditional Access if a P1 or P2 license becomes available.
