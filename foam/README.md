# INTRODUCTION

Foam will be a Python / MongoDB based database / backtester for
financial data / trading algorithms. It will have a simple mechanism
to download data from the open sources, and be able to update itself
incrementally. We will share the latest dump of the database through a
public link here (see below), so that anyone is able to download the
data and start testing algorithms right away.

## Requirements

`conda install` or `pip install`

* pandas
* scipy
* numpy
* pymongo 3.x 
* quandl

## Data

Symbols are retrieved from seperate csv files under `data`
folder. [Details](data/README.md).

* fred.csv: (Selected) Macroeconomic data series from St. Louis FED
  ([FRED][2]) database (whole list is in fred.zip)  
* amex.csv: Companies listed in the AMEX exchange
* nyse.csv: Companies listed in the NYSE exchange
* etfs.csv: All ETFs (exchange traded funds)
* futures.csv: Commodity Futures (from Quandl)
* hft.dat: High-frequency data in 5-minute bars for selected symbols.

Composite unique Id for ticker is comprised of the symbol `sym` and
the date `dt`.

Futures data is retrieved from Quandl; we use their API access, for
which you need to create a `.quandl` file with the API access key in
it (no newlines) under foam's main directory.

## Usage

Simplest usage for mass download is `python foam.py`. This will read
all symbols from under `data` and start downloading them.

For parallel execution, we provided a chunking ability,

```
python foam.py 0 4
```

This divides the symbol list into 4 chunks, and processes the 0th one
(it could have been 1,2,etc). For parallel execution 4 processes of
`foam.py` would be started, each for / using a different chunk number.
These processes would ideally be run under a monitoring tool, we
prefer [dand][1]. A sample configuration for this tool can be found in
[dand.conf](dand.conf) which can simply be executed with `python
dand.py dand.conf`.

Parallel inserts into Mongo are no problem, both in terms of
scalability and collisions; key value stores are particularly good at
parallel inserts, and since we are dividing the symbol list
before handing it over to a processor, no two processes can insert or
update on the same unique id. 

For research, data exploration purposes, there is a utility
function. To see all data for symbol DIJA,

```
import foam
df = foam.get("DJIA")
```

For multiple symbols in one Dataframe,

```
df = foam.get_multi(['DJIA','GOOG'])
```

This returns a Pandas dataframe which can be processes, plotted.

To see all earnings announcements for a particular date, use

```
db.earnings.find( {"_id": 20110126 } )
```

which gives all announcements as a list of tuples for January 26, 2011. 

For some symbols we retrieve high-frequency data. Minute level tick
data for a symbol and specific day can be accessed with,

```
df = foam.get_hft("AHP", 20160213)
```

## Indexing

In the case of composite Ids, indexes might not be created properly in
MongoDB. In this case, simply create them with

```
db.tickers.create_index("_id")
```

To check indexing is working properly

```
print db.tickers.find( {"_id.sym": "DDD", "_id.dt": 20070101 } ).limit(1).explain()
```

This should say something about BTrees, and indicate the table is not
fully scanned. 

## Database Dump

The data dump is from Mongo DB; simply unzip the file below and use
`mongorestore` to load the files into your local instance. The
database has ticker data for all stocks in major exchanges, all ETFs,
all earnings announcement dates per company, after 2016-01-01, for 10+
years.

https://dl.dropboxusercontent.com/u/1570604/data/dump/foam/system.indexes.bson

https://dl.dropboxusercontent.com/u/1570604/data/dump/foam/tickers.metadata.json

https://dl.dropboxusercontent.com/u/1570604/data/dump/foam/earnings.metadata.json

https://dl.dropboxusercontent.com/u/1570604/data/dump/foam/tickers-aa

https://dl.dropboxusercontent.com/u/1570604/data/dump/foam/tickers-ab

https://dl.dropboxusercontent.com/u/1570604/data/dump/foam/tickers-ac

https://dl.dropboxusercontent.com/u/1570604/data/dump/foam/tickers-ad

https://dl.dropboxusercontent.com/u/1570604/data/dump/foam/tickers-ae

https://dl.dropboxusercontent.com/u/1570604/data/dump/foam/tickers-af

https://dl.dropboxusercontent.com/u/1570604/data/dump/foam/earnings.bson.gz

The `tickers-` files are split pieces of `tickers.bson`, the main
database file. In order to construct it from the pieces, download the
files then on Unix simply run `cat tickers-* > tickers.bson.gz`, then
run gunzip on the result which should give `tickers.bson` file. Now
`mongorestore` will load all the files.

[1]: https://github.com/burakbayramli/dand

[2]: https://www.stlouisfed.org
