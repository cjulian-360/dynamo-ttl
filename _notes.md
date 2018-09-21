# VR-725

## AWS-CLI

```
// list all items in table
aws dynamodb scan --table-name vicky-uat-spreadsheet --endpoint-url http://localhost:8000

// query for item with specific ID
aws dynamodb query --table-name orca-uat-spreadsheet \
    --key-condition-expression "id = :id" \
    --expression-attribute-values '{ ":id" : { "S": "1f8c9489-be02-42a2-80be-64ce234c9235" } }'  \
    --endpoint-url https://dynamodb.us-east-1.amazonaws.com
```

## CONFIG

**vicky.config.resources.orca.timeToLiveInSeconds**

determines `timeToLiveInSeconds` and `expirationDate` values in vicky-spreadsheet items

## Report generation flow
- open report generation page in vicky, click generate report
- browser calls `/api/spreadsheet/create` with the following info:
```json
{
  "exportName": "COOPER_DEALER_PERFORMANCE_SUMMARY",
  "filename": "DealerPerformanceSummaryReport_20180918T144726Z",
  "filterParameters": {
    "years": [
      2018
    ],
    "quarters": [
      1,
      2,
      3,
      4
    ],
    "dealerLocations": [
      1811801
    ]
  },
  "outputFormat": "xlsx"
}
```

- server createSpreadsheetHandler adds userId, clientId
- request sent to POST http://orca.test.360incentives.io/create
```json
{
  "exportName": "COOPER_DEALER_PERFORMANCE_SUMMARY",
  "clientCode": "CMUS",
  "filename": "DealerPerformanceSummaryReport_20180918T144726Z",
  "filterParameters": {
    "years": [
      2018
    ],
    "quarters": [
      1,
      2,
      3,
      4
    ],
    "dealerLocations": [
      1811801
    ],
    "userId": 2236648,
    "clientId": 238
  },
  "locale": "en",
  "outputFormat": "xlsx",
  "userId": "2236648"
}
```

- 'spreadsheet' object received in response
```json
{
  "timeToLiveInSeconds": 600,
  "createdDate": "2018-09-18T18:42:44.034Z",
  "modifiedDate": "2018-09-18T18:42:44.034Z",
  "exchangeName": "vicky",
  "exportName": "COOPER_DEALER_PERFORMANCE_SUMMARY",
  "exportParameters": {
    "filename": "DealerPerformanceSummaryReport_20180918T144128Z",
    "locale": "en",
    "outputFormat": "xlsx",
    "clientCode": "CMUS",
    "filterParameters": {
      "clientId": 238,
      "userId": 2236648,
      "years": [2018],
      "quarters": [4],
      "dealerLocations": [1811801]
    }
  },
  "id": "a94e0913-9e60-4cce-a297-0c93d65dafbd",
  "routingKey": "vicky-spreadsheet-status",
  "expirationDate": "2018-09-18T18:52:44.034Z",
  "status": "Accepted"
}
```

- spreadsheet is supplemented with `userId` and `isDownloaded` and put in vicky database
```json
{
  "timeToLiveInSeconds": 600,
  "createdDate": "2018-09-18T18:52:01.075Z",
  "modifiedDate": "2018-09-18T18:52:01.075Z",
  "exchangeName": "vicky",
  "exportName": "COOPER_DEALER_PERFORMANCE_SUMMARY",
  "exportParameters": {
    "filename": "DealerPerformanceSummaryReport_20180918T144726Z",
    "locale": "en",
    "outputFormat": "xlsx",
    "clientCode": "CMUS",
    "filterParameters": {
      "clientId": 238,
      "userId": 2236648,
      "years": [
        2018
      ],
      "quarters": [
        1,
        2,
        3,
        4
      ],
      "dealerLocations": [
        1811801
      ]
    }
  },
  "id": "8ee1a4c7-22e2-42f5-a348-3b3f91baa0c9",
  "routingKey": "vicky-spreadsheet-status",
  "expirationDate": "2018-09-18T19:02:01.075Z",
  "status": "Accepted",
  "userId": "2236648",
  "isDownloaded": false
}
```

- `id` from above is then sent to `http://orca.test.360incentives.io/generate`
- response:
```json
{
  "timeToLiveInSeconds": 600,
  "createdDate": "2018-09-18T18:52:01.075Z",
  "modifiedDate": "2018-09-18T18:58:27.925Z",
  "exchangeName": "vicky",
  "exportName": "COOPER_DEALER_PERFORMANCE_SUMMARY",
  "exportParameters": {
    "filename": "DealerPerformanceSummaryReport_20180918T144726Z",
    "locale": "en",
    "outputFormat": "xlsx",
    "clientCode": "CMUS",
    "filterParameters": {
      "clientId": 238,
      "userId": 2236648,
      "years": [
        2018
      ],
      "quarters": [
        1,
        2,
        3,
        4
      ],
      "dealerLocations": [
        1811801
      ]
    }
  },
  "id": "8ee1a4c7-22e2-42f5-a348-3b3f91baa0c9",
  "routingKey": "vicky-spreadsheet-status",
  "expirationDate": "2018-09-18T19:08:27.925Z",
  "status": "Queued"
}
```
- item in vicky dynamodb is updated with values from response (updated `status`)



## Report status lookup flow
- open reports page in vicky
- GeneratedReports component calls asyncReportsManager.getReports
- getReports calls /api/spreadsheet/status
-


## expiry
- at regular intervals orca scans dynamo table for expired records (expirationDate < currentDate)
- deletes files from s3
- updates status of expired spreadsheets to 'EXPIRED'
- publishes a status update message (rabbit)
- vicky picks up status update message, sees 'EXPIRED' status and deletes the spreadsheet
