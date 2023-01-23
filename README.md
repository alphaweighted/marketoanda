# marketoanda

The `marketoanda` tool is a market data module for reliable, high performance and easy-to-use access to Oanda market data.

It acts as a locally hosted 'smart proxy' between your trading code and Oanda's APIs - allowing for easier
integration, 
 

## You will need

To use this component, you will need:

1. a Linux-based machine with a working [Docker](https://www.docker.com/) install
2. An account with [Oanda](https://www.oanda.com/)
3. An account ID and API key for your Oanda account (see [Oanda's instructions here](https://developer.oanda.com/rest-live-v20/authentication/))

## Installation/quickstart

Market data modules are provided as Docker images, you can run them in one hit using the command below.  This will start the market data service, and display some basic debug info to stdout.

Assuming your account ID and API key are correct, you'll see the corresponding info in the console and the service will keep running indefinitely.

```shell
docker run \
  --name marketoanda \
  -e MKT_ACCOUNT=<your Oanda Account ID>\
  -e MKT_APIKEY=<your Oanda API key>\
  -p 6011:6011 \
  ghcr.io/alphaweighted/marketoanda:latest
```

Check the service is operating correctly by using `awcli` to interact with it.  For example, the below command will retrieve the current price of the `GBP_USD` instrument.  See the [awcli docs](https://github.com/alphaweighted/awcli) for how to discover instruments, stream prices, get candles, etc.

```shell
> awcli --data localhost:6011 price OANDA:GBP_USD
Symbol: OANDA:GBP_USD
Price: 1.21958
```

Find out more about `awcli` [here](https://github.com/alphaweighted/awcli).

## Running as a service

The command given above can be modified with Docker's `-d` switch to daemonize the process and run in the background.

Alternatively you can run the service using Docker Compose - see [a simple example here](docker-compose.yml).


## Integrating

`marketoanda` exposes a gRPC API for high performance access to market data, including real-time streaming.

In addition, following the UNIX philosophy of small components that do one thing well, `awcli` can be used
as a production integration to stream price data to `stdout`, emit alerts, and/or various other things in
your trading workflow.


## Further configuration

The following environment variables are supported for configuration:

### MKT_ACCOUNT

This should be the numeric account ID of one of your Oanda accounts, it look be similar to `001-004-1234567-001`

### MKT_APIKEY

The API key for your Oanda account, instructions for how to get an API key are provided by Oanda: https://developer.oanda.com/rest-live-v20/authentication/

Note that your can provide the API key directly as the environment variable - or (recommended!) store the key as a secret in your hosting infrastructure and specify the secrets file here, e.g. `MKT_APIKEY=/var/run/secrets/oanda_secretkey.txt` 

### MKT_PAPER

Defaults to 0 for live accounts.  If you connect to a Paper/Demo account, you must set this to 1 - as the URLs Oanda uses for API connections are different for paper/demo accounts.

### MKT_VERBOSITY

By default, only key setup information, warnings and errors are emitted as logs.  Set `MKT_VERBOSITY` to `debug` or `trace` for more verbose debug output if you wish.  `trace` will include output for each request
made to `marketoanda`, which can be helpful in debugging your own trading systems.

## Future enhancements

This service is stateless at present - so no volumes nor other persistence is required.  Future versions may (optionally) support a disk volume for more effective caching of historic market data.

## Security

### Secrets

The environment variable `MKT_APIKEY` may contain your Oanda API key directly, or - in line with Docker's
standard methods for handling secrets - you may set this to the path to a secrets file containing the API key - e.g. `MKT_APIKEY=/var/run/secrets/oanda_secretkey.txt` where the file  `/var/run/secrets/oanda_secretkey.txt` is automatically populated by your hosting infrastructure. 


### Networks security

Outbound connections are made only to Oanda's API services, nowhere else.  If you prefer, you can safely blocks outbound connections to anywhere else - either via Docker's networking configuration, or via a separate firewall.

It listens on only one port (6011 by default), with a single HTTP/2 server to expose the gRPC market data API for `awcli` and any other clients you create.  Note that for simplicity this API service has **no authentication**; so you should run this on a private network only, and restrict access to the host and port that this service runs on.

## License

By using this component you agree to [the license](LICENSE), including the notices below:

LICENSEE ACKNOWLEDGES AND UNDERSTANDS THAT THE SERVICE AND ANY SOFTWARE MAY CONTAIN ERRORS, OMISSIONS, AND PROBLEMS. LICENSEE HEREBY ACCEPTS THE SERVICE AND SOFTWARE, "AS IS" AND WITH ALL FAULTS, DEFECTS AND ERRORS AND LICENSEE UNDERSTANDS THAT IT ASSUMES ALL RISKS OF USE, QUALITY, AND PERFORMANCE. NEITHER ALPHAWEIGHTED NOR ANY OF ALPHAWEIGHTED'S LICENSORS MAKE ANY EXPRESS WARRANTIES, AND EACH OF THEM DISCLAIMS ALL IMPLIED WARRANTIES, INCLUDING IMPLIED WARRANTIES OF ACCURACY, MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE, TITLE AND NON-INFRINGEMENT.

LICENSEE AGREES AND ACKNOWLEDGES THAT NEITHER ALPHAWEIGHTED NOR ANY OF ITS LICENSORS MAY BE HELD LIABLE FOR ANY CLAIM, LOSS, DAMAGES, EXPENSES OR COSTS OF AN INDIRECT NATURE, INCLUDING CONSEQUENTIAL OR SPECIAL DAMAGES, LOST PROFITS OR OTHERWISE AND IN NO EVENT SHALL THEY BE LIABLE FOR ANY DAMAGES IN EXCESS OF THE AMOUNT OF FEES PAID TO ALPHAWEIGHTED BY LICENSEE (IF ANY) UNDER THIS AGREEMENT DURING THE IMMEDIATELY PRECEDING SIX MONTHS. THIS LIMITATION APPLIES TO ALL CAUSES OF ACTION OR CLAIMS IN THE AGGREGATE, INCLUDING, WITHOUT LIMITATION, BREACH OF CONTRACT, BREACH OF WARRANTY, INDEMNITY, NEGLIGENCE, STRICT LIABILITY, MISREPRESENTATION AND OTHER TORTS. THE LIMITATIONS IN THIS SECTION APPLY TO YOU ONLY TO THE EXTENT THEY ARE LAWFUL IN YOUR JURISDICTION.

THIS LIMITATION OF LIABILITY IS INTENDED TO APPLY WITHOUT REGARD TO WHETHER OTHER PROVISIONS OF THIS AGREEMENT HAVE BEEN BREACHED OR HAVE PROVEN INEFFECTIVE OR IF A REMEDY FAILS OF ITS ESSENTIAL PURPOSE. YOU ACKNOWLEDGE THAT IF THE ABOVE LIMITATION WERE NOT INCLUDED HEREIN, ALPHAWEIGHTED WOULD NOT LICENSE THE SERVICE OR SOFTWARE TO YOU.
