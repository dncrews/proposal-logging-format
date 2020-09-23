# Allē Logging Format Proposal

## Current status

Stage 0  
Mimicking the [TC39 Process](https://tc39.es/process-document/)

---
## Abstract

This is a proposal for creating a standardized Logging format for Allē applications.

This proposal does not mandate implementation, language, etc., but only describesthe final output
that is sent to our logs and to our log aggregation provider.

## Rationale

In Allē logs, we do not have any consistency around our logging formats. We now have more logs, but
the inconsistency makes it difficult to be able to chart specific problems that we may be facing
and to break them down by aggregate in our logging platform. The addition of a standardized format
will simplify our logs and allow us to create parsers that can properly process and analyze our
systems.

### Additional Considerations

- Thoughtful fields: in some log aggregation platforms, there are limits on the number of indexable
fields.
- Datatypes: numbers should be numbers; strings should be strings; objects should be shallow


## Gathered Data

When we want to track an event or a log, we should consider each log as a metric, and vice versa.
All data can be represented in the form of a metric. Rather than a log:

```
"login successful: id 22d08057-ce47-499e-aa41-903e2c77990e"
```

A parseable metric:

```
{ "name": "users.login.success", "type": "COUNT", "value": 1, "data": { "id": "22d08057-ce47-499e-aa41-903e2c77990e" }}
```

The three types of metrics are `count`, `measure`, and `sample`. A `count` or `measure` represents
non-dynamic "truths", and the difference between them generally comes down to the need for units.
The most common use of `count` will likely be for when you would otherwise use logs (as in the)
login success example above. The most common use of `measure` will likely be in timing, where the
units would be `ms`. `sample` should be for measuring a snapshot of ever-changing data. Take these
examples of metrics from a queue-driven system:

#### Count
> I processed _this item_ 1 time  
> I processed 10 items  

#### Measure
> It took me 120 ms to process one item  
> This file was 100 MB in size  
> It took me 1022 ms to process all items

#### Sample
> There are 100 items remaining on the queue  
> I am currently consuming 106.2 MB of memory


## Recommended Output Format

```typescript
{
  name: string; // This should include enough information to be unique and identifiable
  type: 'COUNT' | 'MEASURE' | 'SAMPLE';
  value?: number;
  units?: string;
  data: Record<string, number | string | number[] | string[] | boolean | void | null>
}
```

Using the previous examples

#### Count
```
{ "name": "email.processEmailQueue.itemComplete", "type": "COUNT", "value": 1, "data": { "id": "22d08057-ce47-499e-aa41-903e2c77990e" }}  
{ "name": "email.processEmailQueue.batchSize", "type": "COUNT", "value": 10 }  
```

#### Measure
```
{ "name": "email.processEmailQueue.itemProcessTime", "type": "MEASURE": "value": 120, "units": "ms", "data": { "id": "22d08057-ce47-499e-aa41-903e2c77990e" }}  
{ "name": "email.processEmailQueue.itemFileSize", "type": "MEASURE": "value": 100, "units": "MB", "data": { "id": "22d08057-ce47-499e-aa41-903e2c77990e" }}  
{ "name": "email.processEmailQueue.batchProcessTime", "type": "MEASURE": "value": 1022, "units": "MB" }  
```

#### Sample
```
{ "name": "email.processEmailQueue.queueSize", "type": "SAMPLE", "value": 100 }  
{ "name": "email.processEmailQueue.memoryUsage", "type": "SAMPLE", "value": 106.2, "units": "MB" }  
```

### Additional Considerations

- Thoughtful fields: in some log aggregation platforms, there are limits on the number of indexable
fields.
- Datatypes: numbers should be numbers; strings should be strings; objects should be shallow
