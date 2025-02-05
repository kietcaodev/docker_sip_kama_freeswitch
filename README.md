# Kamailio + FreeSWITCH

The projects demonstrates how tenants softphones and FreeSWITCH PBXs can be connected to each other using Kamailio SIP router to enable advanced calls routing, logging, measurements and NAT traversal.

## Main Use Case

Remote softphones register on edge SIP routers (Kamailio), that accept registrations and route SIP requests to pre-configured softphone PBX (FreeSWITCH). In response to SIP invite, PBX simultaneously rings extension on the corresponding local and remote softphones or sends SIP to PSTN gateway (Twilio). Local PBX softphones are able to reach out to remote softphones via the same SIP router.

## Build Kamailio && Postgresql
docker run --name postgres -e POSTGRES_PASSWORD=admin -p 5432:5432 -d postgres:15

docker build -t kamailio:local .

docker run --name kamailio --network host -v $(pwd)/etc:/etc/kamailio -d kamailio:local

docker exec -it kamailio sngrep

## Build Freeswitch
docker build --build-arg TOKEN=pat_9GLDnLG9uxxH524XFp4nM4Wk -t freeswitch:local .

docker run --name freeswitch --network host --cap-add SYS_NICE -v $(pwd)/etc:/etc/freeswitch -v ~/Documents/freeswitch/scripts/perl:/scripts -d freeswitch:local

docker exec -it freeswitch fs_cli

## Implementation Details

-   scalable SIP router configuration
-   persistent TCP/TLS connections with softphone clients and PBXs
-   SIP requests are proxied to the exact instance where target softphone/PBX is registered
-   locations and extension to PBX mappings are stored in shared PostgreSQL database
-   only standard Kamailio modules used: usrloc, registrar, postgres, sqlops, nathelper, tls
-   tested with Linphone and Zoiper softphones on Linux and Android

## Usage

To build Docker images and start Kamailio and FreeSWITCH go through readme files. Make sure your local network is 192.168.0.0/24, otherwise update the configs. If all good, Kamailio and FreeSWITCH should be listening on ports 5080 and 5060, 5061 (`netstat -ntlp`). Then configure PBX mappings in database and start registering softphones on Kamailio or FreeSWITCH with extensions 1000-1002 and password 12345.
