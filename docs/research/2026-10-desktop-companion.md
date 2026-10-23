# OpenDAP Desktop as the companion: library, sync and providers

- **Date:** 2026-10-23
- **Status:** Research report. Nothing here is decided and no technology is chosen; it is input for DESK requirements to be agreed with the maintainers.
- **Question:** what should OpenDAP Desktop do so that, together with the T1, the listener has the best complete experience? Desktop is for the administrative work: organising the library, buying music, advanced configuration, looking after devices. The T1 is for listening and operating audio. The question covers how the two share one library, and how optional, proprietary providers fit in without ever being required.
- **Companion documents:**
  - the [ecosystem experience research](https://github.com/opendap-project/t1/blob/main/docs/research/ux/2026-10-ecosystem-experience.md) in `t1`: principles E1 to E20, personas, jobs, journeys and the responsibility matrix this report details for the desktop;
  - the [protocol research](https://github.com/opendap-project/system/blob/main/docs/research/2026-10-device-ecosystem-protocol.md) in `system`: how devices meet and talk.
- **Inputs:** DESK-001 to DESK-009, TK-009, the [CLI spec](https://github.com/opendap-project/toolkit-cli/blob/main/docs/specs/opendap-cli.md), the [audio profiles spec](https://github.com/opendap-project/t1/blob/main/docs/specs/audio-profiles.md).
- **Source labels:** **[doc]** official documentation; **[review]** press or reviewer; **[user]** a forum report, not a measured fact. **UNVERIFIED** marks what was not checked against a primary source.

## 1. Summary

1. **The desktop keeps; the T1 plays.** The desktop is where the collection is curated, acquired, backed up and configured in depth. The T1 is complete on its own and never needs the desktop to play (E1).
2. **One library model, shared.** The desktop and the CLI share one core and one domain model (DESK-004), and the T1 holds a replica of the same model. A playlist, a rating, a play, a profile is the same object everywhere.
3. **Files are the library.** The listener's audio files, in folders they choose, are the source of truth. The desktop's database indexes and enriches them, and writes tags only when the listener asks. Purchased files are imported like any other, with their provenance.
4. **Identity is open.** Items are keyed by open identifiers found in the files' tags or added by the listener (a MusicBrainz ID, an ISRC, a UPC). Lookups against MusicBrainz, AcoustID or any other service are providers the listener enables, never the core (SYS-045). Provider identifiers are kept only as links. Playlists hold the identifiers, so they survive files moving and providers leaving.
5. **Providers are optional adapters.** Each one declares what it can do: search, hand off to its checkout, download, list purchases, metadata, scrobbling, playlist import. The listener enables the ones they want; none is a default and none is privileged. With no provider at all, nothing is lost.
6. **Two-way sync with no subscription and no destructive default.**
   - Selections go to the T1 by rule, always with a preview.
   - Plays, ratings and playlists come back as timestamped events.
   - Conflicts are kept and shown.
   - Deletions are recoverable.
   - Nothing about it is a paid feature.
7. **Fast with any library.** A slow desktop manager that loads the whole library before it can be used is the most common complaint about vendor desktop apps. Bounded memory and time (SYS-033, T1-046) apply to the desktop too.

## 2. Benchmark

### 2.1 What makers do

| Product | Model | Sync | Users praise | Users complain |
|---|---|---|---|---|
| **Apple** (iTunes, then Finder and Music) | Desktop library; device as a subset | Whole library or selections; "Automatically fill free space with songs"; USB or Wi-Fi [doc] ([Apple](https://support.apple.com/en-au/guide/mac-help/mchlbf6a1fab/11.0/mac)). On iPods, ratings and play counts came back on connection [user] ([Apple Discussions](https://discussions.apple.com/thread/7527920)) | One place for everything; fill free space; smart playlists fed by plays | Deleting on the Mac removes from the device [doc]; connecting to another library offers "Erase and Sync" [user] ([Apple Discussions](https://discussions.apple.com/thread/255700966)); cloud matching in 2015 changed artwork and metadata and deleted playlists [review] ([Kirkville](https://kirkville.com/why-itunes-12-2-changed-metadata-artwork-and-icloud-status-for-files-in-some-users-itunes-libraries/)); play-count sync now needs a subscription [user] ([MacRumors](https://forums.macrumors.com/threads/music-app-syncing-playcount-issues.2470106/)) |
| **Sony** Music Center for PC, Content Transfer | Desktop transfer tool | Songs not yet on the device or updated since; optional auto-sync on connect (UNVERIFIED excerpt) | — | "So slow its unusable" [user] ([Sony community](https://community.sony.es/t5/pcs-accessories/music-centre-for-pc-too-slow-to-use/td-p/3826961)); the Mac tool broke with macOS 10.15 [user] ([Sony community](https://community.sony.nl/t5/portable-audio/apple-mac-compatibility-with-content-transfer/m-p/2674673)) |
| **Astell&Kern** AK File Drop, AK Connect | DAP as an FTP server; phone app over UPnP | Drag and drop over the LAN [review] ([Moon Audio](https://moon-audio.com/how-to-astell-kern-file-drop)) | No cable | Default `admin`/`admin` credentials [review] |
| **FiiO** FiiO Link, FiiO Control | DAP as server, phone as client over Bluetooth or Wi-Fi [doc] ([FiiO FAQ](https://www.fiio.com/newsinfo/891504.html)) | Wi-Fi transfer from the phone app [doc] ([FiiO](https://fiio.com/newsinfo/626107.html)) | Phone as a remote with library browsing | iPhone limited to Wi-Fi [doc] |
| **HiBy** HiByLink | Phone remote over Bluetooth or LAN; the player plays its own files [doc] ([HiBy](https://guide.hiby.com/en/docs/guides/hibylink/hibylink)) | Browser upload on the LAN | HiBy states the power and speed trade-off of each link [doc] | "A wired transfer is quite faster" [review] ([Headfonia](https://www.headfonia.com/hiby-r6-iii-review/2)) |
| **Shanling** Eddict | Phone remote; settings for USB DACs [doc] ([Shanling](https://en.shanling.com/article-EddictPlayer.html)) | — | — | Device support differs by platform [doc] |

**Pattern:** every maker keeps files and the database on the player, and the phone is a thin remote. None offers a desktop library that syncs ratings and play counts both ways.

### 2.2 Library managers

| Tool | Strength to learn from | Weakness to avoid |
|---|---|---|
| **MusicBee** | Copies only missing or newer files; conversion "only for lossless" on the fly; ratings and play counts synced [review] ([Well-Tempered Computer](https://thewelltemperedcomputer.com/SW/Players/MusicBee/MusicBee_Sync.htm)) | Windows only |
| **foobar2000** | Tagging, ReplayGain, converter; Onewaysync exports playlists and files to a device [doc] ([foo_ows](https://foobar2000.org/components/view/foo_ows)) | Power lies in components |
| **beets** | Import matches against MusicBrainz; copy, move, in place or read-only modes [doc] ([beets](https://docs.beets.io/en/stable/guides/main.html)); `beets-alternatives` keeps a query-defined device collection with add, remove, move and refresh [doc] ([GitHub](https://github.com/geigerzaehler/beets-alternatives)) | Command line only |
| **MusicBrainz Picard** | AcoustID fingerprints, cover art, naming scripts [doc] ([Flathub](https://flathub.org/apps/org.musicbrainz.Picard)) | Tags only, no device |
| **Roon** | One search across local files and services; rich metadata; versions grouped, primary by quality [doc] ([Roon](https://help.roonlabs.com/portal/en/kb/articles/architecture)) | Albums bound to the service they came from; playlists break when a version is withdrawn [user] ([Roon community](https://community.roonlabs.com/t/playlist-tracks-unavailable-after-tidal-to-qobuz-switch-ref-374bt5/322688)); artwork lost offline in ARC [user] ([Roon community](https://community.roonlabs.com/t/arc-offline-mode-removes-album-art/322626)) |
| **Plex, Plexamp** | Server-side analysis | 24 h of downloads per playlist [user/staff] ([Plex forum](https://forums.plex.tv/t/plexamp-full-offline-controls-and-support/575495.md)); TIDAL integration ended in 2024 leaving unplayable entries [user] ([Plex forum](https://forums.plex.tv/t/tidal-integration-ending/886456)) |
| **Navidrome** | Multi-user with per-user plays and playlists, smart playlists, M3U auto-import, scrobbling [doc] ([Navidrome](https://www.navidrome.org/docs/overview/)) | Offline depends on each client |
| **Music Assistant** | Provider mappings per item; matching by ISRC or MusicBrainz ID first, then title, artist and length; cross-provider playlists in its own store [doc] ([usage](https://www.music-assistant.io/usage)) | Cross-provider playlists live only in its database |
| **MediaMonkey** | Two-way sync with its Android app since 2014 [doc] ([Wikipedia](https://en.wikipedia.org/wiki/MediaMonkey)) | Unlimited Wi-Fi sync is paid [review] ([Liliputing](https://liliputing.com/?p=63714)) |
| **Rockbox** | Runtime data (plays, rating, last played) exportable to text because a new database format cannot read the old one; a playback log with dedupe rules [doc] ([tagcache](https://raw.githubusercontent.com/Rockbox/rockbox/master/manual/rockbox_interface/tagcache.tex), [scrobbler](https://raw.githubusercontent.com/Rockbox/rockbox/master/manual/plugins/lastfm_scrobbler.tex)) | — |

### 2.3 Moving files

| Way | Limit | Source |
|---|---|---|
| Mass storage | The host owns the filesystem while mounted | [kernel](https://docs.kernel.org/usb/mass-storage.html) [doc] |
| MTP | One application at a time; no macOS client; Android File Transfer hidden in 2024, with a 4 GB file limit and frequent disconnections | [9to5Google](https://9to5google.com/2024/02/16/android-file-transfer-mac-hidden/) [review], [OpenMTP](https://cdn.jsdelivr.net/gh/ganeshrvel/openmtp@master/README.md) [doc] |
| Syncthing | Keeps both versions of a conflict, but does not notify the listener | [Syncthing](https://docs.syncthing.net/users/syncing.html) [doc], [forum](https://forum.syncthing.net/t/questions-about-sync-conflict-with-receive-only-folder/13544) [user] |
| rsync to FAT | Re-copies everything (2 s timestamps) without `--modify-window` | [Guy Rutenberg](https://www.guyrutenberg.com/2017/01/23/rsync-and-fat-file-systems/) [review] |

### 2.4 Stores and services

| Finding | Source |
|---|---|
| DRM-free stores (Qobuz, Bandcamp, NativeDSD, Presto, HDtracks, ProStudioMasters, iTunes, Amazon) offer **no open purchase or entitlement API** to third-party clients; Qobuz's API is partner-only, Bandcamp's covers labels and fulfilment only | [Bandcamp developer](https://bandcamp.com/developer) [doc]; Qobuz app IDs revoked [user] ([moOde](https://moodeaudio.org/forum/showthread.php?tid=2208)) |
| Re-downloads are **not guaranteed**: Qobuz recommends keeping a backup; Apple says purchases may become unavailable | [Qobuz](https://help.qobuz.com/en/articles/10166-can-i-re-download-my-purchases) [doc], [Apple](https://support.apple.com/en-us/120552) [doc] |
| NativeDSD's downloader runs on Linux | [NativeDSD](https://help.nativedsd.com/en/articles/93762-downloading-and-troubleshooting) [doc] |
| Subscription offline copies stay inside each service's own app or SDK; streaming downloads cannot be handed to third-party apps | [Qobuz](https://help.qobuz.com/en/articles/10133-is-it-possible-to-listen-to-music-without-internet-connection) [doc], [Roon](https://blog.roonlabs.com/offline-listening-in-roon-arc/) [doc] |
| Spotify restricted its Web API for new apps (2024) and reserved extended access for established services (2025) | [Spotify](https://developer.spotify.com/blog/2024-11-27-changes-to-the-web-api) [doc], [Spotify](https://developer.spotify.com/blog/2025-04-15-updating-the-criteria-for-web-api-extended-access) [doc] |
| MusicBrainz core data is CC0, supplementary data non-commercial; AcoustID and Last.fm free for non-commercial use; ListenBrainz listens published as CC0 | [MusicBrainz](https://musicbrainz.org/doc/About/Data_License), [AcoustID](https://acoustid.org/webservice), [Last.fm](https://www.last.fm/api/tos), [ListenBrainz](https://listenbrainz.readthedocs.io/en/latest/users/listenbrainz-dumps.html) [doc] |
| One recording can carry several ISRCs (remixes, some remasters), so ISRC alone is not a safe key | [IFPI](https://isrc.ifpi.org/faq) [doc] |
| XSPF and JSPF separate a track's identifier from its location, so a playlist can be resolved against any source | [XSPF](https://www.xspf.org/spec), [JSPF](https://musicbrainz.org/doc/jspf) [doc] |
| Affiliate income is not plannable: Apple pays a one-time bounty for Apple Music sign-ups and nothing on downloads; no programme found for most stores | [Apple Performance Partners](https://performance-partners.apple.com/commissions-payments) [doc] |

## 3. The library

### 3.1 Model

| Object | What it is | Identity | Lives |
|---|---|---|---|
| **File** | An audio file in a folder the listener chose | Content hash and path | On disk, the listener's |
| **Recording, release, work, artist** | What the file is | Identifiers from tags (MusicBrainz ID; ISRC and UPC as secondary); a fingerprint if a provider adds one | In the library model |
| **Edition** | A version of a release (remaster, format, resolution) | Release ID; grouping by release group | In the model; the listener chooses the primary |
| **Provenance** | Where a file came from | Provider, order, date, original format | Metadata on the file entry |
| **Playlist** | An ordered list of entries | Its own ID; entries hold identifiers plus title, artist and length | Owner's state; exported as JSPF and M3U8 |
| **Event** | A play, a skip, a rating change, a bookmark | Device, time, item | Owner's state, append-only, merged |
| **Profile** | An audio profile (audio profiles spec) | Its own ID; optional headphone and output | Owner's state |
| **Device selection** | What a device carries | Rules per device | Owner's state |

The listener's **personal state** (playlists, events, profiles, selections) is separate from the **collection** (files and what they are). That makes the household case (one collection, several people, ecosystem research §7.11) possible from the start.

### 3.2 Identification

A ladder, as Music Assistant and Roon show, with a manual fallback at the end. The core uses only what is in the files and the listener's own data. Every lookup against an outside service is an opt-in provider (SYS-045):

1. A MusicBrainz ID in the file's tags (core).
2. An ISRC or UPC in the tags, checked against the length (core).
3. Fuzzy title, artist, album and length within the library (core).
4. Lookups through enabled metadata providers: MusicBrainz, an AcoustID fingerprint, Discogs, a store's metadata (providers).
5. A manual link by the listener (core).

Uncertain matches are queued for review, never applied silently (E6).

### 3.3 Curation

- **A health view:** missing or low-resolution art, inconsistent tags, duplicates, several editions, unidentified files, files not on any device, playlists with unresolved entries.
- **Batch fixes with a preview** of every file and tag that would change, and an undo per batch (E6, E7). The writing modes follow beets: write tags in place, or keep files untouched and hold changes in the library only. The listener chooses the mode.
- **Never automatic:** reorganising folders, renaming files and replacing art happen only on request.

### 3.4 Playlists across providers

- Entries are identifiers, resolved when played: local files first, then any enabled provider that can play them, in an order the listener sets.
- An entry that cannot be resolved stays in place, greyed, with the reason ("not in your library; available from…"). It never vanishes; this avoids the Roon and Plex failures in §2.2.
- Smart playlists are rules over the model (added, played, rated, tags), evaluated on every device that has the data.
- Import from M3U8, XSPF, JSPF and service export files; export to JSPF (with identifiers) and M3U8 (paths, for compatibility).

## 4. Device sync

### 4.1 What travels

| Rule | Example | From |
|---|---|---|
| Everything | The whole library, if it fits | Apple |
| Selections | Artists, albums, playlists | Apple, Sony, MusicBee |
| Smart rules | Recently added, most played, rated four or more | Apple's smart playlists; beets queries |
| Fill the free space | The rest of the card, by a rule the listener chooses | Apple |
| Space options | Optional transcoding **per device**, never changing the library; off by default; lossless kept unless the listener chooses | iTunes, MusicBee, beets-alternatives |

Every change is shown before it is applied (what will be added, removed, moved, and the space it needs). Removing something from a selection removes it from the device, not from the library, and says so.

### 4.2 What comes back

| State | How | Why |
|---|---|---|
| Plays and skips | Timestamped events, merged; each keeps the time it happened | The Rockbox and Finamp pattern; Jellyfin's stamping all offline plays at reconnection is the failure |
| Ratings, loves | The latest change wins, per item, with its time; both values kept in history | No consumer tool documents a per-field rule (UNVERIFIED for iTunes); this makes it explicit |
| Playlists | Merged per entry; a real conflict (the same playlist reordered on both sides offline) keeps both versions and is shown once | E9 |
| Bookmarks, resume points | The latest per item wins | T1-051 |
| Profiles | Merged by profile; edits on both sides keep both versions | E9 |
| New files on the T1 | Recordings, HAT captures, files copied by other means come back into the library if the listener wants | Two directions on every channel |

### 4.3 When and how

- **Without a "sync" button.** When the T1 and the desktop meet (cable, LAN, a relay the owner chose), sync starts in the background. Heavy transfers wait for charging or idle on the T1 (ecosystem research §12.3). "Sync now" exists as the visible alternative.
- **Resumable and verified.** Transfers resume after interruption and are checked by content, not by timestamp (FAT's two-second granularity).
- **State in one line, detail one step down:** "On the T1 · synced 2 min ago over USB", then what arrived, what is waiting, what failed and why (E8).
- **No version lock.** An older T1 and a newer desktop agree on what both support and disable the rest (SYS-040). Sony's "keep both current or they may not connect" (UNVERIFIED excerpt) is the failure to avoid.

## 5. Providers

### 5.1 The model

A provider is an optional adapter behind a port of the application core (the [Rust crate architecture](https://github.com/opendap-project/system/blob/main/docs/specs/rust-crate-architecture.md)). It declares what it can do, and the core never assumes any provider exists.

| Capability | What it means | Realistic today |
|---|---|---|
| `catalog.search` | Search releases and recordings, with identifiers when available | Public catalogue APIs and metadata services (MusicBrainz, Discogs); some stores to partners only |
| `store.checkout` | Hand off to the provider's own checkout (its site or flow); the provider takes the payment | Every store, through its site |
| `store.download` | Fetch DRM-free files in the chosen format and resolution | Stores' own download pages or tools; few APIs |
| `store.purchases` | List what the listener owns there, re-download | Only where the provider offers an API (partner terms, UNVERIFIED for most) |
| `stream.play` | Play full tracks from the provider | Only under partner terms |
| `metadata.enrich` | Tags, art, credits | MusicBrainz, Cover Art Archive, Discogs, provider metadata |
| `history.scrobble` | Send plays to a listening history service | ListenBrainz (open data), Last.fm (non-commercial terms) |
| `playlist.import` | Bring playlists from a service | Export files always; APIs where allowed |

### 5.2 Rules

1. **Optional and isolated.** A provider is a separate, optional package (SYS-001 capabilities on the desktop's side). Any proprietary SDK stays out of process and out of the core, as Clementine's out-of-process Spotify library did ([Arch bug](https://bugs.archlinux.org/task/27750)).
2. **Purchases become files.** Everything bought lands as files in the library with provenance. A provider leaving, a store withdrawing a title or a region changing never touches the listener's copy (E5).
3. **No default, no privilege.** The listener enables providers; results from several providers appear side by side, ordered by the listener's criteria (format, price, provider), never by commercial arrangement. Any commercial relationship with a provider is disclosed where it applies (E18).
4. **Back up what was bought.** Since re-downloads are not guaranteed, purchases are included in the desktop's backups by default.
5. **The first step needs no contract.** A provider that only searches, hands off to checkout and imports the downloaded files works with any store's site, as Roon imports Qobuz purchases. Deeper integration (purchases list, streaming) depends on each provider's terms.
6. **Every third party is a provider** (SYS-045): metadata services (MusicBrainz, AcoustID, Discogs), stores, streaming and scrobbling are all opt-in providers, each following its service's terms. The core works with none.

## 6. Everything else the desktop does

| Area | What it does | Why on the desktop |
|---|---|---|
| **Audio profiles in depth** | Curves, targets, measurements (AutoEq, own), PEQ, tone macros, crossfeed; A/B on the T1 with the desktop as remote; share as files | Graphs and fine editing need a large screen (E3); the T1 keeps choosing and comparing |
| **Device management** | Pairing and revocation, updates staged for the T1, backups with versions, restore to a new device, diagnostics and logs the owner reads and chooses to send | Trust-sensitive work needs attention and a large screen |
| **Work files** | A drop folder that reaches the T1 at once, outside the library (the sound engineer, ecosystem research §7.7) | Speed |
| **Remote and handoff** | Control the T1; "continue on the T1" and back | Complementary and continuous (E4) |
| **Engineering** | Measurement sessions with the T1's tools (meters, signal path, captures) | SYS-041: one step away |
| **Export and leaving** | Everything in documented formats | E5 |
| **Background service** | Device discovery, sync, backups, update downloads, notifications with the app closed (DESK-006 to DESK-008) | Sync without a button |

The CLI exposes every one of these from the same core (DESK-004), for automation and agents (TK-009).

## 7. Lessons

### 7.1 Copy

1. A player complete on its own; the desktop optional.
2. Plays as an append-only, timestamped event log.
3. Plain, versioned exports of statistics and playlists.
4. Selection by rule, playlist or "fill the free space", with a preview.
5. Incremental, resumable, content-verified transfer.
6. Conflicts kept and shown.
7. Heavy identification and tagging on the desktop.
8. Transcoding per device, opt-in, never touching the library.
9. The power and speed trade-off of each link stated, as HiBy does.
10. A wired or file-based recovery path next to updates over the network.

### 7.2 Avoid

1. Destructive defaults: delete that propagates, "Erase and Sync", silent cloud matching.
2. Updates that need a music backup first or cannot be reverted.
3. A version lock between the desktop and the device.
4. A desktop that loads the whole library before it can be used.
5. Caps on offline content, and files duplicated per playlist.
6. Sync features behind a subscription or a paid link.
7. MTP as the main transfer.
8. Timestamp-only change detection on FAT media.
9. Indexing that disturbs playback (Tangara reports, [cooltech.zone forum](https://forum.cooltech.zone/showthread.php?pid=946) [user]).
10. A proprietary on-device database others must reverse-engineer (iTunesDB, [libgpod](https://git.stg.centos.org/source-git/libgpod/blob/c8/f/src/itdb_itunesdb.c)).
11. A weaker Mac build.

## 8. Open points

1. Measured transfer speeds over USB and Wi-Fi on the T1 (no DAP publishes them; reviewers' opinions only).
2. The retention window for recoverable deletion.
3. Which writing mode is the default for tags (in place, or library only).
4. Candidate DESK requirements will be drafted from §3 to §6 once the maintainers review this report.

Decided on 2026-10-23: transcoding per device is offered (DESK-010); every third party is an opt-in provider (SYS-045); the household and its members are in the model (SYS-043, SYS-044); playback syncs in real time across devices (SYS-042).
