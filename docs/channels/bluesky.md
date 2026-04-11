---
summary: "Bluesky DM channel via AT Protocol chat API"
read_when:
  - You want OpenClaw to receive DMs on Bluesky
  - You're setting up a Bluesky bot account
title: "Bluesky"
---

# Bluesky

**Status:** Optional bundled plugin (disabled by default until configured).

Bluesky is a decentralized social platform built on the AT Protocol. This channel enables OpenClaw to receive and respond to direct messages via the `chat.bsky.*` lexicons.

## Bundled plugin

Current OpenClaw releases ship Bluesky as a bundled plugin, so normal packaged builds do not need a separate install.

### Older/custom installs

- Onboarding (`openclaw onboard`) and `openclaw channels add` still surface Bluesky from the shared channel catalog.
- If your build excludes bundled Bluesky, install it manually:

```bash
openclaw plugins install @openclaw/bluesky
```

Restart the Gateway after installing or enabling plugins.

## Quick setup

### Interactive (recommended)

```bash
openclaw configure bluesky
```

The wizard prompts for your handle and app password, detects existing env vars, and writes the config. You can also reach it via `openclaw onboard` when Bluesky has not been configured yet.

### Environment variables

```bash
export BLUESKY_HANDLE="yourbot.bsky.social"
export BLUESKY_APP_PASSWORD="xxxx-xxxx-xxxx-xxxx"
```

### Config file

```json
{
  "channels": {
    "bluesky": {
      "handle": "yourbot.bsky.social",
      "appPassword": "xxxx-xxxx-xxxx-xxxx"
    }
  }
}
```

Then bind the channel to an agent and restart the Gateway:

```json
{
  "bindings": [{ "agentId": "your-agent", "channel": "bluesky" }]
}
```

## App Password

Bluesky App Passwords are separate credentials scoped to third-party apps:

1. Go to **Settings → Privacy and Security → App Passwords**
2. Click **Add App Password**, give it a name
3. Copy the generated password — it is shown only once

**Never use your main account password.** App Passwords can be revoked individually from Bluesky settings without affecting your account.

## Configuration reference

| Key           | Type    | Default                 | Description                                     |
| ------------- | ------- | ----------------------- | ----------------------------------------------- |
| `handle`      | string  | `$BLUESKY_HANDLE`       | Bot account handle (e.g. `yourbot.bsky.social`) |
| `appPassword` | string  | `$BLUESKY_APP_PASSWORD` | App password (supports SecretRef)               |
| `pdsUrl`      | string  | `https://bsky.social`   | Personal Data Server URL (self-hosted PDS only) |
| `enabled`     | boolean | `true`                  | Enable or disable the channel                   |
| `accounts`    | object  | —                       | Named accounts for multi-account setups         |

## Multiple accounts

Use `accounts` to run separate bots for different agents from the same Gateway:

```json
{
  "channels": {
    "bluesky": {
      "accounts": {
        "sales-bot": {
          "handle": "sales.bsky.social",
          "appPassword": "xxxx-xxxx-xxxx-xxxx"
        },
        "support-bot": {
          "handle": "support.bsky.social",
          "appPassword": "yyyy-yyyy-yyyy-yyyy"
        }
      }
    }
  },
  "bindings": [
    { "agentId": "sales-agent", "channel": "bluesky", "accountId": "sales-bot" },
    { "agentId": "support-agent", "channel": "bluesky", "accountId": "support-bot" }
  ]
}
```

## Self-hosted PDS

If your account is on a self-hosted Personal Data Server, set `pdsUrl`:

```json
{
  "channels": {
    "bluesky": {
      "handle": "yourbot.example.com",
      "appPassword": "xxxx-xxxx-xxxx-xxxx",
      "pdsUrl": "https://pds.example.com"
    }
  }
}
```

## How it works

### Authentication

The plugin authenticates with the Bluesky PDS using your handle and app password to create an XRPC session. Chat API calls use short-lived service auth tokens scoped to each `chat.bsky.*` lexicon (tokens are cached for 25 minutes and refreshed automatically). A `401` response evicts the cached token and retries the request once.

### Polling

The plugin uses an adaptive polling loop:

- **2 seconds** after a new message arrives (to catch quick replies)
- Backs off by **1.5×** per idle cycle up to a maximum of **90 seconds**
- Resets to 2 seconds immediately on any new activity

### Slash commands

Inbound messages starting with `/` are forwarded to the agent as slash commands. Any DM sender can issue slash commands — Bluesky has no server-side allowlist system so the default policy is open.

## Troubleshooting

### `Bluesky: not configured` in `openclaw doctor`

- Check that `BLUESKY_HANDLE` and `BLUESKY_APP_PASSWORD` are set, or that a named account is configured under `channels.bluesky.accounts`
- When using named accounts without top-level credentials, ensure the account is bound to an agent in `bindings`

### Bot not receiving messages

1. Verify the app password is correct — generate a new one if unsure
2. Confirm the handle includes the domain (e.g. `yourbot.bsky.social`, not just `yourbot`)
3. Check that `enabled` is not `false`
4. Verify the channel is bound to an agent in `bindings`
5. Check Gateway logs for login errors

### `401` errors at startup

The app password may have been revoked. Generate a new one from Bluesky Settings → Privacy and Security → App Passwords.

Custom PDS users: verify `pdsUrl` is correct and reachable.

## Security

- Use App Passwords — never your main Bluesky account password
- App Passwords can be revoked individually without affecting your account
- Credentials are never logged
- Use environment variables or SecretRef config values; avoid committing credentials to config files

## Limitations

- Direct messages only (no group chats or public posts)
- No media attachments
- No server-side allowlist; any DM sender can reach the agent

## Related

- [Channels Overview](/channels) — all supported channels
- [Pairing](/channels/pairing) — DM authentication and pairing flow
- [Channel Routing](/channels/channel-routing) — session routing for messages
- [Security](/gateway/security) — access model and hardening
