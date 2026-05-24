# Country Calling Codes MCP

Metadata-only submission repository for the hosted Country Calling Codes MCP
server.

This repository intentionally does not include the website source code or the
full country calling-code dataset. It exists so MCP directories and registries
can discover, inspect, and list the public hosted MCP endpoint.

## Hosted MCP

- MCP endpoint: `https://countrycalling.codes/api/mcp`
- Transport: Streamable HTTP
- Authentication: none
- Tool: `lookup_country_calling_code`
- Website: `https://countrycalling.codes`
- LLM guide: `https://countrycalling.codes/llms-full.md`
- Skill manifest: `https://countrycalling.codes/skill.json`
- OpenAPI schema: `https://countrycalling.codes/api/calling-codes/openapi.json`

## What It Does

The MCP server provides read-only lookup access to international telephone
calling-code reference data:

- Country and territory names
- ISO 3166-1 alpha-2 and alpha-3 codes
- E.164 international dial codes
- Regions and subregions
- Trunk prefixes where relevant
- NANP area-code disambiguation where relevant

## MCP Client Configuration

For MCP clients that support remote Streamable HTTP servers:

```json
{
  "mcpServers": {
    "country-calling-codes": {
      "url": "https://countrycalling.codes/api/mcp"
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
        "https://countrycalling.codes/api/mcp"
      ]
    }
  }
}
```

## Tool

### `lookup_country_calling_code`

Look up countries and territories by free-text query, ISO code, E.164 dial code,
or region.

Input parameters:

- `q`: free-text search, such as `United Kingdom`, `London`, or `AUS`
- `code`: ISO 3166-1 alpha-2 or alpha-3 code, such as `GB` or `GBR`
- `dialCode`: calling code with or without plus sign, such as `+44` or `44`
- `region`: one of `Africa`, `Americas`, `Asia`, `Europe`, `Oceania`
- `limit`: maximum results to return, from `1` to `250`

Example tool input:

```json
{
  "code": "GB"
}
```

Expected result includes the United Kingdom record with dial code `+44`.

## Directory Submission Metadata

```text
Name: Country Calling Codes
Description: Read-only MCP server and JSON API for international country calling codes, ISO country codes, E.164 dial codes, regions, trunk prefixes, and NANP area codes.
Category: Data / Search / Reference / API
Website: https://countrycalling.codes
MCP URL: https://countrycalling.codes/api/mcp
OpenAPI URL: https://countrycalling.codes/api/calling-codes/openapi.json
Skill manifest: https://countrycalling.codes/skill.json
LLM guide: https://countrycalling.codes/llms-full.md
GitHub: https://github.com/Kajdep/countrycallingcodesmcp
Tool: lookup_country_calling_code
Auth: None
Cost to user: Free/public read-only lookup
```

## Data and Privacy

This repository contains no private data and no full dataset export. The hosted
MCP server is read-only and returns public reference data from
`countrycalling.codes`.

