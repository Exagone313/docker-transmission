# Local transmission patches

Temporary local build of transmission `4.1.3` with fixes for the
"torrent stalls at 99.9x%" regression.

Upstream issues: [#8935](https://github.com/transmission/transmission/issues/8935),
[#7919](https://github.com/transmission/transmission/issues/7919),
[#7834](https://github.com/transmission/transmission/issues/7834).

## Why

`refactor: faster wishlist` (#7027, shipped in 4.1.0-beta.1) rewrote the block
scheduler and dropped *endgame*. Since then a block is handed to exactly one
peer and only returns to the pool on choke / reject / cancel / a 25s request
timeout / peer disconnect. At the tail of a download that lets a single slow or
unresponsive peer hold the last few blocks hostage while every other peer sits
idle, which is why the torrent sits at 99.9x% with 0 B/s for minutes to hours
and why "pause/resume" or "verify local data" unsticks it (both rebuild the
wishlist).

4.1.3 already has every wishlist fix that upstream has published; the remaining
cause is still open upstream.

## Patches

| file | what |
| --- | --- |
| `0001` | Restore the 3.00/4.0.x endgame policy: once everything we still want is already in flight, let a second peer race for those blocks. |
| `0002` | Partial backport of upstream #7026 (`fa832041c`, in `main`, not in `4.1.x`): ask for blocks as soon as we can, and top up requests *before* handing a rejected block back to the wishlist. |
| `0003` | Remember per-peer which blocks a peer rejected, for 60s, so we stop looping on peers that refuse a block. |
| `0004` | Guard `requested_block_span()` against an empty `unrequested` set, which `0001` makes reachable. |

Generated with `git format-patch` from a branch based on the `4.1.3` tag; they
apply with `patch -p1` to the contents of the official
`transmission-4.1.3.tar.xz` release tarball.

## Impact on state

None. Only the binaries in `/usr/bin` are replaced; the web assets, runtime
libraries, s6 services and settings handling all still come from the Alpine
packages. Nothing in `/config` or the download directories changes format, so
moving back to a stock `lscr.io/linuxserver/transmission` image is just a
`docker pull`.
