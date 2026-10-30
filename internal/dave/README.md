# Native DAVE boundary

This package is compiled only with `voice && cgo`. It owns ephemeral MLS, sender,
receiver and prepared-transition handles; it contains no Gateway, RTP, UDP, job,
media decoder, persisted identity, or replacement cryptographic implementation.

## Exact native input

- Upstream: official `discord/libdave` release `v1.2.2/cpp`.
- Source commit: `6396c90aedbb9a6ee117ea9ce1ce9832af6a3375`.
- Required source patch: [`tools/native/libdave.patch`](../../tools/native/libdave.patch).
- Patch SHA-256: `4f315a8af3f4b464b846644aeca5054010493241762f8f2ffc85695eb3016511`.
- This is a **source-patched fork**, not an unmodified upstream release artifact.
- Exact dependency pins: [`tools/native/native-lock.json`](../../tools/native/native-lock.json).
- Go module: `github.com/chatanium/discordgo`; no FlexModule checkout or local
  DiscordGo replacement is required.

Pending-self export is a project integration inference from the public protocol
and pinned native state model, not a documented upstream activation API or a
claim about recovered proprietary Discord-client behavior. Interoperability
acceptance remains an explicit integration gate.

The patch adds `dave/flex_dave_ext.h` and two C symbols:

```c
bool flexDaveSessionSetProposalLimits(DAVESessionHandle, size_t, size_t);
bool flexDaveSessionActivatePendingGroup(DAVESessionHandle);
```

The first configures mandatory positive queued-proposal count and serialized-byte
limits in the official parser/queue path. Its outer list is decoded once using
MLSPP's TLS parser, each parsed message is validated and processed normally, and
queue entries retain their exact wire-byte charge. Revoke subtracts the removed
entry's charge; rejected batches discard partial proposal caches and stale
outbound candidates, without discarding the current/pending group or latest join
package. Empty queues generate no synthetic commit. Reset clears the old roster
so the first new-group change map cannot omit a member whose signature key was
retained across the reset.

The second authorizes **pending-self media export only** after a sanctioned
immediate transition zero. It checks supported nonzero version, pending epoch
zero, exactly one basic self credential with the expected signature public key,
expected group/ciphersuite, exactly the expected external sender, and a fresh
join package/private-key pair. It does **not** promote pending state into
`currentState_`, choose a commit, consume the join package, or change the official
exporter, ratchet derivation, cipher, or wire format. Reset/recreation clears its
flag. The Go owner additionally requires current-generation package production,
the recognized participant set to contain only self, and an explicit
`AuthorizePendingReset(ctx, version)` stamp. The Gateway owner may grant that
stamp only after an authenticated opcode-24 epoch-one reset and successful
opcode-26 write for the matching generation/version. Initial Session Description
or invalid-commit recovery alone cannot authorize pending-self activation.
The matching pending opcode-21 transition zero consumes the stamp. Reset, fresh
package replacement, `SetProtocolVersion` (including unchanged versions), control
failure and genuine establishment invalidate it. A duplicate already-active
transition zero is an idempotent no-op, never a ratchet/nonce reinstall. The fresh
join package and private key remain native for a later genuine Commit/Welcome.

An external-sender package can arrive after that generation's fresh key package
was sent. Pinned `SetExternalSender`/`CreatePendingGroup` preserve the generated
join package, private join key and local leaf. Installing the public sender
clears any native-owner authorization stamp, but the Gateway owner can defer or
refresh authorization after successful opcode 25 using its still-unconsumed
matching authenticated-reset/successful-write record. A generated package or a
new external sender alone never creates that record.

## Headers, link and build responsibilities

CGO needs the include root containing both `dave/dave.h` from the pinned source
and the installed patched `dave/flex_dave_ext.h`. It compiles the C callback bridge
as C11 and links `-ldave`. Supply include/library search paths through the native
builder's `CGO_CFLAGS`/`CGO_LDFLAGS` or installed toolchain configuration. No MLSPP
C++ headers are needed by CGO; they are needed by the separate native source
build. Source-build requirements are C++17, CMake >= 3.20, the locked MLSPP/vcpkg
revisions and their matching BoringSSL closure. Do not mix crypto closures or
load an ABI-incompatible prebuilt library. The lock's EL9 builder metadata
describes the original runtime distribution target, not a claim that every
SDK consumer platform has been validated.

Native download/digest/header-symbol/loader checks, target-compatible source
builds, native license/runtime closure, formatter, builds, tests and live Discord
acceptance are integration-owner responsibilities. Source implementation and
fixtures alone are not evidence of playable or received voice.

