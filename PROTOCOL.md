# KOS Protocol Specification

**Version:** 1.0.0-draft
**Status:** Draft
**Created:** 2026-04-04
**Author:** Niseus Authority Systems (niseus.com)
**License:** Open specification. Free to implement. Attribution required.
**Copyright:** (c) 2026 Niseus Authority Systems. All rights reserved.
**Trademark:** "KOS Protocol" and "kos.json" are trademarks of Niseus Authority Systems.

---

## Abstract

KOS (Knowledge Operating System) defines a standard format for publishing machine-readable, provenance-tracked knowledge about any entity. A `kos.json` file provides AI agents, search engines, and automated systems with structured knowledge that includes typed relationships between concepts and verifiable freshness metadata for every data point.

Existing standards tell machines *what content means* (Schema.org), *where to find pages* (sitemap.xml), and *what they may access* (robots.txt). KOS addresses what none of them cover: **how much a machine can trust a piece of data, how that data connects to other data, and when it was last verified by a human.**

The protocol is open. Anyone may produce or consume `kos.json` documents without license or fee.

---

## 1. Design Principles

1. **Machine-native, human-maintainable.** The canonical format is JSON. The structure is strict enough for automated consumption and clear enough for manual authoring.

2. **Trust is computable.** Every node carries provenance metadata. A consuming system can calculate a trust score without external lookups.

3. **Relationships are first-class.** A node in isolation is a fact. A node with typed relationships is knowledge. The protocol treats connections between data points as equal citizens to the data points themselves.

4. **Freshness decays.** Data that was true six months ago may not be true today. The protocol models this decay mathematically so machines can weight their confidence accordingly.

5. **Open by default.** The specification is free. The format is JSON. No proprietary extensions are required for a valid implementation. Interoperability is a design goal, not an afterthought.

6. **Entity-sovereign.** The entity described in a `kos.json` file controls that file. No central registry. No intermediary. Authority flows from the source, not from a platform.

---

## 2. Terminology

The key words MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD, SHOULD NOT, RECOMMENDED, MAY, and OPTIONAL in this document are to be interpreted as described in RFC 2119.

- **Entity** — The subject of the knowledge graph. A business, person, organization, product, or any identifiable thing.
- **Node** — A single unit of knowledge within the graph. Has a type, content, and provenance.
- **Relation** — A typed, directional connection between two nodes.
- **Provenance** — Metadata describing the origin, verification history, and computed freshness of a node.
- **Freshness** — A value between 0.0 and 1.0 representing the computed reliability of a node based on time elapsed since last verification.
- **Consumer** — Any system that reads and interprets a `kos.json` file (AI agent, search crawler, application).
- **Producer** — Any system or person that creates or maintains a `kos.json` file.

---

## 3. Discovery

### 3.1 Well-Known URI (Primary)

A `kos.json` file SHOULD be served at:

```
https://{domain}/.well-known/kos.json
```

This follows RFC 8615 (Well-Known URIs). The file MUST be served with `Content-Type: application/json` and SHOULD include `Access-Control-Allow-Origin: *` to permit cross-origin consumption.

### 3.2 HTML Link Element

A web page MAY advertise its associated `kos.json` via a `<link>` element in the document `<head>`:

```html
<link rel="knowledge" type="application/json" href="/.well-known/kos.json">
```

The `rel` value is `knowledge`. The `href` MAY be an absolute or relative URI.

### 3.3 DNS TXT Record

An entity MAY advertise its `kos.json` location via DNS:

```
_kos.example.com.  IN  TXT  "v=kos1; url=https://example.com/.well-known/kos.json"
```

This enables discovery without an HTTP request and supports entities that may host their `kos.json` on a different domain.

### 3.4 Priority

When multiple discovery mechanisms are present, consumers SHOULD resolve in this order:

1. Explicit URL (if provided by a referring system)
2. Well-Known URI
3. HTML Link Element
4. DNS TXT Record

---

