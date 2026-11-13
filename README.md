# Country Calling Codes MCP

Metadata-only listing repository for the hosted Country Calling Codes MCP
server (service **1.4.0**).


| Agent skill | [Kajdep/countrycalling-codes-skill](https://github.com/Kajdep/countrycalling-codes-skill) |

Official registry name: `codes.countrycalling/countrycallingcodes`.

## Hosted MCP

- Endpoint: `https://www.countrycalling.codes/api/mcp`
- Transport: Streamable HTTP
- Authentication: none
- Protocol: `2025-11-25`
- Tools: 11 read-only tools (see below)
- Resources: `countrycalling://capabilities`, `countrycalling://coverage`, and the template `countrycalling://countries/{code}/phone-input`
- Website: `https://www.countrycalling.codes`
- Developer docs: `https://www.countrycalling.codes/developers#mcp`
- LLM guide: `https://www.countrycalling.codes/llms-full.md`
- Skill manifest: `https://www.countrycalling.codes/skill.json`
- OpenAPI: `https://www.countrycalling.codes/api/calling-codes/openapi.json`
- Status: `https://www.countrycalling.codes/api/calling-codes/status`
- Registry manifest (live): `https://www.countrycalling.codes/server.json`

## Guides

Reviewed articles on the website that complement the MCP tools:

- [International dialing format (E.164)](https://www.countrycalling.codes/guides/international-dialing-format)
- [Exit codes and trunk prefixes](https://www.countrycalling.codes/guides/exit-codes-and-trunk-prefixes)
- [Phone number validation](https://www.countrycalling.codes/guides/phone-number-validation)
- [Common phone number formatting errors](https://www.countrycalling.codes/guides/common-phone-number-formatting-errors)
- [Missed-call and wangiri scams](https://www.countrycalling.codes/guides/missed-call-scams)

The full set is at [countrycalling.codes/guides](https://www.countrycalling.codes/guides).

The `server.json` in this repository is the same document the website serves
and the official MCP Registry already publishes as 1.4.0. Registry publication
is driven from the implementation repo (`.github/workflows/publish-mcp.yml` in
`v0-countrycallingcodes`), not from this listing.

## What it does

Read-only international phone workflows for apps and agents:

- Country and territory calling-code lookup (ISO codes, E.164 dial codes, regions, trunk prefixes, NANP)
- Phone analysis and ephemeral formatting against versioned numbering metadata
- Origin-specific dialing instructions for reviewed country pairs
- Batch normalization (up to 100 records) with duplicate grouping
- Number comparison, region resolution, form-input hints, and format diagnosis
- Overlapping calling windows across explicit IANA time zones

Every MCP tool has a REST twin that returns the same structured payload. Results
carry source `evidence` and a `limitations` array. A well-formed number is not
proof that it is allocated, active, reachable, owned, or consented for contact.

Agent-facing usage rules (when to call which tool, how to treat ambiguity, how
to clean a CRM column) live in the [skill](https://github.com/Kajdep/countrycalling-codes-skill),
not here.

## MCP client configuration

For clients that support remote Streamable HTTP servers — this is the same
config the skill and the live discovery document use:

```json
{
  "mcpServers": {
    "country-calling-codes": {
      "type": "streamable-http",
      "url": "https://www.countrycalling.codes/api/mcp"
    }
  }
}
```

For clients that require a local stdio bridge, use `mcp-remote`:

```json
{
  "mcpServers": {
    "country-calling-codes": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://www.countrycalling.codes/api/mcp"
      ]
    }
  }
}
```

A ready-to-paste copy is in [`mcp-config.json`](./mcp-config.json).

## Tools

All tools are annotated `readOnlyHint`, non-destructive, idempotent, and
closed-world. Phone strings are capped at 64 characters; tool arguments at
64 KiB; the JSON-RPC envelope at 128 KiB.

| MCP tool | REST twin | Purpose |
| --- | --- | --- |
| `lookup_country_calling_code` | `GET /api/calling-codes` | Free-text / ISO / dial-code / region lookup across 233 countries and territories. Set `includeExamples` for reviewed patterns. |
| `format_international_phone_number` | `POST /api/phone/format` | Lighter ephemeral formatter. Prefer `analyze_phone_number` when you need status, checks, candidates, or evidence. |
| `get_phone_capabilities` | `GET /api/capabilities` | Per-country review status, supported operations, evidence, dataset versions, and request limits. Call this first for an unfamiliar country. |
| `analyze_phone_number` | `POST /api/phone/analyze` | Strict parse against versioned numbering metadata. National numbers need `numberCountryCode`; exit prefixes need `originCountryCode`. |
| `get_dialing_instructions` | `POST /api/dialing/instructions` | Origin-specific dial string, steps, and pattern for reviewed country pairs. |
| `normalize_phone_numbers` | `POST /api/phone/batch` | Up to 100 uniquely identified records, order preserved, per-record errors, optional canonical duplicate grouping. |
| `compare_phone_numbers` | `POST /api/phone/compare` | Equality after analysis. `extensionPolicy` is `exact` by default; suffix-only matches are never equal. |
| `resolve_phone_region` | `POST /api/phone/region` | Metadata-backed country candidates or non-geographic classification. Not geolocation or carrier lookup. |
| `get_phone_input_rules` | `GET /api/countries/{code}/phone-input` | Display placeholders and trunk-prefix guidance. Not a validation mask. |
| `diagnose_dialing_problem` | `POST /api/phone/diagnose` | Explain a formatting failure and propose a bounded, reviewed prefix repair for confirmation. |
| `find_calling_windows` | `POST /api/calling-windows` | Overlapping working hours for up to 8 participants, explicit IANA zones, 1–31 days, DST-aware. |

Example (`lookup_country_calling_code`):

```json
{
  "code": "GB",
  "includeExamples": true
}
```

Expected result includes the United Kingdom record with dial code `+44`.

Full parameter and response shapes belong in the
[skill API reference](https://github.com/Kajdep/countrycalling-codes-skill/blob/main/references/api.md)
and the [hosted OpenAPI](https://www.countrycalling.codes/api/calling-codes/openapi.json).
Do not fork a third copy of those contracts in this repository.

## Resources

| URI | Contents |
| --- | --- |
| `countrycalling://capabilities` | Full capability document (same payload as `GET /api/capabilities`) |
| `countrycalling://coverage` | Coverage view of the same document |
| `countrycalling://countries/{code}/phone-input` | Reviewed input rules for one ISO alpha-2 country |

## Coverage and limits

- Numbering metadata: libphonenumber-js 1.13.12, max metadata, 245 countries and territories plus supported non-geographic numbering.
- Reviewed dialing guides and input examples: AE, AU, BR, CN, DE, FR, GB, IN, IT, JP, SG, US, ZA. Dialing instructions require both origin and destination to be reviewed.
- Directory dataset: 233 countries and territories.
- No SMS, calls, HLR/HSS, MNP, carrier lookup, contact mutation, or outbound data submission.
- Phone-bearing and MCP transport responses are `no-store`. No application-level per-client quota; honour `429` + `Retry-After` if hosting enforces one.

## Directory submission metadata

```text
Name: Country Calling Codes
Description: Calling codes, phone analysis, dialing guidance, batch normalization and calling windows.
Category: Data / Search / Reference / API
Website: https://www.countrycalling.codes
MCP URL: https://www.countrycalling.codes/api/mcp
Registry: codes.countrycalling/countrycallingcodes
Version: 1.4.0
OpenAPI URL: https://www.countrycalling.codes/api/calling-codes/openapi.json
Skill: https://github.com/Kajdep/countrycalling-codes-skill
Skill manifest: https://www.countrycalling.codes/skill.json
LLM guide: https://www.countrycalling.codes/llms-full.md
Implementation: https://github.com/Kajdep/v0-countrycallingcodes
GitHub listing: https://github.com/Kajdep/countrycallingcodesmcp
Tools: 11 (see table above)
Auth: None
Cost to user: Free/public read-only lookup and analysis
```

## Data and privacy

This repository contains no private data and no dataset export. The hosted MCP
server is read-only. Phone values are processed ephemerally and are not stored
by the service; MCP clients and third-party gateways may retain their own
conversation history.
