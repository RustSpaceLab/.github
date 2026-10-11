# Rust Space Lab

Software for the data a spacecraft produces: the standard that describes it, the framing that
carries it, and the two ends that write and read it.

Three repositories, one dependency chain. Every number below was measured on this machine and
the command that produced it is in the repository.

## The projects

**[xtce-rs](https://github.com/RustSpaceLab/xtce-rs)** — parse XTCE, decode CCSDS telemetry.

XTCE is the OMG and CCSDS standard that says which bits of a packet are which parameter, how
to calibrate them, and which container a packet matches. This reads a real mission database
and decodes a real downlink against it, and it is tested the only way that means anything:
packet for packet against [`lasp/space_packet_parser`](https://github.com/lasp/space_packet_parser),
the Python implementation, over nine recorded streams. Where they disagree, this one is wrong
until proven otherwise.

It is about 100× that reference on decode and 11× on load, and it also compiles a definition
into a static Rust decoder — every offset a literal, nothing consulted at run time — which is
another two orders of magnitude over its own interpreter.

**[xtce-flight](https://github.com/RustSpaceLab/xtce-flight)** — the other half, for the
spacecraft.

Ground software reads packets, and the public tooling reflects that: parsers, decoders,
displays. Flight software *writes* them, on a part with no heap, no operating system and a
hard rule against a code path that can panic — and the generators that produce that half live
inside the companies that fly, and stay there. This one compiles an XTCE definition into a
`no_std` encoder, with a bare-metal probe that proves no panic path exists in the generated
code for `thumbv7em-none-eabihf`.

**[xtce-gs](https://github.com/RustSpaceLab/xtce-gs)** — a ground station in one binary.

A socket, a definition, and a window. UDP, TCP or a file replay; CCSDS transfer frames with
Reed-Solomon and the pseudo-randomiser; CSP; decoding through `xtce-rs`; and a live interface
with plots that decimate to the pixel budget rather than to the ring. Nine megabytes, no JVM,
no Node, no database, no browser.

## Why

The open-source mission control that exists is excellent and enormous. Yamcs runs on a JVM;
OpenMCT is a front end that needs a backend behind it. Both are the right answer for an
operations centre flying a constellation, and both are a lot to carry into a room where four
people are testing a cubesat on a bench.

These are built for that room.

## What they are not

Not flight-proven. Not an operations centre. Not a parameter archive — history is a ring in
memory and a recording is a file of bytes. One process, one operator, one window; two
operators want Yamcs.

## Licence

MIT or Apache-2.0, at your option. Vendored test data is BSD-3-Clause from the reference
implementation, with provenance recorded in each repository.
