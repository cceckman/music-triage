# music-triage: Emplace music files

This is a program to take music files - CD rips, downloads, etc. - and put them
in the right place: `<Library>/<Album>/<Artist>/<Track>.<format>`.

There's some variance in how files are tagged, formats, etc., though, so we try
to do some other things at the same time:

- [x] Any files for which the sort keys aren't available get put into a
  "quarantine" directory for manual action. This makes it ~easy to handle e.g.
  imports of another music system's database; lyrics, cover art, get sorted out
  to a different directory.
- [ ] #1: Duplicate detection: Multiple copies of the same track can be deduplicated
  by format, preferring lossless codecs.
- [ ] #2: DRM detection: Files with DRM protection (M4P, old iTunes files) can be
  quarantined.

(Unchecked items are WIP.)
