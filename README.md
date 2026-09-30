# Response Format Negotiation for Multi-Agent Communication

A minimal, dependency-free demonstration of how agents in a multi-agent system stay interoperable when they don't natively speak the same data format — by declaring capabilities, negotiating a shared format at runtime, and auto-converting payloads (JSON ↔ XML) when needed.

A reference demo inspired by my Virginia Tech Agentic AI System Design program (Lesson 6, agent-to-agent design). It runs on sample data only.

## What this shows

- **Capability declaration**: each agent states which formats it supports and in what preference order (metadata, not code)
- **Negotiation**: a `negotiate()` function finds the overlap between sender and receiver supported formats, honoring sender preference first, then receiver preference, before falling back to any shared format
- **Conversion**: when the negotiated format differs from the sender's native format, the payload is decoded to a plain dict and re-encoded in the agreed format — shown working in both directions (XML→JSON and JSON→XML)
- **Safe fallback**: when there's no common format at all, the exchange aborts and logs the reason rather than guessing or silently corrupting the payload

No external packages, no API keys, no network calls — everything runs on Python's standard library (`json`, `xml.etree.ElementTree`, `typing`).

## Prerequisites

- Python 3.10+ (no other dependencies)

## Files

| File | Scenario |
|---|---|
| `format_negotiation_demo.py` | Base case — sender prefers XML, receiver is JSON-only. Negotiation agrees on JSON, the XML payload is converted, and delivery succeeds. Also runs a second pair of agents with no format overlap at all, showing the fallback path. |
| `format_negotiation_json_to_xml.py` | Reversed case — sender prefers JSON, receiver is XML-only. Negotiation agrees on XML, the JSON payload is converted the other direction. Also re-runs the same no-overlap fallback check. |

(Negotiation was also verified to hold symmetrically — running the XML→JSON base case a second time produced identical results, confirming the logic isn't direction-dependent.)

## Usage

```
python format_negotiation_demo.py
python format_negotiation_json_to_xml.py
```

Each script prints the full exchange to the terminal: the negotiation decision, the conversion step (if any), the payload before and after conversion, and the delivery result.

## Example output (base case)

```
=== Success path (mismatch auto-heals) ===
NEGOTIATE: sender=A, receiver=B, sender_supported=['XML', 'JSON'], receiver_supported=['JSON'], agreed=JSON
CONVERT: XML -> JSON
PAYLOAD BEFORE (XML):
<message><id>101</id><title>Quarterly Report</title><amount>123.45</amount></message>
PAYLOAD AFTER (JSON):
{"id":101,"title":"Quarterly Report","amount":123.45}
DELIVER: content_type=application/json, ok=True
RESULT: {'ok': True, 'content_type': 'application/json', 'agreed_format': 'JSON'}

=== Fallback path (no overlap) ===
NEGOTIATE: sender=C, receiver=D, sender_supported=['XML'], receiver_supported=['JSON'], agreed=None
FALLBACK: No common format. Action=abort_and_log
RESULT: {'ok': False, 'reason': 'no_common_format'}
```

## Notes

- The converters here are demo-grade: XML values round-trip through Python's `float`/`int` parsing, so they're fine for flat key-value messages like the one used here but would need a real schema (e.g. XSD) for nested or typed production data.
- The negotiation logic treats format names case-insensitively and doesn't assume any particular agent is authoritative — either side's preference order can win, depending on overlap.

## Next steps

- Extend the message payload to nested structures and adapt the XML converter accordingly
- Layer this negotiation pattern on top of the Agent-to-Agent (A2A) protocol's Agent Card capability declarations from the same lesson
