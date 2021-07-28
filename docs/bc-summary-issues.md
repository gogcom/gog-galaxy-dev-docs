# Summary Issues

Build Creator verifies your project state on the fly and displays possible errors here. Every issue displayed is either an error or a warning.

## Solving Issues

**Errors** are strict, logical problems. Please resolve them in order to be able to build and upload your game.

**Warnings** suggest that something is probably wrong in your project, though the raw logic of the project is still valid and you can build the game.

!!! Important
    We cannot verify the correctness of your game content. Please check all of your packages twice before building your game.

When you encounter any issues in the *Summary* window, clicking the issue message should take you to the most probable place to fix it.

| Code |  Type   | Explanation                                                  |
| :--: | :-----: | ------------------------------------------------------------ |
| 1011 |  error  | [Installation Directory](bc-installation-dir.md) property is invalid as a folder name (e.g., is empty or contains forbidden characters) |
| 1024 |  error  | Specific [package](bc-one-package.md) in the base product has no **primary task**. Create [tasks](bc-file-tasks.md) and add them to the corresponding packages |
| 1025 |  error  | Specific [package](bc-one-package.md) in the base product has more than one **primary task**. Go to *Packages* window and deselect unnecessary [tasks](bc-file-tasks.md) |
| 1026 |  error  | Specific [package](bc-one-package.md) in the base product has no [depot](bc-adding-depots.md). Create depots and add them to corresponding packages |
| 1651 | warning | Content of a specific [depot](bc-adding-depots.md) can’t be read or watched by GOG Build Creator. This might affect building your game. Make sure you have access and read permissions to folders specified as depot paths |
| 2212 | warning | [Depot](bc-adding-depots.md) you created is not added to any [package](bc-one-package.md) |
| 2213 | warning | [Task](bc-file-tasks.md) you created is not added to any [package](bc-one-package.md) |
| 2351 |  error  | Specific [package](bc-one-package.md) contains depots with duplicated file(s). There would be a conflict during installation on a user’s machine. Remember that the [installation directory](bc-installation-dir.md) on a user’s machine is common for each [depot](bc-adding-depots.md) you create |
| 2777 |  error  | [Depot](bc-adding-depots.md) Folder attribute refers to a directory that does not exist on your computer. Most commonly you will see this error after importing project prepared on a different machine |
| 4323 |  error  | [File Tasks](bc-file-tasks.md) *Executable* attribute refers to a file that cannot be found within a specific [package](bc-one-package.md). Either you forgot to add the [depot](bc-adding-depots.md) to that package or the *Executable* attribute is wrong |
| 4400 |  error  | The [package](bc-one-package.md) doesn’t contain a [depot](bc-adding-depots.md) that has the *Contents* subfolder. Open *[Files Preview](bc-package-preview.md)* window for that package to review your file structure. See [Installation Directory](bc-installation-dir.md) to learn more. |

