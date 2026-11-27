# ABC Connect authentication and Entra External ID architecture

Review date: November 27, 2026. Reference version 0.1.0.

Start with ABC_Connect_Authentication_Entra_External_ID_Architecture.pdf. It documents the recommended platform identity / supplier authorization split, both ABC integration tracks, the scope matrix, OAuth contracts, Entra registration model, account linking, token lifecycle, optional OIDC federation, operation policy and acceptance plan.

The baseline is Entra External ID sign-in plus a server-side ABC API authorization broker. Direct ABC-to-Entra OIDC federation is a gated feasibility candidate: both official issuer discovery endpoints returned public OIDC metadata, but no registered client, identity token, consent or token exchange was tested. The report identifies the current Entra federation client-authentication restrictions and the need for an ABC-approved compatible client. Federation does not itself establish supported forwarding of ABC API refresh tokens to the platform.

## Evidence hierarchy

- D: product documentation establishes intended integration tracks and their documented permissions.
- M: observed public authorization-server metadata establishes generic advertised capabilities only.
- T: client-level authenticated verification establishes what an actual provisioned registration can do. None was performed.
- P: proposed platform design and local controls, explicitly separated from provider requirements.

Public discovery was retrieved with unauthenticated HTTPS GETs for sandbox and production on the review date. Generic metadata and written documentation differ on client_credentials and several invoice/order scope names. These disagreements are retained; the pack does not silently infer new entitlements from metadata.

## Reference files

- abc-oauth-profiles.json: grant and scope profiles, lifetime rules and discovery caveats.
- api-authorization-matrix.json: 33 operation variants with ABC and proposed platform permission requirements; runtime enablement is false pending verification.
- configuration.example.json: verified public issuer/endpoint values, proposed Entra fields and unresolved client settings. Placeholder IDs are intentional; no credentials are present.
- protocol-requests.http: request templates only, with server-resolved token/credential placeholders.
- connection-record.schema.json: connection owner, independent customerOrgId, client/environment binding, opaque token reference, granted scopes and state.
- oauth-transaction.schema.json: single-use link transaction bound to Entra session/user/org, issuer/client, state digest and PKCE secret reference.
- organization-membership.schema.json and account-binding.schema.json: application organization roles and ABC account/branch ACLs.
- authorization-decision.schema.json: server-created operation decision and supporting evidence references; a caller cannot assert authorization by sending this schema.
- orchestration-and-local-routes.json: proposed platform endpoints and renewal/state-machine rules. These local routes are not ABC API endpoints.
- verification-register.json: 14 provider/client/architecture verification items.
- acceptance-plan.json: 18 unexecuted authenticated test scenarios and expected outcomes.
- discovery/: unmodified public metadata values reserialized as JSON; no secret credentials.
- sources.json: primary documentation URLs, retrieval observations and hashes.
- validation-results.json: structural checks of this reference pack, not runtime security tests.

## Identity and authorization rules

One External ID tenant may represent many contractor organizations. Entra tenant ID is not customerOrgId. Use stable validated Entra identity, current per-organization membership, provider connection ownership and verified account/branch bindings. Email equality and local roles do not prove ABC ownership or permissions. Supplier tokens are kept server-side; apps and agents receive redacted connection and operation results.

Entra OBO can serve supported Entra API chains, but no Entra-to-ABC token exchange trust was established. Azure managed identity protects Azure resource access; it is not an ABC registered-client credential. Local role membership, the ABC granted scope set and resource eligibility are evaluated together for every operation.

All schemas are proposed internal contracts. JSON Schema does not prove session ownership, hash correctness, freshness, actor identity, single-use transaction enforcement, token authenticity or account authorization; the runtime must implement those invariants and pass the acceptance plan.

## Next implementation decision

Confirm whether this deployment is a Third-Party Aggregator serving independent contractors or an Individual Business integration for ABC-approved enrolled accounts. The architecture covers both; the working recommendation for a multi-contractor product is TPA with delegated customer connections. Keep API scope profiles, identity-federation client profiles and secrets separate by environment and intended use.