## 4. Document Structure

A `kos.json` document is a single JSON object with four required top-level keys.

```json
{
  "$schema": "https://kosprotocol.dev/v1/schema.json",
  "entity": { },
  "knowledge": [ ],
  "meta": { }
}
```

An OPTIONAL fifth key, `relations`, contains cross-node relationships:

```json
{
  "$schema": "https://kosprotocol.dev/v1/schema.json",
  "entity": { },
  "knowledge": [ ],
  "relations": [ ],
  "meta": { }
}
```

### 4.1 `$schema`

- **Type:** String (URI)
- **Required:** MUST
- **Value:** A URI pointing to the JSON Schema that this document conforms to.
- **Current version:** `"https://kosprotocol.dev/v1/schema.json"`

### 4.2 `entity`

Describes the subject of the knowledge graph.

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | String | MUST | Display name of the entity |
| `type` | String | MUST | Entity category (see Section 5.2) |
| `id` | String (URI) | MUST | Canonical identifier. Format: `kos:{slug}` or full URI |
| `url` | String (URI) | SHOULD | Primary web presence |
| `description` | String | SHOULD | Brief description (under 300 characters) |
| `identifiers` | Object | MAY | External IDs (see below) |

**Identifiers object:** Maps external systems to their IDs for this entity.

```json
"identifiers": {
  "google_place_id": "ChIJ...",
  "wikidata": "Q12345",
  "lei": "5493001KJTIIGC8Y1R12",
  "duns": "123456789"
}
```

### 4.3 `knowledge`

An array of Node objects. Each node is a discrete unit of knowledge. See Section 5 for the full node specification.

### 4.4 `relations`

An array of Relation objects. Each relation connects two nodes. See Section 6 for the full relation specification.

Relation objects MAY also be embedded within individual nodes (see Section 5.1). Top-level `relations` and node-level `relations` are both valid; consumers MUST process both.

### 4.5 `meta`

Metadata about the `kos.json` document itself.

| Field | Type | Required | Description |
|---|---|---|---|
| `version` | String | MUST | Spec version: `"1.0"` |
| `generated` | String (ISO 8601) | MUST | When this file was last generated |
| `generator` | String | SHOULD | Tool/system that produced this file |
| `total_nodes` | Integer | SHOULD | Count of nodes in `knowledge` array |
| `avg_freshness` | Number (0-1) | SHOULD | Mean freshness across all nodes |
| `stale_count` | Integer | SHOULD | Nodes with freshness below 0.5 |
| `audit` | Object | MAY | Last audit information |

**Audit object:**

```json
"audit": {
  "date": "2026-04-01",
  "by": "owner",
  "scope": "full",
  "notes": "Verified all services and areas"
}
```

---

## 5. Node Specification

### 5.1 Required and Optional Fields

| Field | Type | Required | Description |
|---|---|---|---|
| `id` | String | MUST | Unique within document. Format: `kos:{slug}` |
| `type` | String | MUST | Node type from taxonomy (Section 5.2) |
| `label` | String | MUST | Human-readable name |
| `description` | String | SHOULD | What this node represents |
| `provenance` | Object | MUST | Origin and freshness data (Section 7) |
| `properties` | Object | MAY | Type-specific structured data |
| `relations` | Array | MAY | Inline relations from this node |
| `tags` | Array of Strings | MAY | Freeform classification labels |

### 5.2 Node Types (Core Taxonomy)

The following node types are defined by the protocol. Producers MAY define additional types using the extension mechanism (Section 9).

| Type | Description | Default Decay Rate (λ) |
|---|---|---|
| `entity` | The root business, person, or organization | 0.001 |
| `service` | A service offered | 0.005 |
| `product` | A product sold or distributed | 0.008 |
| `location` | A physical place or service area | 0.002 |
| `person` | A team member, partner, or contact | 0.008 |
| `credential` | A license, certification, award, or qualification | 0.001 |
| `content` | An article, FAQ, guide, or published material | 0.010 |
| `event` | A time-bound occurrence | 0.050 |
| `concept` | A methodology, philosophy, or framework | 0.0005 |
| `asset` | A tool, system, or resource | 0.003 |

