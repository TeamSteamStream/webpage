This document holds instructions on how the webpage is built and how to make changes to it.

# Webpage structure

The webpage is based on [Hugo](https://gohugo.io/).
This framwork generates static HTML pages from indata,
and updates crosslinks and main page when new content is added.

The webpage is stored in two github repositories.

TeamSteamStream/webpage
: contains the source code

TeamSteamStream/TeamSteamStream.github.io
: contains the rendered webpage as pure html


# Getting started

1. Get access to the TeamSteamStream organization.
Make sure you have a public key associated with your account.
2. Clone down the repository
3. The webpage depends on an external theme, download it by running `git submodule init` and `git submodule update` in the root of the directory
3. Make sure you have Hugo installed.
The version in the normal apt repositoro is not updated, but recent packages
can be found here: https://github.com/gohugoio/hugo/releases
To install, download the .deb file and run `sudo apt install ./hugo_versionNumber.deb`
4. Run `hugo server` from the root directory of the webpage repository.
This will start a local webserver that you can access to watch how changes look before you push them online.

# Creating content

Create a new markdown file in the correct subfolder of the `contents` directory.
As of now, there are three categories:

news
: for general news items

notes
: meeting notes, for technical reference

writeups
: solutions from CTF competitions

# Pushing content online

TODO
