# JSON-stat Command Line Conversion Tools

The **JSON-stat Command Line Conversion Tools** contain command line tools for converting to and from [JSON-stat](https://json-stat.org/). They support different JSON flavors, CSV (including [CSV-stat](https://github.com/jsonstat/csv)) and [SDMX-JSON](https://github.com/sdmx-twg/sdmx-json/blob/master/data-message/docs/1-sdmx-json-field-guide.md). They are built upon the [JSON-stat Javascript Toolkit](https://www.npmjs.com/package/jsonstat-toolkit) and the [JSON-stat Javascript Utilities Suite](https://www.npmjs.com/package/jsonstat-suite).

## Installation

```
npm install -g jsonstat-conv
```

More in the [Installation](https://github.com/jsonstat/conv/blob/master/docs/INSTALL.md) page.

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

Check the [API Reference](https://github.com/jsonstat/conv/blob/master/docs/API.md) page for more.

## Example

Get unemployment rate time series by country from Eurostat and convert it to CSV.

```
curl "https://ec.europa.eu/eurostat/api/dissemination/statistics/1.0/data/tesem120" -o unr.jsonstat

jsonstat2csv unr.jsonstat unr.csv
```

Or using the stream interface:

```
curl "https://ec.europa.eu/eurostat/api/dissemination/statistics/1.0/data/tesem120" | jsonstat2csv > unr.csv -t
```

More in the [Examples](https://github.com/jsonstat/conv/blob/master/docs/EXAMPLES.md) page.