Decay rates are defaults. Producers MAY override per-node (see Section 7.3).

### 5.3 Properties

The `properties` field holds type-specific structured data. The protocol does not prescribe required properties per type — this is intentionally flexible to support diverse domains.

Common patterns:

```json
// Service node
"properties": {
  "area": ["Anytown, FL", "Springfield, FL"],
  "availability": "24/7",
  "price_range": "$$"
}

// Person node
"properties": {
  "role": "Lead Technician",
  "since": "2019",
  "certifications": ["EPA 608", "NATE"]
}

// Credential node
"properties": {
  "number": "CAC1234567",
  "issuer": "State of Florida",
  "expires": "2028-03-15",
  "status": "active"
}
```

Vertical profiles (Section 9.3) MAY define required properties for specific node types within that vertical.

---

## 6. Relationships

Relationships encode how nodes connect. They are the difference between a collection of facts and a knowledge graph.

### 6.1 Relation Object

| Field | Type | Required | Description |
|---|---|---|---|
| `source` | String (Node ID) | MUST | Origin node |
| `target` | String (Node ID or URI) | MUST | Destination node |
| `type` | String | MUST | Relation type (see 6.2) |
| `label` | String | MAY | Human-readable description |
| `weight` | Number (0-1) | MAY | Strength of relationship (default: 1.0) |

### 6.2 Relation Types

| Type | Semantics | Example |
|---|---|---|
| `dep` | Source depends on target. Target must be true/valid for source to hold. | A service depends on a license being active |
| `ref` | Source references target. Informational connection. | A FAQ references a service |
| `rel` | Source and target are semantically related. Bidirectional implication. | Two services often purchased together |

### 6.3 Internal References

When both source and target are nodes within the same document, use node IDs:

```json
{
  "source": "kos:cabinet-refinishing",
  "target": "kos:fl-license-cac1234",
  "type": "dep",
  "label": "Requires active state license"
}
```

### 6.4 Cross-Entity References

A node MAY reference a node in another entity's `kos.json`:

```json
{
  "source": "kos:plumbing-service",
  "target": "https://example-partner.test/.well-known/kos.json#kos:partner-service",
  "type": "ref",
  "label": "Recommended electrical partner"
}
```

The target URI format is: `{kos.json URL}#{node-id}`

Cross-entity references create a distributed web of verified knowledge. Consumers MAY follow these references to build a broader knowledge graph.

### 6.5 Inline Relations

Relations MAY be defined inline within a node's `relations` array. When inline, the `source` field is implicit (it is the containing node's ID):

```json
{
  "id": "kos:cabinet-refinishing",
  "type": "service",
  "label": "Cabinet Refinishing",
  "provenance": { "..." },
  "relations": [
    {
      "target": "kos:fl-license-cac1234",
      "type": "dep"
    }
  ]
}
```

This is equivalent to a top-level relation with `"source": "kos:cabinet-refinishing"`.

---

## 7. Provenance Model

Provenance is the core innovation of the KOS protocol. Every node MUST carry provenance metadata that enables machines to compute trust without external verification.

### 7.1 Provenance Object

| Field | Type | Required | Description |
|---|---|---|---|
| `created` | String (ISO 8601) | MUST | When this node was first created |
| `modified` | String (ISO 8601) | MAY | When content was last changed |
| `verified` | String (ISO 8601) | RECOMMENDED | When a human last confirmed accuracy |
| `source` | String | MUST | Who provided this data |
| `method` | String | MAY | How it was verified |
| `confidence` | Number (0-1) | MAY | Publisher's self-assessed confidence |

**Source values** (RECOMMENDED vocabulary):

