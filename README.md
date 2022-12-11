# recon-definition-language
Examples/Documentation of Reconciliation Definition Language (RDL)

# First mandatory step is to define one or many data sources.
## Define Data source called 'sales'
```
DEFINE DATASOURCE sales WITH BUCKET FIELD entityId AS (name='sales' order by id)
```

## Define Another Data source called 'tdsledger'
```
DEFINE DATASOURCE tdsLedger WITH BUCKET FIELD entityId AS (name='tdsLedger' order by id)
```

## Define a rule that checks the transactions (one to one matches) at every entry level.
```
DEFINE RULE EntryChecks OF TYPE EntryWiseOneToOneChecks AS  
(sales.invoiceNo == tdsLedger.invoiceNo AND valueWithinTolerance(sales.tdsOnSales, tdsLedger.tds, 1.0))
TAGS WHEN MATCHED (TDS_MATCHED_BETWEEN_SALES_AND_TDS_LEDGER) TAGS WHEN NOT MATCHED (TDS_UNMATCHED_BETWEEN_SALES_AND_TDS_LEDGER)
```

## Define a rule that checks the transactions (one to many) at every entry level.
```
DEFINE RULE EntryOneToManyChecks A OF TYPE EntryWiseOneToManyChecks AS 
(sales.invoiceNo == tdsLedger.invoiceNo AND valueWithinTolerance(sales.tdsOnSales, tdsLedger.tds, 1.0))
TAGS WHEN MATCHED (TDS_MATCHED_BETWEEN_SALES_AND_TDS_LEDGER) TAGS WHEN NOT MATCHED (TDS_UNMATCHED_BETWEEN_SALES_AND_TDS_LEDGER)
```

## Define a rule that checks the fields for a sales datasource.
```
DEFINE RULE FieldChecks A OF TYPE FieldChecks AS 
(valueWithinTolerance(sales.tdsOnSales, 0.1 * sales.transactionAmount, 1.0))
TAGS WHEN MATCHED (CORRECT_TDS_ON_SALES) TAGS WHEN NOT MATCHED (INCORRECT_TDS_ON_SALES)
```

## Define a rule that checks the fields for a tdsLedger datasource.
```
DEFINE RULE FieldChecks B OF TYPE FieldChecks AS 
(tdsLedger.tds > 0)
TAGS WHEN MATCHED (CORRECT_TDS_ON_TDS_LEDGER) TAGS WHEN NOT MATCHED (INCORRECT_TDS_ON_TDS_LEDGER)
```

## Define a rule that checks the totals.
```
DEFINE RULE TotalsChecks OF TYPE TotalsChecks AS 
(sales.invoiceNo == tdsLedger.invoiceNo AND valueWithinTolerance(sales.tdsOnSales, tdsLedger.tds, 1.0))
TAGS WHEN MATCHED (TDS_TOTALS_MATCHED_BETWEEN_SALES_AND_TDSLEDGER) TAGS WHEN NOT MATCHED (TDS_TOTALS_UNMATCHED_BETWEEN_SALES_AND_TDSLEDGER)
```

## Putting all together, A sample definition file would look like this. 
`Every rule must start in a new line`
```
DEFINE DATASOURCE sales WITH BUCKET FIELD entityId AS (name='sales' order by id),

DEFINE DATASOURCE tdsLedger WITH BUCKET FIELD entityId AS (name='tdsLedger' order by id)

DEFINE RULE EntryChecks OF TYPE EntryWiseOneToOneChecks AS 
(sales.invoiceNo == tdsLedger.invoiceNo AND valueWithinTolerance(sales.tdsOnSales, tdsLedger.tds, 1.0))
TAGS WHEN MATCHED (TDS_MATCHED_BETWEEN_SALES_AND_TDS_LEDGER) TAGS WHEN NOT MATCHED (TDS_UNMATCHED_BETWEEN_SALES_AND_TDS_LEDGER)

DEFINE RULE EntryOneToManyChecks A OF TYPE EntryWiseOneToManyChecks AS 
(sales.invoiceNo == tdsLedger.invoiceNo AND valueWithinTolerance(sales.tdsOnSales, tdsLedger.tds, 1.0))
TAGS WHEN MATCHED (TDS_MATCHED_BETWEEN_SALES_AND_TDS_LEDGER) TAGS WHEN NOT MATCHED (TDS_UNMATCHED_BETWEEN_SALES_AND_TDS_LEDGER)

DEFINE RULE FieldChecks A OF TYPE FieldChecks AS 
(valueWithinTolerance(sales.tdsOnSales, 0.1 * sales.transactionAmount, 1.0))
TAGS WHEN MATCHED (CORRECT_TDS_ON_SALES) TAGS WHEN NOT MATCHED (INCORRECT_TDS_ON_SALES)

DEFINE RULE FieldChecks B OF TYPE FieldChecks AS (tdsLedger.tds > 0)
TAGS WHEN MATCHED (CORRECT_TDS_ON_TDS_LEDGER) TAGS WHEN NOT MATCHED (INCORRECT_TDS_ON_TDS_LEDGER)

DEFINE RULE TotalsChecks OF TYPE TotalsChecks AS 
(sales.invoiceNo == tdsLedger.invoiceNo AND valueWithinTolerance(sales.tdsOnSales, tdsLedger.tds, 1.0))
TAGS WHEN MATCHED (TDS_TOTALS_MATCHED_BETWEEN_SALES_AND_TDSLEDGER) TAGS WHEN NOT MATCHED (TDS_TOTALS_UNMATCHED_BETWEEN_SALES_AND_TDSLEDGER)
```

## Functions Reference:
```
equals(String, String)
```

```
startsWith(String, String)
```

```
contains(a: String, b: String) - If a contains b
```

```
containsAny(a: String, b: String) - If a contains b or b contains a
```

```
dateMatch(dateStringInAnyFormatA, dateStringInAnyFormatB, toleranceInDaysDefaultedToZero)
```

```
fuzzyMatch(stringA, stringB, thresholdScoreLessThan1, ["stopWords1", "stopWords2"]) - stop words are optional
```

```
valueWithinTolerance(numericValueA, numericValueB, numericTolerance)
```




Feedback welcome.

