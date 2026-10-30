<img align="right" alt="DiscordGo logo" src="/docs/img/discordgo.svg" width="200">

## DiscordGo Airhorn Example

This example demonstrates how to utilize DiscordGo to listen for an !airhorn
command in a channel and then play a sound to that user's current voice channel.

**Join [Discord Gophers](https://discord.gg/0f1SbxBZjYoCtNPP)
Discord chat channel for support.**

### Build

Use Go 1.27.3 and build the pinned patched libdave as described in
[the native instructions](../../internal/dave/README.md). Export the documented
CGO include/library and loader paths, then run this from the example directory:

```sh
CGO_ENABLED=1 go build -tags=voice
```

### Usage

```
Usage of ./airhorn:
  -t string
        Bot Token
  -voice-config string
        Required JSON file with explicit discordgo.VoiceConfig bounds
```

The below example shows how to start the bot from the airhorn example folder.

```sh
./airhorn -t YOUR_BOT_TOKEN -voice-config bounds.json
```

`bounds.json` must provide every required finite `discordgo.VoiceConfig` bound;
durations are encoded as integer nanoseconds and `VoiceConfig.Validate` rejects
missing or invalid values. Keep `airhorn.dca` in the working directory. This
example sends existing Opus packets and does not need a separate libopus decoder.

### Creating sounds

Airhorn bot uses [DCA](https://github.com/bwmarrin/dca) files, which are 
pre-computed files that are easy to send to Discord.


See the below example of creating a DCA file from a MP3 file.  This also works
with WAV, FLAC, and many other file formats. Of course, you will need to [install](https://github.com/bwmarrin/dca/tree/master/cmd/dca#Getting-Started)
FFmpeg and the DCA CLI first.

```sh
ffmpeg -i test.mp3 -f s16le -ar 48000 -ac 2 pipe:1 | dca > test.dca
```