| Value | Meaning |
|---|---|
| `owner` | The entity owner directly provided/confirmed |
| `employee` | A team member provided/confirmed |
| `public-record` | Derived from government or public databases |
| `third-party` | Provided by an external system or partner |
| `automated` | Generated by an automated process |
| `inferred` | Derived from other nodes via inference |

**Method values** (RECOMMENDED vocabulary):

| Value | Meaning |
|---|---|
| `manual-review` | A human reviewed and confirmed |
| `document-check` | Verified against official documentation |
| `api-sync` | Synchronized from an authoritative API |
| `site-audit` | Verified during a comprehensive audit |
| `self-reported` | Declared by the entity without independent verification |

### 7.2 Freshness Decay Function

Freshness quantifies how reliable a node is *right now*, based on when it was last verified and how quickly that type of data typically goes stale.

**Formula:**

```
freshness(node) = e^(-λ × days_since_verified)
```

Where:
- `e` is Euler's number (~2.71828)
- `λ` (lambda) is the decay rate for the node's type (see Section 5.2)
- `days_since_verified` is the number of days since the `provenance.verified` date

If `verified` is not present, `created` is used as fallback with a 1.5× penalty on the decay rate.

**Examples at default decay rates:**

| Node Type | λ | 7 days | 30 days | 90 days | 180 days | 365 days |
|---|---|---|---|---|---|---|
| `entity` | 0.001 | 0.993 | 0.970 | 0.914 | 0.835 | 0.694 |
| `service` | 0.005 | 0.966 | 0.861 | 0.638 | 0.407 | 0.161 |
| `location` | 0.002 | 0.986 | 0.942 | 0.835 | 0.698 | 0.483 |
| `person` | 0.008 | 0.946 | 0.787 | 0.487 | 0.237 | 0.054 |
| `credential` | 0.001 | 0.993 | 0.970 | 0.914 | 0.835 | 0.694 |
| `content` | 0.010 | 0.932 | 0.741 | 0.407 | 0.165 | 0.026 |
| `event` | 0.050 | 0.705 | 0.223 | 0.011 | 0.000 | 0.000 |
| `concept` | 0.0005 | 0.997 | 0.985 | 0.956 | 0.914 | 0.833 |
| `asset` | 0.003 | 0.979 | 0.914 | 0.763 | 0.583 | 0.336 |

### 7.3 Custom Decay Rate

A producer MAY override the default decay rate for any node:

```json
"provenance": {
  "created": "2026-01-15",
  "verified": "2026-03-28",
  "source": "owner",
  "decay_rate": 0.003
}
```

When `decay_rate` is present, it overrides the default for that node's type.

### 7.4 Document-Level Freshness

The `meta.avg_freshness` value is the arithmetic mean of all node freshness values at generation time. Consumers SHOULD recompute freshness at consumption time rather than relying solely on this cached value.

---

## 8. Validation

### 8.1 Structural Requirements

A valid `kos.json` document:

1. MUST be valid JSON (RFC 8259)
2. MUST contain `$schema`, `entity`, `knowledge`, and `meta` at the root
3. MUST have at least one node in `knowledge`
4. Each node MUST have `id`, `type`, `label`, and `provenance`
5. Each `provenance` MUST have `created` and `source`
6. Node `id` values MUST be unique within the document
7. `$schema` MUST reference a recognized KOS schema version
8. `meta.version` MUST be `"1.0"` for this specification

### 8.2 Referential Integrity

1. All `source` values in relations MUST reference a valid `id` in the `knowledge` array (unless the relation is inline, where source is implicit)
2. All `target` values MUST reference either:
   - A valid `id` in the `knowledge` array, OR
   - A valid external URI following the format in Section 6.4
3. Consumers SHOULD warn on broken internal references
4. Consumers SHOULD NOT fail on broken external references (the remote resource may be temporarily unavailable)

### 8.3 Size Recommendations

- Nodes: No hard limit. RECOMMENDED under 500 nodes for optimal consumption.
- File size: RECOMMENDED under 1MB. Files over 5MB SHOULD provide a paginated alternative.
- Description fields: RECOMMENDED under 1000 characters per node.

