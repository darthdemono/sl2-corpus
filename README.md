# sl2-corpus

> **Offline use only.** These are PC save files. Loading a copied, edited, or otherwise unfamiliar save and then going online can trigger an invalid-data flag, a softban, or an account restriction. Do not use your ordinary online account as the test environment. That is not a dramatic disclaimer; FromSoftware games really do care about save state, and they do not explain their decisions very well.

This repository is a documented corpus of FromSoftware `.sl2` save files. The files are the payload, but a pile of binaries is not a corpus by itself. The useful part is that each file has a place in a sequence, a game-specific interpretation, and a generated report that says exactly what the current analyzer can prove from it.

The corpus exists because save data is rarely preserved as a timeline. Most public saves are either a single endgame mule or a half-described upload that expects you to trust a title such as `SAVE_FINAL_REAL.zip`. That is useful for somebody, but it is not enough for regression testing, progress research, or understanding how a game represents a playthrough. These files were preserved as intermediate snapshots, then copied into this repository without modifying their backup sources. The result is imperfect, sparse, and much more useful than one final save.

## What is here now

The current snapshot contains 153 individual PC `.sl2` files. They remain as individual files on purpose: an editor author can take one exact fixture, a researcher can cite one exact snapshot, and a player can inspect one exact state without extracting an archive just to find it.

- **Dark Souls II: Scholar of the First Sin** contains 28 snapshots: Joy's progression from level 127 to 197, plus a level-10 `test char` fixture. See [`Dark Souls II SOTFS/README.md`](Dark%20Souls%20II%20SOTFS/README.md).
- **Sekiro: Shadows Die Twice** contains 125 files: 124 readable Wolf snapshots from Attack Power 1 through 14, plus one empty save container. See [`Sekiro Shadows Die Twice/README.md`](Sekiro%20Shadows%20Die%20Twice/README.md).

[`Combined.md`](Combined.md) is the dense, generated cross-game report. It is deliberately not this README. A first-time visitor needs to know what the archive is, whether it is safe enough for their purpose, and how to read it. They do not need 2,000 lines of boss tables before they know what they downloaded.

DS1, DS3, and Elden Ring are not included in this snapshot. Their absence is not a mysterious gap in the documentation; those collections are still being curated and tested. A corpus should state its boundary instead of quietly pretending that it has none.

## Why individual files do not mean an enormous Git repository

The saves themselves are fixed-size game containers. A valid DS2 or Sekiro save cannot simply have nine unused slot blocks cut out of it and remain a trustworthy `.sl2`; the games index those blocks at fixed positions. So this corpus does not pretend that zero-filling padding is physical trimming. It is not.

Git is the space-saving layer here. The 153 individual raw files occupy about 1.7 GB after checkout, but Git's native delta compression packs this exact set to about 38.8 MiB because snapshots share almost all of their bytes. You get real, individually usable `.sl2` files after checkout; the repository does not need to store 1.7 GB of repeated padding as separate history objects. This is the boring solution. It is also the correct one.

The cost is local disk space. A clone or checkout expands the files. That is unavoidable if the objective is to hand somebody actual game-valid `.sl2` files rather than a custom compressed format. Storage on disk is the trade; individual inspectable files are the benefit.

## Who this is for

**Save-editor authors and reverse engineers** can use the files as labelled fixtures across real progression states rather than making two endgame characters and hoping they cover every edge case. A parser that reads only a pristine final save has not really been tested.

**Progression researchers** can follow what appears between snapshots: boss evidence, bonfires or idols, inventory, Soul Memory, NG state, and endings where the format exposes them. The sequence is a record of what was present by a point in time, not a perfect diary of every button press.

**Players and build testers** can inspect an offline state without repeating an entire run. This is the shallowest use, but it is legitimate. Just do not confuse convenient with online-safe.

**Preservationists** get files with context. If the analyzer improves later, these saves can be re-read. If a field turns out to have been interpreted wrongly, the evidence has not disappeared behind a dead file-hosting link.

## How the corpus is made

Each corpus file starts as a backup copy. The backup is never written. A new copy is made outside the backup location, its BND4 container checksums are verified, and the result is parsed again by [`sl2-analyzer`](https://github.com/darthdemono/sl2-analyzer). Sekiro copies have only their provably inactive padding canonicalised. DS2 keeps its inactive encrypted slot data because a safe general DS2 slot-rewrite format has not been demonstrated. The correct response to uncertain save surgery is to leave it alone. A smaller bad file is still bad.

The generated reports are intentionally conservative. A listed boss, item, bonfire, idol, covenant, or ending is evidence the parser can read. A missing one is not proof that it never happened. Likewise, the first snapshot containing an event gives an **upper bound**: it happened sometime after the previous backup and no later than this one. This is reconstruction from sparse saves, not a fabricated event log.

The game-folder `README.md` files are generated timelines with a short collection-specific introduction. They include the recorded environment metadata that matters for reproduction: edition, version where retained, PC platform, launcher, compatibility layer, OS, and DLC where applicable. Those facts were supplied from the historical run environment, not extracted from the save bytes.

## How to use a file with sl2-analyzer

Choose a save, copy it somewhere safe, and run the analyzer against the copy. The analyzer reads locally; it does not upload the save. The Markdown report is the easiest format to inspect or share with a tool author, while JSON is useful for scripting.

```powershell
git clone https://github.com/darthdemono/sl2-analyzer.git
Set-Location .\sl2-analyzer
python .\sl2_to_md.py 'C:\path\to\save.sl2' -o .\report.md
python .\sl2_to_md.py 'C:\path\to\save.sl2' -o .\report.json
```

If you want the overview instead, point the analyzer at one of the game directories and write a new combined report. The checked-in reports are the reference render for this corpus snapshot; regenerate only when the parser or the collection changes.

```powershell
python .\sl2_to_md.py 'C:\path\to\sl2-corpus\Sekiro Shadows Die Twice' -o .\sekiro-combined.md
```

These are PC-format saves. Console conversion, account migration, and online compatibility are outside the scope of this repository. The files do not become safe merely because a conversion tool accepts them.

## Account and provenance policy

The public Steam account associated with this corpus is `76561199030416229`. When a supported save format contained a different account number, its corpus copy was normalised to that public account after verification. No account identifier is guessed from a folder name, and no backup is modified. This makes the stored identity consistent without hiding that save formats handle account data differently.

Package and source provenance are deliberately not published here. They are not useful to somebody evaluating the save contents, and they are not needed to reproduce an analysis. The public reports retain the environment metadata that is useful: game edition, verified version where known, platform, launcher, Proton version, OS, and DLC.

## Limits and status

This is an ongoing archive, not a claim to have every FromSoftware game, every branch, or every possible state. The current scope is explicit above. New games enter only after their copies can be made safely and the analyzer can read them consistently. That standard is slower than dropping files into a folder. It is also the whole point.

Use the individual saves when you need a fixture. Use the game README when you need one playthrough's evidence. Use `Combined.md` when you need the broader sequence. Those are three different questions, and keeping them separate makes the archive usable.
