---
tags: [reference, kql]
---

# KQL Cheat Sheet

Every query starts with a table, then pipes (`|`) results through operators, top to bottom.

```kusto
TableName
| where column == "value"
| project col1, col2
| order by col1 asc
```

## Exploring a table

| Goal | Query |
|---|---|
| See the columns and their types | `DnsEvents \| getschema` |
| See a few sample rows | `DnsEvents \| take 5` |
| Count rows | `DnsEvents \| count` |

> **`take` returns rows in no set order.** Use `sort by timestamp asc` (or `order by`) when order matters.

## Filtering with `where`

| Operator | Meaning | Example |
|---|---|---|
| `==` | Exact match (case-sensitive) | `where name == "DNS-Server"` |
| `!=` | Not equal | `where vlan != "workstations"` |
| `in (...)` | Exact match to **any** value in a list | `where ip_address in ("10.10.16.14", "10.10.16.7")` |
| `has` | Contains a whole word/term (fast) | `where resolved_ips has "185.174.137.42"` |
| `has_any (...)` | Contains **any** of these terms | `where email_addr has_any ("alex", "jessica")` |
| `has_all (...)` | Contains **all** of these terms **in the same field** | `where subject has_all ("invoice", "urgent")` |
| `contains` | Contains a substring anywhere (slower, catches partial matches) | `where query_name contains "update-cdn-service.xyz"` |
| `startswith` | Begins with | `where ip_address startswith "10.10.1."` |
| `endswith` / `!endswith` | Ends with / doesn't end with | `where sender !endswith "@whiskersandwonders.org"` |
| `between (a .. b)` | Inside a time range | `where timestamp between (datetime(2025-06-03 08:50) .. datetime(2025-06-03 09:00))` |

### Common mistakes I've made

- **One string with commas is still one string.** `has_all ("a, b, c")` searches for the whole text `a, b, c`. Each value needs its own quotes: `in ("a", "b", "c")`.
- **`has_all` across rows doesn't work.** Each row has one IP, so `has_all` with three IPs matches nothing. Use `in`.
- **Columns differ between tables.** `domain`, `url`, `method`, `status_code` are ProxyEvents columns, not DnsEvents. Check with `getschema` or `take 1`.
- **`contains` vs `==`.** `contains "update-cdn-service.xyz"` also catches `www.update-cdn-service.xyz`; `==` doesn't.

## Shaping results

| Operator | What it does | Example |
|---|---|---|
| `project` | Keep only these columns | `project timestamp, src_ip, url` |
| `distinct` | Unique values | `distinct client_ip` |
| `summarize count() by` | Count per group | `summarize count() by client_ip` |
| `summarize name=count() by` | Count with a custom column name | `summarize device_count=count() by vlan` |
| `summarize min()` / `max()` | Earliest / latest per group | `summarize first_query=min(timestamp) by client_ip` |
| `order by` / `sort by` | Sort (`asc` or `desc`) | `order by count_ desc` |
| `take` / `limit` | First N rows (no set order) | `take 10` |

> `summarize count()` names its column `count_`, so sort with `order by count_ desc`.

## Time

| Goal | Query |
|---|---|
| Whole hours between two times | `print hours = datetime_diff('hour', datetime(2025-06-02 23:52:31), datetime(2025-06-02 16:00:00))` → **7** |
| Time range filter | `where timestamp between (datetime(2025-06-02) .. datetime(2025-06-08))` |

> `datetime_diff` counts **whole units crossed**, not rounded time. 7 h 52 m returns 7, not 8.

## Investigation patterns

**IP → domain** (what name points to this IP?)
```kusto
DnsEvents
| where resolved_ips has "185.174.137.42"
| distinct query_name
```

**Domain → which machines looked it up, and when first**
```kusto
DnsEvents
| where query_name contains "update-cdn-service.xyz"
| summarize first_query=min(timestamp), lookups=count() by client_ip
| order by first_query asc
```

**Did they actually connect?**
```kusto
NetworkFlow
| where dest_ip == "185.174.137.42"
| project timestamp, src_ip, dest_ip, dest_port, protocol, bytes
| sort by timestamp asc
```

**What did they request?**
```kusto
ProxyEvents
| where domain == "update-cdn-service.xyz"
| project timestamp, src_ip, url, method, status_code
| order by timestamp asc
```

**Lookalike sender domain**
```kusto
Email
| where sender !endswith "@whiskersandwonders.org"
| where sender contains "whiskersandwonders"
| project timestamp, sender, recipient, subject
```

**IP → machine → person**
```kusto
DeviceInfo
| where ip_address in ("10.10.16.14", "10.10.16.7", "10.10.16.24")
```

## KC7 tables I've used

| Table | What it holds | Columns I've used |
|---|---|---|
| Employees | Staff: names, emails, roles, machines | `name`, `email_addr`, `role` |
| Email | Every email sent/received | `timestamp`, `sender`, `recipient`, `subject` |
| DeviceInfo | Every device on the network | `name`, `ip_address`, `vlan`, `host_type` |
| DnsEvents | Every DNS lookup | `timestamp`, `client_ip`, `query_name`, `query_type`, `resolved_ips`, `ttl`, `dns_server` |
| NetworkFlow | Connections between IPs | `timestamp`, `src_ip`, `dest_ip`, `dest_port`, `protocol`, `bytes` |
| ProxyEvents | Web requests through the proxy | `timestamp`, `src_ip`, `domain`, `url`, `method`, `status_code` |
| PassiveDns | Outside DNS history: which domains have pointed to an IP | |
| AuthenticationEvents | Logins | |
| FileCreationEvents | Files written to disk | |
| ProcessEvents | Programs that ran | |
| InboundNetworkEvents / OutboundNetworkEvents | Web traffic in and out (older KC7 modules) | |

> **DnsEvents vs PassiveDns:** DnsEvents = lookups made by *the company's* machines. PassiveDns = outside DNS history for research on attacker infrastructure.
