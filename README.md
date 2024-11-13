This node-red flow is able to control the heating system with multiple heating circuits.
The heating system could be integrated by the ems-esp gateway using mqtt or other ways of integration.

The heat demand flow is able to decide if a heat demand for a boiler / heatpump is active by reading thermostat entities (actual temp vs. settemp) and using parameters to switch a heating circuit on/off depending on the heatdemand calculated.

***

The following technical prerequisites are needed:
1.	HA Node-Red addon is installed and active.

2.	MQTT Broker is installed and discovery prefix is set to standard “homeassistant”

3.	additional “axios” module needs to be configured within NR as additional npm package and added in settings.js for node-red under
    functionGlobalContext using the file editor add-on from HA (see PDF or Word documentation)

4.	a longterm api access token is generated in HA

***

With these prerequisites the data processing flow consists of:

1. The Node-Red flow for heat demand:
 
A configuration file hd.yaml has to exist in the config directory of HA.
The following entries within hd.yaml:
1.	server local ha api access
2.	the longterm access token generated
3.	outdoor temp entity
4.	outdoortemp_threshold: hd active if outdoortemp is above threshold 

5.	thermostats per room with entity, settemp and actualtemp
- deltam: defining minimum delta temp for heatdemand
- hc: heating circuit (hc1 to hc4)
- weight: weight of this thermostat

6.	heatingcircuits
- hc: hc1 to hc4
- weigthon and weigthoff
- state: for mqtt write
- entity: entity within HA
- on /off: writing values for hc on/off (-1= auto ; 0 = off)
- savesettemp: saving previous settemp for floorheating when overwritten by 0 (off):        
                         true/false

Example hd.yaml: https://github.com/tp1de/home-assistant-node-red-heatdemand/blob/main/hd.yaml

***

Flow Logic:

Once on Start the heat demand entities are created by using mqtt discovery api calls.
These entities are grouped under the device “Heatdemand” within mqtt integration:

Please note that entities are not automatically deleted when you change names. This has to be done using mqtt explorer or a similar tool.
.
.
The heatdemand logic is described by:
For each thermostat actualtemp is compared to settemp. 
If (settemp-actualtemp) > deltam then there is a heatdemand for this thermostat / climatate entity. The demand value is given by the weight.
When actualtemp is >= settemp the then demand for this thermostat is set to zero. 
Please select deltam with care - This defines the correct hysteresis.

All demands for all thermostats of one heating circuit are aggregated and compared to the parameters of the heating circuit. 
If sum(weigths) >= weigthon then hc will be switched on using the on value. 
If sum(weigths) <= weigthoff then hc will be switched off using the off value. 

For floorheating the change of settemp to off will overwrite the former settemp. 
For floorheating savesettemp could be be then set to true. 
In this case the former settemp will be stored and used for comparison of temperatures. 

***

NR Flows:
The following flow can be copied and imported to node-red:

https://github.com/tp1de/home-assistant-node-red-heatdemand/blob/76909f1e963174fb54bd8f2a0f0f714b45bee87c/flownr.txt

****


