# Papaya
This repository contains various assets which are used by the [Papaya Games Launcher](https://play.google.com/store/apps/details?id=com.sebschaef.papaya) Android app:
- Translations
- Icons for the various supported gaming platforms
- Configuration for the various gaming platforms, supported emulators and game streaming apps

You are more than welcome to contribute to any of these assets :)

To contribute, you can either directly open a Pull Request with your additions or you can send me them somewhere else and I'll add them to this Repository.

## Translations
How to add translations for another Locale:
1. Copy the translations file of any other existing Locale (`default.xml` = `en`)
2. Name the file according to your Locale's country code ([See here](https://saimana.com/list-of-country-locale-code/), but with a `-` instead of `_`). E.g. `pt` = Portugese (any country), `pt-BR` = Brazilian Portugese.
3. Translate the texts:
	- Translate only `<string "...">[this part]</string>` and do not modify the `name` etc
	- Translations marked with `translatable="false"` do not need to be translated. You can remove the whole line then.
	- `%1$s`, `%d` etc are placeholders for texts which are filled at runtime (`%1$s games` -> `PSP games`). Please keep them and place them where it makes sense in your language.
    - Please note that certain characters need to be escaped: `'` → `\'`, `&` → `&amp;`, `<` → `&lt;`, `>` → `&gt;` etc.

