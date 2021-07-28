# publish-build

Publishes a selected build of a specified product to a selected branch. This solution can only be used to publish builds to private branches.

## Windows

```
GOGGalaxyPipelineBuilder.exe publish-build <base_product_id> <build_id> <branch> <optional arguments>
```

!!! Example
    ```
    GOGGalaxyPipelineBuilder.exe publish-build 0123456789 01234567890123456 Staging --branch_password="xyz" --log_level=debug
    ```

## macOS and Linux

```
./GOGGalaxyPipelineBuilder publish-build <base_product_id> <build_id> <branch> <optional arguments>
```

!!! Example
    ```
    ./GOGGalaxyPipelineBuilder.exe publish-build 0123456789 01234567890123456 Staging --branch_password="xyz" --log_level=debug
    ```

## Positional Arguments

| Argument            | Description                                         |
| ------------------- | --------------------------------------------------- |
| `<base_product_id>` | ID of a product that you want to publish a build    |
| `<build_id>`        | ID of a build that you want to publish              |
| `<branch>`          | ID of a branch to which you want to publish a build |

!!! Attention
    For security reasons, you cannot publish to the Master or any other public branch directly from Pipeline Builder.

## Optional Arguments

| Argument                            | Description                                                  |
| ----------------------------------- | ------------------------------------------------------------ |
| `-h`, `--help`                      | Displays help for the `publish-build` command and exits      |
| `--log_level=debug`                 | Enables generating debug logs during command execution; the logs are saved in *C:\Users\<username>\AppData\Local\Temp\__gog\logs* (**Windows**) or *$TMPDIR/__gog/logs* (**macOS**) |
| `--username USERNAME`               | Username used for authentication to the GOG.com server. If not provided, cached credentials will be used. If there are no cached credentials found, you will be prompted to enter them |
| `--password PASSWORD`               | Password used for authentication to the GOG.com server. If not provided, cached credentials will be used. If there are no cached credentials found, you will be prompted to enter them |
| `--version VERSION`                 | Overrides the *versionName* property from the .json file of the project |
| `--branch_password BRANCH_PASSWORD` | Required when you are publishing to a password protected branch. On Windows, the best practice is to embrace the branch password with double quotation marks (`"`). On macOS or Linux, either embrace the branch password with single quotation marks (`'`), or make sure that all special characters, including white-spaces, are escaped properly |
