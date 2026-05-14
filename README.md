# ADF Repro — APP-10891

## Setup

- **ADF instance:** `sagie-repro-app10891` (project `653296462edd6d1437510dba`)
- **ADF config:** [`adf-config.json`](adf-config.json)
- **Azure container:** `sagie-test-app10891` (storage account `sttabulardatadev1`, eastus)

## Data files

3 files copied from staging, placed at:

```
f87b919c-63b2-4d32-b676-0b55393938d7/2574ics2q6/4406b6c3-ffd4-4f99-b01e-45e26a561ade/7f635c4d-b3f8-4cc7-8410-f119bb7b20b4/rdk:component:movement_sensor/etai/AngularVelocity/2025-04-16/data-1.bson.gz
f87b919c-63b2-4d32-b676-0b55393938d7/2574ics2q6/4406b6c3-ffd4-4f99-b01e-45e26a561ade/7f635c4d-b3f8-4cc7-8410-f119bb7b20b4/rdk:component:movement_sensor/etai/CompassHeading/2025-04-16/data-1.bson.gz
f87b919c-63b2-4d32-b676-0b55393938d7/2574ics2q6/4406b6c3-ffd4-4f99-b01e-45e26a561ade/7f635c4d-b3f8-4cc7-8410-f119bb7b20b4/rdk:component:movement_sensor/etai/LinearAcceleration/2025-04-16/data-1.bson.gz
```

The files in this repo (`AngularVelocity.bson.gz`, `CompassHeading.bson.gz`, `LinearAcceleration.bson.gz`) are those same files.

## Repro

Connect to the ADF instance and run:

### Bug 1: Internal error

```js
db.readings.aggregate([{
  "$match": {
    "$expr": {
      "$and": [
        { "$not": [{ "$eq": [{ "$type": ["$locationId"] }, { "$literal": "null" }] }] },
        { "$eq": ["$locationId", { "$literal": "FOO" }] }
      ]
    }
  }
}])
// MongoServerError: an internal error occurred, correlationID = 18af8200ce10ec3628dcf0b0
```

Note: the value (`"FOO"`) does not need to match any document — the 500 triggers as long as `locationId` (or any real field) is referenced in the second `$eq`.

### Bug 2: Zero results

```js
db.readings.aggregate([
  {
    "$match": {
      "$expr": {
        "$and": [
          { "$gt": ["$locationId", { "$literal": null }] },
          { "$eq": ["$locationId", { "$literal": "2574ics2q6" }] }
        ]
      }
    }
  },
  { "$limit": 10 }
])
// Returns empty — expected 10 documents matching locationId = "2574ics2q6"
```

Both queries work correctly on a standard MongoDB collection with the same data.
