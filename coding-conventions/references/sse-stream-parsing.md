# SSE Stream Parsing

Load this reference only when working on Server-Sent Events or similar chunked streaming transports.

## Goals

- Treat chunk boundaries as arbitrary
- Keep wire-format handling at the transport edge
- Convert parsed events into typed domain messages before the rest of the system sees them

## Rules

- Never assume a chunk boundary matches a message boundary
- Buffer partial lines and partial events until a full event delimiter is present
- Handle `\r\n`, `\n`, and split `\r` / `\n` boundaries safely
- Parse the stream incrementally instead of concatenating unbounded data
- Reset or cap buffers when the protocol allows it so malformed input cannot grow memory forever
- Distinguish transport parse failures from application-level event failures

## Design Guidance

- Keep one small parser responsible for turning byte or text chunks into complete SSE events
- Expose a typed event shape to downstream code instead of leaking raw SSE lines everywhere
- Test chunk-splitting edge cases explicitly when changing parser behavior
- If the system supports more than one streaming protocol, isolate shared buffering logic from protocol-specific field parsing
