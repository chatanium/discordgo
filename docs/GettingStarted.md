# Getting Started

This page is dedicated to helping you get started on your way to making the
next great Discord bot or client with DiscordGo. Once you've done that please
don't forget to submit it to the 
[Awesome DiscordGo](https://github.com/bwmarrin/discordgo/wiki/Awesome-DiscordGo) list :).


**First, lets cover a few topics so you can make the best choices on how to 
move forward from here.**

#### Bot Application
A bot application is a special program that interacts with the Discord servers
to perform some form of automation or provide some type of service.  Examples 
are things like number trivia games, music streaming, channel moderation, 
sending reminders, playing loud airhorn sounds, comic generators, YouTube 
integration, Twitch integration... You're *almost* only limited by your imagination.

Bot applications require the use of a special Bot account.  These accounts are
tied to your personal user account. Bot accounts cannot login with the normal
user clients and they cannot join servers the same way a user does. They do not 
have access to some user client specific features however they gain access to
many Bot specific features.

To create a new bot account first create yourself a normal user account on 
Discord then visit the [My Applications](https://discord.com/developers/applications/me)
page and click on the **New Application** box.  Follow the prompts from there
to finish creating your account.


**More information about Bot vs Client accounts can be found [here](https://discord.com/developers/docs/topics/oauth2#bot-vs-user-accounts).**

# Requirements

This fork requires Go 1.27.0 or newer and uses Go 1.27.3 in CI.
Ordinary Gateway and REST builds do not require a native voice library.
Voice builds additionally require CGO and the pinned patched libdave described
in [Native DAVE boundary](../internal/dave/README.md); platform compatibility
must be verified with that native toolchain.

You must already have a working Go environment setup to use DiscordGo.  If you 
are new to Go and have not yet installed and tested it on your computer then 
please visit [this page](https://golang.org/doc/install) first then I highly
recommend you walk though [A Tour of Go](https://tour.golang.org/welcome/1) to
help get your familiar with the Go language.  Also checkout the relevant Go plugin 
for your editor &mdash; they are hugely helpful when developing Go code.

* Vim &mdash; [vim-go](https://github.com/fatih/vim-go)
* Sublime &mdash; [GoSublime](https://github.com/DisposaBoy/GoSublime)
* Atom &mdash; [go-plus](https://atom.io/packages/go-plus)
* Visual Studio &mdash; [vscode-go](https://github.com/Microsoft/vscode-go)


# Install DiscordGo

Use the canonical fork module from your application's module directory:

```sh
go get github.com/chatanium/discordgo@v0.30.0
```

Import it as `github.com/chatanium/discordgo`. Do not add a DiscordGo `replace`
directive or depend on a copy inside another repository. Go resolves the tagged
SDK and its pinned dependencies through the usual module mechanisms.

For native voice, build the patched library first, configure its include and
library paths as documented above, and compile your application with
`CGO_ENABLED=1 go build -tags=voice`. Default builds expose voice as unavailable
rather than substituting an unencrypted implementation.


# Next...
More coming soon.
