# Luno Streams

[![Build Status](https://travis-ci.org/jacoduplessis/luno_streams.svg?branch=master)](https://travis-ci.org/jacoduplessis/luno_streams)


Python library to connect to the [Luno Streaming API](https://www.luno.com/en/api#streaming) with websockets. 

Includes example app to replicate the Luno Exchange interface as well 
as proxy server for easy consumption of practical market data.

Requires Python 3.6+.

## Install

```
pip install luno_streams
```

## Example usage

```python

import asyncio
from luno_streams import Updater

def print_it(consolidated_order_book, trades):
    print(consolidated_order_book, trades)

updater = Updater(
    pair_code='XBTZAR',
    api_key='123',
    api_secret='456',
    hooks=[print_it],
)

loop = asyncio.get_event_loop()
loop.run_until_complete(updater.run())
```

See `cli.py` in the source code for an example of running multiple 
websocket connections in parallel, as well as how hooks are used 
to store results and proxy to other websockets.

## Hooks

The `Updater` accepts a `hooks` parameter - a list of functions or coroutines that will be called
whenever the order book is updated. This is where you will probably add some code to store
the data in redis or do some calculations and make some API calls. 

Each hook will receive two arguments:

1. a consolidated order book, which groups all orders by price. See [Order Book Structure](#order-book-structure) below.
2. a list of trades that were performed during the last update. See [Trade Structure](#trade-structure) below.

Scan the list of trades if you need to determine whether your order was fulfilled without making 
an API call. Pro tip: if you have an open order, you are a `maker`.

## Order Book Structure

An order book is a dict with two keys: `bids` and `asks`. Each side contains a list
of entries, where each entry is a list of the form `[price, volume]`.

Entries are ordered — `bids` are ordered with the highest price on top, and `asks` are ordered with the lowest
price on `top`.

Example:

```json
{
    "bids": [
      ["500", "0.5"],
      ["480", "0.7"]
    ],
    "asks": [
      ["520", "0.4"],
      ["540", "0.6"]
    ]
}
```

## Trade Structure

A trade is a dict with the following keys: 

* `type` - `buy` or `sell`
* `price`
* `base` - volume
* `taker_order_id`
* `maker_order_id`

## Running the socket server

```
luno_streams api_key api_secret pairs [pairs ...]
```

Extras:

* `--app` - Serve a single page javascript app to render live order books
* `--depth` - specify an integer `n` to trim the order book to at most `n` orders on each side

Run `luno_streams --help` to view all options.

It is highly recommended to use the `--depth` option, usually somewhere between 100 - 200 is a good choice.

## Example app

An example app made with [Vue](https://vuejs.org) is included. See `app.html` in the source code
and run it using the `--app` flag when running the server. This will serve the `app.html` in a 
new thread.
