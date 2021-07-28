# download-repository

Downloads the game repository.

## Windows

```
GOGGalaxyPipelineBuilder.exe download-repository <base_product_id> <output> <optional arguments>
```

!!! Example
    ```
    GOGGalaxyPipelineBuilder.exe download-repository 0123456789 c:\downloads\ --branch=staging --branch_password="xyz"
    ```

## macOS and Linux

```
./GOGGalaxyPipelineBuilder download-repository <base_product_id> <output> <optional arguments>
```

!!! Example
    ```
    ./GOGGalaxyPipelineBuilder.exe download-repository 0123456789 ~/Downloads/ --branch=staging --branch_password="xyz"
    ```

## Positional Arguments

| Argument            | Description                                                  |
| ------------------- | ------------------------------------------------------------ |
| `<base_product_id>` | Base productID of the repository to be downloaded            |
| `<output>`          | Path to a directory, where the repository will be downloaded to |

## Optional Arguments

| Argument                            | Description                                                  |
| ----------------------------------- | ------------------------------------------------------------ |
| `-h`, `--help`                      | Displays help for the `build-game` command and exits         |
| `--log_level=debug`                 | Enables generating debug logs during command execution; the logs are saved in *C:\Users\<username>\AppData\Local\Temp\__gog\logs* (**Windows**) or *$TMPDIR/__gog/logs* (**macOS**) |
| `--username USERNAME`               | Username used for authentication to the GOG.com server. If not provided, cached credentials will be used. If there are no cached credentials found, you will be prompted to enter the credentials |
| `--password PASSWORD`               | Password used for authentication to the GOG.com server. If not provided, cached credentials will be used. If there are no cached credentials found, you will be prompted to enter the credentials |
| `--version VERSION`                 | Overrides the *versionName* property from the project .json file |
| `--branch BRANCH_NAME`              | Selects a branch to download from. If a branch is password protected, don’t forget to include `--branch_password` argument |
| `--branch_password BRANCH_PASSWORD` | Required when you are publishing to a password protected branch. On Windows, the best practice is to embrace the branch password with double quotation marks (`"`). On macOS or Linux, either embrace the branch password with single quotation marks (`'`), or make sure that all special characters, including white-spaces, are escaped properly |
| `--build_id BUILD_ID`               | Specifies an identifier of a build to be downloaded. By default, uses the newest build |
| `--dump_build_info`                 | Saves the selected build information file to the output directory (default: `false`) |
| `--platform {windows,osx}`          | Target platform for the game                                 |
| `--silent`                          | Performs extraction without issuing prompts, except for credentials, unless they’re cached. If you have one or more repositories already downloaded in the output path, they will not be removed. Two or more repositories can be downloaded to the same output path. By default, this is disabled (`false`) |