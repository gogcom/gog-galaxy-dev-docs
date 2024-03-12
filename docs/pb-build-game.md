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
| `--log_level=debug`                 | Enables generating debug logs during command execution; the logs are saved in *C:\Users\<username>\AppData\Local\Temp\__gog\logs* (**Windows**) or *$TMPDIR/__gog/logs* (**macOS**) |
| `--username USERNAME`               | Username used for authentication to the GOG.com server. If not provided, cached credentials will be used. If there are no cached credentials found, you will be prompted to enter them |
| `--password PASSWORD`               | Password used for authentication to the GOG.com server. If not provided, cached credentials will be used. If there are no cached credentials found, you will be prompted to enter them |
| `--dump_upload_info`                | Saves the manifest of the upload data for debug purposes (default: `false`) |
| `--version VERSION`                 | Overrides the *versionName* property from the .json file of the project |
| `--output OUTPUT`                   | Intermediate path where the repository will be built into. Particularly useful when creating a build locally, using the `--offline` command; by default, the builds are saved in *C:\Users\<username>\AppData\Local\Temp\__gog\builds* (**Windows**) or *$TMPDIR/__gog/builds* (**macOS**) |
| `--offline`                         | Do not upload the repository to servers, only build the game project (default: `false`) |
| `--silent`                          | Performs extraction without issuing prompts, except for credentials, unless they’re cached. If you have one or more repositories already downloaded in the output path, they will not be removed. Two or more repositories can be downloaded to the same output path. By default, this is disabled (`false`) |
| `--branch BRANCH_NAME`              | Selects a branch for publishing the build. If a branch is password protected, don’t forget to include `--branch_password` argument |
| `--branch_password BRANCH_PASSWORD` | Required when you are publishing to a password protected branch. On Windows, the best practice is to embrace the branch password with double quotation marks (`"`). On macOS or Linux, either embrace the branch password with single quotation marks (`'`), or make sure that all special characters, including white-spaces, are escaped properly |
