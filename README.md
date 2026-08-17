# manager-account-ai — releases

Installers for **Claude Account Manager**. Nothing else lives here: no source, no
issues, no history worth reading.

## Why a separate repository

The application's source repository is private. Everything GitHub serves for a
private repo — the releases API, the asset download URL, `latest.yml` — answers
`404` without an `Authorization` header, so an app updating itself from there
would have to carry a token, handed to every machine it is installed on.

Publishing the artifacts somewhere public lets the app check for a new version
anonymously, and the source stays private.

## What each release contains

| File | Role |
|------|------|
| `ClaudeAccountManager-Setup-<version>.exe` | the Windows installer |
| `latest.yml` | version + checksum; how an installed app learns a newer one exists |
| `<installer>.exe.blockmap` | lets an update download only the changed parts |

## Installing

Download the `.exe` from [Releases](../../releases) and run it.

It is **not code-signed**, so SmartScreen will say the publisher is unknown:
choose *More info*, then *Run anyway*. It installs for the current user only, so
Windows never asks for an administrator. Uninstalling leaves your accounts in
`%APPDATA%\manager-account-ai`, so reinstalling picks them straight back up.

From 0.2.0 onward the app checks for new versions itself, at startup and from
*Settings › Updates*.
