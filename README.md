
# BloodForge Web - Microsoft App Verification Information

This page is provided for Microsoft and Minecraft Services application review.

## Application Summary

BloodForge Web is the account center for BloodForge, a private Minecraft Java Edition multiplayer server.

The application uses Microsoft authentication only to verify ownership of a Minecraft Java Edition profile before allowing a player to bind that Minecraft profile to their BloodForge website account.

## Why Microsoft Authentication Is Needed

BloodForge Web allows players to bind a Minecraft player name to their website account.

For offline-mode players, the server can generate an offline UUID from the player name.

For premium Minecraft Java Edition players, a public Mojang profile lookup can show that a name exists, but it cannot prove that the current website user owns that profile.

Therefore, Microsoft authentication is required so that BloodForge Web can verify that the user signing in is the real owner of the Minecraft Java Edition profile being bound.

This prevents impersonation of premium Minecraft player names.

## User Flow

1. The user logs in to their BloodForge Web account.
2. The user chooses to bind a premium Minecraft account.
3. BloodForge Web starts Microsoft OAuth Device Code Flow.
4. The user opens the Microsoft verification page and signs in with their Microsoft account.
5. BloodForge Web receives a Microsoft access token after the user completes verification.
6. BloodForge Web exchanges the Microsoft token through Xbox Live and XSTS.
7. BloodForge Web calls Minecraft Services to retrieve the authenticated Minecraft Java Edition profile.
8. If Minecraft Services returns a Java Edition profile, BloodForge Web binds the returned Minecraft name and UUID to the user's BloodForge account.

## Minecraft Services Usage

BloodForge Web uses Minecraft Services only for account ownership verification.

The application needs to retrieve:

- Minecraft Java Edition profile name
- Minecraft UUID

The application does not:

- Modify the user's Microsoft account
- Modify the user's Minecraft account
- Access multiplayer sessions
- Access Realms data
- Store Microsoft access tokens
- Store Xbox Live tokens
- Store XSTS tokens

Tokens are used only during the verification request and are discarded after the binding flow completes.

## Data Stored By BloodForge Web

BloodForge Web stores only the data required for website login and Minecraft profile binding:

- Website account email
- Password hash
- Email verification state
- Minecraft profile name
- Minecraft UUID
- Binding metadata
- Player point balance and recharge order records

Microsoft OAuth tokens are not persisted.

## Requested Permission Purpose

The requested Microsoft/Minecraft authentication access is used only to confirm that a BloodForge Web user owns the Minecraft Java Edition profile they want to bind.

The application does not use Microsoft authentication for advertising, tracking, analytics, account resale, automation, or any unrelated purpose.

## OAuth Flow

BloodForge Web primarily uses Microsoft OAuth Device Code Flow:

```text
https://login.microsoftonline.com/consumers/oauth2/v2.0/devicecode
```

Requested scopes:

```text
XboxLive.signin offline_access
```

The application may also keep an authorization-code callback as a fallback:

```text
https://your-domain.example/api/player-profile/microsoft/callback
```

For local development only:

```text
http://localhost:3000/api/player-profile/microsoft/callback
```

## Contact / Brand

Application name:

```text
BloodForge Web
```

Server name:

```text
BloodForge
```

Application type:

```text
Minecraft Java Edition server website and player account center
```

Primary purpose:

```text
Verify Minecraft Java Edition profile ownership before binding a Minecraft profile to a BloodForge website account.
```

