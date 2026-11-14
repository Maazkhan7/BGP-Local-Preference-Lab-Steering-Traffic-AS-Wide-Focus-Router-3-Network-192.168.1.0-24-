<div align="center">

# 🌐 BGP Local Preference Lab — Steering Traffic AS-Wide (Focus: Router 3, Network 192.168.1.0/24)

### EBGP + Full-Mesh IBGP | AS 10 ⇄ AS 20 | `bgp default local-preference 200` | GNS3 + Cisco IOS

![Cisco](https://img.shields.io/badge/Cisco-IOS-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![BGP](https://img.shields.io/badge/Protocol-BGP-orange?style=for-the-badge)
![Attribute](https://img.shields.io/badge/Attribute-LOCAL__PREF-yellow?style=for-the-badge)
![Default](https://img.shields.io/badge/Default-100-lightgrey?style=for-the-badge)
![Lab Value](https://img.shields.io/badge/Lab%20Value-200-brightgreen?style=for-the-badge)
![Scope](https://img.shields.io/badge/Scope-AS--Wide%20(IBGP)-blueviolet?style=for-the-badge)
![RIB](https://img.shields.io/badge/Flag-r%20RIB--Failure-critical?style=for-the-badge)
![Lab](https://img.shields.io/badge/Lab-GNS3-informational?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Completed-success?style=for-the-badge)
![Level](https://img.shields.io/badge/Level-CCNA%20%E2%86%92%20CCNP-purple?style=for-the-badge)

*Router 3 hears `192.168.1.0/24` from two IBGP peers. By raising the **Local Preference to 200** on one border router (R4), every router in AS 20 learns a preferred exit — the best path on R3 changes to the next hop `4.1.1.1`.*

</div>

---

## 📑 Table of Contents

1. [Project Overview](#-project-overview)
2. [BGP Introduction](#-bgp-introduction)
3. [Types of BGP & Administrative Distance](#-types-of-bgp--administrative-distance)
4. [BGP Path Attributes](#-bgp-path-attributes)
5. [BGP Best-Path Selection Algorithm](#-bgp-best-path-selection-algorithm)
6. [Local Preference (Lab Focus)](#-local-preference-lab-focus)
7. [Local Preference vs Weight vs MED vs AS-Path](#-local-preference-vs-weight-vs-med-vs-as-path)
8. [Lab Topology](#-lab-topology)
9. [Addressing Table](#-addressing-table)
10. [Configuration](#-configuration)
11. [Verification (Stage by Stage)](#-verification-stage-by-stage)
12. [The `r` Flag — BGP RIB-Failure](#-the-r-flag--bgp-rib-failure)
13. [Lab Series Comparison](#-lab-series-comparison)
14. [Command Cheat Sheet](#-command-cheat-sheet)
15. [Key Takeaways](#-key-takeaways)
16. [Troubleshooting Checklist](#-troubleshooting-checklist)
17. [Skills Demonstrated](#-skills-demonstrated)
18. [Author](#-author)

---

## 🎯 Project Overview

| Item | Details |
|------|---------|
| **Lab Name** | BGP Local Preference — choosing the exit from AS 20 |
| **Platform** | GNS3 (Cisco IOS routers), accessed over Telnet |
| **Autonomous Systems** | **AS 10** (R1 originates `192.168.1.0/24`) and **AS 20** (R2, R3, R4) |
| **Sessions** | **EBGP:** R1 ⇄ R2 and R1 ⇄ R4 · **IBGP:** full mesh inside AS 20 |
| **Focus Router** | **R3** (BGP Router ID `3.1.1.1`) |
| **Focus Prefix** | **`192.168.1.0/24`** |
| **Configured on** | **R4:** `bgp default local-preference 200` |
| **Question** | Can one setting on a border router change the best path chosen by **another** router in the AS? |

### 🧪 The Experiment

| Stage | Action | R3's best path for `192.168.1.0` |
|-------|--------|----------------------------------|
| **1** | Baseline — both IBGP paths: LocPrf **100**, Weight 0 | via **1.1.1.1** (`r>i`) |
| **2** | On R4: `bgp default local-preference 200` | — |
| **3** | R3 re-evaluates | via **4.1.1.1** — **LocPrf 200** (`r>i`) |

---

## 📘 BGP Introduction

**BGP (Border Gateway Protocol)** is the routing protocol of the Internet and the only **EGP (Exterior Gateway Protocol)** in use today (RFC 4271).

| Property | Description |
|----------|-------------|
| **Category** | Exterior Gateway Protocol — routing *between* autonomous systems |
| **Algorithm** | **Path Vector** (tracks the ASNs a route has crossed) |
| **Transport** | **TCP port 179** |
| **Neighbors** | **Manually configured** |
| **Updates** | Incremental, triggered — no periodic refresh |
| **Metric** | No single metric — decisions use **path attributes** |
| **Loop prevention** | AS-Path check (EBGP), split-horizon rule (IBGP) |
| **Focus** | **Policy, control and scalability** |

### 🆚 IGP vs EGP

| | IGP (OSPF, EIGRP, RIP) | EGP (BGP) |
|---|---|---|
| Scope | Inside **one** AS | **Between** ASes |
| Goal | Fast, optimal paths | Policy-based path selection |
| Neighbor discovery | Automatic | Manual `neighbor` |
| Transport | Own protocol / IP | TCP 179 |

---

## 🔀 Types of BGP & Administrative Distance

| | **EBGP** | **IBGP** |
|---|---|---|
| **Neighbors in** | Different AS | **Same AS** |
| **Administrative Distance** | **20** | **200** |
| **AS-Path** | Local ASN prepended | Not changed |
| **Local Preference** | **Not carried** | **Carried (default 100)** ⭐ |
| **Next-Hop** | Changed to the advertiser | **NOT changed** |
| **Loop prevention** | AS-Path check | Split horizon → full mesh / route reflectors |

> ⭐ Local Preference **only travels inside an AS over IBGP**. It is never sent to an EBGP neighbor.

---

## 🧩 BGP Path Attributes

| Category | Examples |
|----------|----------|
| **Well-known mandatory** | ORIGIN, AS_PATH, NEXT_HOP |
| **Well-known discretionary** | **LOCAL_PREF** ⭐, ATOMIC_AGGREGATE |
| **Optional transitive** | COMMUNITY, AGGREGATOR |
| **Optional non-transitive** | MED, ORIGINATOR_ID, CLUSTER_LIST |
| **Cisco-proprietary (local only)** | Weight |

### 🔎 Attributes of the two paths on R3 (baseline)

```
     Network          Next Hop            Metric LocPrf Weight Path
 r i 192.168.1.0      4.1.1.1                  0    100      0 10 i
 r>i                  1.1.1.1                  0    100      0 10 i
```

| Attribute | Path via 4.1.1.1 | Path via 1.1.1.1 |
|-----------|------------------|------------------|
| NEXT_HOP | `4.1.1.1` (R1's link to R4) | `1.1.1.1` (R1's link to R2) |
| **LOCAL_PREF** | **100** | **100** |
| AS_PATH | `10` | `10` |
| ORIGIN | `i` | `i` |
| MED (Metric) | 0 | 0 |
| Weight | 0 | 0 |
| Type | IBGP (`i`) | IBGP (`i`) |

---

## 🏆 BGP Best-Path Selection Algorithm

| Step | Attribute | Prefer |
|------|-----------|--------|
| 0 | **Next hop reachable** | Required |
| 1 | **Weight** | Highest |
| 2 | **Local Preference** ⭐ | **Highest** |
| 3 | **Locally originated** | Network/aggregate on this router |
| 4 | **AS-Path** length | Shortest |
| 5 | **Origin** | IGP (`i`) < EGP (`e`) < Incomplete (`?`) |
| 6 | **MED** | Lowest |
| 7 | **EBGP over IBGP** | EBGP |
| 8 | **IGP metric to next hop** | Lowest |
| 9 | **Oldest EBGP path** | Oldest |
| 10 | **Router ID** | Lowest |
| 11 | **Neighbor IP** | Lowest |

### 📍 Where Local Preference sits

- Checked **second** — only **Weight** is stronger.
- It is checked **long before** the Router ID tie-breaker (Step 10) that decided the previous lab.
- With identical Weight (0), the path with the **higher LocPrf wins immediately**.

---

## 🎚️ Local Preference (Lab Focus)

| Property | Description |
|----------|-------------|
| **Type** | Well-known **discretionary** attribute |
| **Scope** | **AS-wide** — shared with all IBGP peers; **never sent to EBGP peers** |
| **Default** | **100** |
| **Range** | 0 – 4,294,967,295 |
| **Preference** | **Highest wins** |
| **Controls** | **Outbound traffic** — which exit the AS uses to reach a destination |
| **Set on** | The router where the route **enters** the AS (the EBGP border router) |
| **Who sees it** | Every IBGP router in the AS, via the UPDATE messages |

### 🧭 How the value reaches R3

```
          AS 10                           AS 20
        ┌────────┐   EBGP     ┌────────────────────┐   IBGP      ┌────────┐
        │   R1   │──────────► │ R4 (border)        │────────────►│   R3   │
        │192.168 │ next hop   │ sets LocPrf = 200  │ LocPrf 200  │ best:  │
        │ .1.0   │ 4.1.1.1    │ (bgp default ...)  │ next hop    │ 4.1.1.1│
        └────────┘            └────────────────────┘ 4.1.1.1     └────────┘
```

1. R1 advertises `192.168.1.0/24` to R4 over **EBGP** (LocPrf is not carried here).
2. R4 assigns **Local Preference 200** to the route (`bgp default local-preference 200`).
3. R4 advertises it to its **IBGP** peers with LocPrf 200 — the next hop (`4.1.1.1`) is unchanged.
4. R3 compares: **200 (via R4) > 100 (via R2)** → the path via R4 wins.

### 🔧 Two ways to set it

| Method | Command | Scope |
|--------|---------|-------|
| **Default for the router** ⭐ (this lab) | `bgp default local-preference 200` | All EBGP-learned and locally originated routes on that router |
| **Per neighbor / per prefix** (production) | `route-map … set local-preference 200` applied **inbound** to the EBGP neighbor | Only the routes you match |

---

## ⚖️ Local Preference vs Weight vs MED vs AS-Path

| Attribute | Scope | Influences | Who sets it | Preferred | Order |
|-----------|-------|------------|-------------|-----------|-------|
| **Weight** | One router only | Outbound (that router) | Local admin | Highest | **1st** |
| **Local Preference** ⭐ | **Whole AS** | **Outbound (AS-wide)** | Local admin | **Highest** | **2nd** |
| **AS-Path prepend** | Between ASes | **Inbound** (to your AS) | Local admin (to influence others) | Shortest | 4th |
| **MED** | Between neighboring ASes | **Inbound** | Neighbor AS suggests | Lowest | 6th |
| **Router ID** | Local tie-break | — | Auto / manual | Lowest | 10th |

> 💡 **Rule of thumb:** use **Local Preference** to choose how **your AS exits**; use **AS-Path prepending / MED** to influence how **others enter** your AS; use **Weight** only for a single-router override.

---

## 🗺️ Lab Topology

```
          ┌──────────────────────── AS 10 ────────────────────────┐
          │                      ┌────────┐                        │
          │       1.1.1.1  ┌─────│   R1   │─────┐  4.1.1.1         │
          │                │     └────────┘     │                  │
          │       192.168.1.0/24 (originated)    │                  │
          └────────────────┼─────────────────────┼──────────────────┘
                       EBGP│                     │EBGP
        ┌──────────────────┼─────────────────────┼───────── AS 20 ──────────────────┐
        │             1.1.1.2                 4.1.1.2   ◄── Local Preference 200     │
        │            ┌────────┐             ┌────────┐       configured here         │
        │            │   R2   │             │   R4   │                               │
        │            └───┬────┘             └───┬────┘                               │
        │           2.1.1.1│                   │3.1.1.2                             │
        │     2.0.0.0/24   │                   │   3.0.0.0/24                       │
        │           2.1.1.2│                   │3.1.1.1                             │
        │            ┌─────┴───────────────────┴──┐                                 │
        │            │           R3 (FOCUS)       │   Router ID 3.1.1.1             │
        │            └────────────────────────────┘                                 │
        │      IBGP full mesh: R2 ⇄ R3 · R3 ⇄ R4 · R2 ⇄ R4                           │
        └────────────────────────────────────────────────────────────────────────────┘
```

> Same topology as the previous **Path Selection (Router ID & Weight)** lab.

| Link | Subnet | Used for |
|------|--------|----------|
| R1 ⇄ R2 | `1.1.1.0/24` | **EBGP** |
| R1 ⇄ R4 | `4.1.1.0/24` | **EBGP** |
| R2 ⇄ R3 | `2.1.1.0/24` | Physical link (IBGP) |
| R3 ⇄ R4 | `3.1.1.0/24` | Physical link (IBGP) |
| R2 ⇄ R4 | — (logical) | IBGP session |

---

## 📋 Addressing Table

| Device | Interface | IP Address | Role |
|--------|-----------|------------|------|
| **R1** (AS 10) | link to R2 | 1.1.1.1 | **NEXT_HOP** of the path learned via R2 |
| **R1** (AS 10) | link to R4 | 4.1.1.1 | **NEXT_HOP** of the path learned via R4 |
| **R2** (AS 20) | link to R1 | 1.1.1.2 | EBGP border router |
| **R2** (AS 20) | link to R3 | 2.1.1.1 | — |
| **R3** (AS 20) | link to R2 | 2.1.1.2 | — |
| **R3** (AS 20) | link to R4 | 3.1.1.1 | Router ID `3.1.1.1` |
| **R4** (AS 20) | link to R3 | 3.1.1.2 | — |
| **R4** (AS 20) | link to R1 | 4.1.1.2 | **EBGP border router where LocPrf 200 is set** |

---

## 🛠️ Configuration

### 🔹 R4 (AS 20) — the change

```bash
R4(config)# router bgp 20
R4(config-router)# bgp default local-preference 200
R4(config-router)# do write
Building configuration...
[OK]
```

### 🔹 Production alternative — route-map on the EBGP neighbor (R4)

```bash
R4(config)# route-map SET-LP permit 10
R4(config-route-map)# set local-preference 200
R4(config-route-map)# exit
R4(config)# router bgp 20
R4(config-router)# neighbor 4.1.1.1 route-map SET-LP in
```

### 📝 Command Explanation

| Command | What it does |
|---------|--------------|
| `router bgp 20` | Enters the BGP process of AS 20 |
| `bgp default local-preference 200` | Changes the default Local Preference from **100 → 200** for routes this router learns via EBGP (and originates) before it advertises them to IBGP peers |
| `do write` | Saves the configuration |
| `route-map … set local-preference` | Targeted version: only the matched routes get the new value |

> ⚠️ `bgp default local-preference` is a blunt tool — it affects **every** route R4 learns from EBGP. A route-map lets you choose the neighbor or prefix.

---

## 🔍 Verification (Stage by Stage)

### 🟡 Stage 1 — Baseline on R3: Two Equal Paths

```
R3# show ip bgp
BGP table version is 2, local router ID is 3.1.1.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete

     Network          Next Hop            Metric LocPrf Weight Path
 r i 192.168.1.0      4.1.1.1                  0    100      0 10 i
 r>i                  1.1.1.1                  0    100      0 10 i
```

#### 📌 Annotated Reading

| Marker | Explanation |
|--------|-------------|
| Two lines for `192.168.1.0` | R3 hears the prefix from **two IBGP peers** (full mesh) |
| `LocPrf 100` on both | **Default** value — equal |
| Weight `0`, AS-Path `10`, Origin `i`, MED `0` | All equal → decided by later tie-breakers |
| `r>i … 1.1.1.1` | Best path (`>`) at baseline — via R2 |
| **`r`** | **RIB-failure** flag on both paths (see [The `r` Flag](#-the-r-flag--bgp-rib-failure)) |
| Router ID `3.1.1.1` | R3's BGP Router ID |

---

### 🟢 Stage 2 — Raise Local Preference on R4

```
R4(config)# router bgp 20
R4(config-router)# bgp defa
R4(config-router)# bgp default lo
R4(config-router)# bgp default local-preference 200
R4(config-router)# do wr
Building configuration...
[OK]
```

| Marker | Explanation |
|--------|-------------|
| `bgp default local-preference 200` | Local Preference changes from the default **100** to **200** |
| Configured on **R4 only** | R2 still advertises its path with LocPrf 100 |

---

### 🟢 Stage 3 — Result on R3: The Best Path Changes

```
R3# show ip bgp
     Network          Next Hop            Metric LocPrf Weight Path
 r>i 192.168.1.0      4.1.1.1                  0    200      0 10 i      ◄── Now the local preference is 200
 r i                  1.1.1.1                  0    100      0 10 i
```

#### 📌 Annotated Reading

| Marker | Before | After |
|--------|--------|-------|
| Best path (`>`) | via `1.1.1.1` (R2) | via **`4.1.1.1` (R4)** |
| Path via 4.1.1.1 — LocPrf | 100 | **200** 🟨 |
| Path via 1.1.1.1 — LocPrf | 100 | 100 |
| Other attributes | Weight 0, AS-Path `10`, MED 0 | **Unchanged** |
| Deciding step | Later tie-breakers | **Step 2: highest Local Preference** |

✅ **Result:** One command on R4 changed the best path **on R3** — Local Preference is an **AS-wide** attribute carried inside IBGP.

---

### 🔄 Stage Summary

| Stage | Change | R3 best path | LocPrf (via 4.1.1.1 / via 1.1.1.1) | Decided by |
|-------|--------|--------------|-----------------------------------|------------|
| 1 | Baseline | **1.1.1.1** | 100 / 100 | Later tie-breakers |
| 2 | `bgp default local-preference 200` on R4 | — | — | Configuration |
| 3 | R3 re-evaluates | **4.1.1.1** | **200** / 100 | **Local Preference** |

---

## 🚩 The `r` Flag — BGP RIB-Failure

Both rows in R3's BGP table carry the status code **`r` (RIB-failure)**.

| Question | Answer |
|----------|--------|
| **What does `r` mean?** | BGP selected a best path, but the **routing table (RIB) refused to install it**. |
| **Common cause** | Another route source offers the same prefix with a **lower Administrative Distance** (e.g., OSPF 110, static 1) than IBGP's **200**. |
| **What is visible in these screenshots?** | The `r` flags only. The routing table entry for `192.168.1.0/24` is **not captured**, so the competing source is not shown. |
| **Is BGP down?** | No — the paths are in the BGP table and Local Preference is evaluated normally. |
| **Why it matters** | Local Preference decides BGP's **best path**, but **forwarding follows the routing table**. If another protocol owns the route, traffic does not follow BGP's choice. |

### 🔎 How to identify the cause

```
R3# show ip route 192.168.1.0
R3# show ip bgp rib-failure
```

`show ip bgp rib-failure` reports the reason (for example, *Better admin distance*) and the winning protocol.

### 🧰 Possible fixes

| Fix | Command / action |
|-----|------------------|
| Stop the competing protocol from carrying the prefix | Remove `192.168.1.0/24` from the OSPF/static/redistribution source |
| Raise the competing AD above 200 | `distance 210` under the IGP (as used in the earlier IBGP + OSPF lab) |
| Filter the prefix from the IGP | `distribute-list`/prefix-list on the IGP |

> ✅ After the RIB-failure is cleared, the status changes from `r>i` to `*>i` and the route appears as `B 192.168.1.0/24 [200/0] via 4.1.1.1` in `show ip route`.

---

## 🧭 Lab Series Comparison

Same topology, three different ways to steer the path for `192.168.1.0/24` on R3:

| Lab | Tool | Where configured | Reach | Step in algorithm |
|-----|------|------------------|-------|-------------------|
| Path Selection | **Router ID** | R2 (`bgp router-id`) | Tie-breaker only | **10** |
| Path Selection | **Weight** | R3 (`neighbor 3.1.1.2 weight 10`) | **R3 only** | **1** |
| **This lab** | **Local Preference** | R4 (`bgp default local-preference 200`) | **Entire AS 20** | **2** |

---

## 🧾 Command Cheat Sheet

| Purpose | Command |
|---------|---------|
| Default Local Preference | `bgp default local-preference <n>` |
| Per-route Local Preference | `route-map <name>` → `set local-preference <n>` |
| Apply a route-map inbound | `neighbor <ip> route-map <name> in` |
| Refresh outbound updates | `clear ip bgp * soft out` |
| Refresh inbound updates | `clear ip bgp <ip> soft in` |
| BGP table | `show ip bgp` |
| One prefix in detail | `show ip bgp 192.168.1.0` |
| RIB-failures | `show ip bgp rib-failure` |
| Routing table entry | `show ip route 192.168.1.0` |
| Save | `write memory` |

---

## 🧠 Key Takeaways

1. 🎚️ **Local Preference is an AS-wide attribute**, carried to all IBGP peers (default **100**, **higher wins**).
2. 🥈 It is the **second** step of best-path selection — only **Weight** is checked earlier.
3. 🚪 It controls the **exit** from your AS (outbound traffic).
4. 🔧 One command on **R4** (`bgp default local-preference 200`) changed the best path on **R3**.
5. 📈 After the change R3 showed **LocPrf 200** for `4.1.1.1` and **100** for `1.1.1.1` — everything else stayed the same.
6. 🚫 Local Preference is **never sent over EBGP**.
7. 🆚 **Weight** = this router only · **Local Preference** = whole AS · **MED / AS-Path** = influence **inbound** traffic.
8. 🎯 `bgp default local-preference` is a blunt, router-wide setting — use a **route-map** for per-neighbor or per-prefix control.
9. 🚩 The **`r` flag** means BGP's best path was **not installed** in the routing table — always check `show ip route` and `show ip bgp rib-failure`.
10. 🧭 BGP's **best path** and the **forwarding path** are only the same when the BGP route is actually installed.

---

## 🩺 Troubleshooting Checklist

| Symptom | Likely Cause | Fix |
|---------|--------------|-----|
| LocPrf didn't change on R3 | Update not re-sent after the change | `clear ip bgp * soft out` on the router where it was set |
| Only some routers prefer the new exit | Value set on a router that isn't the entry point | Set it on the **EBGP border router** that learns the route |
| Local Preference has no effect | Higher **Weight** on a path | Compare Weight first (Step 1) |
| `r` flag on a path | A better-AD route owns the prefix | `show ip bgp rib-failure`; fix the AD or the source |
| Best path correct but traffic goes elsewhere | RIB-failure — forwarding follows another protocol | Resolve the RIB-failure |
| All routes from one EBGP peer changed | `bgp default local-preference` is router-wide | Use a route-map for targeted control |
| LocPrf visible on an EBGP neighbor | Not possible — LocPrf is not sent over EBGP | Check IBGP only |

---

## 💡 Ideas to Extend This Lab

- 🔬 Capture `show ip route 192.168.1.0` and `show ip bgp rib-failure` to document the cause of the `r` flags
- 🎛️ Use a **route-map** to set Local Preference only for `192.168.1.0/24`
- ➕ Add **AS-Path prepending** on R1 to influence the **inbound** path
- 📊 Set **MED** on R1 and compare its effect with Local Preference
- 🔁 Make R2 the preferred exit with Local Preference **300** and verify that R3 switches back
- 🧪 Test end-to-end connectivity with `traceroute` to confirm the real forwarding path

---

## 🧰 Skills Demonstrated

`BGP` · `BGP Local Preference` · `BGP Path Attributes` · `Best-Path Selection` · `EBGP` · `IBGP Full Mesh` · `Traffic Engineering` · `BGP RIB-Failure` · `Route-Maps` · `Cisco IOS` · `GNS3` · `Routing Table Analysis` · `Network Troubleshooting` · `Network Documentation`

---

## 👤 Author

<div align="center">

**Maaz Khan**
CCNA-Certified Network & NOC Engineer
📍 Lower Dir, KPK, Pakistan

![CCNA](https://img.shields.io/badge/CCNA-Certified-blue?style=flat-square&logo=cisco)
![NOC](https://img.shields.io/badge/NOC-Engineer-orange?style=flat-square)

⭐ If this lab helped you, give the repo a star!

</div>

---

<div align="center">

**#BGP #LocalPreference #BGPAttributes #IBGP #EBGP #TrafficEngineering #CCNA #CCNP #Cisco #GNS3 #Networking #NetworkEngineer #NOC**

</div>
