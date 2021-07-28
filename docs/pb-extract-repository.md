# extract-repository

Extracts the game repository with all languages and products.

### Windows

```
GOGGalaxyPipelineBuilder.exe extract-repository <repository_path> <output> <optional arguments>
```

!!! Example
    ```
    GOGGalaxyPipelineBuilder.exe extract-repository c:\gamerepo\ c:\gamerepo\extracted\ --version="1.02" --log_level=debug
    ```

## macOS and Linux

```
./GOGGalaxyPipelineBuilder extract-repository <repository_path> <output> <optional arguments>
```

!!! Example
    ```
    ./GOGGalaxyPipelineBuilder.exe extract-repository ~/gamerepo/ ~/gamerepo/extracted/ --version="1.02" --log_level=debug
    ```

## Positional Arguments

| Argument            | Description                                                  |
| ------------------- | ------------------------------------------------------------ |
| `<repository_path>` | Path to a directory containing the game repository           |
| `<output>`          | Path to a directory, where the game files will be extracted to |

## Optional Arguments

| Argument              | Description                                                  |
| --------------------- | ------------------------------------------------------------ |
| `-h`, `--help`        | Displays help for the `publish-build` command and exits      |
| `--log_level=debug`   | Enables generating debug logs during command execution; the logs are saved in *C:\Users\<username>\AppData\Local\Temp\__gog\logs* (**Windows**) or *$TMPDIR/__gog/logs* (**macOS**) |
| `--version VERSION`   | Overrides the *versionName* property from the .json file of the project |
| `--build_id BUILD_ID` | Extracts the game with a given build ID                      |
| `--silent`            | Performs extraction without asking questions. By default, this is disabled (`false`) |