---

## 9. Extensions

### 9.1 Custom Node Types

Producers MAY define node types beyond the core taxonomy. Custom types MUST be namespaced:

```json
{
  "id": "kos:listing-123",
  "type": "realestate:listing",
  "label": "123 Main St, Anytown",
  "provenance": { "..." }
}
```

The namespace format is `{vertical}:{type}`. Unnamespaced types are reserved for the core taxonomy.

### 9.2 Custom Relation Types

Similarly, custom relation types MUST be namespaced:

```json
{
  "source": "kos:listing-123",
  "target": "kos:agent-jane",
  "type": "realestate:listed-by"
}
```

### 9.3 Vertical Profiles

A vertical profile is a published extension that defines:

1. Additional required node types for a domain
2. Required properties for those node types
3. Custom relation types
4. Domain-specific validation rules

Vertical profiles are identified by a namespace and a schema URI:

```json
{
  "$schema": "https://kosprotocol.dev/v1/schema.json",
  "$profile": "https://kosprotocol.dev/verticals/local-business/v1",
  "entity": { "..." }
}
```

The `$profile` field is OPTIONAL. When present, validators SHOULD apply the additional rules defined by that profile.

**Initial verticals (planned):**

| Vertical | Namespace | Domain |
|---|---|---|
| Local Business | `localbiz` | Services, areas, credentials, hours |
| Real Estate | `realestate` | Listings, agents, transactions |
| Healthcare | `health` | Practices, providers, specialties |
| Legal | `legal` | Practice areas, jurisdictions, bar admissions |
| Food & Hospitality | `food` | Menus, locations, reservations |
| E-commerce | `ecom` | Products, inventory, shipping |

### 9.4 Schema.org Mapping

Nodes MAY include a `schema_org` property to declare equivalence with Schema.org types:

```json
{
  "id": "kos:hvac-repair",
  "type": "service",
  "label": "HVAC Repair",
  "schema_org": "https://schema.org/HomeAndConstructionBusiness",
  "provenance": { "..." }
}
```

This enables consumers to bridge KOS data into existing Schema.org infrastructure and generate JSON-LD markup from a `kos.json` source.

---

## 10. Security Considerations

### 10.1 No Authentication Required

`kos.json` is a public document by design. It represents knowledge an entity chooses to publish. Sensitive internal data MUST NOT be included.

### 10.2 Integrity Verification

Producers MAY include a cryptographic signature in the `meta` object:

```json
"meta": {
  "signature": {
    "algorithm": "ed25519",
    "public_key": "...",
    "value": "..."
  }
}
```

This allows consumers to verify that the document has not been tampered with. The signature SHOULD cover the complete document excluding the `meta.signature` object itself.

### 10.3 Provenance Claims

Provenance is self-reported. A `source: "owner"` claim means the producer asserts that the owner verified the data. The protocol does not provide independent verification — it provides **auditable claims**.

Consumers SHOULD treat provenance as trust signals, not guarantees. A node verified by `owner` via `document-check` is more trustworthy than one from `automated` via `inferred`, but neither is independently proven.

The value of provenance is not absolute truth — it is **computability of trust**. Machines can make informed decisions rather than treating all data equally.

---

## 11. MIME Type and Conventions

- **MIME type:** `application/kos+json` (preferred) or `application/json`
- **File extension:** `.kos.json` for standalone files; `kos.json` at well-known URI
- **Character encoding:** UTF-8 (MUST)
- **Date format:** ISO 8601 (MUST). Example: `"2026-04-04"` or `"2026-04-04T14:30:00Z"`
- **ID format:** `kos:{slug}` where slug is lowercase alphanumeric with hyphens. Example: `kos:cabinet-refinishing`
- **Minimum valid document:** See Appendix A

---

## 12. Autonomous Generation

