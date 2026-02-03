 

---

# Security

This document explains how security works in the MEMPOOL-RAM MVP and where it does not.
It is written so that a future incident report can cite it.

## 1. Security goals

We are protecting memory contents that move between machines.
This matters because memory can include secrets like API keys, customer data, or encryption material.

We are also protecting the system from misuse.
This matters because a single unsafe machine can harm the rest of the pool.

We are not trying to hide the existence of the system or who is using it.
We are trying to prevent unauthorized access, leakage, and silent tampering.

## 2. Threat model

The following threats are realistic for this MVP:

* Malicious provider: a machine that offers memory but tries to read or alter it.
* Malicious consumer: a machine that uses memory but tries to access others or abuse the pool.
* Compromised machine: a provider, consumer, or control plane host that has been taken over.
* Network interception: an attacker who can observe or alter traffic on the network.
* Software bugs: mistakes in code that allow access, corruption, or denial of service.

## 3. Zero-trust approach

Zero trust means we do not assume any machine or network is safe.
Every connection must prove identity, and every request must be limited to the minimum needed.

This is required because providers and consumers do not trust each other.
It is also required because we assume some machines and networks are hostile.

## 4. Authentication between machines

Each machine has a long-lived identity key.
A key is a secret value used to prove identity.

When two machines connect, they prove possession of their keys to each other.
The control plane also proves its identity before it can instruct providers or consumers.

If identity cannot be proven, the connection is rejected.
No machine is trusted just because it is on the same network.

## 5. Encryption in transit

All memory data is encrypted while it moves across the network.
Encryption means data is transformed so outsiders cannot read it in transit.

Each connection uses fresh session keys.
Session keys are temporary keys used only for one connection.

If the network is intercepted, the attacker should see only encrypted data.
This does not protect data after it reaches a compromised machine.

Encryption adds some latency, which is acceptable because this system targets burst and spillover workloads, not ultra-low-latency paths.

## 6. Memory isolation

Each consumer receives memory blocks that are marked as belonging to that consumer.
Isolation means one consumer cannot read or write another consumer’s memory blocks.

The provider enforces this by tracking ownership and refusing cross-tenant access.
The control plane also checks that allocations match the consumer identity.

Isolation reduces accidental or casual access.
It does not stop a fully compromised provider from reading memory it hosts.

MEMPOOL-RAM does not attempt to protect data from the owner of the machine that physically hosts the memory.

## 7. Memory lifetime and wiping

Allocations are time-bound.
Time-bound means each allocation has a set expiry time.

When the time limit expires, the provider reclaims the memory automatically.
Automatic reclaim means the memory is taken back without relying on the consumer.

When memory is released or reclaimed, it is cleared before reuse.
Clearing means overwriting so the next user cannot see previous data.

This matters because memory can hold sensitive values long after a workload ends.
Without clearing, data could leak to the next consumer.

## 8. Failure handling from a security view

If a provider crashes, allocations on that provider are considered lost.
The system treats that memory as unsafe and does not try to reuse it until the provider recovers and reinitializes.

If a consumer crashes, its allocations are reclaimed after the time limit.
This prevents abandoned memory from staying exposed.

If the network disconnects, allocations are frozen and then reclaimed on timeout.
This limits the risk of stale or hijacked sessions.

If a time limit expires, the allocation is terminated and cleared.
This prevents long-lived hidden usage.

## 9. Kill switches and emergency controls

Operators can revoke a consumer or provider identity immediately.
Revocation means future connections from that identity are refused.

Operators can also force-stop allocations on a provider.
Force-stop means the provider wipes memory and stops serving the consumer.

These controls exist to contain damage during an incident.
They do not fix data that has already been exposed.

Security-relevant events such as allocations, revocations, and force-stops are logged for later review.

## 10. Known limits and accepted risks

A compromised provider can read any memory it hosts.
This is a fundamental risk in the MVP.

A compromised consumer can misuse its own allocations.
We can limit abuse, but we cannot prevent all malicious use.

Bugs can still lead to leakage or denial of service.
This is likely in early-stage software.

Encryption in transit does not protect data at rest on a hostile machine.
We do not yet provide trusted hardware isolation or verified execution.

The control plane is a high-value target.
If it is compromised, attackers could redirect or disrupt allocations.

These risks are accepted for the MVP and must be revisited before any production use.

---
