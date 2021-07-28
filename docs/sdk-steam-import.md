# Importing Achievements Using the Steam VDF File

To generate the file containing achievement information, including localized data, please do the following:

1. Go to https://partner.steamgames.com/.

2. Log in to the system.

3. Go to *Apps & Packages→All Applications*.

    ![Steam VDF File 1](_assets/sdk-steam-vdf-1.jpg)

4. Select the *Steamworks Admin* option for your game.

5. Go to *Misc→View Raw Settings* option.

    ![Steam VDF File 2](_assets/sdk-steam-vdf-2.jpg)

6. In the *Raw Configuration Data* section, you’ll see an option to *Choose app info section*: select *Stats* from the drop-down menu.

    ![Steam VDF File 3](_assets/sdk-steam-vdf-3.jpg)

7. Copy all the presented information to a text editing program, such as Notepad or Notepad++, and save the data as a .txt file.

    !!! Important
        Make sure to encode your .txt or .vdf as UTF-8 when saving; this ensures special characters, and characters not featured in the English language are correctly saved and imported.

8. With the .txt file ready, log in to the [GOG Developer Portal](https://devportal.gog.com/) and click *Games* in the main menu.

9. On the resulting [Games](developer-portal.md#games-screen-product-buttons) screen, click the *Galaxy Features* button for the game you want to import Steam achievements to, then select Achievements from the displayed menu. The [Achievements](achievements.md) screen will appear.

10. Follow the steps outlined in [*Adding Achievements Imported From Steam*](achievements.md#adding-achievements-imported-from-steam) to import the data from the .txt file to the Developer Portal.

