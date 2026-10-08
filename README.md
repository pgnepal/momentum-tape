# Momentum Tape

A public, append-only fingerprint of a momentum swing-trading desk's nightly calls.

Each night the desk freezes its actionable calls (entry trigger, stop, targets)
into a private record and publishes **only its fingerprint** here: one line in
[`tape.txt`](tape.txt) per night, plus an [OpenTimestamps](https://opentimestamps.org)
receipt in [`ots/`](ots/). No symbols and no prices appear in this repository.

```
<seq> <date> r<revision> calls=<n> entry=<entry_hash> payload=<payload_sha256>
```

## Why it can't be rewritten

- `payload` is the SHA-256 of that night's calls, serialised canonically
  (sorted keys, no spaces, UTF-8).
- `entry` is the SHA-256 of `{seq, asof, revision, payload_sha256, prev_hash, tape_version}`,
  where `prev_hash` is the previous night's `entry`. Every night therefore
  commits to every night before it: changing any past night changes every
  later fingerprint.
- Each night's `entry` is also submitted to OpenTimestamps calendars, which
  anchor it in the Bitcoin blockchain. The receipt proves the fingerprint
  existed by that time, independently of GitHub.
- A night that was re-run with different calls gets a new `r<revision>` line;
  earlier lines are never edited.

## Verifying a revealed night

When a night's calls are revealed, anyone can check them:

1. Serialise the revealed payload canonically and take its SHA-256; it must
   equal that night's `payload`.
2. Recompute `entry` from the fields above with the previous line's `entry`
   as `prev_hash`; it must equal that night's `entry`.
3. Run `ots verify` on the receipt (after `ots upgrade`) to see when it was anchored.

Nothing here is investment advice. Calls are research published on a regular
schedule, and the record exists so its history can be checked, not to promise
results.
