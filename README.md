# Chatanium DiscordGo

[![Go Reference](https://pkg.go.dev/badge/github.com/chatanium/discordgo.svg)](https://pkg.go.dev/github.com/chatanium/discordgo) [![Go Report Card](https://goreportcard.com/badge/github.com/chatanium/discordgo)](https://goreportcard.com/report/github.com/chatanium/discordgo) [![CI](https://github.com/chatanium/discordgo/actions/workflows/ci.yml/badge.svg)](https://github.com/chatanium/discordgo/actions/workflows/ci.yml)

<img align="right" alt="DiscordGo logo" src="docs/img/discordgo.svg" width="400">

DiscordGo is a [Go](https://golang.org/) package that provides low level 
bindings to the [Discord](https://discord.com/) chat client API. DiscordGo 
has nearly complete support for all of the Discord API endpoints, websocket
interface, and voice interface.

This independently versioned fork is based on
[bwmarrin/discordgo](https://github.com/bwmarrin/discordgo) commit
`f43dd94faaacd5b163e9e783f14b5bd8be639fc9` (after upstream `v0.29.0`).
Its first independent release is `v0.30.0`; upstream tags are not releases of
this module identity. The BSD license and upstream history are retained.

The imported Chatanium patch set adds context-aware Gateway dispatch, bounded
ingress and observer hooks, partial-message/state handling, and the native DAVE
voice transport. Voice uses explicit configuration and authenticated Opus
packets; upstream voice helper packages are not drop-in replacements for this API.
Source migration alone is not evidence of live Discord interoperability.

* See [dca](https://github.com/bwmarrin/dca) for an **experimental** stand alone
tool that wraps `ffmpeg` to create opus encoded audio appropriate for use with
Discord (and DiscordGo).

**For help with this package or general Go discussion, please join the [Discord 
Gophers](https://discord.gg/golang) chat server.**

## Getting Started

### Installing

Use Go 1.27.3 (the CI toolchain). From your application's Go module, require
the canonical fork directly; no `replace` directive or local SDK checkout is
needed:

```sh
go get github.com/chatanium/discordgo@v0.30.0
```

### Usage

Import the package into your project.

```go
import "github.com/chatanium/discordgo"
```

Construct a new Discord client which can be used to access the variety of 
Discord API functions and to set callback functions for Discord events.

```go
discord, err := discordgo.New("Bot " + "authentication token")
```

See Documentation and Examples below for more detailed information.

### Native voice

The default build does not load native voice code. Building with `-tags=voice`
requires CGO, a C/C++ toolchain, and the pinned, source-patched `libdave` library.
See [the native build instructions](internal/dave/README.md) for the exact source,
patch, headers, linker and loader configuration. An unpatched upstream binary is
not ABI-compatible with the required extension symbols.

### Repository checks

CI owns SDK tests and examples here, including the independent
`examples/linked_roles` and `examples/voice_receive` Go modules. CI runs ordinary
builds, race tests and vet; it does not claim native voice coverage. Native
voice checks require the real patched library and remain integration-owner
responsibilities, as described in the native instructions. Unit checks do not
require bot credentials or establish live Discord acceptance. Nested examples
consume the published fork version, not a local replacement.


## Documentation

**NOTICE**: This library and the Discord API are unfinished.
Because of that there may be major changes to library in the future.

The DiscordGo code is fairly well documented at this point and is currently
the only documentation available. Go reference (below) presents that information in a nice format.

- [Go reference](https://pkg.go.dev/github.com/chatanium/discordgo)
- [Getting started](docs/GettingStarted.md)


## Examples

Below is a list of examples and other projects using DiscordGo.  Please submit 
an issue if you would like your project added or removed from this list. 

- [DiscordGo Examples](examples) - Example programs maintained with this fork
- [Awesome DiscordGo](https://github.com/bwmarrin/discordgo/wiki/Awesome-DiscordGo) - A curated list of high quality projects using DiscordGo

## Troubleshooting
For help with common problems please reference the 
[Troubleshooting](https://github.com/bwmarrin/discordgo/wiki/Troubleshooting) 
section of the project wiki.


## Contributing
Contributions are very welcomed, however please follow the below guidelines.

- First open an issue describing the bug or enhancement so it can be
discussed.  
- Try to match current naming conventions as closely as possible.  
- This package is intended to be a low level direct mapping of the Discord API, 
so please avoid adding enhancements outside of that scope without first 
discussing it.
- Create a Pull Request with your changes against the master branch.


## List of Discord APIs

See [this chart](https://abal.moe/Discord/Libraries.html) for a feature 
comparison and list of other Discord API libraries.

## Special Thanks

[Chris Rhodes](https://github.com/iopred) - For the DiscordGo logo and tons of PRs.
