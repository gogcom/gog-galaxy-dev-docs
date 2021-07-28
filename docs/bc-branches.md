# Branches

There are two types of branches: **Master** and **Beta**.

- **Master** is the default branch for GOG GALAXY used whenever a user initializes an installation in the client (the GOG GALAXY application).
- **Beta** branches can be also accessed in the GOG GALAXY client, but are hidden by default. Beta branches may or may not be protected with a password.

If a Beta branch is not password protected, then it is considered a **Public Beta** branch, and can be selected by any user. Otherwise, it is considered a **Private Beta** branch, and a user must enter a password to access this channel.

Branch passwords can be viewed on the [GOG Developer Portal](https://devportal.gog.com/panel/games) and additional ones can be created on *Branches* tab of the *[Builds & Branches](build-branches.md)* page (accessible after clicking *Builds* button for a particular game).

By default, two password protected beta branches are created: **GOG-use only** and **Staging**.

- **GOG-use only** is a GOG internal branch, and we ask that you refrain from publishing to this branch.
- **Staging** is a branch for you to test the stability of your uploaded build in the GOG GALAXY client without publishing to the Master branch.

!!! Attention
    For security reasons, you cannot publish to the Master or any Public branch directly from Build Creator.

You have full discretion to publish a build to the Master branch at any time from the GOG Developer Portal, but we ask that you conduct a preliminary check to ensure that the upload is functional and downloading correctly; this should be done using the Beta channels in the GOG GALAXY client.

Once a Linux build is uploaded to the Master branch, we will treat it as a Release Candidate and prepare installers for it. If your game is not released yet, don’t worry — we will only make the installers available after the release.