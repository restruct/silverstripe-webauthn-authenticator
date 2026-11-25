# silverstripe/webauthn-authenticator on webauthn-lib 5

*Maintained by [Restruct](https://github.com/restruct). If this fork saves you time, you can [support ongoing maintenance](https://github.com/sponsors/restruct).*

This is a fork of [silverstripe/silverstripe-webauthn-authenticator](https://github.com/silverstripe/silverstripe-webauthn-authenticator), the security key (WebAuthn) method for [silverstripe/mfa](https://github.com/silverstripe/silverstripe-mfa). Upstream requires `web-auth/webauthn-lib ^4.4` on every supported line, and webauthn-lib 4 receives no fixes for the advisories published against it in 2026. This fork moves the module to `web-auth/webauthn-lib ^5.3.13`, where they are fixed, for both CMS 6 and CMS 5. Nothing else changes.

> **What to expect from this fork**
>
> - **One change only: webauthn-lib 5.** No features, no general bug fixes. Requests for those will normally be declined.
> - **Meant to be temporary.** The change is offered upstream ([#228](https://github.com/silverstripe/silverstripe-webauthn-authenticator/issues/228)). Once upstream supports webauthn-lib 5, switch back and this fork will stop being updated.
> - **It may be discontinued at any time.** It exists mainly to keep our own projects secure. Anyone is welcome to use it as is, under the same license and with no warranty.
> - **Not affiliated with or endorsed by Silverstripe.** Any mistake in the port is ours, not upstream's.

## Quick start

| CMS | Branch | Tag | PHP |
|---|---|---|---|
| 6 | `6.0-restruct` | `6.0.1-p1` (upstream 6.0.1) | ^8.3 |
| 5 | `5.3-restruct` | `5.3.3-p1` (upstream 5.3.3) | ^8.2 |

1. Add this repository to the project's `composer.json`, ahead of any other entry that could provide the package, and exclude the package from Packagist:

   ```json
   "repositories": [
       {"type": "vcs", "url": "https://github.com/restruct/silverstripe-webauthn-authenticator", "only": ["silverstripe/webauthn-authenticator"]},
       {"type": "composer", "url": "https://repo.packagist.org", "exclude": ["silverstripe/webauthn-authenticator"]},
       {"packagist.org": false}
   ]
   ```

   A CMS 5 project that already uses the [CMS 5 security fork set](https://github.com/restruct/silverstripe-framework) has such a Packagist entry: add `silverstripe/webauthn-authenticator` to its `exclude` list and the `vcs` entry above to the others.

2. Remove any `config.policy.advisories.ignore-id` (or `audit.ignore`) entries for `web-auth/webauthn-lib` advisories: they no longer apply, and an ignore would hide future ones.

3. Update only the WebAuthn packages (`-W` would also update unrelated shared libraries):

   ```bash
   composer update silverstripe/webauthn-authenticator web-auth/webauthn-lib web-auth/cose-lib spomky-labs/cbor-php spomky-labs/pki-framework
   composer show silverstripe/webauthn-authenticator web-auth/webauthn-lib
   composer audit
   ```

   `composer show` should list `6.0.1-p1` or `5.3.3-p1`, and webauthn-lib 5.3.13 or later. Then run `dev/build flush=1`.

4. On a test site, register a security key and log in with it in a real browser, and log in with a key that was registered before the update.

To go back to upstream, remove the two repository entries again. Keys registered with this fork keep working on the upstream release (see below).

## What changed

The commits on top of the upstream tag list each webauthn-lib 5 API change. In short:

- The browser's credential JSON is read through webauthn-lib 5's serializer, and the checks are configured through its ceremony steps; login still accepts ES256 only.
- Registered keys are still stored in the webauthn-lib 4 JSON format, so **existing registrations keep working** and no data migration is needed. Keys registered after the update also still load if a site rolls back to the upstream release (checked on webauthn-lib 4.4.3 and 4.9.3).
- What the browser receives is unchanged, apart from an empty `excludeCredentials` list (the WebAuthn default).

## Testing

Upstream's own tests mock webauthn-lib, so they cannot show that stored registrations survive a library change. This fork adds `tests/SoftwareAuthenticator.php`, a software security key (real ES256 keys and signatures, `none` and `fido-u2f` attestation, encoded as the module's client code encodes it), and `tests/StoredCredentialTest.php`, which registers and logs in through the real handlers. `tests/fixtures/` holds registrations as webauthn-lib 4.9.3 stored them; they must keep verifying.

Both branches pass the full suite (53 tests) on PHP 8.3, and [restruct/silverstripe-mfa-bundle](https://github.com/restruct/silverstripe-mfa-bundle)'s suite passes against them. Not tested: a hardware security key in a real browser (hence step 4), and PHP 8.2.

A member who is in the middle of an MFA login while the update is deployed may have to start the login again.

## Upstream documentation

Installation, configuration and usage are as upstream: see the [upstream README](https://github.com/silverstripe/silverstripe-webauthn-authenticator/blob/6/README.md) and the [MFA user guide](https://userhelp.silverstripe.org/en/6/optional_features/multi-factor_authentication/).

## License

BSD-3-Clause, as upstream. See [LICENSE](LICENSE).
