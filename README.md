# Offline QR Capture PWA — V2

Version 2.0.0 is a fully offline-first mobile PWA for photographing an oil-sample bottle carrying exactly one Label A (Bottle/Location) QR and one Label B (Mission/Sample) QR.

## V2 workflow

1. First launch registers the device (Device Code, Device Name, optional location) and creates the local Administrator.
2. Admin creates local users. Passwords are stored as PBKDF2 verifiers, not plaintext.
3. Collector logs in, opens New Scan, photographs the complete bottle, and keeps Label A, Label B and handwritten Label B fields visible.
4. The app decodes exactly two QRs offline and validates them by schema, not by detection order.
5. Label A example: `00001/01/01/A/1/M/zz;` → Bottle UUID, Rack, Shelf, Crate, X, Y, Future/Reserved.
6. Label B example: `00001/SB-000/11-00-000/LH;` → Sticker UUID, Aircraft Tail Number, Engine Number, Engine Side.
7. The app adds Oil Spec (default `Turbonic210A`), logged-in collector, registered device, system timestamp, timezone and GPS.
8. The JPEG is hashed with SHA-256. Images are stored in OPFS where supported, with IndexedDB Blob fallback. Metadata/indexes are in IndexedDB.
9. Engine Hours can be entered later from Record Details and records who/when entered it.
10. Export New Records creates a ZIP containing only pending records for that user. Admin can Export All Pending Records across local users.

## Device identity

Each installation has:
- random `device_uuid`
- human-readable `device_code` such as `TAB-001`
- `device_name` such as `Storage Lab Tablet 01`
- optional device location/department

Every record and export contains device identity. The internal record key remains a UUID. A readable reference such as `TAB-001-20260920-000001` is also generated.

## QR and bottle requirements

The app accepts exactly one valid Label A and one valid Label B. Two Label A codes, two Label B codes, unknown QRs, one QR, or more than two QRs are rejected.

Bottle curvature can distort QR codes. Test the exact printed label material, QR size, bottle curvature, camera distance, lighting and device models before field rollout. Torch control is shown when the browser/device camera exposes torch capability.

This build uses the browser's offline `BarcodeDetector` API. Test the exact Android Chrome/Edge version to be deployed. A future decoder can replace `assets/qr-reader.js` if field tests require stronger cylindrical-distortion handling (for example a locally bundled computer-vision decoder). No runtime CDN is used.

## Storage and durability

- IndexedDB: users, metadata, indexes, settings and fallback image Blobs.
- OPFS: preferred captured JPEG storage when supported.
- `navigator.storage.persist()` is requested where supported.
- Dashboard shows usage/quota and warns at the configured 70% ratio.
- Browser-managed storage is not the permanent archive. Export pending records regularly and transfer ZIPs to the central PC/archive.
- Clearing site data, uninstall/reset behavior, device loss or browser-profile damage can still destroy local-only records.

## Record lifecycle

New records start `pending`. Successful incremental export marks them `exported` with `exported_at` and `export_batch_id`. Export does not delete records. Delete actions are pseudo-delete: the record is retained locally with `deleted_at` and `deleted_by` and excluded from normal record lists/exports.

## Export format

ZIP contains:
- `records.json`
- `manifest.json`
- `images/*.jpg`

The export includes App Version, DB Schema Version, QR Schema Version, device identity, batch UUID and image SHA-256 values.

## Versions

- App Version: 2.0.0
- DB Schema Version: 2
- QR Schema Version: 1
- Service-worker cache: `offline-qr-pwa-v3`

Versions are shown on the Dashboard and stored with each new capture.

## Installation

Serve the complete folder through HTTPS, e.g. `https://YOUR-DOMAIN/Offline_QR_Capture/`. Complete Device Registration + Administrator setup, install/add to home screen, then test the full workflow in airplane mode.

Camera, geolocation, service worker and persistent storage APIs depend on browser/device support and secure context (HTTPS).

## Important multi-device note

Each tablet has its own offline user database. This V2 build supports device identity but does not silently synchronize users between tablets because there is no server. Create/provision authorized users on each required device. If centralized user provisioning is later required, add an authenticated/signed provisioning workflow rather than browser fingerprinting or hardware-ID collection.

## Acceptance tests before deployment

Test: first setup; device registration; admin/user role restrictions; camera; torch where supported; real Label A + Label B bottles; cylindrical curvature; poor lighting; QR schema rejection cases; GPS required/optional behavior; OPFS and IndexedDB fallback; persistence after restart; Engine Hours later entry; pseudo-delete; incremental user export; Admin all-pending export; ZIP contents/hashes; USB transfer; storage warning; and complete airplane-mode operation.