If you don't know how to use Git, you can also download a translation file, translate it and send it to me somewhere else. Also, don't hesistate to reach out to me if you need more information to contribute with translations. Thank you for contributing! :)
- [mail@sebschaef.com](mailto:mail@sebschaef.com)
- [Discord](https://discord.gg/bv9HTAepTP)
- [Reddit](https://www.reddit.com/user/IllegalArgException/)

## Icons
Any new icons should match the current style:
- Monochrome
- Relatively simple, not too detailled
- They should be recognizable even if they are displayed in a small size

The art style: 
- The current icons usually show the controller for the platform or the device itself if it is a Handheld
- Usually, the icon resembles the shape with cutouts where the screen is located and where the analog sticks are located. Consider this just as a guideline, because for older consoles without screens and analog sticks this scheme does not work.

## Config - Presets
The `presets.json` file is split into three parts:
- `platforms`: Gaming platforms. E.g. PlayStation Portable, Android etc.
- `emulatorApps`: The supported Emulators. E.g. PPSSPP, Dolphin etc.
- `streamingApps`: The supported Apps for the Game Streaming section.

See the sections below for more details about each part.

### Platforms
`platforms` is a list of objects, where each object refers to a gaming platform.

#### Platform
Each platform defintion contains some Metadata like the name of the gaming platform. Also, it contains detailled information of the supported game file types ("Roms") and references to their respective IDs on various Scraper APIs.

- **`key` (Required):** The internal identifier for this platform. Needs to be unique and should be at max 4 characters long (Preferred: 3).
- **`shortName` (Required):** Abbreviation of the platform which is visible to the user. Should be uppercase and at max 4 characters long (Preferred: 3).
- **`name` (Required):** Full name of the platform which is visible to the user. Should be according to this scheme: "\[Manufacturer\] \[Name\] (\[Alternative name\])"
- **`supportedFiles` (Required):** List of supported file types. See below.
- **`scraperPlatformId`:** Obkject with references to IDs of this platforms on the various Scraper APIs. See below.

Example:
```json
{
    "key": "psp",
    "shortName": "PSP",
    "name": "Sony PlayStation Portable",
    "supportedFiles": [...],
    "scraperPlatformId": ...
}
```

##### SupportedFile
Defines rules for a file type.

- **`type` (Required):** The file type (Aka the letters behind the dot). Should be lowercase for consistency, but the case is ignored by Papaya.
- **`nonDescriptiveNames`:** A list of file names (without the type) which don't hold any useful information for the scrapers. If such a name is matched, Papaya considers this file as a game, but will try to get of the search term for scraping from somwhere else (See samples below)
- **`ignoredNames`:** A list of file names (without the type). If such a name is matched, Papaya does not consider this file as a game file -> The file is ignored.
- **`idBy`:** If set, the value must be either `content` or `folder`. If this is set, Papaya considers a matching file not as a Rom, but as a definition of a Game ID. Depending on the set value, Papaya tries to extract Game ID either from the file `content` or the `folder` name in which the file lies (See samples below).

**Examples:**

A simple Rom file
```json
{
    "type": "iso"
}
```

A Rom file with occasionally not useful names. For example, the standard executable file for PSP games had the name `EBOOT.PBP` which is why they often might be found like this: `Driver 76/EBOOT.PBP`. With the definition below, we tell Papaya that the file name is not useful and we will try to get the game name from the folder name instead.
```json
{
    "type": "pbp"
    "nonDescriptiveNames": [
        "eboot",
        "param"
    ]
}
```

For very common file types, we might want to ignore a file if it has a certain name. For example, for PC games on GameHub Lite we can define `txt` as a valid Game ID type where we can find the ID inside the file and extract the game name from the file name (e.g. `GTA IV.txt`). However, the user might use the same folder with other Frontends like ES-DE, so we would like to ignore the `systeminfo.txt` which could be located in the same folder.
```json
{
    "type": "txt",
    "ignoredNames": [
        "systeminfo"
    ],
    "idBy": "content"
}
```

##### ScraperPlatformId
An optional object which contains the IDs of the platform on various scraper platforms. All fields are optional. If an ID for a scraper is set, Papaya saves an extra network call to get the platform by name: Less load for the API, shorter scraping time for the user.

- **`igdb`:** The ID of the platform on the "Internet Games Database".
- **`tgdb`:** The ID of the platform on the "The Games DB".
- **`rawg`:** The ID of the platform on Rawg.

### EmulatorApps
`emulatorApps` is a list of objects which refer to an Emulator.

#### Emulator
Each Emulator definition contains some Metdata like the name of the App. More importantly, it references the platforms it can emulate and holds information about how game can be launched in it.

- **`packageName` (Required):** The package name of the Emulator App. This is the unique identifier of an Android App. Is used to launch the specific App and to display the information that it is installed on the device.
- **`name` (Required):** The name of the App(s) which is visible to the user in the Emulator selection screen. Note: Although the package name is a unique identifier, often there are a few forks of the same Emulator which share the same package name: Lime3DS/Azahar or Winlator/Winlator CMOD etc.
- **`supportedPlatforms` (Required):** The list of platforms which can be emulated in this App. Each entry refers to the unique platform `key`s. The list can not be empty and the referenced `key`s must exist (See `platforms` list).
- **`core`:** Some Apps can emulate multiple platforms where each emulated platform is targeted through different launch parameters. E.g. Retroarch, GameHub Lite. This field is a secondary attribute to differentiate Emulators inside the App. See samples below.
- **`flag`:** Currently supports only one flag: `deprecated`. This can be used to remove the Emulator from Papaya's on-device database: If the entry does not exist in the database, it is not inserted. If it exists and is referenced by a game from the user, it is kept in the database. If it exists and is not referenced, it is removed from the database. If set, the `name` should be modified to "\[App name\] (Deprecated)", so the user knows that this configuration is outdated and to be removed.
- **`launchConfiguration`:** An object which defines how a game can be launched in this App. If not set, the default launch method is used: Explictly launching the App with the `packageName`, with the action `android.intent.action.VIEW` and passing the Rom's `Content URI` in the `data` parameter. More details, see below.

> [!IMPORTANT]  
> The uniqueness of an Emulator is defined by the combination of the `packageName`, the `supportedPlatforms` and the `core`. It is not possible to define multiple entries where these fields are identical.

**Examples**

Simple Emulator App definition where a game is started with the default launch functionality: xplictly launching the App with the `packageName`, with the action `android.intent.action.VIEW` and passing the Rom's `Content URI` in the `data` parameter.
```json
{
    "packageName": "info.cemu.cemu",
    "name": "Cemu",
    "supportedPlatforms": [
        "wiiu"
    ]
}
```

For Apps which can emualate multiple platforms and where each platform needs to be started differently, we can differentiate them with the `core` attribute:
```json
{
    "packageName": "gamehub.lite",
    "name": "GameHub Lite - Steam game",
    "core": "steam",
    "supportedPlatforms": [
        "win"
    ],
    "launchConfiguration": {...},
},
{
    "packageName": "gamehub.lite",
    "name": "GameHub Lite - Local game",
    "core": "local",
    "supportedPlatforms": [
        "win"
    ],
    "launchConfiguration": {...}
}
```

##### LaunchConfiguration
An object which contains the definition on how a game is launched in an EmulatorApp

- **`action`:** The `Intent` action (See Android documentation). If not set, the default action `android.intent.action.VIEW` is used.
- **`explicitActivity`:** Some Apps do not support handling the action in their default launcher `Activity`. In this case, we want to send the action to a specific `Activity` which can handle the game launch. If not set, only the action is passed to the App with no specific `Activity` as a target.
- **`fileParameterName`:** Some Apps require the file path to be passed through a specific parameter. If set, the file path will be passed through this parameter. If not set, it will be passed through the `data` parameter (standard).
- **`fileParameterType`:** The name is misleading: Is only used in combination with the `idParameterName` and defines how the parameter is passed: `string` or `int` are allowed. If not set, `string` is used.
- **`filePathType`:** The scheme of the passed file path. Possible values are `contentUri` or `absolute`. On modern Android, a file is referenced through a `contentUri` which could be an abstract ID which refers to a specific file or folder and most Apps will be able to handle this. Older Apps and Apps with full access to the storage however might refer to a file by an `absolute` file path (e.g. `storage/emulated/0/games/...`). If set to `absolute`, Papaya tries to construct such an old fashioned file path and sends this to the App instead. If not set, `contentUri` is used.
- **`launchById`:** Can be `true` or `false`. Default is `false`. If enabled, the game will be launched through an ID instead of a file path to a Rom file. This requires however that the platform has at least on ID file type (See "Platform" section above).
- **`idParameterName`:** Is currently set in combination with `launchById`. Similar to `fileParameterName` this will cause to send the ID through the specified parameter. If not set, but `launchById` is set, the ID is passed through the `data` parameter.
- **`additionalParameters`:** Contains a list of addtional parameters. Each parameter object requires the `key` to be set and at least one value: `booleanValue`, `stringValue`, `intValue`.

**Examples**

PPSSPP expects a specific Activity, but otherwise uses the standard way to open files: action `android.intent.action.VIEW` and passing the Content URI to the file in the `data` parameter:
```json
{
    "explicitActivity": "org.ppsspp.ppsspp.PpssppActivity"
}
```

"Pizza Boy C Pro" expects the file path in the custom `rom_uri` parameter for a specific `Activity`. The path is expected as a Content URI.
```json
{
    "explicitActivity": "it.dbtecno.pizzaboypro.MainActivity",
    "fileParameterName": "rom_uri"
}
```

ScummVM expects an unusual action, a specific `Activity` and wants to be launched with a game ID instead of file path:
```json
{
    "action": "android.intent.action.MAIN",
    "explicitActivity": "org.scummvm.scummvm.SplashActivity",
    "launchById": true
}
```

GameHub Lite expects a custom action, specific `Activity`, a game ID and even additional parameters:
```json
{
    "action": "gamehub.lite.LAUNCH_GAME",
    "explicitActivity": "com.xj.landscape.launcher.ui.gamedetail.GameDetailActivity",
    "idParameterName": "localGameId",
    "launchById": true,
    "additionalParameters": [
        {
            "key": "autoStartGame",
            "booleanValue": true
        }
    ]
}
```


### StreamingApps
`streamingApps` is a list of objects which refer to Apps in which a user can launch game streams.

#### StreamingApp
Contains the `packageName` as the unique reference to App and an optional platform reference for the scraping function.

- **`packageName` (Required):** The package name of the Streaming App. This is the unique identifier of an Android App. If the App with this `packageName` is installed on the device, it will automatically appear in the "Streaming" category.
- **`supportedPlatforms`:** Contains references to the platforms the streamed games are running on. The list references the unique `keys` of the platforms in the `platforms` list. A platform with that `key` must exist. The platform is used for finding better matches for game covers when scraping.

> [!IMPORTANT]  
> Although `supportedPlatforms` is a list, only one platform reference is supported at the moment. Only the first reference is considered.

**Examples**

Mark "Artemis" as a game streaming App and specify that the scraper should search in Windows games:
```json
{
    "packageName": "com.limelight.noir",
    "supportedPlatforms": [
        "win"
    ]
}
```

Mark "PXPlay" as a game streaming App. No platform is specified, because the games can be streamed from both a PlayStation 4 and PlayStation 5, but currently we only support one platform reference. If no reference is set, the scraper searches on all platforms.
```json
{
    "packageName": "psplay.grill.com"
}
```
