# API Reference
> [JSON-stat Command Line Conversion Tools](https://github.com/jsonstat/conv/blob/master/README.md) ▸ **API Reference**

## Shared options

### --help (-h)

Shows command's help.

```
jsonstat2csv --help
```

### --stream (-t)

Use the stream interface:

```
jsonstat2csv < oecd.json > oecd.csv -t
```

### --version

Shows jsonstat-conv version.

```
jsonstatslice --version
```

## Available commands

* [arrow2jsonstat](https://github.com/jsonstat/conv/blob/master/docs/API.md#arrow2jsonstat) - converts an Apache Arrow file to JSON-stat
* [csv2jsonstat](https://github.com/jsonstat/conv/blob/master/docs/API.md#csv2jsonstat) - converts CSV into JSON-stat
* [jsonstat2array](https://github.com/jsonstat/conv/blob/master/docs/API.md#jsonstat2array) - converts JSON-stat into an array of arrays
* [jsonstat2arrobj](https://github.com/jsonstat/conv/blob/master/docs/API.md#jsonstat2arrobj) - converts JSON-stat into an array of objects
* [jsonstat2arrow](https://github.com/jsonstat/conv/blob/master/docs/API.md#jsonstat2arrow) - converts JSON-stat to the Apache Arrow format
* [jsonstat2csv](https://github.com/jsonstat/conv/blob/master/docs/API.md#jsonstat2csv) - converts JSON-stat into CSV
* [jsonstat2objarr](https://github.com/jsonstat/conv/blob/master/docs/API.md#jsonstat2objarr) - converts JSON-stat into an object of column-oriented arrays
* [jsonstat2object](https://github.com/jsonstat/conv/blob/master/docs/API.md#jsonstat2object) - converts JSON-stat into a DataTable object
* [jsonstatdice](https://github.com/jsonstat/conv/blob/master/docs/API.md#jsonstatdice) - creates JSON-stat from JSON-stat
* [jsonstatslice](https://github.com/jsonstat/conv/blob/master/docs/API.md#jsonstatslice) (deprecated) - creates JSON-stat from JSON-stat
* [sdmx2jsonstat](https://github.com/jsonstat/conv/blob/master/docs/API.md#sdmx2jsonstat) - converts SDMX into JSON-stat

## arrow2jsonstat

Converts an Apache Arrow file created with [jsonstat2arrow](#jsonstat2arrow) back to JSON-stat.

```
csv2jsonstat oecd.arrow oecd.json
```

## csv2jsonstat

Converts a CSV created with [jsonstat2csv](#jsonstat2csv) back to JSON-stat.

```
csv2jsonstat galicia.csv galicia.json
```

All options are ignored when the input is a rich CSV in the [JSON-stat Comma-Separated Values format](https://github.com/jsonstat/csv) (CSV-stat or JSV for short). See [jsonstat2csv](#jsonstat2csv).

### Options

#### --vlabel (-v)

String. Specifies the name of the value column. When not provided, the value column must be the last one.

```
csv2jsonstat galicia.csv galicia.json --vlabel val
```

#### --slabel (-l)

String. Specifies the name of the status column. Default is "Status". If no column has the specified name, no status information will be included in the output.

```
csv2jsonstat oecd.csv oecd.json --slabel stat
```

#### --column (-c)

String. Column separator. Default is ",".

```
csv2jsonstat galicia.ssv galicia.json --column ";"
```

#### --decimal (-d)

String. Decimal separator. Default is ".", unless column separator is ";" (then default is ",").

```
csv2jsonstat galicia.tsv galicia.json --column "\t" --decimal ","
```

#### --label (-a)

String. Dataset label.

```
csv2jsonstat galicia.csv galicia.json --label "Imported from galicia.csv"
```

#### --ovalue (-o)

Boolean. Includes a "value" property of type array instead of object.

#### --ostatus (-s)

Boolean. When status information is available, includes a "status" property of type array instead of object.

## jsonstat2array

Converts JSON-stat into an array of arrays.

```
jsonstat2array oecd.json oecd-array.json
```

### Options

#### --status (-s)

Boolean. Includes status information.

```
jsonstat2array oecd.json oecd-array.json --status
```

#### --vlabel (-v)

String. Specifies the label of the value field. Default is "Value".

```
jsonstat2array oecd.json oecd-array.json --vlabel val
```

#### --slabel (-l)

String. Specifies the label of the status field. Default is "Status".

```
jsonstat2array oecd.json oecd-array.json --status --slabel stat
```

#### --fid (-f)

Boolean. Identifies dimensions, value and status by ID instead of label.

```
jsonstat2array oecd.json oecd-array.json --fid
```

#### --cid (-c)

Boolean. Identifies categories by ID instead of label.

```
jsonstat2array oecd.json oecd-array.json --cid
```

## jsonstat2arrobj

Converts JSON-stat into an array of objects.

```
jsonstat2arrobj oecd.json oecd-arrobj.json
```

### Options

#### --status (-s)

Boolean. Includes status information.

```
jsonstat2arrobj oecd.json oecd-arrobj.json --status
```

#### --vlabel (-v)

String. Specifies the label of the value field. Default is "Value".

```
jsonstat2arrobj oecd.json oecd-arrobj.json --vlabel val
```

#### --slabel (-a)

String. Specifies the label of the status field. Default is "Status".

```
jsonstat2arrobj oecd.json oecd-arrobj.json --status --slabel stat
```

#### --flabel (-f)

Boolean. Identifies dimensions, value and status by label instead of ID.

```
jsonstat2arrobj oecd.json oecd-arrobj.json --fid
```

#### --cid (-c)

Boolean. Identifies categories by ID instead of label.

```
jsonstat2arrobj oecd.json oecd-arrobj.json --cid
```

#### --unit (-u)

Boolean. Includes unit information in a property called "unit". If the dataset does not include unit information, the value of the "unit" property will be null.

```
jsonstat2arrobj oecd.json oecd-arrobj.json --unit
```

#### --meta (-m)

Boolean. Returns a metadata-enriched output (object of objects, instead of array of objects).

```
jsonstat2arrobj oecd.json oecd-objobj.json --meta
```

#### --comma (-k)

Boolean. Represents values as strings instead of numbers with comma as the decimal mark.

```
jsonstat2arrobj canada.json canada-comma.json --comma
```

#### --by (-b)

String. Transposes data by the specified dimension. String must be an existing dimension ID. Otherwise it will be ignored.

```
jsonstat2arrobj oecd.json oecd-transp.json --by area
```

#### --bylabel (-l)

Boolean. Uses labels instead of IDs to identify categories of the transposed dimension, unless --cid has been specified.

```
jsonstat2arrobj oecd.json oecd-transp.json --by area --bylabel
```

#### --prefix (-p)

String. Text to be used as a prefix in the transposed categories. Only valid in combination with --by.

```
jsonstat2arrobj oecd.json oecd-transp.json --by area --prefix "AREA:"
```

#### --drop (-d)

String. Comma-separated dimension IDs to be dropped from the output. Dimensions with more than one category cannot be dropped. Only valid in combination with --by.

```
jsonstat2arrobj canada.json canada-transp.json --by concept --drop country,year
```

## jsonstat2arrow

Converts JSON-stat to the Apache Arrow format.

```
jsonstat2arrow oecd.json oecd.arrow -c -0
```

Take into account that only basic metadata will be kept.

### Options

#### --binary (-0)

Boolean. Saves data in binary format.

When this option is not specified,

```
jsonstat2arrow oecd.json byte-array.json
```

the result is an array of integers representing bytes. That is not probably what you want. To get a file in the Apache Arrow IPC format use **--binary**.

```
jsonstat2arrow oecd.json oecd.arrow -0
```

#### --cid (-c)

Boolean. Identifies categories by ID instead of label.

#### --flabel (-f)

Boolean. Identifies dimensions, value and status by label instead of ID.

## jsonstat2csv

Converts JSON-stat into CSV.

```
jsonstat2csv oecd.json oecd.csv
```

### Options

#### --status (-s)

Boolean. Includes status information.

```
jsonstat2csv oecd.json oecd.csv --status
```

#### --vlabel (-v)

String. Specifies the label of the value field. Default is "Value".

```
jsonstat2csv oecd.json oecd.csv --vlabel val
```

#### --slabel (-l)

String. Specifies the label of the status field. Default is "Status".

```
jsonstat2csv oecd.json oecd.csv --status --slabel stat
```

#### --na (-n)

String. Not available text. Default is "n/a".

```
jsonstat2csv oecd.json oecd.csv --na ":"
```

#### --column (-c)

String. Column separator. Default is ",".

```
jsonstat2csv oecd.json oecd.ssv --column ";"
```

#### --decimal (-d)

String. Decimal separator. Default is ".", unless column separator is ";" (then default is ",").

```
jsonstat2csv oecd.json oecd.tsv --column "\t" --decimal ","
```

#### --by (-y)

String. Dimension ID. When a valid dimension ID is provided, the resulting CSV is transposed by that dimension and the value column is replaced by as many columns as categories in the specified dimension.

When this option is set with a valid ID, **--status**, **--vlabel** and **--slabel** are ignored. When **--rich** is set or an invalid ID is specified, **--by** is ignored.

#### --drop (-o)

String. Comma-separated list of dimension IDs. When a valid dimension ID is provided in **--by** parameter, ***drop*** can provide a list of constant dimension IDs to remove from the resulting CSV.

```
jsonstat2csv eurostat.json eurostat.csv --by unit --drop freq,age
```

#### --rich (-r)

Boolean. Output is a rich CSV in the [JSON-stat Comma-Separated Values format](https://github.com/jsonstat/csv) (CSV-stat, or JSV for short). CSV-stat is CSV plus a metadata header. CSV-stat supports all the JSON-stat dataset core semantics. This means that CSV-stat can be converted back to JSON-stat (using [csv2jsonstat](#csv2jsonstat)) without loss of information (only the *note*, *link*, *child*, *coordinates* and *extension* properties are not currently supported).

When this option is set, **--status**, **--vlabel**, **--slabel**, **--by** and **--drop** are ignored.

```
jsonstat2csv oecd.json oecd.jsv --rich
```

#### --big (-b)

Boolean. Trying to convert a very big dataset into a CSV can produce several errors. The default Node.js memory can be insufficient (Node.js *max-old-space-size* will need to be increased). But even with enough memory, an *Invalid string length* error could stop the execution of **jsonstat2csv**. To avoid this, use the **--big** option.

This option will be ignored if the stream interface is enabled (**--stream**).

## jsonstat2objarr

Converts JSON-stat into an object of column-oriented arrays.

## Options

#### --status (-s)

Boolean. Includes status information.

#### --vlabel (-v)

String. Specifies the label of the value field. Default is "Value".

#### --slabel (-a)

String. Specifies the label of the status field. Default is "Status".

#### --flabel (-f)

Boolean. Identifies dimensions, value and status by label instead of ID.

#### --cid (-c)

Boolean. Identifies categories by ID instead of label.

#### --unit (-u)

Boolean. Includes unit information in a property called "unit". If the dataset does not include unit information, the value of the "unit" property will be null.

#### --meta (-m)

Boolean. Returns a metadata-enriched output (object of objects, instead of array of objects).

#### --comma (-k)

Boolean. Represents values as strings instead of numbers with comma as the decimal mark.

#### --by (-b)

String. Transposes data by the specified dimension. String must be an existing dimension ID. Otherwise it will be ignored.

#### --bylabel (-l)

Boolean. Uses labels instead of IDs to identify categories of the transposed dimension, unless --cid has been specified.

#### --prefix (-p)

String. Text to be used as a prefix in the transposed categories. Only valid in combination with --by.

#### --drop (-d)

String. Comma-separated dimension IDs to be dropped from the output. Dimensions with more than one category cannot be dropped. Only valid in combination with --by.


## jsonstat2object

Converts JSON-stat into an object of arrays in the [Google DataTable format](https://developers.google.com/chart/interactive/docs/reference#dataparam).

```
jsonstat2object oecd.json oecd-object.json
```

### Options

#### --status (-s)

Boolean. Includes status information.

```
jsonstat2object oecd.json oecd-object.json --status
```

#### --vlabel (-v)

String. Specifies the label of the value field. Default is "Value".

```
jsonstat2object oecd.json oecd-object.json --vlabel val
```

#### --slabel (-l)

String. Specifies the label of the status field. Default is "Status".

```
jsonstat2object oecd.json oecd-object.json --status --slabel stat
```

#### --fid (-f)

Boolean. Identifies dimensions, value and status by ID instead of label.

```
jsonstat2object oecd.json oecd-object.json --fid
```

#### --cid (-c)

Boolean. Identifies categories by ID instead of label.

```
jsonstat2object oecd.json oecd-object.json --cid
```

## jsonstatdice

Creates JSON-stat from JSON-stat. It can be used to convert old JSON-stat to JSON-stat version 2.0. Because it supports a filter parameter, it can also be used to create a JSON-stat subset from a JSON-stat dataset. A JSON-stat subset has the same dimensions as the original dataset but with only a selection of the original dimension categories.

```
jsonstatdice oecd.json oecd-subset.json -f area=BE/DE,year=2013/2014
```

In the previous example, oecd-subset.json only contains data for Belgium and Germany and years 2013 and 2014.

### Options

#### --filter (-f)

String. Specifies a filter. When no filter is specified, the original JSON-stat will be returned unless it was not JSON-stat 2.0: in such case, it will be updated to version 2.0.

```
jsonstatdice jsonstat10.json jsonstat20.json
```

A filter is a comma-separated list of selection criteria. Each criterion must follow the pattern *{dimension id}={category ids}* where *{category ids}* is a list of category ids separated by "/". Because dimension ids and category ids are strings that can contain whitespaces, they should be double-quoted. For example,

```
"area"="BE"/"DE","year"="2013"/"2014"
```

forces the subset to keep only categories "BE" and "DE" from dimension "area" and categories "2013" and "2014" from dimension "year". Because these ids do not contain whitespaces, double quotes are not strictly necessary.

```
jsonstatdice oecd.json oecd-subset.json -f area=BE/DE,year=2013/2014
```

#### --drop (-d)

Filters are considered positive statements: they state what should be kept in the dataset. Use **--drop** if the specified filter must be read negatively.

For example, to keep all areas but Belgium and Germany and all years but 2013 and 2014, type:

```
jsonstatdice oecd.json oecd-subset.json -f area=BE/DE,year=2013/2014 -d
```

#### --ovalue (-o)

Boolean. By default, the returned dataset includes an array property for "value". Use **--ovalue** to get a "value" object instead of an array.

#### --ostatus (-s)

Boolean. By default, the returned dataset includes an array property for "status". Use **--ovalue** to get a "status" object instead of an array. This is recommended because it usually means a smaller file (usually status is only set for some values).

## jsonstatslice

Deprecated: use jsonstatdice instead.

Creates JSON-stat from JSON-stat. It can be used to convert old JSON-stat to JSON-stat version 2.0. Because it supports a filter parameter, it can also be used to create a JSON-stat subset from a JSON-stat dataset. A JSON-stat subset produced by jsonstatslice has the same dimensions as the original dataset but with some of them fixed for a certain category.

```
jsonstatslice oecd.json oecd-subset.json -f area=DE,year=2014
```

In the previous example, oecd-subset.json only contains data for Germany in 2014.

### Options

#### --filter (-f)

String. Specifies a filter. When no filter is specified, the original JSON-stat will be returned unless it was not JSON-stat 2.0: in such case, it will be updated to version 2.0.

```
jsonstatslice jsonstat10.json jsonstat20.json
```

A filter is a comma-separated list of selection criteria. Each criterion must follow the pattern *{dimension id}={category id}*. Because dimension ids and category ids are strings that can contain whitespaces, they should be double-quoted. For example,

```
"area"="DE","year"="2014"
```

forces the subset to keep only category "DE" from dimension "area" and category "2014" from dimension "year". Because these ids do not contain whitespaces, double quotes are not strictly necessary.

```
jsonstatslice oecd.json oecd-subset.json -f area=DE,year=2014
```

#### --modify (-m)

Boolean. When this option is not set, the *value* and *status* types of the input dataset are kept. When this option is set, the type of these two properties is transformed taking into account **--ovalue** and **--ostatus**.

#### --ovalue (-o)

Boolean. When **--modify** is set, it includes a "value" property of type object instead of array. When **--modify** is set and **--ovalue** is not, it includes a "value" property of type array.

#### --ostatus (-s)

Boolean. When status information is available and **--modify** is set, includes a "status" property of type object instead of array. When **--modify** is set and **--ostatus** is not, it includes a "value" property of type array.

## sdmx2jsonstat

Creates JSON-stat from SDMX-JSON. The input must be in the SDMX-JSON format. Inputs with more than one dataset are not supported. Intermediate grouping ("time series", "cross-section") is not supported, either. Only SDMX-JSON with a flat list of observations (*dimensionAtObservation=allDimensions*) is supported. This is sometimes referred to as the **SDMX-JSON flat format flavor**.

```
sdmx2jsonstat sdmx.json stat.json
```

### Options

#### --ovalue (-o)

Boolean. Includes a "value" property of type object instead of array.

#### --ostatus (-s)

Boolean. When status information is available, includes a "status" property of type object instead of array.
