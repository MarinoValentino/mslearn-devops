# Implementation TODO: Azure Boards bugs 981 and 982

## Scope and blocker

This document records the requirements from both bugs and their discussions,
plus proposed implementation and validation steps. It does not fix either bug
or establish that any acceptance criterion has passed.

The target storefront source is missing from this repository's `main` branch.
There are no Razor views, including the layout referenced by bug 981, and no
Google/Facebook login implementation in the included application sources.
The `Azure.Identity` dependency in the catalog API is not the ASP.NET Core
Identity social-login implementation described by bug 982.

Before implementation:

- [ ] Obtain the storefront repository and branch.
- [ ] Reproduce both bugs and identify the existing menu and authentication flow.
- [ ] Confirm the storefront's framework version and existing test tooling.
- [ ] Obtain a test deployment and Google/Facebook test authentication
      configuration through the project's approved secret-management process.
      Do not commit provider secrets.

Both bugs should remain open until their corrections are implemented and
validated in the actual storefront.

## Bug 981: Mobile menu overlapping content on iOS Safari

Source: [Azure Boards bug 981](https://dev.azure.com/marinovalentinonexplore/AZ-401/_workitems/edit/981/).

### Reported problem and acceptance criteria

Navigation overlaps page content after an orientation change on iOS Safari.
The discussion requests:

- [ ] A full-screen overlay menu.
- [ ] Closing the menu when clicking outside it.
- [ ] Smooth animation using CSS transitions.
- [ ] Correct behavior on iOS Safari 15+ and Chrome Mobile.

### Implementation TODO

The discussion identifies `Views/Shared/_Layout.cshtml` and
`wwwroot/css/site.css`; these paths must be verified in the storefront.

- [ ] Implement a full-viewport overlay with a distinct menu panel and backdrop.
      Keep the backdrop outside the panel's click target so tapping it dismisses
      the menu without treating taps inside the panel as outside clicks.
- [ ] Inspect ancestor stacking contexts and position the overlay above the
      content. Account for iOS viewport resizing, safe areas, orientation changes,
      and menus taller than the viewport.
- [ ] Prevent background scrolling while open and restore normal scrolling on
      dismissal. Preserve existing desktop navigation behavior.
- [ ] Add CSS transitions, respecting reduced-motion preferences.
- [ ] Preserve keyboard access: update the toggle's expanded state, support
      Escape to close, and manage focus while open and after dismissal.

### Validation TODO

Manual testing on actual iOS Safari is required; browser emulation alone does
not satisfy this requirement.

1. On iOS Safari 15 and a current supported iOS Safari version, load a page with
   enough content to scroll. Record device and browser versions.
2. In portrait, open the hamburger menu. Verify the backdrop covers the viewport,
   the panel remains usable, and background content cannot be scrolled.
3. Rotate to landscape while the menu is open, then back to portrait. Verify
   coverage, panel scrolling, and that links are not clipped or obscured.
4. Tap inside the panel and verify it stays open unless navigating. Tap the
   backdrop and verify it closes and background scrolling is restored.
5. Repeat after scrolling the page and changing the browser toolbar visibility.
   Verify reopening and the close control still work.
6. Repeat the open, rotate, outside-tap, and close checks on Chrome Mobile.
   Check keyboard dismissal and focus restoration with a keyboard.
7. Verify desktop navigation and reduced-motion behavior. Record results.

- [ ] Extend existing UI tests for open/close state and outside-click behavior.
- [ ] Add portrait/landscape visual regression cases if the storefront already
      has visual regression tooling.
- [ ] Attach manual device results before marking the bug fixed.

## Bug 982: Social login failing with popup blocker

Source: [Azure Boards bug 982](https://dev.azure.com/marinovalentinonexplore/AZ-401/_workitems/edit/982/).

### Reported problem and acceptance criteria

Default browser settings block the popup-based social-login flow.
The discussion requests:

- [ ] Redirect-based OAuth instead of a popup.
- [ ] Preservation of the return URL after authentication.
- [ ] Updates to both Google and Facebook login flows.
- [ ] A user-friendly error message when OAuth fails.

### Implementation TODO

The discussion identifies ASP.NET Core Identity as the authentication framework;
verify its configuration and existing external-login handlers in the storefront.

- [ ] Replace popup launches for both providers with navigation to the existing
      server-side external-login challenge flow.
- [ ] Reuse the framework's authentication properties, callback handling, and
      correlation/state validation rather than implementing a separate OAuth
      protocol flow.
- [ ] Carry the intended local return URL through the challenge and callback,
      preserving its path and query string. Validate it as local before the
      final redirect; reject external return targets using the application's
      standard error handling or explicitly documented safe navigation.
- [ ] Preserve existing account linking and sign-in behavior. Confirm each
      provider's registered callback URI matches the test deployment.
- [ ] Handle provider errors, cancellation, and invalid callback state with a
      clear localized message and a retry path, using existing logging patterns
      without exposing tokens or sensitive provider responses.

### Validation TODO

Run the following checks separately for Google and Facebook:

1. Enable the browser's popup blocker and start login from a deep link with a
   query string. Verify navigation uses a redirect, not a popup.
2. Complete authentication and verify the user returns to the original local
   path with its query string intact.
3. Start login without a return URL and verify the application's documented
   default destination.
4. Cancel at the provider, then exercise a provider failure and invalid/expired
   callback state. Verify a useful error and retry path with no false success.
5. Supply an external return target and verify there is no redirect to that
   target. Confirm normal local deep links still work.
6. Verify existing-account sign-in and the application's supported new-account
   or account-linking path remain functional.

- [ ] Add focused tests for local return URL preservation, external target
      rejection, and authentication failure handling.
- [ ] Run end-to-end checks for both providers on the configured test deployment
      with popup blocking enabled and record the results.
- [ ] Do not mark the bug fixed based only on mocked authentication tests.
