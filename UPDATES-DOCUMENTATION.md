# ReeMate Updates Documentation

This document explains the update flow for ReeMate.
Scope: update logic only, not full CI/CD setup.

## 1) Overview

ReeMate updates are composed of two parts:

1. App side (macOS app): checks for updates, validates update metadata, downloads, installs.
2. Site side (GitHub Pages): publishes update metadata and release notes, exposes stable URLs.

Recommended stack for direct macOS distribution:

1. Sparkle in the app.
2. appcast.xml on the website.
3. EdDSA signature for each update artifact.

Code signing + notarization remain mandatory and are complementary.

## 2) App Side Logic (macOS)

## 2.1 Check policy

Use both automatic and manual checks:

1. Automatic check at launch (with a small delay).
2. Periodic check every 12-24 hours.
3. Manual menu action: Check for Updates.

## 2.2 Update channels

Optional but useful:

1. stable channel for all users.
2. beta channel for testers.

Each channel points to a dedicated appcast URL.

## 2.3 User experience states

The app should handle these states clearly:

1. No update available.
2. Update available with release notes.
3. Download in progress.
4. Ready to install.
5. Install failed with actionable error.

## 2.4 Security checks

For each update candidate:

1. Read appcast over HTTPS only.
2. Verify EdDSA signature from appcast metadata.
3. Refuse install if signature is invalid.
4. Optionally enforce minimum supported macOS version.

## 2.5 Versioning rules

Use semantic versioning style:

1. Marketing version: 1.0.1, 1.1.0, 2.0.0.
2. Build number monotonic and always increasing.

Do not reuse the same version with different binaries.

## 2.6 Fallback behavior

If any update step fails:

1. Keep current installed app.
2. Show concise error and retry option.
3. Provide direct download fallback URL.

## 3) Site Side Logic (GitHub Pages)

## 3.1 Required files

Recommended static structure:

1. updates/appcast.xml
2. updates/latest.json (optional for website UI)
3. index.html with a latest update section
4. Binary artifacts hosted on GitHub Releases (dmg or zip)

## 3.2 appcast.xml responsibilities

appcast.xml is the source of truth for Sparkle.
For each release item include:

1. version and build.
2. published date.
3. release notes URL.
4. enclosure URL.
5. enclosure length in bytes.
6. enclosure EdDSA signature.

## 3.3 Caching strategy

Use this behavior when possible:

1. appcast.xml: short cache or no-cache.
2. versioned artifacts on GitHub Releases: CDN-managed cache.
3. index.html: standard static-page cache.

On GitHub Pages, CDN cache may delay visibility for a few minutes.

## 3.4 URL strategy

Always use immutable and versioned URLs for artifacts.
Good: https://github.com/MAvizzano/ReeMateApp/releases/download/1.0.1/ReeMateServer-1.0.1.dmg
Avoid: https://github.com/MAvizzano/ReeMateApp/releases/latest/download/ReeMateServer.dmg

## 3.5 Release notes strategy

Keep a latest update section in index.html with:

1. changes summary.
2. fixes.
3. compatibility notes.
4. known issues (if any).

## 4) Release Checklist (for each new version)

1. Build app for release.
2. Sign and notarize app.
3. Produce final artifact (dmg or zip).
4. Generate EdDSA signature for artifact.
5. Upload artifact to GitHub Releases with versioned filename and tag.
6. Update the latest update section in index.html.
7. Update appcast.xml with new item.
8. Update optional latest.json.
9. Validate download URL, signature, and release notes URL.
10. Trigger in-app manual check and confirm update flow.

## 5) Common Mistakes

1. appcast points to latest URL instead of versioned URL.
2. Wrong enclosure length.
3. Wrong EdDSA signature.
4. Reusing an old version number.
5. Publishing appcast before artifact is actually reachable.

## 6) Minimal Governance Rules

1. Keep private signing key outside source control.
2. Keep appcast changes reviewed.
3. Keep index latest update details synchronized with appcast item.
4. Keep one source of truth for current stable version.
