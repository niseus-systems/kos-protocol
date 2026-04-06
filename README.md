# KOS Protocol

**The knowledge layer machines trust.**

KOS is an open protocol for publishing machine-readable, provenance-tracked knowledge about any entity via a single `kos.json` file.

- **Specification:** [kosprotocol.dev/PROTOCOL](https://kosprotocol.dev/PROTOCOL)
- **Generator:** [kosprotocol.dev/GENERATOR](https://kosprotocol.dev/GENERATOR) — AI reads this → produces valid kos.json
- **Consumer:** [kosprotocol.dev/CONSUMER](https://kosprotocol.dev/CONSUMER) — AI reads this → knows how to use kos.json
- **Proof:** [kosprotocol.dev/PROOF](https://kosprotocol.dev/PROOF) — 4/14 vs 14/14 signals comparison
- **Registry:** [kosprotocol.dev/registry.json](https://kosprotocol.dev/registry.json) — index of live kos.json files
- **Structure Checker:** [kosprotocol.dev](https://kosprotocol.dev)
- **JSON Schema:** [kosprotocol.dev/v1/schema.json](https://kosprotocol.dev/v1/schema.json)

## What it does

Existing standards tell machines *what content means* (Schema.org), *where to find pages* (sitemap.xml), and *what they may access* (robots.txt). None of them address **how much a machine can trust a piece of data, how that data connects to other data, and when it was last verified by a human.**

KOS fills that gap.

## How it works

A `kos.json` file is placed at `/.well-known/kos.json` on any domain. It contains:

- **Typed nodes** — 10 types: entity, service, product, location, person, credential, content, event, concept, asset
- **Provenance** — every node carries metadata: who provided it, when verified, how checked, confidence score (0-1)
- **Freshness decay** — `f = e^(-lambda * days)` computes reliability over time. Different node types decay at different rates
- **Typed relationships** — dep (dependency), ref (reference), rel (semantic)
- **Cross-entity references** — nodes can link to other entities' kos.json files

## Quick start

```json
{
  "$schema": "https://kosprotocol.dev/v1/schema.json",
  "entity": {
    "name": "Your Entity Name",
    "type": "Organization",
    "id": "kos:your-entity"
  },
  "knowledge": [
    {
      "id": "kos:main-offering",
      "type": "service",
      "label": "Your Main Offering",
      "provenance": {
        "created": "2026-04-04",
        "source": "owner"
      }
    }
  ],
  "meta": {
    "version": "1.0",
    "generated": "2026-04-04"
  }
}
```

Publish at `yourdomain.com/.well-known/kos.json`. Done.

## Repository contents

```
PROTOCOL.md          Full specification (v1.0)
GENERATOR.md         AI-executable generation instructions
CONSUMER.md          AI-executable consumption guide (10 steps)
generator.json       Machine-readable generator
kos.json             KOS Protocol self-description
LICENSE              Open spec + MIT code license
schema/
  v1.schema.json           JSON Schema for validation
  v1.generator.schema.json Generator schema
```

## The 0.8 boundary

AI-generated kos.json files are capped at 0.8 confidence. The range 0.8-1.0 is reserved exclusively for human-verified data. This boundary is inviolable.

## First AI citation — 19 hours

KOS Protocol was cited by Perplexity and Grok (xAI) within 19 hours of publication, with zero marketing or promotion. The protocol did not need marketing. It needed existence. [See the evidence](https://kosprotocol.dev/#the-challenge).

## Live implementations

| Domain | Entity | Nodes |
|---|---|---|
| [kosprotocol.dev](https://kosprotocol.dev/.well-known/kos.json) | KOS Protocol | 11 |
| [niseus.com](https://niseus.com/.well-known/kos.json) | Niseus LLC (creator) | 8 |

Full registry: [kosprotocol.dev/registry.json](https://kosprotocol.dev/registry.json)

## License

The specification is open. Anyone may implement it without fee. Attribution to Niseus Authority Systems is required. See [LICENSE](LICENSE) for full terms.

"KOS Protocol" and "kos.json" are trademarks of Niseus Authority Systems.

---

Created by [Niseus Authority Systems](https://niseus.com) | [kosprotocol.dev](https://kosprotocol.dev)
