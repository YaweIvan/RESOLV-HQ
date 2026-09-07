# RESOLV-HQ — Initial System Architecture

**Owner:** Yawe Ivan — Application/Integration Lead
**Week:** 1 (Problem Framing and AI-Native Requirements)

## Architecture Diagram

```
   +-------------------------+
   |  CUSTOMER INPUT /       |
   |  FRONTEND (Chat UI)     |
   +------------+------------+
                |
                | authenticated request
                v
   +-------------------------+
   |   REACT AGENT CORE      |
   |  (Sense - Plan - Act -  |
   |   Observe - Respond)    |
   +------------+------------+
                |
      +---------+----------+
      |                    |
      v                    v
+-----------------+  +--------------------------+
| RAG ENGINE /     |  | TOOLS (CSV / DB)          |
| VECTOR KB        |  | - Account Status Lookup   |
| (Policy & FAQ    |  | - Outage Status Checker   |
|  documents)      |  | - Draft Ticket Generator  |
+-----------------+  +--------------------------+
      |                    |
      +---------+----------+
                |
                | grounded answer  OR  drafted escalation
                v
   +-------------------------+
   | DETERMINISTIC SAFETY    |
   | LAYER (auth, validation,|
   | logging, action gating) |
   +------------+------------+
                |
     low-risk   |   high-risk / financial / account-change
     (auto)     |   (gated)
                v
   +-------------------------+
   | HUMAN ADMIN APPROVAL    |
   | DASHBOARD               |
   | (review / approve /     |
   |  reject / edit)          |
   +-------------------------+
```

## Component Interaction & Safety Boundaries

The Customer Input / Frontend captures the client's natural-language request and forwards it, together with a verified session token issued by deterministic authentication middleware, to the ReAct Agent Core. The agent core is the only component permitted to reason freely: on each turn it decides whether to retrieve policy context from the RAG Engine / Vector Knowledge Base, call one of the read-only diagnostic tools backed by the mock CSV/database layer, ask a clarifying question, or produce a final response.

Both the RAG Engine and the Tools layer are strictly read-only and return evidence — retrieved passages or lookup results — rather than taking any action themselves.

All agent output then passes through a Deterministic Safety Layer, which is the sole component authorised to log the interaction, validate the proposed action against the AI Boundary Matrix, and gate anything financial, account-altering, or otherwise high-risk. Low-risk, fully-grounded responses (e.g., an FAQ answer with a valid citation) are delivered directly to the client.

Anything involving ambiguity, dissatisfaction, refunds, or account changes is routed instead to the Human Admin Approval Dashboard, where a human reviews, edits, approves, or rejects the drafted action before it is ever executed.

This separation ensures the agent can reason and draft freely while remaining structurally unable to execute a consequential action on its own.