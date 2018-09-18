# VR-725

## AWS-CLI

```
// list all items in table
aws dynamodb scan --table-name vicky-uat-spreadsheet --endpoint-url http://localhost:8000
```

## CONFIG

**vicky.config.resources.orca.timeToLiveInSeconds**

determines `timeToLivInSeconds` and `expirationDate` values in vicky-spreadsheet items

## Report generation flow
- open report generation page in vicky, click generate report
- calls /api/spreadsheet with the following info
```js
{
  "exportName": "DEALER_PERFORMANCE_SUMMARY",
  "filename": "DealerPerformanceSummaryReport_20180918T112355Z",
  "filterParameters": {
    "years": [ 2018 ],
    "quarters": [ 4 ],
    "dealerLocations": [ 1811801 ]
  },
  "outputFormat": "xlsx"
}
```
-

## Report status lookup flow
- open reports page in vicky
- GeneratedReports component calls asyncReportsManager.getReports
- getReports calls /api/spreadsheet/status
-