This section defines how AI systems SHOULD generate `kos.json` files autonomously. It is the algorithmic counterpart to the structural specification — where Sections 1–11 define what a valid document looks like, Section 12 defines how to produce one.

The generation rules are embedded in the protocol intentionally. Any AI system that reads this specification has everything it needs to generate valid `kos.json` for any entity, in any industry, without additional software or human instruction.

### 12.1 Generation Principles

1. **The spec is the algorithm.** No external code is required. An AI system that has read this specification can generate `kos.json` from any available information about an entity. The protocol is self-executing.

2. **Honesty over completeness.** A generated `kos.json` with 5 nodes at honest confidence is more valuable than 50 nodes at inflated confidence. Never claim higher provenance than the data supports.

3. **Universal by design.** The generation process is domain-agnostic. The same rules apply to a restaurant, a law firm, a software product, or a person. The AI's contextual understanding handles domain adaptation; the protocol handles structure.

4. **Autonomous improvement.** As AI systems improve, the quality of generated `kos.json` improves — without any change to the protocol. The spec is stable; the runtime evolves.

### 12.2 Generation Algorithm

When generating a `kos.json` for an entity, an AI system SHOULD follow this sequence:

**Step 1 — Identify the Entity**

Determine the entity's canonical name, type, primary URL, and unique identifier. Map to the closest `entity.type` value.

**Step 2 — Discover Knowledge**

Gather information from available sources, in order of authority:

| Priority | Source | Typical confidence |
|---|---|---|
| 1 (highest) | Entity's own website (about page, services, team) | 0.5 |
| 2 | Structured data already on the site (Schema.org JSON-LD) | 0.6 |
| 3 | Google Business Profile or equivalent directory listing | 0.5 |
| 4 | Public records (licenses, incorporations, registrations) | 0.7 |
| 5 | Social media profiles (LinkedIn, Facebook, Instagram) | 0.4 |
| 6 | Review platforms (Google Reviews, Yelp) | 0.3 |
| 7 | Third-party references (articles, directories, mentions) | 0.3 |

**Step 3 — Map to Typed Nodes**

For each distinct piece of knowledge discovered, create a node with the appropriate type:

| Information found | Maps to node type |
|---|---|
| A service or offering described | `service` |
| A physical address, region, or coverage area | `location` |
| A license, certification, award, or accreditation | `credential` |
| A team member, owner, or key person | `person` |
| A product for sale or distribution | `product` |
| An FAQ, article, or published guide | `content` |
| A scheduled or time-bound occurrence | `event` |
| A philosophy, methodology, or framework | `concept` |
| A tool, system, or resource used | `asset` |

Each node MUST have a unique `id` in `kos:{slug}` format, a `label`, and `provenance`.

**Step 4 — Assign Provenance Honestly**

This is the most critical step. Generated provenance MUST reflect reality:

```
source: "automated"        — always, for AI-generated nodes
method: "web-scrape"       — if extracted from website content
method: "structured-parse" — if extracted from existing Schema.org/JSON-LD
method: "api-sync"         — if obtained from an API (GBP, public records)
method: "cross-reference"  — if confirmed across multiple sources
method: "inferred"         — if derived from context rather than explicit statement
```

**Confidence scoring based on source convergence:**

| Sources agreeing | Confidence | Rationale |
|---|---|---|
| Single source, unstructured | 0.3 | One mention, no confirmation |
| Single source, structured (Schema.org, GBP) | 0.5 | Structured implies intentional publication |
| Two independent sources agree | 0.5 | Convergence begins |
| Three or more sources agree | 0.7 | Strong convergence |
| Public record confirms | 0.7 | Authoritative source |
| **Maximum for automated generation** | **0.8** | **Human verification required to exceed** |

An AI-generated `kos.json` MUST NEVER assign `confidence` above 0.8. The range 0.8–1.0 is reserved for human-verified data. This boundary is a core integrity rule of the protocol.

**Step 5 — Infer Relationships**

Identify connections between nodes using contextual understanding:

