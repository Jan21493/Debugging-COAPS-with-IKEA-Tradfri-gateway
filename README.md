# Debugging COAPS with IKEA Tradfri gateway
These notes are an add-on to the excellent introduction [Ikea Tradfri CoAP Docs](https://github.com/glenndehaan/ikea-tradfri-coap-docs). With the linked, very well-described instructions and the coap-client tool, you can easily take the first steps yourself and query your own devices.

The Constrained Application Protocol (CoAP) was developed for the Internet of Things (IoT) and submitted to the Internet Engineering Task Force (IETF) in 2016. It is specially designed for the scarce memory and CPU resources of IoT devices and adopts the basics of REST, but also HTTP. Here is a link to the protocol definition: [Constrained Application Protocol (CoAP)](https://tools.ietf.org/id/draft-ietf-core-coap-09.html#rfc.section.3). Also of interest is the following link, which contains notes on integration into programming languages and tools (https://coap.technology/).

The protocol runs over UDP and is therefore connectionless. For IoT devices, this has the advantage that complex TCP communication with acknowledgments and buffers can be omitted. The IKEA Gateway only uses the encrypted variant COAPS, which uses DTLS as transport.

The encrypted communication is not negotiated via certificates as with SSL, but initially via a security code that is printed on the underside of the gateway. With this securtiy code, a CoAP client can, for example, request a pre-shared-key for a desired username when setting up the communication. This username / pres-shared-key are not used to encrypt each packet, but are used when a "session" ist established. A "session" is comparable to a TLS session where each side is sending a "hello" and dynamic keys are negotiated between both parties.

**TIP**: If you work in parallel with Node-RED and the node-red-contrib-ikea-tradfri library, you should adopt the user name and pre-shared key from Node-RED. At least I had problems when several clients with different keys accessed the gateway at the same time.

With a pipe to the tools *sed* and then to  *jq*, the JSON data can be output in a more readable manner. This is shown below for one of my IKEA GU10 spots:

````
coap-client -m get -u my_tradfri_username -k "my_tradfri_key" "coaps://192.168.1.73:5684/15001/65550" | sed "1,3d" | jq
{
  "9001": "Spot A1",               // user friendly name
  "9003": 65550,                   // instanceID
  "3": {                           // object with vendor information
    "0": "IKEA of Sweden",         // manufacturer name
    "1": "TRADFRIbulbGU10WS345lm", // model name
    "2": "",
    "6": 1,                        // power source (1=internal ?)
    "3": "1.0.012",                // firmware version of light bulb
    "7": 8709,                     // OTA image type
    "8": 5                         // unknown
  },
  "9002": 1641675281,              // creation timestamp
  "9054": 0,
  "9020": 1642111763,              // last seen timestamp
  "9019": 1,                       // reachability state
  "5750": 2,                       // device type (2=light bulb)
  "3311": [                        // light bulb object with data
    {
      "5850": 1,                   // onOff - type boolean
      "5849": 2,                   // action after power restored (1=remember on/off setting 2=always on/default), settable via IKEA Smart Home app
      "5851": 188,                 // brightness/dimmer - number from 0=dark to 254=bright
      "5717": 0,                   
      "9003": 0,
      "5711": 345,                 // color temperature in mireds - number from 250=warm to 454=cold
      "5709": 29090,               // colorX
      "5710": 26728,               // colorY
      "5706": "feb465"             // color rgb type rgb hex string
    }
  ]
}
````
As a comment, I have added the above properties manually. These are partly from the instructions linked above and partly from (https://github.com/AlCalzone/node-tradfri-client/blob/master/src/lib/light.ts), a low-level library from AlCalzone that is included in e.g. *node-red-contrib-tradfri*. Devices other than light bulbs are also included. IKEA has not officially published the details, but almost all parameters are known today. These are probably based on the definition of the Open Mobile Alliance (OMA), see (https://devtoolkit.openmobilealliance.org/OEditor/default.aspx9) for details.

In addition to a list of end devices, a list of groups, scenes/moods, notifications and smart tasks can be output via different URIs. You can then use the respective instanceIDs to get details about an individual element in a list. Everything is more detailed in the linked instructions, so I'll save the details here.

It gets really exciting when you want to sniff the communication between the client, e.g. Node-RED, and the IKEA Tradfri gateway via Wireshark. You can get a packet capture, for example, via port mirroring on your own network switch or a Fritzbox (popular in Germany as an Internet router) and calling it up via the URL /html/capture.html. At first you only see encrypted packets, because the IKEA Tradfri Gateway only speaks COAPS (S=secure). However, there is the option of storing the pre-shared key in Wireshark in order to be able to read the data traffic in plain text.

To do this, the pre-shared key that e.g. Node-RED uses, for example, must first be read out of the config-node. If you are using the coap-client on the CLI, you have that pre-shared key already. With Node-RED You can call up the embedded tradfri-config node via a tradfri-monitor node (click on the edit symbol) and read out the pre-shared key very easily.

As a comment, I have added the above properties, which are all numbered in CoAP. These are partly from the instructions linked above and partly from https://github.com/AlCalzone/node-tradfri-client/blob/master/src/lib/light.ts , a low-level library from AlCalzone that is included in node-red-contrib-tradfri. Devices other than light bulbs are also included. IKEA has not officially published the details, but almost all parameters are now known. These are probably based on the definition of the Open Mobile Alliance (OMA), see https://devtoolkit.openmobilealliance.org/OEditor/default.aspx

In addition to a list of end devices, a list of groups, scenes/moods, notifications and smart tasks can be output via different URIs. You can then use the respective instanceIDs to get details about an individual element in a list. Everything is more detailed in the linked instructions, so I'll save the details here.

It gets really exciting when you want to sniff the communication between the client, e.g. Node-RED, and the IKEA Tradfri gateway via Wireshark. You may also sniff the communication between the IKEA Tradfri app and the Tradfri gateway! Of course, it's not THAT exciting, if you use the coap-client utility on the CLI, because you can see all in- and output directly, but even with that tool, you may get additional information about the CoAP protocol itself.

You can get a recording, for example, via port mirroring on your own network switch or a Fritzbox and calling it up via the URL /html/capture.html. At first you only see encrypted packets, because the IKEA Tradfri Gateway only speaks COAPS. However, there is the option of storing the pre-shared key in Wireshark in order to be able to read the data traffic in plain text.

![Read Identity and Pre-shared-key from config-node in Node-RED](/assets/images/Read Identity and Pre-shared-key from config-node in Node-RED.png)
  
To do this, the pre-shared key that Node-RED uses, for example, must first be read out of the config-node. You can call up the embedded tradfri-config node via a tradfri-monitor node (click on the edit symbol) and read out the pre-shared key very easily.

xxx

**After** you have started a packet capture, you have to tell your client to start a new session, e.g. by triggering a restart of the flows in Node-RED. On the CLI a new session is automatically created each time you send a URL, so you don't have to care about this.

xxx

From this point on, the CoAP communication is displayed in plain text.

**NOTE**: after each new capture you have to restart the flows, since a dynamic key is generated for each "*session*" via the pre-shared key and identity name, which Wireshark unfortunately does not remember when you start a new capture. But you can certainly live with that.

The following is an example of the PUT command that Node-RED uses to set the values of a lamp with instanceID 65550 (Spot A1):


xxx

The parameters and values are passed in JSON format. In the example below, the brightness is set to 25% and the light color to “ warm ”:

xxx

This message is acknowledged by the IKEA Tradfri Gateway as most messages. By using ACKs on the application layer the CoAP protocol compensates for the build-in reliability that is provided by TCP.

I find it very interesting to read the CoAP communication, for example to check that events are being sent between Node-RED and the Tradfri Gateway and the values don't have to be polled at a regular short interval. It is not clear to me why Node-RED is constantly sending empty *CON* messages. Maybe it is used as a health-check. However, these messages are all acknowledged with an RST (reset) by the gateway.

If you examine a complete "*session*" more closely, you can see that Node-RED initially queries the device list and all devices from it in order to be able to select the devices by user-friendly name - at least for the "control" nodes. In addition, all instanceIDs with an Observe: Register option are subscribed with the queries and therefore the CoAP client is able to get an event when values for these instance IDs have changed. A publisher-subscriber communication is used here, which avoids constant polling.

xxx

If you change the lamps via the IKEA remote control, for example, an event is sent to Node-RED, as can be seen in the example below:

xxx

Here's another idea that I haven't tested yet: If you can read the pre-shared key from the IKEA Smart Home app or initially record the coupling via the security code, then maybe you could also improve communication between the IKEA Smart Home app and read along with Tradfri Gateway.

Have fun decoding the CoAP communication!
