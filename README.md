# humio-community-iocs
This project uses Free and OpenSource IoC/Tracker to update Humio Repository and View Lookup files. This will allow Humio Search results to be enriched with Threat data.

Currently implemented lists:
- Feodo Tracker,  Botnet C2 Indicators Of Compromise (IOCs) - (https://feodotracker.abuse.ch/)

## How to Use this tool
The humio-community-iocs tool is available as a docker container (alatham99/humio-community-iocs) hosted in Docker Hub. It is recommended to use the docker container where possible.

The first step for either option is to gather the required inputs for use with humio-community-iocs.

From Humio, you will need the following information:
- The URL of your Humio server (e.g. https://cloud.community.humio.com:443/)
- The name of the Repository/View you want to load the lookup File into
- an API Token with rights to manage Lookup Files

   From the Humio console Click on the logged in User icon (Top Right) then Manage Your Account | API Tokens | Generate New API Token. Copy the API Token as we will need that later.

Clone this Github repository:
> git clone https://github.com/nthnthcandrew/humio-community-iocs.git

Browse to the cloned repository and edit the feodo2humio.conf file:
>[HUMIO]
>
>Humio_Host = https://cloud.community.humio.com:443/
>
>Humio_API_Token = [HUMIO USER TOKEN]
>
>Humio_Lookup_File = ipblocklist.csv
>
>Humio_Repository = [HUMIO REPOSITORY NAME]
>
>[FEODO]
>
>Feodo_URL = https://feodotracker.abuse.ch/downloads/ipblocklist.json
>
>[GENERAL]
>
>debug = False

Ensure you update the following values specific to your environment:
- Humio_Host
- Humio_API_Token
- Humio_Repository

You can now start the docker container using the feodo2humio.conf to intialise the container:

> docker run -it -v ~/humio-community-iocs/feodo2humio.conf:/app/feodo2humio.conf alatham99/humio-community-iocs:latest

**Make sure that you specify the full path to the feodo2humio.conf file**

If you are successful you will receive a message such as:
>SUCCESS: 557 entries added to Lookup File "ipblocklist.csv"

## Using the IoC lookups in Humio
- Log into your Humio console
- In the search field use the match() query function and pass to it the fields you want to lookup and against with column in the Lookup File you want to match against. For example:
> match(file="ipblocklist.csv", column=FEODO_ip, field=id.resp_h)

AND for hostnames
> match(file="ipblocklist.csv", column=FEODO_hostname, field=server_name)

For more information on how to use Lookup Files please refer to - https://library.humio.com/reference/query-functions/functions/match/

### Triggering Results
In general you may find that you have no events that will match to an entry in the Lookup File. To validate your queries/Alerts/Dashboards are working use the configuration file to enable Triggering. This will allow you to specify IP addresses and Hostnames that you know exist in your events to produce some enriched search results. You enable the functionality, disabled by deafult, by setting the "Trigger_Result" value to **True** and providing relevant data in the "Trigger_Data" key. 
- **Ensure that you adhere to the format of the Trigger_Data. It must be comma separated and comprise of 10 fields ONLY.**

To disable this functionality set the "Trigger_Result" value to **False** and run the docker container again to reload the lookup file.
