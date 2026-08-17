# manager-account-ai — releases

[![version](https://img.shields.io/github/v/release/BaoDoan-AI/manager-account-ai-releases?label=version&color=22c55e)](https://github.com/BaoDoan-AI/manager-account-ai-releases/releases/latest)
[![platform](https://img.shields.io/badge/platform-Windows-1b2336)](#installing)

Installers for **Manager Account AI**. Nothing else lives here: no source, no
issues, no history worth reading.

The badge above reads the latest release of this repository directly, so it is
never a number someone forgot to bump.

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
| `ManagerAccountAI-Setup-<version>.exe` | the Windows installer |
| `latest.yml` | version + checksum; how an installed app learns a newer one exists |
| `<installer>.exe.blockmap` | lets an update download only the changed parts |

## Installing

Download the `.exe` from [the latest release](https://github.com/BaoDoan-AI/manager-account-ai-releases/releases/latest)
and run it.

It is **not code-signed**, so SmartScreen will say the publisher is unknown:
choose *More info*, then *Run anyway*. It installs for the current user only, so
Windows never asks for an administrator. Uninstalling leaves your accounts in
`%APPDATA%\manager-account-ai`, so reinstalling picks them straight back up.

From 0.2.0 onward the app checks for new versions itself, at startup and from
*Settings › Updates*.

## Changes

[CHANGELOG.md](CHANGELOG.md) — written by the release workflow, one section per
published version.
