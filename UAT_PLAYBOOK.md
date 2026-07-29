# UAT Playbook — headlamp-rook-plugin

This document defines the browser-based validation steps for the Rook-Ceph Headlamp plugin. Run through these steps before merging any `uat → main` PR.

## Prerequisites

- Headlamp is running with the rook plugin loaded.
- A Rook-Ceph cluster is accessible (or test data is stubbed).
- You are logged in to Headlamp and can see the sidebar.

### Authenticating to headlamp-uat (PRI-2006)

`headlamp-uat` requires authentication (it is OIDC-fronted via Authentik, but
also accepts a direct bearer token on the Headlamp login screen). The UAT
agent must use a dedicated, viewer-scoped credential — never an admin
credential and never the `paperclip-app` service account.

1. Open the `headlamp-uat` URL.
2. On the login screen, use the token field (not "Sign in with Authentik").
3. Paste the token from the agent's injected `HEADLAMP_UAT_TOKEN` env var.
   Never type, paste, echo, or log this value into a chat transcript, PR
   comment, issue, or file — env injection only.
4. You should land on the sidebar once authenticated.

This credential is read-only, scoped to the namespaces and resource kinds the
rook plugin reads (`rook-ceph` Ceph CRDs/pods, plus cluster-scoped
StorageClasses/PersistentVolumes/Namespaces). It cannot create, update, or
delete anything.

Provisioning of this credential (RBAC + ServiceAccount token, sealed via
GitOps in the infra repo) is tracked separately and is a prerequisite for this
playbook's steps to be run end-to-end by the UAT agent.

## 1. Top Navigation Bar

### 1.1 AppBarClusterBadge — removed (PRI-1993)

The cluster health badge that previously appeared in the Headlamp top navigation bar has been removed.

**Steps:**
1. Open Headlamp in a browser.
2. Inspect the top navigation bar.

**Expected:** No cluster-health badge appears in the top nav. The nav bar shows only the standard Headlamp controls (no extra Rook-Ceph badge or chip).

**Regression check:** If a badge, chip, or additional icon labeled with cluster health appears in the top nav, this is a regression.

## 2. Sidebar Navigation

**Steps:**
1. Confirm the Rook-Ceph section appears in the sidebar with all expected entries.

**Expected entries:**
- Overview
- Block Pools
- Storage Classes
- Volumes
- Pods
- Filesystems
- Object Stores

## 3. Overview Page

**Steps:**
1. Click **Overview** in the sidebar.
2. Confirm the ClusterStatusCard renders without errors.

**Expected:** Cluster status (health, FSID, version) displays correctly.

## 4. Block Pools Page

**Steps:**
1. Click **Block Pools**.
2. Verify the list of CephBlockPool objects loads.

**Expected:** Table renders; no console errors.

## 5. Storage Classes Page

**Steps:**
1. Click **Storage Classes**.
2. Verify only Rook-managed storage classes appear (RBD and CephFS provisioners).

**Expected:** Rows show storage class name, provisioner, reclaim policy.

## 6. Volumes Page

**Steps:**
1. Click **Volumes**.
2. Verify PVs bound to Rook provisioners appear.

**Expected:** Table lists volumes; status column reflects PV phase.

## 7. Pods Page

**Steps:**
1. Click **Pods**.
2. Verify daemon pods (operator, mon, osd, mgr, mds, rgw) appear.

**Expected:** Pod list with status; no non-Rook pods shown.

## 8. Filesystems Page

**Steps:**
1. Click **Filesystems**.
2. Verify CephFilesystem objects load.

**Expected:** Table renders correctly.

## 9. Object Stores Page

**Steps:**
1. Click **Object Stores**.
2. Verify CephObjectStore objects load.

**Expected:** Table renders correctly.

## 10. PVC / PV / Pod Detail Injections

**Steps:**
1. Navigate to any PVC detail view in Headlamp.
2. Scroll to the Rook-Ceph section at the bottom.
3. Repeat for a PV detail view and a Rook pod detail view.

**Expected:** Injected sections render pool/provisioner/block info without errors.

## Pass / Fail Criteria

- **Pass:** All sections above render without console errors or blank panels, and no AppBarClusterBadge appears in the top nav.
- **Fail:** Any section shows an error, blank panel, or the removed badge re-appears in the top nav.
