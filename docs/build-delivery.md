# Build Delivery

The final build ready to be published on GOG.com is a **Release Candidate**. It is a build that was **already tested by you**, e.g. published on a [Beta or Staging branch](bc-branches.md), then installed on a local machine with the GOG GALAXY client to ensure the build downloads properly and is fully functional. And if you implemented our SDK, please check if its features work — these [testing scenarios](sdk-testing-implementation.md) should help.

!!! Attention "Games Using EasyAntiCheat"
    If your game makes use of EasyAntiCheat, then please let your Product Manager know so that they can put in a request to turn on EAC compatibility features in the [overlay](gc-overlay.md) for your game.

Once this final build passed your internal tests, submit it to us for routine QA checks before the planned release date: just upload this build to the Staging branch (or any pre-release branch of your choice; see [*Build branches*](build-branches.md)). You select a branch in either of our two tools: 

- [**GOG GALAXY Build Creator** ](bc-quick-start.md)— a fully featured graphical (GUI) tool to upload and publish new game builds
- [**GOG GALAXY Pipeline Builder**](pb-quick-start.md) — a command line (CLI) tool recommended for build process automation.

!!! Important
    Please inform your Product Manager of the version number of the build and the branch that it is uploaded to, after your QA is finished and you are comfortable submitting the build as a Release Candidate.

Ideally, you should submit your build at least 3-5 days before the launch of your game. That said, we completely understand the somewhat unpredictable nature of game development, and simply ask that you keep your Product Manager informed of any delays, along with any updated ETAs, should meeting the build delivery deadline not be possible.

