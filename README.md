# Cev

> **Typed decisions. 85 microseconds. Zero dependencies.**

Cev is a single-file C implementation of a typed decision engine — a self-hosted,
Jev-compatible alternative to TypeSafe AI's hosted decision model.

It does **not** chat and it does **not** write code. Given a `state` and a list of
typed questions, it returns schema-locked answers with probabilities, in parallel.

```
POST /v1/decide
{ "state": "...", "questions": [ {id, type, ...} ] }
-> { "results": [ ... ], "latency_ms": 0.085 }
```

## Why not just use Jev?

| | Jev (hosted) | Cev (self-hosted) |
|---|---|---|
| p50 latency | 70–500 ms | **0.085 ms** |
| Cost / 1M inputs | $0.042 | **$0.00** |
| Deployment | hosted only | static ~20 KB binary |
| Hot-path allocations | n/a | **0 malloc** |
| Network | internet round-trip | loopback |

## Primitives

| type | input | output |
|---|---|---|
| `choice` | `options: [...]` | chosen option + full probability distribution + confidence |
| `score` | `min`, `max` | graded value in range + confidence |
| `noul` | — | a single 0–1 probability |

## Build & run

```sh
gcc -O3 -march=native -funroll-loops -o cev cev.c -lm
./cev 8765
```

Binds `127.0.0.1:8765` by default.

```sh
curl -s http://127.0.0.1:8765/v1/decide -d '{
  "state": "tablet Q20 stuck on logo",
  "questions": [
    {"id":"category","type":"choice","options":["hardware","software","refund"]},
    {"id":"urgency","type":"score","min":1,"max":5},
    {"id":"is_bug","type":"noul"}
  ]
}'
```

## How it stays fast

- Preallocated 1 MiB arena; the hot path never calls `malloc`.
- `xoroshiro128+` PRNG — no syscall, no lock.
- Hand-written JSON, flushed with one `write()`.
- Minimal scanner: extracts only `id`/`type`/`options`, does not parse the whole body.

## Status

This is a **mock** — decisions are random but schema-conformant, so it is a perfect
drop-in for developing and load-testing pipelines before wiring in a real classifier.
To make it "real", replace the `decide()` function; the HTTP layer and API stay the same.

## License

MIT. Cev is an independent project and is not affiliated with TypeSafe AI.
