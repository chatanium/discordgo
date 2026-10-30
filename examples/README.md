<img align="right" src="../docs/img/discordgo.svg">

# DiscordGo Examples

These examples demonstrate how to utilize DiscordGo.

Please explore the individual folders and give them a try!

Use Go 1.27.3. Most examples belong to the SDK root module; `linked_roles` and
`voice_receive` are separate modules requiring `github.com/chatanium/discordgo`
at `v0.30.0`, with no local replacement. Run their checks from each directory;
root `go test ./...` does not include nested modules.

`airhorn` and `voice_receive` require a `voice`-tagged CGO build and the patched
native library described in [the native instructions](../internal/dave/README.md).
They exchange already-encoded Opus packets; they do not decode PCM or link a
separate libopus codec. Building libdave still requires its pinned C++/BoringSSL
dependency closure. Ordinary CI checks do not exercise native voice; use the
real patched library and `-tags=voice` for native integration checks, rather
than disabling CGO to make voice examples appear to pass.

**Join [Discord Gophers](https://discord.gg/0f1SbxBZjYoCtNPP)
Discord chat channel for support.**