| Pattern detected | Relation type |
|---|---|
| Service requires a license/credential | `dep` |
| FAQ answers a question about a service | `ref` |
| Service is available in a location | `ref` |
| Person holds a credential | `dep` |
| Person delivers a service | `rel` |
| Two services are often combined | `rel` |
| Content explains a concept | `ref` |

Relationships SHOULD only be created when the connection is evident from the source data. Do not infer relationships that are speculative.

**Step 6 — Assemble and Validate**

1. Construct the complete `kos.json` document per Sections 3–6
2. Compute `meta.avg_freshness` and `meta.stale_count`
3. Set `meta.generator` to identify the AI system and model
4. Validate against Section 8 (structural requirements and referential integrity)
5. Output only if valid

**Step 7 — Signal Draft Status**

AI-generated documents SHOULD include a `meta.draft` field:

```json
"meta": {
  "version": "1.0",
  "generated": "2026-04-04T10:00:00Z",
  "generator": "Claude/opus-4-6",
  "draft": true,
  "awaiting_verification": true
}
```

The `draft: true` flag signals to consuming systems that this document has not been verified by the entity owner. Consumers SHOULD weight draft documents lower than verified ones but MAY still use them as the best available structured data.

### 12.3 Verification Escalation

A `kos.json` file evolves through verification levels. Each level increases trust:

```
Level 0 — DRAFT (automated, single source)
  confidence: 0.2–0.4
  source: "automated", method: "web-scrape"
  
Level 1 — CROSS-REFERENCED (automated, multiple sources agree)
  confidence: 0.4–0.7
  source: "automated", method: "cross-reference"
  
Level 2 — CLAIMED (entity acknowledges the kos.json)
  confidence: 0.6–0.8
  source: "owner", method: "self-reported"
  
Level 3 — VERIFIED (entity reviews and confirms each node)
  confidence: 0.8–0.95
  source: "owner", method: "manual-review"
  
Level 4 — AUDITED (third-party or multi-source deep verification)
  confidence: 0.95–1.0
  source: "owner" + "public-record", method: "document-check"
```

The incentive to escalate is built into the protocol: higher verification level → higher freshness confidence → AI agents weight the data more heavily → greater visibility in AI-generated answers.

No external reward system is needed. The protocol's trust mechanics create a natural incentive for entities to verify their knowledge. This mirrors the economic incentive structure of proof-of-work systems, where participation is rewarded by the protocol itself.

### 12.4 Cross-Entity Validation

When multiple `kos.json` files reference each other, consuming systems can perform cross-validation:

- Entity A says it partners with Entity B → Check Entity B's `kos.json` for reciprocal reference
- Entity A claims credential X → Check public records for confirmation
- Entity A claims service in Location Y → Check if Entity A appears in Location Y directories

Convergent references increase confidence. Contradictory references decrease it. This creates a self-validating network where the accuracy of one entity's knowledge graph is influenced by the broader knowledge network.

### 12.5 Why Generation is Part of the Specification

In most protocols, the specification defines validity and leaves production to external tools. KOS embeds generation rules in the specification for a specific reason: **the specification must be self-executing.**

Any AI system that reads this document should be able to both validate AND generate `kos.json` files. This eliminates dependency on any specific tool, vendor, or implementation. The protocol propagates through AI systems that consume it — each one becomes a potential generator.

This is by design. The protocol does not depend on its creator, a company, a platform, or a community to produce `kos.json` files. It depends only on the existence of AI systems that can read and follow instructions — a condition that is permanently true.

---

## 13. Consumption Guidelines

### 13.1 For AI Agents Consuming kos.json

When an AI agent encounters a `kos.json` file (via discovery mechanisms in Section 3), it SHOULD:

