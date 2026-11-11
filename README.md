# CoGS documentation for builders

The CoGS Technical Reference and Owner's Manual as plain Markdown, plus one page of notes for coding assistants. Clone this beside your project, or point your assistant at the files, and it has the whole contract for the card in a form it can read.

- `techref.md`: the Technical Reference, the book a program is written against.
- `owners.md`: the Owner's Manual, what the user sees and the share protocol a LAN server speaks.
- `AGENTS.md`: the rules that bite when code is written from the book without debugging on a real Apple II. Copy it into your project as `AGENTS.md`, `CLAUDE.md`, or a `.cursor/rules` file.

The files are regenerated from the manual sources on every manual build and match https://a2cogs.com/docs/ (HTML and PDF there, `llms.txt` at the site root). The firmware is pre-release and its source is not published; the register map, protocol, commands and events documented here are stable and programs written against them run on every released firmware.

Licenses. The two books are Creative Commons Attribution-NoDerivatives 4.0 (`LICENSE`): copy, quote, mirror, and hand to an assistant freely, with credit; do not publish modified versions. `AGENTS.md` and any example or data files are MIT (`LICENSE-MIT`) so you can edit them into your own project. CoGS is a trademark of Rob Perissi; neither license grants trademark rights.

Bug reports, technical questions, and corrections go to Rob at info@a2cogs.com, or #cogs on the Apple2Infinitum Slack.

---


> CoGS is a co-processor and network card for the Apple II (IIGS first, 8-bit Apple II too). A Raspberry Pi Pico 2 W on an Apple II slot card gives the machine Wi-Fi or wired Ethernet, TCP, TLS 1.2 with HTTPS terminated on the card, HTTP and JSON commands an 8-bit or 65816 program drives through sixteen slot registers, network disk images streamed and booted over HTTP, audio streaming, and a GS/OS desk accessory. The card talks a small frame protocol: an opcode, a two-byte length, and a payload, over two FIFOs. Site: https://a2cogs.com

This file indexes the CoGS documentation in forms an assistant or a coding agent can read whole. The Technical Reference is the contract a program is written against; it is about 32,000 words and fits in one context window. The firmware is pre-release and its source is not published yet; the card's register map, protocol, commands, events and timing rules are fully documented and stable, and programs written against the Technical Reference run on every released firmware. Current firmware 1.42 (11/10/2026), manual revision 36.

## Documentation

- [CoGS Technical Reference (Markdown)](https://a2cogs.com/docs/techref.md): the whole book. Bus interface and register map, frame protocol, every command and event with its payload, the network disk, injected desk accessory and tool sets, bus timing rules host code must follow, a programming guide with 65816, ORCA/C and cc65 examples, hardware, the emulator, and appendices of error codes, URL and catalog formats, and a glossary. Read this before writing code for the card.
- [Notes for coding assistants](https://a2cogs.com/docs/cogs-agent-notes.md): one page of the rules that go wrong when code is written from the book without debugging on a real Apple II. Copy it into a project as `AGENTS.md` or `CLAUDE.md`. MIT.
- [CoGS Owner's Manual (Markdown)](https://a2cogs.com/docs/owners.md): installing and using the card. The power-on menu, Browse and boot, drives, audio, pictures, the Game Room, GS/OS and the CoGS Config desk accessory, the CoGS Image Manager Mac app that shares a folder to the card, updating, troubleshooting, specifications. Useful for what the user sees and for the share protocol a LAN server speaks.
- [Documentation page](https://a2cogs.com/docs/): the same two books as HTML and PDF.
- [Firmware notes](https://a2cogs.com/firmware/): what changed in each firmware, for users. Cards update themselves from About / Update.

## Code and examples

- [cogs-gameroom](https://github.com/rperissi/cogs-gameroom): the Game Room lobby for two-player network play through the card. Lobby spec, the card protocol subset a client needs, a porting guide, a worked ORCA/Pascal network layer, a Python reference. The place to start for a networked game.
- [cogs-fujinet-lib](https://github.com/rperissi/cogs-fujinet-lib): fujinet-lib with a CoGS card bus (GPL-3). A program written against fujinet-lib links against it and runs from a CoGS card with no source change. Worked example of HTTP, JSON and appkeys on an 8-bit Apple II.
- [RetroWRITE (CoGS Share fork)](https://github.com/rperissi/RetroWRITE): a ProDOS 8 word processor with Send and Open to a CoGS share, in cc65 C. Worked example of HTTP PUT in pieces, GET with the retained-document stream, and running from a volume the card itself serves.
- [iigs-doom](https://github.com/rperissi/iigs-doom), [iigs-fly](https://github.com/rperissi/iigs-fly), [iigs-llm](https://github.com/rperissi/iigs-llm), [iigs-bitcoin-miner](https://github.com/rperissi/iigs-bitcoin-miner): IIGS programs that use the card (deathmatch over the internet, neurons over TLS, an LLM client, a Stratum miner).

## Catalogs and services the card reads

- [Disk catalog](https://a2cogs.com/disks/cogs-hub.json): the hub the card's Browse Disks reads; the page and search shapes are in Technical Reference Appendix B. A LAN share (the Image Manager, or any HTTP server with range requests) serves the same shapes.
- Game Room lobby at `https://lobby.a2cogs.com/`: the matchmaking service, plain HTTPS with flat JSON. Endpoints and the client contract are in Technical Reference Chapter 10 and the cogs-gameroom repo.

## Optional

- [Full text, both books](https://a2cogs.com/llms-full.txt): the Owner's Manual followed by the Technical Reference as one file, about 51,000 words.
- [a2cogs.com](https://a2cogs.com): the card, the project, and the interest list.
- [cogs-docs](https://github.com/rperissi/cogs-docs): these files as a repository to clone beside a project (`AGENTS.md`, `techref.md`, `owners.md`), updated with each manual build. Books CC BY-ND 4.0, notes MIT.
- Bug reports, technical questions, and corrections go to Rob at info@a2cogs.com, or #cogs on the Apple2Infinitum Slack.
