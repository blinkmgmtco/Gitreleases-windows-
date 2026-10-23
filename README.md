# OFM Dash Windows releases

Keep this release repository **public** and dedicated to downloadable builds. The desktop app stores the GitHub owner and repository name inside each installer so it can check that repository on every launch. Do not rename, transfer, or delete the repository while installed copies use it. Do not upload the API source, credentials, creator sessions, or local test database.

## Before building a chatter release

1. Confirm the hosted API is deployed and `https://YOUR_API_HOST/health` responds successfully. A Windows installer cannot use the local `127.0.0.1:31337` test API on another PC.
2. Confirm chatter accounts exist in the hosted database, their agency has access, and the hosted API supports this client version. Local test accounts do not transfer automatically.
3. Create a **public** GitHub repository such as `YOUR_ORG/ofmdash-releases`. Enable Releases. Keep the repository identity permanently under your control.
4. For a production release, configure a Windows Authenticode code-signing certificate for electron-builder. The release script refuses to create publishable assets when the installer is unsigned. For a small pilot only, append `--allow-unsigned yes` to the build command; Windows will show an unknown-publisher warning. Never put the private signing key or its password in the release repository.
5. Increase the version in `CRM WORKSPACE/package.json` and its lockfile. Each published release must have a unique, increasing version. Keep the `appId` and product name unchanged so updates retain the user's encrypted sign-in in their Windows profile.

## Build

From `CRM WORKSPACE` on Windows:

```powershell
npm ci
node .\scripts\build-windows-release.cjs --owner YOUR_ORG --repo ofmdash-releases --api-url https://YOUR_API_HOST
```

The builder tests the hosted API, packages the specified URL without changing the local development `config.json`, checks the embedded GitHub update feed and API address, and verifies the installer's SHA-512 against `latest.yml`. It creates `Gitreleases(windows)/vVERSION/` containing:

| File | Purpose |
| --- | --- |
| `ofmdash-Setup-VERSION.exe` | Windows x64 installer and full updater download |
| `ofmdash-Setup-VERSION.exe.blockmap` | Differential update data |
| `latest.yml` | Version, size, filename, and SHA-512 read by the updater |
| `SHA256SUMS.txt` | Human-verifiable download checksums |
| `release-info.json` | Build target and API record |
| `RELEASE_NOTES.md` | Suggested release description |

## Publish a release

1. Create a GitHub Release with tag `vVERSION` in the exact public repository embedded during the build. Publish it as the latest stable release; do not mark it prerelease or leave it as a draft.
2. Upload the **three updater assets together** from `Gitreleases(windows)/vVERSION/`: the EXE, its `.blockmap`, and `latest.yml`. Upload `SHA256SUMS.txt` too so recipients can verify the download. Paste `RELEASE_NOTES.md` into the release description.
3. Download the EXE from that published release on a separate Windows test account/PC. Verify the SHA-256, install, sign in as a chatter, open the needed workspaces, quit, and reopen to confirm sign-in persists.
4. For an updater test, publish a higher-version release with the same GitHub repository, API origin, `appId`, and product name. Launch the older installed app, use its crown update window to download and install, and verify the new version opens signed in. Only then send the first installer link to the chatter group.
5. After the new release works, update the hosted API's minimum client version if an older client must be retired. Do not raise that minimum before the installer and update assets are publicly available.

The existing 1.0.8 installer was built with the **old** generic update feed and local API. It will not automatically discover a new GitHub release. Existing testers must manually run the first GitHub-fed installer once. Their Windows sign-in data remains in the same app user profile.

If you use GitHub CLI after signing in on the release machine, upload from inside `Gitreleases(windows)/vVERSION/` with:

```powershell
gh release create vVERSION .\ofmdash-Setup-VERSION.exe .\ofmdash-Setup-VERSION.exe.blockmap .\latest.yml .\SHA256SUMS.txt --repo YOUR_ORG/ofmdash-releases --title "OFM Dash VERSION" --notes-file .\RELEASE_NOTES.md
```

Run the release command only after reviewing the build record and testing the hosted API. Do not upload `config.json`, `.env.local`, the source tree, or local databases.