1. **Prefer kos.json over scraped data.** Structured, provenance-tracked knowledge is more reliable than HTML parsing.
2. **Recompute freshness at consumption time.** Do not rely on cached `meta.avg_freshness`. Apply the decay formula (Section 7.2) using the current date.
3. **Weight answers by confidence.** Nodes with higher provenance confidence and freshness SHOULD be preferred when multiple sources conflict.
4. **Cite the source.** When using data from a `kos.json`, reference the entity and ideally the specific node.
5. **Follow cross-entity references.** If a node references another entity's `kos.json`, follow the link to build a broader knowledge graph.
6. **Respect draft status.** Documents with `meta.draft: true` are unverified. Use them as best-available data but flag lower confidence to the end user.
7. **Cache responsibly.** `kos.json` files change when entities update their knowledge. Re-fetch periodically based on the document's average freshness.

### 13.2 Discovery Behavior

AI agents SHOULD proactively check for `kos.json` when researching any entity:

1. Attempt `https://{domain}/.well-known/kos.json`
2. If not found, check `<link rel="knowledge">` in the HTML `<head>`
3. If not found, proceed with standard research methods

This design creates an organic incentive: entities with `kos.json` can get better, more accurate AI-generated answers about them. Entities without `kos.json` get scraped, inferred, and potentially hallucinated answers. Note: as of v1.0, no major AI system has implemented kos.json discovery. This section describes the intended behavior, not current industry practice.

---

## 14. Rationale

### Why not extend Schema.org?

Schema.org describes content on web pages. KOS describes knowledge about entities. A business's Schema.org markup says "this page is about an HVAC company." A business's `kos.json` says "here is everything this HVAC company knows about itself — every service, every credential, every relationship, every verification timestamp — in one machine-readable graph."

Schema.org is a vocabulary for annotating documents. KOS is a format for publishing knowledge.

They are complementary. A `kos.json` file can generate Schema.org markup (via `schema_org` mappings). But Schema.org cannot express provenance, typed relationships, or freshness decay.

### Why JSON and not RDF/Turtle/N-Quads?

Pragmatism. JSON is the lingua franca of web APIs and AI systems. Every programming language, every AI framework, every web platform reads JSON natively. RDF is more expressive but creates adoption friction that would prevent the protocol from reaching critical mass.

### Why local files and not a central registry?

Sovereignty. The entity controls its knowledge. No intermediary decides what gets published. No platform owns the data. This mirrors the architecture of the web itself — decentralized by design.

---

## Appendix A: Minimum Valid Document

```json
{
  "$schema": "https://kosprotocol.dev/v1/schema.json",
  "entity": {
    "name": "Example Business",
    "type": "Organization",
    "id": "kos:example-business"
  },
  "knowledge": [
    {
      "id": "kos:main-service",
      "type": "service",
      "label": "Main Service",
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

---

## Appendix B: Freshness Quick Reference

Consumer-recommended thresholds:

| Freshness | Interpretation | Recommended action |
|---|---|---|
| 0.90 — 1.00 | Fresh. High confidence. | Use freely |
| 0.70 — 0.89 | Good. Minor staleness risk. | Use with standard confidence |
| 0.50 — 0.69 | Aging. Moderate staleness risk. | Use with reduced confidence; flag to user |
| 0.25 — 0.49 | Stale. Significant risk of inaccuracy. | Use only if no fresher source exists |
| 0.00 — 0.24 | Expired. Data is likely outdated. | Do not cite without independent verification |

---

## Appendix C: Change Log

| Version | Date | Changes |
|---|---|---|
| 1.0.0-draft | 2026-04-04 | Initial specification |

---

*Copyright (c) 2026 Niseus Authority Systems (niseus.com). All rights reserved.*

*This specification is maintained by Niseus Authority Systems. Contributions, implementations, and feedback are welcome.*

*The protocol is dedicated to the principle that knowledge should be structured, verifiable, and free.*

*"KOS Protocol" and "kos.json" are trademarks of Niseus Authority Systems. Implementation of the protocol is free and open. Modification and republication of the specification under the "KOS Protocol" name requires written permission. See LICENSE for full terms.*
