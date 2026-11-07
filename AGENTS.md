# Building for the CoGS card: notes for coding assistants

Copy this file into your project as `AGENTS.md`, `CLAUDE.md`, or a rule in `.cursor/rules/`, next to the Technical Reference (https://a2cogs.com/docs/techref.md). The Technical Reference is the contract; this file is the short list of things that go wrong when code is written from the book without having debugged against a real Apple II.

CoGS is a slot card for the Apple II (IIGS and 8-bit). A program talks to it through sixteen registers at `$C080 + slot*16` and a frame protocol over two FIFOs. The card does Wi-Fi or Ethernet, DNS, TCP, TLS 1.2, HTTP and HTTPS, JSON extraction, SHA-256, random numbers, network disk images, and audio streaming. TLS 1.3 is not implemented. The firmware is pre-release and its source is not published; the register map and protocol are stable and documented, and programs written against them run on every released firmware.

## The frame

Every command is `opcode, LEN_LO, LEN_HI, payload`, written one byte at a time to DATA (+0). Every reply and event comes back the same way. Command payloads are at most 2048 bytes. A `$91` DATA event payload is at most 1024 bytes; other replies have no cap (STREAM_READ can return 6 + 4096), so read the two length bytes first and discard what you cannot keep. The card never interleaves the bytes of two frames, and never fragments one.

Replies to network commands are deferred: the command is accepted at once, the reply arrives when the work is done, and events (`$91` DATA, `$93` ERROR, `$95` HTTP_DONE, link state) can arrive before it. Read frames until you see the opcode you are waiting for, and consume or discard everything else. Never assume the next frame is your reply.

## Register rules that bite

- Check STATUS (+1) bit 7 before every pop of DATA, or RXLO/RXHI for a count. A pop with nothing queued returns `$00` and your parser is now off by one.
- Check STATUS bit 6 before writing. The host-to-card FIFO is 4 KB, so a whole command fits.
- On the 65816, never read a card register with `abs,X` or `abs,Y` while the index register is 16 bits wide (x = 0). The processor adds a dummy read of the same address and the FIFO pops twice; you see every second byte. Use `sep #$30` and 8-bit indexes around register access, `lda [dp]` long indirect, or a long absolute `lda $E0C0n0`. Some accelerator cores and emulators do not show the fault, so passing there proves nothing.
- `sta DATA,x` is fine. The card compensates for the indexed-store dead-cycle read. Read-modify-write instructions on DATA (`inc`, `asl`) are not compensated and have no use.
- Detect the card with three reads and no timing: ID (+5) is `$A2`, VERSION (+6) is nonzero, STATUS is not `$FF`. Never probe by sending a frame to an unknown slot.
- One context owns the card at a time. If an interrupt handler (the card's background audio heartbeat, your own) can touch the FIFO, bracket each foreground round trip with `sei`/`cli`. Do not wait for a key with interrupts masked on a IIGS; mask only around the round trip.
- Code that executes from `$C800` must not let an interrupt handler touch another slot's `$CnXX` or `$CFFF`.
- After a parse error (`$93` with socket `$FF`), resynchronize: CONTROL (+4) bit 3, then bit 2, then read SYNC (+A) and wait for it to change.

## HTTP

The HTTP command (`$20`) takes socket, flags, method, URL, extra headers, and a body under 2 KB. Flags: STREAM (deliver as it arrives), HDRECHO (header lines in the first DATA event), RETAIN (keep the body on the card for JSON_GET, up to 32 KB), KEEPALIVE (park the connection), DISCARD. The body arrives as `$91` DATA events of up to 1024 bytes and ends with `$95` HTTP_DONE carrying the status. A body bigger than 32 KB with RETAIN set ends with ERROR 9, BUF_OVERRUN.

To pull one value out of a JSON reply, make the request with RETAIN and call JSON_GET (`$32`) with a path like `pl[0].n`. The whole document never crosses the bus.

Uploads over 2 KB go in pieces: repeat the PUT with `Content-Range: bytes a-b/total` on each, 1 KB at a time. A server that understands this (the CoGS Image Manager's `files/` folder does) joins them.

## Programs that run from a disk the card serves

When the user booted from the card or mounted your program's volume from it, the ProDOS disk driver and your program share the FIFO. Every disk read or write is a frame round trip. So:

- Drain every frame whole before any disk I/O, and never do disk I/O between the frames of one response.
- Do not take a streamed body (`$91` DATA events) while writing it to disk. Make the GET with RETAIN and then read the retained document through STREAM_READ (`$71`) on handle 2, one piece at a time, writing each piece between reads. Handle 2 needs no STREAM_OPEN; END (flags bit 1) marks the last piece; STREAM_CLOSE (`$72`) handle 2 when done. This limits a download to 32 KB.
- Expect frames that are not yours (an event, a reply to the driver's own command) and skip them by length.

An emulator that serves the disk from a file will not show any of this. Test on a volume the card actually serves.

## Platform

- 8-bit hosts (][+, //e, //c) and the IIGS in emulation mode have 8-bit indexes and none of the 16-bit index rule applies. Everything else does.
- Slot I/O runs at 1 MHz on every host, accelerated or not. Register timing never changes; only the instruction time between accesses does.
- GS/OS programs can use `cogslib` (ORCA/C). 8-bit programs in cc65 use the same frame protocol; the Technical Reference has a cc65 host and the RetroWRITE fork (https://github.com/rperissi/RetroWRITE, `src/ovl_net.c`) is a complete 8-bit client for GET, PUT, and the retained stream. Programs written against fujinet-lib link against cogs-fujinet-lib (https://github.com/rperissi/cogs-fujinet-lib) with no source change. Two-player network games use the Game Room lobby (https://github.com/rperissi/cogs-gameroom).
- Error codes are in Technical Reference Appendix A. The ones you will meet first: 1 NOT_JOINED, 2 DNS_FAIL, 3 CONN_TIMEOUT, 4 CONN_REFUSED (also a non-2xx on STREAM_OPEN), 6 CERT_VERIFY, 9 BUF_OVERRUN, 10 BAD_FRAME, 11 UNSUPPORTED, 13 NO_RETAINED, 14 JSON_NOPATH.

## Things not to do

- Do not claim or rely on TLS 1.3, HTTP/2, or any command not in the Technical Reference for the firmware you are targeting. WebSocket exists only as CONNECT flag bit 3 (firmware 1.36, used by the Game Room lockstep tier); there is no WebSocket API beyond that socket. PING reports the firmware version and a caps byte; gate features on it.
- Do not poll DATA in a loop as a wait. Poll STATUS.
- Do not send a second command on the same socket before the first has resolved.
- Do not treat the card's User-Agent, card ID, or the a2cogs.com catalog JSON as stable API for your own server unless the Technical Reference documents the shape.

Bug reports, technical questions, and corrections go to Rob at info@a2cogs.com, or #cogs on the Apple2Infinitum Slack. If something in the card does not match the book, that is a report he wants.
