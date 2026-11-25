# MyGeotab Fleet Booking Add-in

A lightweight, browser-hosted MyGeotab custom page add-in for booking shared fleet vehicles. Booking records are stored in MyGeotab as `AddInData`; no external database is required.

## Features

- Create, view, edit, and cancel vehicle booking requests.
- **Administrator-controlled assignment**: Users submit booking requests; administrators review, assign vehicles and drivers from actual MyGeotab records, and approve or reject.
- Vehicle and driver conflict validation before approval.
- Browse the vehicles available to the signed-in MyGeotab account.
- Review booking history with date-range, vehicle, status, and requester filters.
- Use a responsive dashboard with **Booking**, **Approval**, **Users**, **Reports**, and **Vehicles** sections.

## Dashboard

| Section | Current behavior |
| --- | --- |
| **Booking** | Submit booking requests (date, time, purpose, notes). Users cannot select vehicles or drivers. Only the request owner can edit or withdraw a pending request. |
| **Approval** | Administrators review pending requests, assign vehicles (from MyGeotab Device records) and drivers (from eligible MyGeotab User records), validate conflicts, and approve or reject. |
| **Vehicles** | View the vehicles available to the current account, including inactive vehicles. |
| **Users** | Read-only list of user records visible to the current MyGeotab session, with search, account status, functional role, driver status, and Clearance names resolved from each user's assigned security-group IDs. |
| **Reports** | Read-only booking history with date-range, vehicle, status, and requester filters; summary counts and pagination. Date filters include bookings that overlap any part of the selected range. |

## Setup

1. **Keep the add-in ID stable.** `ADDIN_ID` in `fleet-summary.html` must remain unchanged across deployments to the same MyGeotab database. If you create a separate add-in in that database, use a unique ID. See [Geotab Storage API instructions](https://developers.geotab.com/myGeotab/addIns/addInStorage/).
2. **Host the required files over HTTPS.** Place `fleet-summary.html`, `booking-domain.js`, and `booking.css` in the same folder. The page loads its assets using relative URLs.
3. **Configure the add-in.** Update the hosted page URL and `supportEmail` in `configuration.json`, then add the configuration in **Administration > System > System Settings > Add-Ins**.
4. **Grant the required clearances.** Users need permission to read `Device` objects and read/write `AddInData`. The Users tab also requires permission to read `User` records; if unavailable, the rest of the add-in remains usable and the tab displays the API error.
5. **Administrator access:** Approval actions require MyGeotab security clearance of Administrator or Supervisor (security group IDs `everythingsecurity` or `supervisorsecurity`).

## Booking behavior and limitations

- Start and end times are entered in the browser's local time zone and stored as ISO 8601 UTC timestamps.
- Users submit **booking requests** with status `pending`; they cannot select vehicles or drivers.
- Administrators assign a vehicle (from MyGeotab Device records) and an eligible driver (from MyGeotab User records where `isDriver=true` and account is Active).
- A booking becomes `confirmed` only after **both** vehicle and driver are assigned and the administrator approves.
- Overlapping confirmed bookings for the same vehicle or driver are rejected during approval.
- Cancelling a confirmed booking sets its status to `cancelled`; the record is retained.
- Withdrawing a pending request sets its status to `cancelled`.
- Rejected requests are retained with status `rejected`.
- Reports include confirmed, cancelled, pending, and rejected records. Date-range filtering uses the booking period, not the record creation time.
- Report filtering and pagination run in the browser over the booking records returned by MyGeotab; pagination does not limit the records fetched from the API.
- The Users tab shows only user records visible to the current session. MyAdmin accounts may not be visible to BasicAuthentication users; account status is derived from `activeFrom` and `activeTo` and does not guarantee successful authentication.
- MyGeotab functional role is optional and may be unassigned. The Users table resolves Clearance names by matching each user's `securityGroups` IDs with MyGeotab security-clearance Group records (`GroupSecurityId`). The built-in Administrator, Supervisor, Default user, Drive App user, View only, and Nothing technical names are normalized to their MyGeotab UI labels; custom clearance names are shown as returned by the API. A group that cannot be resolved is shown as `Unmapped clearance`, and a failed Group lookup is shown as `Not available`.
- Conflict checking runs in the client before saving and is not atomic. Two users can still create overlapping booking requests at the same time. The add-in checks again after saving and warns if a conflict is detected. Workflows that require a strict guarantee need a transactional backend.
- Administrator approval also re-validates vehicle and driver availability immediately before saving to minimize race conditions.

## Data model (AddInData.details)

Booking records store the following fields:

- `bookingKey` — unique identifier for the booking
- `deviceId` / `deviceName` — legacy fields for confirmed bookings created before approval workflow
- `assignedVehicleId` / `assignedVehicleName` — vehicle assigned by administrator
- `assignedDriverId` / `assignedDriverName` — driver assigned by administrator
- `startUtc` / `endUtc` — booking period in ISO 8601 UTC
- `status` — `pending`, `confirmed`, `cancelled`, or `rejected`
- `purpose` — user-provided purpose
- `notes` — user-provided additional notes
- `approvalNotes` — administrator notes during approval
- `createdBy` / `createdAtUtc` — requester and creation timestamp
- `approvedBy` / `approvedAt` — administrator and approval timestamp (when confirmed)
- `rejectedBy` / `rejectedAt` / `rejectionReason` — administrator and rejection details (when rejected)
- `updatedAtUtc` — last modification timestamp

## Development

Booking time validation, overlap checks, and report filters are in `booking-domain.js`. Run the tests with Node.js:

```sh
node --test tests/booking-domain.test.js
```

### Project files

- `fleet-summary.html` — dashboard markup and MyGeotab add-in behavior.
- `booking.css` — responsive dashboard and booking styles.
- `booking-domain.js` — booking time, conflict, and report filter logic.
- `configuration.json` — MyGeotab add-in configuration template.
- `tests/booking-domain.test.js` — focused tests for booking and report domain logic.

## References

- [Developing MyGeotab Add-Ins](https://developers.geotab.com/myGeotab/addIns/developingAddIns/)
- [Geotab Storage API](https://developers.geotab.com/myGeotab/addIns/addInStorage/)
- [AddInData object reference](https://developers.geotab.com/myGeotab/apiReference/objects/AddInData/)
- [User object reference](https://developers.geotab.com/myGeotab/apiReference/objects/User/)