### Standalone source build

The following Linux/amd64 recipe uses the same source-backed CMake flags and
dependency pins as the original native integration. It is not a prebuilt
library download or a cross-platform packaging guarantee. Install Go 1.27.3,
a C/C++ compiler, CMake >= 3.20, Ninja, Git, make, nasm, Perl, pkg-config,
autoconf, automake, libtool, zip/unzip, tar and gzip first. BoringSSL uses Go
during code generation. Run from this SDK checkout's root:

```sh
SDK_ROOT="$PWD"
WORK="$(mktemp -d)"
git clone https://github.com/discord/libdave.git "$WORK/libdave"
git -C "$WORK/libdave" checkout --detach 6396c90aedbb9a6ee117ea9ce1ce9832af6a3375
printf '%s  %s\n' \
  4f315a8af3f4b464b846644aeca5054010493241762f8f2ffc85695eb3016511 \
  "$SDK_ROOT/tools/native/libdave.patch" | sha256sum -c -
git -C "$WORK/libdave" submodule update --init --depth=1 cpp/vcpkg
test "$(git -C "$WORK/libdave" rev-parse HEAD:cpp/vcpkg)" = \
  16c71a39e5a0fc0bdb3fad03beef8f38ee00ee3b
if [ "$(git -C "$WORK/libdave/cpp/vcpkg" rev-parse --is-shallow-repository)" = true ]; then
  git -C "$WORK/libdave/cpp/vcpkg" fetch --unshallow --no-tags origin \
    16c71a39e5a0fc0bdb3fad03beef8f38ee00ee3b \
    7adc2e4d49e8d0efc07a369079faa6bc3dbb90f3
else
  git -C "$WORK/libdave/cpp/vcpkg" fetch --no-tags origin \
    16c71a39e5a0fc0bdb3fad03beef8f38ee00ee3b \
    7adc2e4d49e8d0efc07a369079faa6bc3dbb90f3
fi
git -C "$WORK/libdave" apply --check "$SDK_ROOT/tools/native/libdave.patch"
git -C "$WORK/libdave" apply "$SDK_ROOT/tools/native/libdave.patch"
"$WORK/libdave/cpp/vcpkg/bootstrap-vcpkg.sh" -disableMetrics
mkdir -p "$WORK/triplets"
cat > "$WORK/triplets/x64-linux-flex.cmake" <<'EOF'
set(VCPKG_TARGET_ARCHITECTURE x64)
set(VCPKG_CRT_LINKAGE dynamic)
set(VCPKG_LIBRARY_LINKAGE static)
set(VCPKG_CMAKE_SYSTEM_NAME Linux)
set(VCPKG_BUILD_TYPE release)
set(VCPKG_C_FLAGS "-fPIC")
set(VCPKG_CXX_FLAGS "-fPIC")
EOF
cmake -S "$WORK/libdave/cpp" -B "$WORK/build" -G Ninja \
  -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=ON -DREQUIRE_BORINGSSL=ON \
  -DPERSISTENT_KEYS=OFF -DTESTING=OFF -DINSTALL_VCPKG_LICENSES=ON \
  -DCMAKE_INSTALL_PREFIX="$WORK/prefix" '-DCMAKE_INSTALL_RPATH=$ORIGIN' \
  -DCMAKE_TOOLCHAIN_FILE="$WORK/libdave/cpp/vcpkg/scripts/buildsystems/vcpkg.cmake" \
  -DVCPKG_MANIFEST_DIR="$WORK/libdave/cpp/vcpkg-alts/boringssl" \
  -DVCPKG_OVERLAY_TRIPLETS="$WORK/triplets" \
  -DVCPKG_TARGET_TRIPLET=x64-linux-flex \
  -DVCPKG_INSTALLED_DIR="$WORK/vcpkg-installed"
cmake --build "$WORK/build" --parallel 2
cmake --install "$WORK/build"
```

The complete vcpkg history is needed for historical port trees, not only its
baseline JSON. Before building, verify the BoringSSL manifest's
`builtin-baseline` and the MLSPP overlay's source commit against
`native-lock.json`. Keep the installed licenses and all runtime dependencies
with any redistributed library.

Set the include path to `$WORK/prefix/include`, and point the linker and loader
to the directory containing the installed `libdave.so` (normally
`$WORK/prefix/lib`). Verify both required extension symbols with `nm -D` and
ensure the target loader can resolve the full library closure before testing:

