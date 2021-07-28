# download-extract

Downloads and extracts a particular game build. Supports subset of parameters, go with a separate download & extract if you need more.

## Windows

```
GOGGalaxyPipelineBuilder.exe download-extract <base_product_id> <output> <build_id> <optional arguments>
```

!!! Example
    ```
    GOGGalaxyPipelineBuilder.exe download-extract 0123456789 c:\downloads\ 01234567890123456 --username="abcd" --password="xyz"
    ```

## macOS and Linux

```
./GOGGalaxyPipelineBuilder download-extract <base_product_id> <output> <build_id> <optional arguments>
```

!!! Example
    ```
    ./GOGGalaxyPipelineBuilder.exe download-extract 0123456789 ~/Downloads/ 01234567890123456 --username="abcd" --password="xyz"
    ```

## Positional Arguments

| Argument            | Description                                                  |
| ------------------- | ------------------------------------------------------------ |
| `<base_product_id>` | Base productID of the repository to be downloaded            |
| `<output>`          | Path to the directory where the repository will be downloaded |
| `<build_id>`        | ID of a build to be downloaded                               |

## Optional Arguments

| Argument              | Description                                                  |
| --------------------- | ------------------------------------------------------------ |
| `-h`, `--help`        | Displays help for the `build-game` command and exits         |
| `--log_level=debug`   | Enables generating debug logs during command execution; the logs are saved in *C:\Users\<username>\AppData\Local\Temp\__gog\logs* (**Windows**) or *$TMPDIR/__gog/logs* (**macOS**) |
| `--username USERNAME` | Username used for authentication to the GOG.com server. If not provided, cached credentials will be used. If there are no cached credentials found, you will be prompted to enter them |
| `--password PASSWORD` | Password used for authentication to the GOG.com server. If not provided, cached credentials will be used. If there are no cached credentials found, you will be prompted to enter them |
| `--version VERSION`   | Overrides the *versionName* property from the .json file of the project |