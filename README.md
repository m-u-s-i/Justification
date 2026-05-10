# Justification

BloodForge Web is the public website and player account center for the BloodForge Minecraft Java Edition server.

The site provides public server information, class descriptions, game mode documentation, and a logged-in player center for game data, point balance, recharge orders, and Minecraft profile binding.

## Project Purpose

BloodForge Web is used by BloodForge server players to:

- View public server information.
- View class and game mode documentation.
- Register a website account with email verification.
- Log in to the player center.
- View game data from the Minecraft server databases.
- Bind Minecraft player profiles to a BloodForge website account.
- Verify ownership of premium Minecraft Java Edition profiles through Microsoft, Xbox Live, XSTS, and Minecraft Services.

The Microsoft login is used only to prove that the logged-in website user owns the Minecraft Java Edition profile they want to bind. This prevents users from manually entering and claiming another premium player's Minecraft name.

## Install And Run

```bash
npm install
copy .env.example .env
npm run dev
```

Default local URL:

```text
http://localhost:3000
```

## Database Tables

The backend initializes the website database automatically on startup.

Main tables:

- `players`: website account table. Stores email, password hash, email verification time, point balance, point spent, point recharged, and timestamps.
- `player_profiles`: Minecraft profile binding table. Stores Minecraft name, server UUID, raw UUID, official Mojang UUID, account type, server UUID mode, and the current selected profile flag.
- `email_verification_codes`: email verification code records.
- `recharge_orders`: player recharge order records.
- `sessions`: website login sessions.

One website account can bind up to two Minecraft player profiles.

## Game Data

BloodForge Web does not store Minecraft game statistics in the website database.

The player center reads game data from external Minecraft server databases configured through environment variables:

- `DUEL_DB_*`
- `ROUND_DB_*`
- `FUN_DB_*`

Round mode and fun mode support the BloodForgeDataStored plugin schema:

```text
data_<data_key>
```

Each data table contains:

```text
uuid
data_value
```

Game data is queried by the UUID of the currently selected Minecraft profile.

## Microsoft And Minecraft Profile Verification

BloodForge Web uses Microsoft authentication to verify premium Minecraft Java Edition ownership.

This is required because public Mojang profile lookup can confirm that a Minecraft name exists, but it cannot prove that the current website user owns that Minecraft account.

The application verifies ownership by letting the user sign in with Microsoft and then retrieving the authenticated Minecraft Java Edition profile from Minecraft Services.

## Microsoft Verification Flow

BloodForge Web uses Microsoft OAuth Device Code Flow, similar to common Minecraft launchers.

Flow:

1. The user logs in to BloodForge Web.
2. The user chooses to bind a premium Minecraft account.
3. BloodForge Web requests a Microsoft device code.
4. BloodForge Web displays the user code and Microsoft verification URL.
5. The user opens the Microsoft verification page, enters the code, and signs in.
6. BloodForge Web polls the Microsoft token endpoint until the user finishes verification.
7. BloodForge Web exchanges the Microsoft token with Xbox Live.
8. BloodForge Web exchanges the Xbox Live token with XSTS.
9. BloodForge Web calls Minecraft Services `login_with_xbox`.
10. BloodForge Web calls Minecraft Services `minecraft/profile`.
11. If a Minecraft Java Edition profile is returned, BloodForge Web binds the returned Minecraft name and UUID to the user's website account.

Tokens are used only during the verification request. Microsoft access tokens, Xbox tokens, and XSTS tokens are not persisted.

## AppID Review Text

The following text can be used when submitting the Azure AppID for Minecraft Services access review.

### Application Name

```text
BloodForge Web
```

### Application Purpose

```text
BloodForge Web is the account center for a private Minecraft Java Edition server named BloodForge. The application uses Microsoft authentication only to verify that a website user owns the Minecraft Java Edition profile they want to bind to their BloodForge website account. This prevents impersonation of premium Minecraft player names.
```

### Why Minecraft Services Access Is Required

```text
After the player signs in with Microsoft, the server exchanges the Microsoft token through Xbox Live and XSTS, then calls Minecraft Services to retrieve the authenticated Minecraft Java Edition profile. BloodForge Web only needs the Minecraft profile name and UUID to bind the player identity to the website account. The application does not access multiplayer services, does not modify the Microsoft account, and does not store Microsoft access tokens after the binding flow finishes.
```

### Data Stored

```text
BloodForge Web stores the website account email, password hash, Minecraft profile name, Minecraft UUID, account type, binding metadata, point balance, and recharge order records. Microsoft access tokens, Xbox tokens, and XSTS tokens are used only during the verification request and are not persisted.
```

### User-Facing Flow

```text
The user logs into BloodForge Web, chooses "Bind premium Minecraft account", receives a Microsoft device code, opens the Microsoft verification page, signs in, and grants access. After Minecraft Services returns the authenticated Minecraft Java Edition profile, the website binds the returned profile name and UUID to the user's BloodForge account.
```

### Security And Privacy

```text
BloodForge Web uses Microsoft authentication only for identity verification. It does not store Microsoft account passwords or OAuth tokens. It stores only the Minecraft profile name and UUID returned by Minecraft Services, plus website account data required for login and server account management.
```

## Azure App Registration Requirements

For Device Code Flow, the Azure application must allow public client flows.

Required settings:

- Supported account types: personal Microsoft accounts.
- Platform: Mobile and desktop applications.
- Redirect URI: `https://login.microsoftonline.com/common/oauth2/nativeclient`
- Allow public client flows: `Yes`

Legacy authorization-code fallback redirect URI for local development:

```text
http://localhost:3000/api/player-profile/microsoft/callback
```

Production redirect URI example:

```text
https://your-domain.example/api/player-profile/microsoft/callback
```

Device code endpoint:

```text
https://login.microsoftonline.com/consumers/oauth2/v2.0/devicecode
```

Scopes:

```text
XboxLive.signin offline_access
```

If Minecraft Services returns:

```text
Invalid app registration
```

then the Azure AppID has not been accepted for Minecraft Services access. Submit the AppID using the Minecraft App Registration information form:

```text
https://aka.ms/AppRegInfo
```

## Environment Variables

Microsoft settings:

```env
MICROSOFT_CLIENT_ID=
MICROSOFT_CLIENT_SECRET=
MICROSOFT_REDIRECT_URI=http://localhost:3000/api/player-profile/microsoft/callback
MICROSOFT_SCOPE=XboxLive.signin offline_access
MICROSOFT_PROMPT=login
MICROSOFT_AUTHORIZE_URL=https://login.microsoftonline.com/consumers/oauth2/v2.0/authorize
MICROSOFT_TOKEN_URL=https://login.microsoftonline.com/consumers/oauth2/v2.0/token
MICROSOFT_DEVICE_CODE_URL=https://login.microsoftonline.com/consumers/oauth2/v2.0/devicecode
```

UUID mode:

```env
PREMIUM_SERVER_UUID_MODE=offline
```

Use `offline` if the Minecraft server stores data using offline UUIDs.

Use `mojang` if the Minecraft server stores data using official Mojang UUIDs.

