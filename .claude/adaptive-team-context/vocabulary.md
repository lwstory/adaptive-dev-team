# Vocabulary

> **Seed content — not definitions.** Every entry below is a placeholder. During `/adaptive-team-init` (the setup wizard), the team customizes this file with the precise meanings used in THIS project. Until filled in, treat every term as undefined. Teammates reading ambiguous terms in code or tasks should consult this file first — if it says `<fill in>`, raise the ambiguity before proceeding.

---

## Identity and tenancy

**tenant**
<fill in during setup — what does 'tenant' mean in THIS project?>
Not: <fill in — what is 'tenant' commonly confused with here? e.g., is it the same as 'organization' or different?>

**account**
<fill in during setup — what does 'account' mean in THIS project?>
Not: <fill in — is 'account' a billing concept, an auth concept, or the same as 'user'? name the collision.>

**workspace**
<fill in during setup — what does 'workspace' mean in THIS project?>
Not: <fill in — is it a tenant sub-division, a UI concept, or something else? what does it NOT include?>

**organization**
<fill in during setup — what does 'organization' mean in THIS project?>
Not: <fill in — is 'organization' synonymous with 'tenant', 'account', or distinct? state clearly.>

**team**
<fill in during setup — what does 'team' mean in THIS project?>
Not: <fill in — is 'team' inside an organization, or is it a different axis entirely?>

**project**
<fill in during setup — what does 'project' mean in THIS project? (yes, the term collides with "this project")>
Not: <fill in — is 'project' a container for resources, a billing unit, a workspace alias, or something else?>

---

## People

**user**
<fill in during setup — what does 'user' mean in THIS project?>
Not: <fill in — is 'user' always authenticated? does it overlap with 'customer' or 'member'?>

**customer**
<fill in during setup — what does 'customer' mean in THIS project?>
Not: <fill in — is 'customer' a billing concept distinct from a user, or are they the same entity?>

**member**
<fill in during setup — what does 'member' mean in THIS project?>
Not: <fill in — is 'member' a user in a specific role context (team/org), or a separate entity type?>

**admin**
<fill in during setup — what does 'admin' mean in THIS project?>
Not: <fill in — is 'admin' a role, a permission set, a separate user type, or a UI concept?>

**owner**
<fill in during setup — what does 'owner' mean in THIS project?>
Not: <fill in — does 'owner' refer to resource ownership, organization admin-level access, or a billing role?>

---

## Sessions and interactions

**session**
<fill in during setup — what does 'session' mean in THIS project?>
Not: <fill in — is 'session' an auth session, a DB transaction, a user interaction window, or an agent turn?>

**request**
<fill in during setup — what does 'request' mean in THIS project?>
Not: <fill in — is 'request' always an HTTP request, or does it also cover async jobs, queue messages, etc.?>

**transaction**
<fill in during setup — what does 'transaction' mean in THIS project?>
Not: <fill in — is 'transaction' always a DB transaction, or does it also mean a financial transaction or a saga?>

---

## Data entities

**entity**
<fill in during setup — what does 'entity' mean in THIS project?>
Not: <fill in — is 'entity' a DDD term, an ORM concept, or used interchangeably with 'record' and 'object'?>

**record**
<fill in during setup — what does 'record' mean in THIS project?>
Not: <fill in — is 'record' always a DB row, or is it used more broadly? does it overlap with 'document'?>

**document**
<fill in during setup — what does 'document' mean in THIS project?>
Not: <fill in — is 'document' a NoSQL document, a user-created artifact, or a generic term for a record?>

---

## Identifiers

**id**
<fill in during setup — what does 'id' mean in THIS project — UUID, integer, slug, or context-dependent?>
Not: <fill in — is 'id' always the primary DB key, or are there external ids, logical ids, and correlation ids?>

**slug**
<fill in during setup — what does 'slug' mean in THIS project?>
Not: <fill in — is 'slug' always URL-safe and human-readable, or is it used for any stable short identifier?>

---

## Authorization

**permission**
<fill in during setup — what does 'permission' mean in THIS project?>
Not: <fill in — is 'permission' an atomic capability, a role assignment, or a scope string? how does it differ from 'role'?>

**role**
<fill in during setup — what does 'role' mean in THIS project?>
Not: <fill in — is 'role' RBAC-style (admin/editor/viewer), a domain role (owner/member), or both? are they the same system?>

**scope**
<fill in during setup — what does 'scope' mean in THIS project?>
Not: <fill in — is 'scope' an OAuth scope, a tenant boundary, a permission namespace, or all three?>

---

## Environments

**environment**
<fill in during setup — what does 'environment' mean in THIS project?>
Not: <fill in — is 'environment' always dev/staging/prod, or do customers also have named environments within their tenant?>

**deployment**
<fill in during setup — what does 'deployment' mean in THIS project?>
Not: <fill in — is 'deployment' the act of releasing code, or also a running instance of the system? can there be multiple per environment?>
