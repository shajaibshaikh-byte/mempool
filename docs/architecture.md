

# MEMPOOL-RAM Architecture (Plain English)

## What this product is

MEMPOOL-RAM is a Memory-as-a-Service system for Linux.
It lets machines temporarily borrow unused RAM from other machines over the network.

Memory is leased, time-limited, wiped on release, and controlled centrally.

---

## The real problem

• Servers waste large amounts of RAM most of the time
• Some workloads need short RAM bursts
• Teams overbuy RAM to handle rare peaks
• RAM is expensive and underutilized

MEMPOOL-RAM makes RAM elastic instead of static.

---

## Big picture

Unused RAM on provider machines is pooled.
A client requests memory from a control plane.
The control plane authorizes the request and selects a provider.
The provider allocates real RAM, serves access over TLS, and enforces a strict time limit.
When time expires, memory is wiped and reclaimed.

---

## Core components

### 1. Control Plane

The brain.

Responsibilities:
• Authenticate clients
• Issue short-lived allocation tokens
• Choose a provider
• Revoke access when needed

The control plane:
• Never stores memory
• Never handles data payloads

If the control plane is down, new allocations stop.

---

### 2. Memory Provider

The authority over memory.

Responsibilities:
• Validate tokens with the control plane
• Allocate real RAM-backed memory using tmpfs and mmap
• Enforce TTL
• Wipe memory on expiry or release
• Serve read and write requests over TLS

Memory always lives on the provider.

If the provider crashes, memory is lost and reclaimed by the OS.

---

### 3. Client

The user of memory.

Responsibilities:
• Request authorization from the control plane
• Use the assigned provider
• Read and write data
• Tolerate sudden memory loss

Clients never assume durability.

---

## How allocation works (step by step)

1. Client requests allocation from control plane
2. Control plane issues a short-lived token and selects a provider
3. Client sends token to provider
4. Provider validates token with control plane
5. Provider allocates RAM-backed memory
6. Client reads and writes data
7. TTL expires or client releases memory
8. Provider wipes memory and reclaims it

---

## Memory model

• Memory is backed by tmpfs (`/dev/shm`)
• Memory is mapped with `mmap`
• No disk persistence
• One allocation per client in MVP

This guarantees fast access and clean failure behavior.

---

## Security model (summary)

• Zero trust between machines
• TLS for all network traffic
• Provider trusts only the control plane
• Short-lived tokens limit blast radius
• Memory wiped before reuse

A compromised provider can read hosted memory.
This is an accepted MVP risk.

---

## Failure behavior

• Control plane down → no new allocations
• Provider down → memory lost, client tolerates loss
• Network drop → client errors, TTL still enforced
• TTL expiry → provider wipes memory immediately

Failures are explicit, not hidden.

---

## What this MVP does NOT do

• Not a replacement for local RAM
• Not transparent memory paging
• Not ultra-low latency
• Not multi-tenant hardened
• Not optimized for scale

---

## Why this architecture works

• Simple authority boundaries
• Provider-enforced lifecycle
• Minimal trusted components
• Clear failure modes
• Incremental extensibility

This design favors correctness over performance.

---

## How this expands later

Future phases may add:
• Multiple providers
• Placement logic
• Quotas and metering
• Scheduler integration
• Pilot customer features

These are intentionally out of scope for the MVP.

---

### Status

Architecture locked for MVP.
Matches implementation up to **Phase 3A**.

 
