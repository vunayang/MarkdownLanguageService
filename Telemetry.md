# Markdown telemetry

All Markdown telemetry data is transmitted within the `vs/editor/properties` event when the text view closes.
Various properties and measures are prefixed with `vs.editor.markdown`

## Measurements

### Repeated operations
To measure duration of a repeated operation, use `TelemetryAggregator.Measure` and pass in `HistogramProfile` which best matches the expected values.

These measurements are aggregated in histograms with buckets named after the largest value it carries, e.g. `.25`, `.50`, `.100`, `.250`, `.1000` [ms] and `.inf` for values greater than 1000ms.
These histograms also have a telemetry measure `.max` which stores the single largest observed value.

Available bucketized measures, where `_` is the bucket name or `max`
- `vs.editor.markdown.parsedocument._`
- `vs.editor.markdown.parsedocumenthandlers._`
- `vs.editor.markdown.getspellcheckablespans._`
- `vs.editor.markdown.preview.gethtml._`
- `vs.editor.markdown.preview.updatecontent.update._`

### Singular operations

To measure duration of a singular operation, use `TelemetryAggregator.Measure` and pass in `HistogramProfile.SingleMeasurement`.
The measured value will not be bucketized and will be included in telemetry in full fidelity.

Available singular measurements which are not in histograms
- `vs.editor.markdown.preview.initializewebview`
- `vs.editor.markdown.preview.createcontrols`
- `vs.editor.markdown.preview.updatecontent.first`

### Counters

To increment a counter, use `TelemetryAggregator.AddUsage`

Available counters
- `vs.editor.markdown.spellchecker` - 1 if spellchecker was used
- `vs.editor.markdown.previewEnabled` - incremented every time preview is shown
- `vs.editor.markdown.previewDisabled` - incremented every time preview is hidden
- `vs.editor.markdown.previewStartsEnabled` - 1 if preview was visible by default
- `vs.editor.markdown.previewStartsDisabled` - 1 if preview is hidden by default
- `vs.editor.markdown.missingLocalBrowsingFilePath` - error condition in preview
- `vs.editor.markdown.objectdisposedexception` - error condition in preview
- `vs.editor.markdown.renderhtmlexception` - error condition in preview

### Set values

To include specific data in telemetry, use `TelemetryAggregator.SetData`

Available data
- `vs.editor.markdown.linecount` - Last encountered line count in file

## Questions we need to answer:

Telemetry helps us answer questions such as:
- What's the current state of "Show preview by default"?
- How often do users interact with the "Show preview" button?
- Preview related delays
- Parse delays

## ShipReady

ShipReady plots a single value of interest.
It allows filtering, but ultimately it's just the single value that's being plotted.
We attempt to recreate a _typical_ value by taking a weighted average of the histogram data.

```
RawEventsVS 
| where AdvancedServerTimestampUtc > ago(1d)
| where EventName == "vs/editor/properties"
| extend b1 = toint(Measures["vs.editor.markdown.parsedocument.8"])
| extend b2 = toint(Measures["vs.editor.markdown.parsedocument.16"])
| extend b3 = toint(Measures["vs.editor.markdown.parsedocument.25"])
| extend b4 = toint(Measures["vs.editor.markdown.parsedocument.50"])
| extend b5 = toint(Measures["vs.editor.markdown.parsedocument.100"])
| extend b6 = toint(Measures["vs.editor.markdown.parsedocument.250"])
| extend b7 = toint(Measures["vs.editor.markdown.parsedocument.inf"])
| extend max = toint(Measures["vs.editor.markdown.parsedocument.max"])
| project b1, b2, b3, b4, b5, b6, max, UserId
| summarize ShipReadyScore = (sum(b1)*4 + sum(b2)*10 + sum(b3)*20 + sum(b4)*37 + sum(b5)*75 + sum(b6)*175 + sum(b7)*500) / (sum(b1)+sum(b2)+sum(b3)+sum(b4)+sum(b5)+sum(b6)+sum(b7)) by UserId, max
```

## Error data
Fault events reported through `VSTaskLibraryHelper.FileAndForget`:
- `vs/editor/markdown/preview/missingLocalBrowsingFilePath`
- `vs/editor/markdown/preview/renderhtmlexception`
- `vs/editor/markdown/preview/objectdisposedexception`
