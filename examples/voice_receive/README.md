<img align="right" alt="DiscordGo logo" src="/docs/img/discordgo.svg" width="200">

## DiscordGo Voice Receive Example

This example experiments with receiving voice data from Discord. It joins
a specified voice channel, listens for 10 seconds and saves .ogg files for each
SSRC that it finds in the channel. An exercise left to the reader is to translate
these SSRCs to user IDs; see speaking update events for this information. :)

This example makes heavy use of the [Pion](https://github.com/pion) family of libraries.
Go check them out for anything to do with voice, video or WebRTC; it's a great
group of people maintaining the project!

Voice receive depends on Discord's evolving voice protocol. Native unit tests
and successful compilation do not establish live multi-member interoperability;
verify that separately before relying on recorded output.

**Join [Discord Gophers](https://discord.gg/0f1SbxBZjYoCtNPP)
Discord chat channel for support.**

### Build

Use Go 1.27.3 and build the pinned patched libdave as described in
[the native instructions](../../internal/dave/README.md). Export the documented
CGO include/library and loader paths. This directory is an independent module
that requires the published `github.com/chatanium/discordgo@v0.30.0`, not a local
replacement.

```sh
CGO_ENABLED=1 go build -tags=voice
```

### Usage

Supply the bot token, guild ID, channel ID, and a JSON file containing every
required finite `discordgo.VoiceConfig` bound. Durations are integer nanoseconds;
`VoiceConfig.Validate` rejects absent or invalid bounds.

```sh
./voice_receive -t MY_TOKEN -g 1234123412341234 -c 5678567856785678 -voice-config bounds.json
```

The example writes authenticated Opus packets into Ogg without PCM decoding.
It requires libdave's C++/BoringSSL build closure, but no separate libopus codec.
