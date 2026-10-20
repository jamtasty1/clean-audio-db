# clean-audio-db

Where the swearing is in TV episodes and films: each listed word, when it is said, and how strong it is. It's the data [TV Compress](https://github.com/jamtasty1/tv-compress)'s Clean Audio uses to silence those words.

Clean Audio finds the words by transcribing the whole soundtrack with Whisper, which takes a long time per title. If this database already has a title, the app takes the words from here and skips Whisper. Every title cleaned with Whisper can be added back, so the next person doesn't have to wait.

## What is here

One file per title:

```
tv/<Show>-tvdb<id>/S01E02.json
films/<Film> (<Year>)-tmdb<id>.json
```

```json
{
  "format": 1,
  "title": "Carrie",
  "kind": "tv",
  "year": 2026,
  "ids": {"tvdb": 455755, "tmdb": 288673, "imdb": "tt11540908"},
  "season": 1,
  "episode": 1,
  "releases": [
    {"runtime": 3311.136, "codec": "eac3", "channels": 6, "detect": 4, "added": "2026-10-20", "hits": [
      [18.99, 19.19, "God,", "mild"],
      [43.05, 43.74, "fucking", "strong"]
    ]}
  ]
}
```

- **ids**: the show's TVDB ID or the film's TMDB ID, as Sonarr and Radarr give them, plus any others they have.
- **releases**: each release of the title that someone has cleaned. Releases differ in cut and intro, so each one has its own timings.
  - **runtime** is the release's length in seconds.
  - **codec** and **channels** describe the audio track the words were heard in.
  - **detect** is the version of Clean Audio's Whisper pass that found the words.
- **hits**: one line per listed word, in this order:
  - start and end in seconds, timed from the audio track's first sample
  - the word as heard
  - its level, `strong` or `mild`

Only the listed words are kept. The rest of what's said is the programme's dialogue and isn't included. There are no file paths and nothing about whose copy it was.

## How a title is matched

A title matches an entry when two things are true:

- **It's the same show and episode, or the same film.** This is decided by its TVDB or TMDB ID. TV Compress gets the ID from Sonarr or Radarr, using the folder the file is in. Without an ID, it matches on the show's or film's name.
- **Its runtime is within a second of the entry's.** A different cut runs longer or shorter, so its words fall at different times. Those titles go to Whisper instead.

## Using it with TV Compress

1. Leave *Settings > Clean Audio > Take a title's words from the shared profanity database* ticked. It's on by default.
2. Press *Clean Audio > Update from GitHub* to fetch the latest entries:
   - If `clean-audio-db` beside the app is a git clone of this repo, the app pulls.
   - Otherwise it downloads the repo.

## Adding titles

1. Press *Clean Audio > Export this machine's titles*. It writes every title Whisper has heard on your machine into the folder. Titles you already have files for get your release added, or brought up to date.
2. Commit the files and open a pull request.

Each title's file is laid out one field per line and one word per line. That way a pull request is a readable diff, and two people's releases of the same title merge cleanly.
