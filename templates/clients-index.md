# Client index

Each ad account maps to one client doc, which holds that client's audience. Update the doc after every run.
Saved audiences aren't usable via the API; the doc's mirrored targeting (with IDs) is the audience.

## Active / managed
| Client | Ad account | ID | Currency | Doc | Sheet tab |
|---|---|---|---|---|---|
| <Client> | <account name> | <ID> | USD | <client>.md | <tab name> |

## Past / dormant (all in dormant-clients.md until one becomes active again)
| Client | Ad account | ID |
|---|---|---|

## Operator's own accounts (testing, customer issues)
- <account name> (<ID>): <what it's used for>

## Misc / not client builds
- <account> (<ID>): <note, e.g. DO NOT TOUCH>

## Not accessible via MCP
- <account>: <reason: unsettled / disabled / not rolled out>

## Built by others (not your discipline)
- <account>: <what to ignore>
