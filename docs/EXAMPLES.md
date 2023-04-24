# Examples
> [JSON-stat Command Line Conversion Tools](https://github.com/jsonstat/conv/blob/master/README.md) ▸ **Examples**


* [A Eurostat Example](#a-eurostat-example)
* [A UNECE Example](#a-unece-example)
* [A Norwegian Example](#a-norwegian-example)
* [An Irish Example](#an-irish-example)
* [A Danish Example](#a-danish-example)
* [An OECD Example](#an-oecd-example)

Please, read the examples in order: **their requirements are incremental**.

Warning: Data in pictures and sample source code may be different from current data.

## A Eurostat Example

Let's assume that we must build a spreadsheet table of unemployment rate by country (columns) and year (rows).

![Sample spreadsheet](https://raw.githubusercontent.com/wiki/badosa/JSON-stat-conv/eurostat.png)

### Steps

#### 1. Retrieve the unemployment rate by country in recent years from Eurostat

You'll need to find the Eurostat dataset identifier. Go to

https://jsonstat.com/eurostat/

and then

```
Tables on EU policy
  > Employment and social policy indicators
    > Employment performance monitor - indicators
      > Unemployment rate by sex [tesem120]
```

Use the "Load Data" button to retrieve the data. You can view the contents of the **tesem120** dataset using the "List" or "Table" buttons. 

Use the "API query" button to get the API URL. The result is:

https://ec.europa.eu/eurostat/api/dissemination/statistics/1.0/data/tesem120?lang=en

To download the dataset from the command line using Eurostat's API, run [cURL](https://curl.haxx.se/dlwiz/?type=bin):

```
curl "https://ec.europa.eu/eurostat/api/dissemination/statistics/1.0/data/tesem120?lang=en" -o eurostat.jsonstat
```

JSON-stat keeps data and metadata completely apart because it is designed to be an efficient transport format. That probably makes it human-unfriendly. Fortunately, we can use jsonstat-conv to convert JSON-stat to other formats.

#### 2. Convert JSON-stat to a more popular JSON data structure

If you don't have jsonstat-conv installed in your system or have a version that is lower than 0.6, please (re)install it:

```
npm install -g jsonstat-conv
```

To check your version of jsonstat-conv:

```
jsonstat2csv --version
```

jsonstat-conv includes a command to translate JSON-stat directly into CSV (**jsonstat2csv**). For example,

```
jsonstat2csv eurostat.jsonstat eurostat.csv
```

will produce a comma-delimited CSV using dot as the decimal mark (_eurostat.csv_).

If you need a semicolon-delimited CSV with comma as the decimal mark, use instead:

```
jsonstat2csv eurostat.jsonstat eurostat-semi.csv --column ";"
```

**jsonstat2csv** comes with several options that allow us to customize the output. For example, to include status information and rename the "Status" and "Value" columns to "Symbol" and "Rate":

```
jsonstat2csv eurostat.jsonstat eurostat-status.csv --status --vlabel Rate --slabel Symbol
```

**jsonstat2csv** also allows you to create a rich CSV document with extra metadata header lines ([CSV-stat](https://github.com/jsonstat/csv)).

```
jsonstat2csv eurostat.jsonstat eurostat.jsv --rich
```

(CSV-stat can be converted back into JSON-stat with **csv2jsonstat**:

```
csv2jsonstat eurostat.jsv eurostat-conv.jsonstat
```

CSV-stat supports all JSON-stat dataset features except _note_, _link_, _child_, _coordinates_ and _extension_. In the previous example, eurostat.jsonstat contained extension information: eurostat-conv.jsonstat is equivalent to eurostat.jsonstat, only the _extension_ property is missing.)

You can also use **jsonstat2arrobj** to convert JSON-stat into an array of objects (_arrobj_, from now on), which is probably the most popular structure used to exchange data in JSON. There are many tools to transform _arrobjs_ into other JSON flavors or even to convert them to CSVs.

So instead of converting Eurostat's JSON-stat directly to CSV using **jsonstat2csv** we are going to translate it to an _arrobj_ using **jsonstat2arrobj** and then convert it to CSV using open source tools.

To convert JSON-stat into an array of objects:

```
jsonstat2arrobj eurostat.jsonstat eurostat.json
```

#### 3. Customize the array of objects

As you probably noticed, the _eurostat.json_ does not have the structure we need in our spreadsheet (country in columns and year in rows). By default, **jsonstat2arrobj** stores the value for each combination of categories in a property called _value_:

```json
[
  {
    "time": "2009",
    "geo": "Austria",
    "sex": "Total",
    "unit": "Percentage of population in the labour force",
    "age": "From 15 to 74 years",
    "freq": "Annual",
    "value": 5.7
  },
  ...
]
```

Instead, we need to have a property for each category of the _geo_ dimension (where the country information is) to store their values for each combination of the categories of the rest of the dimensions:

```json
[
  {
    "unit": "Percentage of population in the labour force",
    "sex": "Total",
    "age": "From 15 to 74 years",
    "time": "2009",
    "AT": 5.7,
    "BE": 8,
    "BG": 7.9,
    "CY": 5.4,
    ...
  },
  ...
]
```

That is, we need to transpose the values by _geo_:

```
jsonstat2arrobj eurostat.jsonstat eurostat-transp.json --by geo
```

Because we are only interested in unemployment as a percentage of active population (PC_ACT) and we don't care about _sex_ or _age_, we need to create a subset of eurostat.jsonstat:

```
jsonstatdice eurostat.jsonstat eurostat-subset.jsonstat --filter sex=T,age=Y15-74,unit=PC_ACT
```

Now that we are sure that _sex_, _age_ and _unit_ are single-category dimensions, we can remove them from the transposed JSON. In the original Eurostat dataset, _freq_ was also a single-category dimension (always "annual"). We can drop all of them:

```
jsonstat2arrobj eurostat-subset.jsonstat eurostat-drop.json --by geo --drop sex,age,unit,freq
```

#### 4. Convert JSON to CSV

For this task, there are many tools out there. We will be using [d3-dsv](https://npmjs.com/package/d3-dsv), which includes a **json2csv** command.

```
npm install -g d3-dsv
```

To convert our last JSON to CSV:

```
json2csv < eurostat-drop.json > eurostat.csv
```

The resulting CSV has comma as the column delimiter and dot as the decimal mark. If what you need is a CSV with comma as the decimal mark, first you must use the **jsonstat2arrobj** _--comma_ (decimal mark) option:

```
jsonstat2arrobj eurostat-subset.jsonstat eurostat-comma.json --by geo --drop sex,age,unit,freq --comma
```

And then specify in **json2csv** a different column delimiter (for example, a semicolon):

```
json2csv < eurostat-comma.json > eurostat-semi.csv -w ";"
```

#### 5. Without external packages

In fact, steps 3 and 4 are not necessary because you can produce the desired CSV directly from JSON-stat. We can use _eurostat-subset.jsonstat_ as input (which was produced with **jsonstatdice**) or we can retrieve the subset directly from Eurostat.

Let's go back to step 1. Visit:

https://jsonstat.com/eurostat/

and then

```
Tables on EU policy
  > Employment and social policy indicators
    > Employment performance monitor - indicators
      > Unemployment rate by sex [tesem120]
```

and make sure only one category is selected in each dimension but "Time" and "Geopolitical entity (reporting)". For this goal, you will need to select "Total" in the "Sex" dimension. Then press the "Load Data" button and the "API query" button. You will get this URL:

https://ec.europa.eu/eurostat/api/dissemination/statistics/1.0/data/tesem120?lang=en&sex=T

Use cURL to download the new dataset:

```
curl "https://ec.europa.eu/eurostat/api/dissemination/statistics/1.0/data/tesem120?lang=en&sex=T" -o eurostat-subset.jsonstat
```

And convert to CSV transposing by the _geo_ dimension, dropping _sex_, _age_, _unit_ and _freq_, requesting IDs instead of dimension labels (--fid):

```
jsonstat2csv eurostat-subset.jsonstat eurostat-subset.csv --by geo --drop sex,age,unit,freq --fid 
```

#### 6. Altogether now

All the process has required three lines and three files (_eurostat.jsonstat_, _eurostat-drop.json_, _eurostat.csv_):

```
curl "https://ec.europa.eu/eurostat/api/dissemination/statistics/1.0/data/tesem120?lang=en&sex=T" -o eurostat-subset.jsonstat

jsonstat2csv eurostat-subset.jsonstat eurostat-subset.csv --by geo --drop sex,age,unit,freq --fid 
```

This is not necessary, though: all the process can be done in a single line, piping the output of a program to another program. For that, we need to enable **jsonstat2arrobj** stream interface (_--stream_) which will allow us to use pipes (|) and redirects (<, >).

In the stream interface, this command

```
jsonstat2csv eurostat-subset.jsonstat eurostat-subset.csv --by geo --drop sex,age,unit,freq --fid
```

must be rewritten as

```
jsonstat2csv < eurostat-subset.jsonstat > eurostat-subset.csv --by geo --drop sex,age,unit,freq --fid --stream
```

So to get a comma-delimited CSV with dot as the decimal mark in a single line:

```
curl "https://ec.europa.eu/eurostat/api/dissemination/statistics/1.0/data/tesem120?lang=en&sex=T" | jsonstat2csv --by geo --drop sex,age,unit,freq --fid --stream > eurostat-subset.csv
```

And to get a semicolon-delimited CSV with comma as the decimal mark:

```
curl "https://ec.europa.eu/eurostat/api/dissemination/statistics/1.0/data/tesem120?lang=en&sex=T" | jsonstat2csv --by geo --drop sex,age,unit,freq --fid --column ";" --decimal "," --stream > eurostat-subset-semi.csv
```

## A UNECE Example

Let's assume that we must build a spreadsheet table of unemployment rate by country (columns) and year (rows), but this time based on UNECE's data.

![Sample spreadsheet](https://raw.githubusercontent.com/wiki/badosa/JSON-stat-conv/unece.png)

The requirements are the same as in [Eurostat's example](#a-eurostat-example), so please read it first if you haven't.

### Steps

#### 1. Retrieve the unemployment rate by country in recent years from UNECE

First, we need to find the UNECE table with such information. Visit the UNECE Statistical Database

http://w3.unece.org/PXWeb2015/pxweb/en/STAT/

and then

```
UNECE Statistical Database
  > Country Profiles
    > UNECE Member Countries in Figures
      > Country Overview by Country and Time
```

On table [Country Overview by Country and Time](https://w3.unece.org/PXWeb2015/pxweb/en/STAT/STAT__10-CountryOverviews__01-Figures/ZZZ_en_CoSummary_r.px/) select *Unemployment rate* as Indicator, all countries for the Country dimension and all years for the Year dimension. Then click **Show table** and expand **Save your query**. In the **Save the result as** field select *JSON-stat 2 file (JSON)*. You will get a URL:

```
https://w3.unece.org/PXWeb2015/sq/6c97a19a-d916-444e-93d9-1c2e951351d8
```

To download the dataset from the command line, let's run [cURL](https://curl.haxx.se/dlwiz/?type=bin):

```
curl https://w3.unece.org/PXWeb2015/sq/6c97a19a-d916-444e-93d9-1c2e951351d8 -o unece.jsonstat
```

#### 2. Convert JSON-stat to CSV

This part is similar to the Eurostat example: we need to transpose by Country and drop Indicator (it's always *Unemployment rate* because that's the only category we selected in the JSON query):

```
jsonstat2csv unece.jsonstat unece-transp.csv --by Country --drop Indicator
```

If we want to use semicolon as the column delimiter and comma as the decimal mark:

```
jsonstat2csv unece.jsonstat unece-semi.csv --by Country --drop Indicator --column ";"
```

#### 3. Altogether now

Comma-delimited CSV with dot as the decimal mark:

```
curl https://w3.unece.org/PXWeb2015/sq/6c97a19a-d916-444e-93d9-1c2e951351d8 | jsonstat2csv -y Country -p Indicator -t > unece.csv
```

Semicolon-delimited CSV with comma as the decimal mark:

```
curl https://w3.unece.org/PXWeb2015/sq/6c97a19a-d916-444e-93d9-1c2e951351d8 | jsonstat2csv -y Country -p Indicator -c ";" -t > unece-semi.csv
```

## A Norwegian Example

Like UNECE's, [Statistics Norway's API](https://www.ssb.no/en/omssb/tjenester-og-verktoy/api/px-api) is rich and powerful and allows a high degree of dataset customization. Flexibility comes, of course, at a price: (JSON query's) complexity. Statistics Norway also offers [ready-made datasets](https://data.ssb.no/api/v0/dataset/?lang=en). Statistics Norway has predefined a collection of datasets, based on its most popular tables. The trade-off: these datasets cannot be customized.

Take, for example, [dataset 1082](https://data.ssb.no/api/v0/dataset/1082?lang=en): it contains the population, by sex and one-year age groups and is perfect to build a population pyramid. But, what if instead we are interested in the total population by age group? Dataset 1082 does contain the male and female population but not the total population. Could we compute it on the fly from the terminal? In a single line? Yes, we can.

Let's assume that we must analyze the differences in the participation rate between men and women in the Norwegian labor market. We want to use as indicator the female participation rate divided by the male participation rate.

![Sample spreadsheet](https://raw.githubusercontent.com/wiki/badosa/JSON-stat-conv/norway2018.png)

### Steps

#### 1. Retrieve the participation rate by sex in recent years from Statistics Norway

[Dataset 1052](https://data.ssb.no/api/v0/dataset/1054?lang=en) from Statistics Norway provides a monthly time series of the main labor market concepts by sex and age (more than 18,000 values). Dataset 1082 in the JSON-stat format is available at:

https://data.ssb.no/api/v0/dataset/1054.json?lang=en

You can view the contents of the dataset at

https://jsonstat.com/explorer/#/https%3A%2F%2Fdata.ssb.no%2Fapi%2Fv0%2Fdataset%2F1054.json%3Flang%3Den

To download the dataset from the command line, run [cURL](https://curl.haxx.se/dlwiz/?type=bin):

```
curl "https://data.ssb.no/api/v0/dataset/1054.json?lang=en" -o no.jsonstat
```

#### 2. Filter data

The dataset is big because it contains many indicators available by sex and age (besides time). Because we are going to calculate our ratio using the "Labour force in per cent of the population" (*ArbStyrkProsBefolkn*), we don't really need any other categories of the *ContentsCode* dimension. And because we are not going to compute results by age (*Alder*), we just need to keep the *15-74* category. Finally, we are going to use "Seasonally adjusted data" (*S*) in the *Justering* dimension ("type of adjustment"). Creating a JSON-stat subset is easy with **jsonstatdice**.

```
jsonstatdice no.jsonstat no-subset.jsonstat --filter ContentsCode=ArbStyrkProsBefolkn,Alder=15-74,Justering=S
```

#### 3. Convert JSON-stat to a more popular JSON data structure

Like in previous examples, we will use **jsonstat2arrobj** to convert JSON-stat to an array of objects. We will need to transpose by sex (*Kjonn*) to be able to compute the ratio.

```
jsonstat2arrobj no-subset.jsonstat no.json --by Kjonn
```

In the output, dimensions are named according to their identifier (age is *Alder*, concept is *ContentsCode*, time is *Tid*). And by default, categories of transposed dimensions are also named according to their identifier (males, females and both sexes are "1", "2" and "0").

```json
[
  {
    "0": 71.4,
    "1": 75,
    "2": 67.7,
    "Alder": "15-74 years",
    "ContentsCode": "Labour force in per cent of the population",
    "Justering": "Seasonally adjusted",
    "Tid": "2006M02"
  },
  ...
]
```

But by default categories are named according to their label. The _--cid_ option allows us to change the default behavior.

 ```
 jsonstat2arrobj no-subset.jsonstat no-id.json --by Kjonn --cid
 ```

 ```json
 [
   {
    "0": 71.4,
    "1": 75,
    "2": 67.7,
    "Alder": "15-74",
    "ContentsCode": "ArbStyrkProsBefolkn",
    "Justering": "S",
    "Tid": "2006M02"
   },
   ...
 ]
 ```

The output is shorter and the rules that must be applied later will be simpler.

#### 4. Filter missing values

To filter missing values we need to be able to process the JSON output, element by element in the array. One way to achieve this is translating JSON into Newline Delimited JSON ([NDJSON](http://ndjson.org/)). For this task we will be using [ndjson-cli](https://npmjs.com/package/ndjson-cli).

```
npm install -g ndjson-cli
```

To convert JSON to NDJSON:

```
ndjson-split < no-id.json > no.ndjson
```

Now, instead of a single line with an array of objects, we have as many lines as objects.

```json
{"0":71.4,"1":75,"2":67.7,"Alder":"15-74","ContentsCode":"ArbStyrkProsBefolkn","Tid":"2006M02", "Justering":"S"}
{"0":71.4,"1":74.9,"2":67.7,"Alder":"15-74","ContentsCode":"ArbStyrkProsBefolkn","Tid":"2006M03", "Justering":"S"}
{"0":71.4,"1":74.9,"2":67.8,"Alder":"15-74","ContentsCode":"ArbStyrkProsBefolkn","Tid":"2006M04", "Justering":"S"}
...
```

ndjson-cli provides a command to filter lines: **ndjson-filter**. We only need to provide the JavaScript filter condition that must be applied to each line. In our example, each line is an object that can be accessed in **ndjson-filter** as *d*.

Because we don't want to keep lines that have missing values, we will filter:

```js
d['0']!==null
```

(The *0* property has the value for both sexes.)

```
ndjson-filter "d['0']!==null" < no.ndjson > no-filtered.ndjson
```

#### 5. Transform data

Now we need to compute the female (*d['2']*) / male (*d['1']*) ratio. We want to transform each line into a JSON that looks like this:

```json
{
  "time": "2006M02",
  "ratio": 0.8996036988110964
}
```

**ndjson-map** is the command for the job:

```
ndjson-map "{time: d.Tid, ratio: d['2']/d['1']}" < no-filtered.ndjson > no-ratio.ndjson
```

But this is still NDJSON: to go back to JSON (an array of objects), use **ndjson-reduce**:

```
ndjson-reduce < no-ratio.ndjson > no-ratio.json
```

(In the [Irish Example](#an-irish-example), we will see that **ndjson-reduce** is not actually required.)

#### 6. Convert JSON to CSV

Like in the previous examples, for this task we will use [d3-dsv](https://npmjs.com/package/d3-dsv), which includes a **json2csv** command:

```
json2csv < no-ratio.json > no.csv
```

```
time,ratio
2009M01,0.9332460732984292
2009M02,0.9357798165137616
2009M03,0.938722294654498
2009M04,0.9384010484927916
2009M05,0.9407894736842105
2009M06,0.9446640316205533
2009M07,0.9455511288180611
2009M08,0.9454787234042552
2009M09,0.9476510067114093
2009M10,0.946524064171123
...
```

#### 7. Altogether now

In a single line:

```
curl "https://data.ssb.no/api/v0/dataset/1054.json?lang=en" | jsonstatdice -f ContentsCode=ArbStyrkProsBefolkn,Alder=15-74,Justering=S -t | jsonstat2arrobj -b Kjonn -t | ndjson-split | ndjson-filter "d['0']!==null" | ndjson-map "{time: d.Tid, ratio: d['2']/d['1']}" | ndjson-reduce | json2csv > no.csv
```

#### 8. Data visualization

What if we wanted to build a line chart like this one

https://visual.js.org/g/?id=783e8ac99d7f0a67c5a9e0c7ee0782d9

?

This was built using the [Visual Maker](https://visual.js.org), a tool based on [Visual](https://github.com/idescat/visual).

To draw a time series in a line chart, Visual expects that we provide two separate arrays: a **data array**

```
[
  2009M01,0.9332460732984292,
  2009M02,0.9357798165137616,
  0.938722294654498,
  ...
]
```

and a **time array**

```
[
  2009M01,
  2009M02,
  2009M03,
  ...
]
```

(Notice that the time format is not the same we have.)

Producing these two arrays from previous **no-ratio.ndjson** is very simple:

```
ndjson-map "d.time.slice(0,4)+d.time.slice(5,7)" < no-ratio.ndjson | ndjson-reduce > time.json
```

```
ndjson-map "d.ratio" < no-ratio.ndjson | ndjson-reduce > data.json
```

(You can see the actual code of the visualization in [this gist](https://api.github.com/gists/783e8ac99d7f0a67c5a9e0c7ee0782d9): see property *files.source.json.content*.)

#### 8. A final warning

Of course, this example was designed for demo purposes: do not retrieve a ready-made dataset with more than 18,000 values when you only care about less than 200! There is a better way! At least, in Norway: build your own customized dataset from table 08931 selecting *ArbStyrkProsBefolkn* as the only concept and *S* as the type of adjustment (male, female, no age groups).

Like this:

```
curl -X POST -d '{ "query": [ { "code": "Kjonn", "selection": { "filter": "item", "values": [ "1", "2" ] } }, { "code": "ContentsCode", "selection": { "filter": "item", "values": [ "ArbstyrkP1" ] } } ], "response": { "format": "json-stat2" } }' https://data.ssb.no/api/v0/en/table/08931 | jsonstat2arrobj -b Kjonn -c -t | ndjson-split | ndjson-filter "d['1']!==null" | ndjson-map "{time: d.Tid, ratio: d['2']/d['1']}" | ndjson-reduce | json2csv > no.csv
```

On Windows use this version instead:

```
curl -X POST -d "{ \"query\": [ { \"code\": \"Kjonn\", \"selection\": { \"filter\": \"item\", \"values\": [ \"1\", \"2\" ] } }, { \"code\": \"ContentsCode\", \"selection\": { \"filter\": \"item\", \"values\": [ \"ArbstyrkP1\" ] } } ], \"response\": { \"format\": \"json-stat2\" } }" https://data.ssb.no/api/v0/en/table/08931 | jsonstat2arrobj -b Kjonn -c -t | ndjson-split | ndjson-filter "d['1']!==null" | ndjson-map "{time: d.Tid, ratio: d['2']/d['1']}" | ndjson-reduce | json2csv > no.csv
```

## An Irish Example

Let's assume that we must build the population pyramid of Ireland.

![Sample spreadsheet](https://raw.githubusercontent.com/wiki/badosa/JSON-stat-conv/ireland.png)

### Steps

#### 1. Retrieve the population by sex from the Central Statistics Office of Ireland

You'll need to find the JSON-stat dataset URL on CSO's PxStat. Go to

https://data.cso.ie/

and then search for "population estimates" and select dataset PE01.

[Dataset PEA01](https://data.cso.ie/table/PEA01) from CSO provides a yearly time series of population by sex and age group. It is available in the JSON-stat format (expand **API Data Query** and then go to the **RESTful** tab) at:

https://ws.cso.ie/public/api.restful/PxStat.Data.Cube_API.ReadDataset/PEA01/JSON-stat/2.0/en

You can view the contents of the dataset at

https://jsonstat.com/explorer/#/https%3A%2F%2Fws.cso.ie%2Fpublic%2Fapi.restful%2FPxStat.Data.Cube_API.ReadDataset%2FPEA01%2FJSON-stat%2F2.0%2Fen

To download the dataset from the command line, run [cURL](https://curl.haxx.se/dlwiz/?type=bin):

```
curl https://ws.cso.ie/public/api.restful/PxStat.Data.Cube_API.ReadDataset/PEA01/JSON-stat/2.0/en -o ie.jsonstat
```

#### 2. Filter data

Many of the categories in the age dimension (*C02076V02508*) in the dataset are not needed to build a population pyramid:

* _All ages_ (-)
* _15 years and over_ (320)
* _65 years and over_ (575)
* _0 - 4 years_ (205)
* _0 - 14 years_ (215)
* _15 - 24 years_ (310)
* _25 - 44 years_ (420)
* _45 - 64 years_ (505)

We need to drop them. In the sex dimension (*C02199V02655*), the total (*-*) is not required either. We can use **jsonstatdice** to drop categories and create a subset of the original dataset:

```
jsonstatdice ie.jsonstat ie-drop.jsonstat --filter C02076V02508=-/320/575/205/215/310/420/505,C02199V02655=- --drop
```

Or using the stream interface:

```
jsonstatdice < ie.jsonstat > ie-drop.jsonstat --filter C02076V02508=-/320/575/205/215/310/420/505,C02199V02655=- --drop --stream
```

The only difference between the previous two lines is that in the stream interface *ie-drop.jsonstat* will be written even though it already exists while in the non-stream interface a new filename is used to avoid losing the content of an existing file.

Because we are only interested in data for the latest year (2022 at the time of writing), we need to apply a new filter and only keep category *2022* in dimension *TLIST(A1)*:

```
jsonstatdice < ie-drop.jsonstat > ie-filtered.jsonstat --filter "TLIST(A1)"=2022 --stream
```

#### 3. Convert JSON-stat to a more popular JSON data structure

In this step, we will convert the JSON-stat file into an array of objects transposing the sex dimension (*C02199V02655*). The dataset contains a dimension (*STATISTIC*) with a single category (*Population Estimates (Persons in April)*): we won't need it. Using **jsonstat2arrobj** like in previous examples:

```
jsonstat2arrobj < ie-filtered.jsonstat > ie.json --drop STATISTIC --by C02199V02655 --bylabel --stream
```

#### 4. Transform data

Many visualization tools do not have pyramids as a type of chart, because they are actually just a special case of a bar chart where the male values have negative values. This is the case of Google Sheets, the tool we are going to use. So the next step is to multiply male values by -1.

To accomplish this, we need to be able to process the JSON output, element by element in the array. As in the Norwegian example, we will use [ndjson-cli](https://npmjs.com/package/ndjson-cli).

First we need to convert JSON to NDJSON:

```
ndjson-split < ie.json > ie.ndjson
```

Then we need to multiply the male values by -1:

```
ndjson-map "{ Age: d['C02076V02508'], Sex: d['C02199V02655'], Male: -1*d.Male, Female: d.Female }" < ie.ndjson > ie-pyram.ndjson
```

In the Norwegian example, we used **ndjson-reduce** to go back from NDJSON to JSON.

```
ndjson-reduce < ie-pyram.ndjson > ie-pyram.json
```

But this is not strictly necessary.

#### 5.  Convert JSON to CSV

It is not strictly necessary because **json2csv** supports NDJSON as input (_-n_ option):

```
json2csv -n < ie-pyram.ndjson > ie.csv
```

leads to the same result as:

```
json2csv < ie-pyram.json > ie.csv
```

We've ended up with a CSV that looks like this:

```
Age,Male,Female
Under 1 year,-31.3,29.7
1 - 4 years,-122.9,117.8
5 - 9 years,-170.6,162.3
10 - 14 years,-187.2,179.1
15 - 19 years,-169.5,163
20 - 24 years,-161.2,157.6
25 - 29 years,-150.7,148.9
30 - 34 years,-156.2,162.1
35 - 39 years,-175.8,193.5
40 - 44 years,-200,212.9
45 - 49 years,-182.5,188.8
50 - 54 years,-164.9,167.7
55 - 59 years,-148.6,154
60 - 64 years,-133.8,138.6
65 - 69 years,-116.1,119.6
70 - 74 years,-97,101.4
75 - 79 years,-72.4,79.1
80 - 84 years,-42.7,51.7
85 years and over,-34.1,54.8
```

#### 6. Altogether now

In a single line:

```
curl https://ws.cso.ie/public/api.restful/PxStat.Data.Cube_API.ReadDataset/PEA01/JSON-stat/2.0/en | jsonstatdice -f C02076V02508=-/320/575/205/215/310/420/505,C02199V02655=- -d -t | jsonstatdice -f "TLIST(A1)"=2022 -t | jsonstat2arrobj -d STATISTIC -b C02199V02655 -l -t | ndjson-split | ndjson-map "{ Age: d['C02076V02508'], Sex: d['C02199V02655'], Male: -1*d.Male, Female: d.Female }" | json2csv -n > ie.csv
```

#### 7. Data visualization

Finally, we import our CSV into Google Sheets. There, we must highlight all the data (including labels), click on the *Insert chart* icon, select the horizontal bar chart in the *Recommendations* tab and, in the *Customization* tab, check the *Stack* and *Reverse* boxes.

And that's all!

## A Danish Example

According to the [World Happiness Report 2016 Update](http://worldhappiness.report/ed/2016/), people who live in Denmark are the happiest in the world. Let's try to build a simple summary report on Danish quality of life key figures like this one:

![Sample spreadsheet](https://raw.githubusercontent.com/wiki/badosa/JSON-stat-conv/denmark.png)

In this example we will use the tools also shown in the previous ones ([jsonstat-conv](https://npmjs.com/package/jsonstat-conv), [ndjson-cli](https://npmjs.com/package/ndjson-cli)) but we will add a new one to the formula: [ndjson-format](https://www.npmjs.com/package/ndjson-format), a module that allows you to format NDJSON using [ES2015 template literals](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Template_literals).

```
npm install -g ndjson-format
```

### Steps

#### 1. Retrieve the quality of life key figures from Statistics Denmark

Dataset [LIVO1](https://api.statbank.dk/v1/tableinfo/LIVO1?lang=en) available in the [Statistics Denmark API](https://www.dst.dk/en/Statistik/statistikbanken/api) has exactly the information we need. At the time of writing, this dataset contained a time series from 2008 to 2018. To simplify the analysis we will compare 2017 with 2010 (the latest year available has missing values and so do the first years). The customized JSON-stat dataset we will use is:

https://api.statbank.dk/v1/data/LIVO1/JSONSTAT?lang=en&Tid=2010%2C2017&AKTP=*

You can view the contents of the dataset at

https://jsonstat.com/explorer/#/https%3A%2F%2Fapi.statbank.dk%2Fv1%2Fdata%2FLIVO1%2FJSONSTAT%3Flang%3Den%26Tid%3D2010%252C2017%26AKTP%3D*

To download the dataset from the command line, run [cURL](https://curl.haxx.se/dlwiz/?type=bin):

```
curl "https://api.statbank.dk/v1/data/LIVO1/JSONSTAT?lang=en&Tid=2010%2C2017&AKTP=*" -o dk.jsonstat
```

#### 2. Convert JSON-stat to a more popular JSON data structure

Because we want to compute the 2017-2010 change in the different key figures, we will transpose the data by time (*Tid*) and drop the constant dimensions (*ContentsCode*):

```
jsonstat2arrobj < dk.jsonstat > dk.json --by Tid --label --drop ContentsCode --stream
```

Nothing new under the sun so far.

#### 3. Transform data

This is the kind of output we are looking for:

```
↑ Life expectancy (years): 80.6 (+1.4)
↓ General medical treatment, 30-59-year-olds (per inhabitant): 6.5 (-0.3)
...
```

The first thing we need is a symbol indicating if the key figure increased (↑) or decreased (↓) between 2010 and 2017. Using [ES2015 template literals](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Template_literals), if *this* represents an element in the array:

```js
`${this["2017"] > this["2010"] ? "\u2191" : "\u2193"}`
```

Then we need to retrieve the key figure name (*AKTP*) and its value in 2017:

```js
`${this["AKTP"]}: ${this["2017"]}`
```

Finally we want to include, in brackets, the change between the two years rounded off to one decimal place, adding a *+* if the difference is positive:

```js
`(${this["2017"] > this["2010"] ? "+" : ""}${(this["2017"]-this["2010"]).toFixed(1)})`
```

The full template literal is:

```js
`${this["2017"] > this["2010"] ? "\u2191" : "\u2193"} ${this["AKTP"]}: ${this["2017"]} (${this["2017"] > this["2010"] ? "+" : ""}${(this["2017"]-this["2010"]).toFixed(1)})`
```

**ndjson-format** allows us to transform NDJSON using template literals. To be able to use **ndjson-format** first we need to convert *dk.json* into NDJSON with **ndjson-split** (included in [ndjson-cli](https://npmjs.com/package/ndjson-cli)):

```
ndjson-split < dk.json | ndjson-format  > dk.txt '${this["2017"]>this["2010"] ? "\u2191" : "\u2193"} ${this["AKTP"]}: ${this["2017"]} (${this["2017"]>this["2010"] ? "+" : ""}${(this["2017"]-this["2010"]).toFixed(1)})'
```

On **Windows**, quotes are treated differently: the previous line must be rewritten as:

```
ndjson-split < dk.json | ndjson-format  > dk.txt "${this['2017']>this['2010'] ? '\u2191' : '\u2193'} ${this['AKTP']}: ${this['2017']} (${this['2017']>this['2010'] ? '+' : ''}${(this['2017']-this['2010']).toFixed(1)})"
```

#### 4. Altogether now

All the process in a single line:

```
curl "https://api.statbank.dk/v1/data/LIVO1/JSONSTAT?lang=en&Tid=2010%2C2017&AKTP=*" | jsonstat2arrobj -b Tid -l -d ContentsCode -t | ndjson-split | ndjson-format > dk.txt '${this["2017"]>this["2010"] ? "\u2191" : "\u2193"} ${this["AKTP"]}: ${this["2017"]} (${this["2017"]>this["2010"] ? "+" : ""}${(this["2017"]-this["2010"]).toFixed(1)})'
```

On Windows:

```
curl "https://api.statbank.dk/v1/data/LIVO1/JSONSTAT?lang=en&Tid=2010%2C2017&AKTP=*" | jsonstat2arrobj -b Tid -l -d ContentsCode -t | ndjson-split | ndjson-format > dk.txt "${this['2017']>this['2010'] ? '\u2191' : '\u2193'} ${this['AKTP']}: ${this['2017']} (${this['2017']>this['2010'] ? '+' : ''}${(this['2017']-this['2010']).toFixed(1)})"
```

## An OECD Example

The OECD does not support JSON-stat: it serves data in the SDMX-JSON format. But jsonstat-conv can convert SDMX-JSON into JSON-stat and benefit from all the tools available for JSON-stat (at the time of writing, they were still very few tools available supporting SDMX-JSON).

In this example, we will be doing several translations.

### Steps

#### 1. Retrieve some key economic indicators from OECD

```
curl "https://stats.oecd.org/SDMX-JSON/data/KEI/PS+PR+PRINTO01+SL+SLRTTO01+SLRTCR03+OD+ODCNPI03+CI+LO+LOLITOAA+LORSGPRT+LI+LF+LFEMTTTT+LR+LRHUTTTT+LC+LCEAMN01+UL+ULQEUL01+PP+PI+CP+CPALTT01+FI+MA+MABMM301+IR+IRSTCI01+IR3TIB01+IRLTLT01+SP+SPASTT01+CCUSMA02+XT+XTEXVA01+XTIMVA01+BP+B6BLTT02+NA+NAEXKP01+NAEXKP02+NAEXKP03+NAEXKP04+NAEXKP06+NAEXKP07.AUS+AUT+BEL+CAN+CHL+CZE+DNK+EST+FIN+FRA+DEU+GRC+HUN+ISL+IRL+ISR+ITA+JPN+KOR+LVA+LTU+LUX+MEX+NLD+NZL+NOR+POL+PRT+SVK+SVN+ESP+SWE+CHE+TUR+GBR+USA+G-7+OECDE+G-20+OECD+NMEC+ARG+BRA+CHN+COL+IND+IDN+RUS+SAU+ZAF.GP.M/all?startTime=2018-01&endTime=2020-01&dimensionAtObservation=allDimensions" -o kei.sdmx.json
```

This line of code produces an SDMX-JSON file with the growth over the previous period of some key economic indicators for several locations. The size of _kei.sdmx.json_ is (at the time of writing) 430 Kb.

#### 2. Convert SDMX-JSON to JSON-stat

**sdmx2jsonstat** can be used to translate SDMX-JSON into JSON-stat.

```
sdmx2jsonstat kei.sdmx.json default.stat.json
```

The JSON-stat file is smaller (250 Kb) than the original SDMX-JSON one. An even smaller file can be produced: by default, **sdmx2jsonstat** uses arrays to express values and statuses. JSON-stat supports both arrays and objects for this purpose. Because usually only a few data have status information, it is generally better to use an object for statuses.

**sdmx2jsonstat** supports objects for status information using the _--ostatus_ option (_-s_).

```
sdmx2jsonstat kei.sdmx.json kei.stat.json -s
```

The new JSON-stat file is now only 181 Kb: less than half the original SDMX-JSON one.

#### 3. Convert JSON-stat to CSV

Because now we have a regular JSON-stat file, it is trivial to convert it to CSV, a format that can be used to import the data in many tools.

```
jsonstat2csv kei.stat.json kei.csv
```

The new file is very big (1,3 Mb) because by default labels, instead of identifiers, are used. **jsonstat2csv** has several options to avoid this. But you don&rsquo;t actually has to choose between labels or identifiers (each serves a different purpose): you can use the ([CSV-stat](https://github.com/jsonstat/csv)) format as the output format: CSV-stat supports the core semantics of JSON-stat using an enriched CSV structure.

You can produce CSV-stat with the _--rich_ option (_-r_):

```
jsonstat2csv kei.stat.json kei.rich.csv -r
```

This command produces a 547 Kb file.

#### 4. Back to JSON-stat

```
csv2jsonstat kei.rich.csv default.json
```

The size of the new JSON-stat is 241 Kb: it is a little smaller than the original JSON-stat because this one had some extension information that was lost in CSV-stat.

This file can be minimized using objects for statuses, thanks to **jsonstat2jsonstat**:

```
jsonstat2jsonstat default.json kei.json -m -s
```

The size of the resulting file is 181 Kb.

#### 5. Producing a key economic indicators CSV for a particular country

Assuming you have installed all the modules used in the previous examples, it is very easy to get a CSV file for a particular location from _kei.json_.

First, we transpose the data using **jsonstat2arrobj**, then we convert it into [NDJSON](http://ndjson.org/) with **ndjson-split** and filter the selected location with **ndjson-filter** to finally translate it into CSV with **json2csv**.

In the example, we will create a CSV for Canada:

```
"d['LOCATION']==='Canada'"
```

but you can choose any location in the original OECD file.

```
jsonstat2arrobj < kei.json -b SUBJECT -d MEASURE,FREQUENCY -l -t | ndjson-split | ndjson-filter "d['LOCATION']==='Canada'" | json2csv -n > kei-ca.csv
```
