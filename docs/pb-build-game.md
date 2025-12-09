# build-game

Builds the game repository from the game data using the *project.json* file as a description and uploads the build to the GOG.com servers.

## Windows

```
GOGGalaxyPipelineBuilder.exe <project_json_file> <optional arguments>
```

!!! Example
    ```
    GOGGalaxyPipelineBuilder.exe build-game "c:\example\project.json" --log_level=debug
    ```

## macOS and Linux

```
./GOGGalaxyPipelineBuilder <project_json_file> <optional arguments>
```

!!! Example
    ```
    ./GOGGalaxyPipelineBuilder.exe build-game project.json --log_level=debug
    ```

## Positional Arguments

| Argument              | Description                                                  |
| --------------------- | ------------------------------------------------------------ |
| `<project_json_file>` | Path to the *project.json* file (including the file itself) of the game |

## Optional Arguments

| Argument                            | Description                                                  |
| ----------------------------------- | ------------------------------------------------------------ |
| `-h`, `--help`                      | Displays help for the `build-game` command and exits         |
| `--ignore_list`                     | Path to a text file that consists of newline separated list of files that should be ignored during the process of game building. File names in the text file can use wildcards. For example, *.info will cause the builder to omit files with the .info extension |
| `--log_level=debug`                 | Enables generating debug logs during command execution; the logs are saved in  `C:\Users\<username>\AppData\Local\Temp\__gog\logs` (**Windows**) or  `$TMPDIR/__gog/logs` (**macOS**) |
| `--username USERNAME`               | Username used for authentication to the GOG.com server. If not provided, cached credentials will be used. If there are no cached credentials found, you will be prompted to enter them |
| `--password PASSWORD`               | Password used for authentication to the GOG.com server. If not provided, cached credentials will be used. If there are no cached credentials found, you will be prompted to enter them |
| `--dump_upload_info`                | Saves the manifest of the upload data for debug purposes (default: `false`) |
| `--version VERSION`                 | Overrides the *versionName* property from the .json file of the project |
| `--output OUTPUT`                   | Intermediate path where the repository will be built into. Particularly useful when creating a build locally, using the `--offline` command; by default, the builds are saved in  `C:\Users\<username>\AppData\Local\Temp\__gog\builds\` (**Windows**) or  `$TMPDIR/__gog/builds` (**macOS**) |
| `--offline`                         | Do not upload the repository to servers, only build the game project (default: `false`) |
| `--silent`                          | Performs extraction without issuing prompts, except for credentials, unless they’re cached. If you have one or more repositories already downloaded in the output path, they will not be removed. Two or more repositories can be downloaded to the same output path. By default, this is disabled (`false`) |
| `--branch BRANCH_NAME`              | Selects a branch for publishing the build. If a branch is password protected, don’t forget to include `--branch_password` argument |
| `--branch_password BRANCH_PASSWORD` | Required when you are publishing to a password protected branch. On Windows, the best practice is to embrace the branch password with double quotation marks (`"`). On macOS or Linux, either embrace the branch password with single quotation marks (`'`), or make sure that all special characters, including white-spaces, are escaped properly |

## Project example

When creating project for the first time, we strongly recommend to create one using [Build Creator](https://docs.gog.com/bc-quick-start/).
But, feel free to check out this example of a *project.json*. Keep in mind that not every option, dependency is going to be used by your game.
This example also contains things that are unavailable in the Build Creator such as: **setupSteps**.

There are two examples of setupSteps in the example below:

- **Configuring Save Path** (see: *saveFolders* from the example) - location where game saves are being stored. Whole content from this directory will be ereased if user selects: "Also remove my local saves and user data" during game uninstall.
- **Configuring registry keys** (see: *registryKey1*) - location where game registry keys are being stored. Whole subkeys will be deleted after game uninstall
!!! Example
    ```
    {
    "project": {
        "baseProductId": "",
        "clientId": "<ClientID_Here>",
        "clientSecret": "",
        "installDirectory": "Cypers Demo Game",
        "name": "Cypers Demo Game",
        "platform": "windows",
        "version": "",
        "scriptInterpreter": true,
        "dependencies": [
            "DOSBox074",
            "nGlide",
            "language_setup",
            "dotNet35",
            "dotNet45",
            "dotNet4",
            "dotNet46",
            "dotNet47",
            "DirectX",
            "openAL",
            "MSVC2005",
            "MSVC2005_x64",
            "MSVC2008",
            "MSVC2008_x64",
            "MSVC2010",
            "MSVC2010_x64",
            "MSVC2012",
            "MSVC2012_x64",
            "MSVC2013",
            "MSVC2013_x64",
            "MSVC2015",
            "MSVC2015_x64",
            "MSVC2017",
            "MSVC2017_x64",
            "XNA",
            "XNA_40",
            "ScummVM",
            "PHYSX",
            "PHYSXLEGACY",
            "UE4REDIST"
        ],
        "products": [
            {
                "depots": [
                    {
                        "folder": ".../languages_p_en-US/__game",
                        "languages": [
                            "en-US",
                            "pl-PL",
                            "de-DE",
                            "ja-JP",
                            "es-ES",
                            "pt-BR",
                            "ru-RU",
                            "ko-KR",
                            "zh-Hant",
                            "fr-FR",
                            "it-IT",
                            "en-US",
                            "es-MX",
                            "cs-CZ",
                            "zh-Hans"
                        ],
                        "name": "TestGame-1.0-win"
                    }
                ],
                "name": "Cypers Demo Game",
                "productId": "<ProductID_Here>",
                "setupSteps": [
            {
                "name": "saveFolders",
                "install": {
                    "action": "savePath",
                    "arguments": {
                        "type": "folder",
                        "savePath": "{userappdata}/../LocalLow/Something/Something"
                    }
                }
            },
            {
                "name": "registryKey1",
                "install": {
                    "action": "setRegistry",
                    "arguments": {
                        "root": "HKEY_CURRENT_USER",
                        "subkey": "Software\\Something\\Something",
                        "deleteSubkeys": true,
                        "saveGameData": true
                    }
                }
            }
        ],
                "tasks": [
                    {
                        "category": "game",
                        "isPrimary": true,
                        "languages": [
                            "en-US"
                        ],
                        "name": "Cypers Renpy Demo Game",
                        "osBitness": [
                            "64"
                        ],
                        "path": "TestGame.exe",
                        "type": "FileTask"
                    },
                    {
                        "category": "other",
                        "languages": [
                            "en-US"
                        ],
                        "name": "32-bit",
                        "path": "TestGame-32.exe",
                        "type": "FileTask"
                    },
                    {
                        "languages": [
                            "*"
                        ],
                        "name": "Support",
                        "type": "URLTask",
                        "url": "https://docs.gog.com/",
                        "category": "document"
                    },
                    {
                        "name": "Cypers Renpy Demo Game",
                        "osBitness": [
                            "64"
                        ],
                        "path": "launcher.exe",
                        "type": "FileTask"
                    },
                    {
                        "category": "game",
                        "isHidden": true,
                        "languages": [
                            "pl-PL",
                            "de-DE",
                            "ja-JP",
                            "es-ES",
                            "pt-BR",
                            "ru-RU",
                            "ko-KR",
                            "zh-Hant",
                            "fr-FR",
                            "it-IT",
                            "en-US",
                            "es-MX",
                            "cs-CZ",
                            "zh-Hans"
                        ],
                        "name": "Cypers Renpy Demo Game - launcher process TestGame.exe",
                        "osBitness": [
                            "64"
                        ],
                        "path": "TestGame.exe",
                        "type": "FileTask"
                    }
                ]
            }
        ]
    }}
    ```