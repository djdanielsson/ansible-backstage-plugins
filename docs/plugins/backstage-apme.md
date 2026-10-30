# @ansible/plugin-backstage-apme

Frontend contributor for Ansible content quality and modernization. Source:
`workspaces/apme/plugins/backstage-apme`.

## Native contributions

The `./alpha` entry point exports the `apme` frontend plugin with Quality,
Quality activity, and Dependencies tabs; overview summaries; scan and
deregistration actions; catalog tabs/columns; Add repository; and a repository
URL scaffolder field.

The plugin contributes through Portal Blueprints rather than importing
self-service page components. Components use the APME API factory registered
through the native plugin.

## Availability and lifecycle

Scanning requires `ansible.apme.enabled: true`, the host-resolved
`apme.scanning` entitlement, and applicable permissions. Configuration alone
does not authorize an operation.

APME owns scan-summary refresh while a background operation is active.
The native scan action and interactive Quality workflow have different
lifecycles; completed submission is not proof that remediation or a PR occurred.

See [NFS composition](../next/portal-sdk-nfs-composition.md),
[APME catalog backend](catalog-apme.md), and
[APME shared contracts](backstage-apme-common.md).

## Shared APME host package integration

The APME host-consolidation branch produces four versioned artifacts:
`@apme/client`, `@apme/ui-workflow`, `@apme/backstage-plugin` and
`@apme/backstage-backend`. The first release must publish those artifacts before
Portal can pin their immutable release URLs. This branch preserves the existing
plugin's API contracts while adding lossless shared-package data routes:

- `/apme/shared/rules` preserves Gateway numeric severity and applicability data.
- `/apme/shared/projects/:projectId` and its `activity`, `violations`,
  `dependencies`, `graph` and `sbom` children authorize the catalog subject and
  scanning entitlement before forwarding. Pagination and query filters survive.
- `/apme/shared/projects/:projectId/activity/:scanId` additionally verifies that
  the returned activity belongs to the selected project.
- `/apme/shared/ai/config`, `/providers` and `/engines` preserve upstream errors
  after settings-view authorization, so failures cannot appear as empty config.
- Existing Galaxy create/update routes validate and forward `validate_certs`,
  including explicit null for deployment policy.

After installing the frontend artifact, use `createPortalProjectApiAdapter`
with the canonical project ID, discovery and FetchApi when embedding the shared
project views. Existing operation writes/SSE retain their current authorization
path. Hosts evaluate settings permissions before passing management props.
No shared request retries against a direct Gateway on failure.

Uploads use the optional backend package's `createUploadSessionRouter`, mounted
under `/apme` on the catalog router. It relays the existing FixSession protocol
through authenticated HTTP commands and an NDJSON response. Its authorization
callback must authenticate the user and check the active organization, scanning
entitlement and `apmeQualityScanPermission` on every HTTP request. Return an owner
key containing both user and active organization, never only a browser-supplied
session ID. Resolve the Gateway URL from the trusted portal settings/config and
register the transport's `close` method with the backend lifecycle shutdown hook.
Do not register the personal Local backend plugin's permissive instance policy.

Set the adapter's `enableUploads` only after mounting that router. Ownership is
transport-local and fails closed after restart; multi-replica deployments need
sticky routing for active channels. No bearer token is sent in a WebSocket URL.
The Gateway/Engine remain the only workflow/session owners.

Current evidence: full TypeScript compilation, backend package build, targeted
ESLint, OpenAPI lint/drift and 110 router tests pass. Actual Portal/RHDH package
installation, authenticated user workflows, browser/theme checks and resource
measurements remain required before claiming runtime acceptance. Existing native
UI removal remains gated on those checks in APME REQ-021.