```sh
export CGO_ENABLED=1
export CGO_CFLAGS="-I$WORK/prefix/include"
export CGO_LDFLAGS="-L$WORK/prefix/lib"
export LD_LIBRARY_PATH="$WORK/prefix/lib${LD_LIBRARY_PATH:+:$LD_LIBRARY_PATH}"
go test -mod=readonly -race -tags=voice ./...
go -C examples/linked_roles test -mod=readonly -race -tags=voice ./...
go -C examples/voice_receive test -mod=readonly -race -tags=voice ./...
```

The two nested modules require the published SDK tag and its checksums even when
these commands are run from a checkout. Ordinary CI does not execute this native
recipe; the integration owner must supply the real C/C++ and loader environment.
The Opus examples forward or record encoded packets and do not use a separate
libopus decoder. No native build, test or live acceptance result is asserted by
these instructions.

## Handle and byte ownership audit

| Native resource | Owner and release |
| --- | --- |
| Session | One `Context`; `daveSessionDestroy` before callback state free. |
| Callback state | C `malloc`-owned atomic flag; C `free` after all admitted calls and native session destruction. No Go callback/pointer is retained. |
| Encryptor | One `Context`; no passthrough; `daveEncryptorDestroy` on close. |
| Prepared self/receiver ratchets | `daveSessionGetKeyRatchet` transfers each new opaque owner to the prepared transition. Destroy after native media installation or discard/close. |
| Installed encrypt/decrypt ratchets | Pinned `bindings_capi.cpp` **copies** borrowed input ratchets through `CopyKeyRatchet`. Destroying the original is correct, not a transferred-owner double free. |
| Decryptors | Recognized sender-specific `Context` map, bounded by configured count; native `TransitionToKeyRatchet` owns its copies and expires old domains using its official ten-second window. |
| Commit/Welcome result | Destroy once after consuming bounded roster deltas. Removed members have an empty signature; this is not a complete roster snapshot. |
| Native returned bytes, roster IDs and signatures | `daveFree`, including rejected oversized outputs. Check size before Go copy; signatures are inspected for removal and freed without Go copying. |
| Frame buffers | Borrowed caller buffers for the duration of a serialized synchronous call only; native maximum-size queries precede writes; no per-frame Go allocation/copy. Failed output is cleared. |

A context lock serializes all native MLS/media operations and close. The only
registered MLS callback is synchronous in the pinned source and writes a C-owned
atomic failure flag, never source/reason text. Asynchronous fingerprint callbacks
are not used. Acquiring the close lock joins admitted calls and their callbacks;
callback memory is freed last. A global C-only discard sink prevents upstream
verbose MLS bytes, key-domain addresses and raw error text from entering logs.
No callback sink memory is freed while libdave could reference it.

Prepared contexts and serialized transition/control memory have explicit finite
bounds. Participant/decryptor counts bound opaque native state; the receiver
ledger also bounds simultaneous retained native key domains, including each
active domain and old domains within the official ten-second window. Sender
changes occur only at Execute Transition or immediate transition zero. A reset
preserves the valid old sender until that boundary. Same-domain transitions do
not reset the encryptor nonce sequence. Unknown/unrecognized senders and failed
native authentication never yield plaintext. The official unauthenticated SFU
silence exception is discarded by this strict authenticated-Opus API.

## API and unexecuted fixtures

`New(Config)` has no defaults; `Config.Validate` checks every required finite
bound. The context-first methods include
`SetParticipants`, `SetProtocolVersion`, `AuthorizePendingReset`,
`PrepareTransition`, `Ready`, `MaxCiphertextSize`, and `MaxPlaintextSize` for the
transport owner. Proposal input excludes its append/revoke byte;
`CommitWelcome.Wire` is the official opaque combined opcode-28 payload. On a
successful call its emptiness is exactly native proposal-queue emptiness; a
nonempty queue synchronously produces its native candidate, while failures
return an error rather than successful empty output. Applied, ignored and failed
commits remain distinct; an ignored result is not a recovery-triggering callback
error and does not revoke a valid pending reset.

`context_test.go` exercises real native fresh packages, pending-only immediate
activation, non-sole rejection, native Opus encryption, sole reset preserving the
old sender until execution, unavailable/plaintext/size boundaries, ignored versus
failed commits, malformed proposal framing, no empty-proposal synthetic commit,
and close racing the native failure callback. `authorization_test.go`
additionally covers generated-only/stale/superseded reset authority, admitted
control-error revocation and ignored-versus-failed semantics. These fixtures
require the patched native library; they have not been run by the source owner.
Integration must also
exercise genuine multi-member Commit/Welcome and receive authentication/replay,
old-epoch in-flight media, actual proposal-count/byte/revoke limits, and an
approved Discord sole-reset-to-later-member-join flow.
