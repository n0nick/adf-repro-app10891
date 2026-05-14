# ADF Repro — APP-10891

## Setup

- **ADF instance:** `sagie-repro-app10891` (project `653296462edd6d1437510dba`)
- **ADF config:** [`adf-config.json`](adf-config.json)
- **Azure container:** `sagie-test-app10891` (storage account `sttabulardatadev1`, eastus)

## Data

A single file with 1 document is sufficient to reproduce both bugs. Both `.bson` and `.bson.gz` reproduce — compression is irrelevant.

The file in this repo ([`AngularVelocity.bson`](AngularVelocity.bson)) is placed in the container at:

```
f87b919c-63b2-4d32-b676-0b55393938d7/2574ics2q6/4406b6c3-ffd4-4f99-b01e-45e26a561ade/7f635c4d-b3f8-4cc7-8410-f119bb7b20b4/rdk:component:movement_sensor/etai/AngularVelocity/2025-04-16/data-1.bson
```

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
// MongoServerError: an internal error occurred, correlationID = 18af83035bb60b9f7b414d34
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
// Returns empty — expected 1 document matching locationId = "2574ics2q6"
```

Both queries work correctly on a standard MongoDB collection with the same data.
