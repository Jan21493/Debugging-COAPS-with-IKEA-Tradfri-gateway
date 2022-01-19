# Debugging COAPS with IKEA Tradfri gateway
These notes are an add-on to the excellent guide / introduction to this topic [Ikea Tradfri CoAP Docs](https://github.com/glenndehaan/ikea-tradfri-coap-docs). With the very well-described guide and the *coap-client* tool, you can easily take the first steps by yourself and query your own devices or set values.

## Constrained Application Protocol
The Constrained Application Protocol (CoAP) was developed for the Internet of Things (IoT) and submitted to the Internet Engineering Task Force (IETF) in 2016. It is specially designed for the scarce memory and CPU resources of IoT devices and adopts the basics of REST, but also HTTP. Here is a link to the protocol definition: [Constrained Application Protocol (CoAP)](https://tools.ietf.org/id/draft-ietf-core-coap-09.html#rfc.section.3). Also of interest is the following link, which contains notes on integration into programming languages and tools (https://coap.technology/).

The protocol runs over UDP and is therefore connectionless. For IoT devices, this has the advantage that complex TCP communication with acknowledgments and buffers can be omitted. The IKEA Gateway only uses the encrypted variant COAPS, which uses DTLS as transport.

The encrypted communication is not negotiated via certificates as with SSL, but initially via a security code that is printed on the underside of the gateway. With this securtiy code, a CoAP client can, for example, request a pre-shared-key for a desired username when setting up the communication. This username and pre-shared-key are not used to encrypt the packets itself, but to establish symmetric keys in the beginning of the communication. This is comparable to a TLS session where each side is sending a "hello" and dynamic keys are negotiated between both parties as well. In fact TLS looks to have been used as a template when COAPS was designed. 

**TIP**: If you work in parallel with Node-RED and the *node-red-contrib-ikea-tradfri* library, you should adopt the user name and pre-shared key from Node-RED. At least I had problems when several clients with different keys are accessing the gateway at the same time. There might be a limitation in the number of concurrent usernames / pre-shared keys, but I couldn't find any details.

## Enhancements
With a pipe to the tools *sed* and then to  *jq*, the JSON data can be output in a more readable manner. This is shown below for one of my IKEA GU10 spots:

````
coap-client -m get -u my_tradfri_username -k "my_tradfri_key" "coaps://192.168.1.73:5684/15001/65550" | sed "1,3d" | jq
````
The output should look like:

````
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
I have added the comments for the properties show above manually! The tool does not provide this (yet). The properties are coded as simple numbers to keep it simple on the IoT devices. The descriptions for the property values are partly from the guide linked above and partly from [AlCalzone's node-tradfri-client](https://github.com/AlCalzone/node-tradfri-client/blob/master/src/lib/light.ts), a low-level library from AlCalzone that is included in e.g. *node-red-contrib-tradfri*. Devices other than light bulbs are also included. IKEA has not officially published the details, but almost all parameters are known today. These are probably based on the definition of the Open Mobile Alliance (OMA), see [Introduction to LightweightM2M](https://wiki.openmobilealliance.org/display/TOOL/Introduction+to+LightweightM2M) and [OMA LightweightM2M (LwM2M) Object and Resource Registry](https://technical.openmobilealliance.org/OMNA/LwM2M/LwM2MRegistry.html) for details.

In addition to a list of end devices, a list of groups, scenes/moods, notifications and smart tasks can be output via different URIs. You can then use the respective instanceIDs to get details about an individual element in a list. Everything is more detailed in the linked guide, so I'll save the details here.

## Sniffing CoAPS Protocol
It gets really exciting when you want to sniff the communication between the client, e.g. Node-RED, and the IKEA Tradfri gateway via Wireshark. You can get a packet capture, for example, via port mirroring on your own network switch or a Fritzbox (popular in Germany as an Internet router) and calling it up via the URL /html/capture.html. At first you only see encrypted packets, because the IKEA Tradfri Gateway only speaks COAPS (S=secure). However, there is the option of storing the pre-shared key in Wireshark in order to be able to read the data traffic in plain text.

To do this, the pre-shared key that e.g. Node-RED uses, for example, must first be read out of the config-node. If you are using the coap-client on the CLI, you have that pre-shared key already. With Node-RED You can call up the embedded tradfri-config node via a tradfri-monitor node (click on the edit symbol) and read out the pre-shared key very easily.

I have added the comments to properties show above manually! This is not (yet) implemented in the tool. These properties that are coded in simple numbers to keep it simple on the IoT devices are partly from the guide linked above and partly from code that I found deep in Al Calzone's node-tradfri-client, e.g. [light.ts code](https://github.com/AlCalzone/node-tradfri-client/blob/master/src/lib/light.ts). Devices other than light bulbs are also included. IKEA has not officially published the details, but almost all parameters are now known today. The property numbers are probably based on the definition of the Open Mobile Alliance (OMA), see [] (https://devtoolkit.openmobilealliance.org/OEditor/default.aspx).

In addition to a list of end devices, a list of groups, scenes/moods, notifications and smart tasks can be output via different URIs. You can then use the respective instanceIDs to get details about an individual element in a list. Everything is more detailed in the linked instructions, so I'll save the details here.

It gets really exciting when you want to sniff the communication between the client, e.g. Node-RED, and the IKEA Tradfri gateway via Wireshark. You may also sniff the communication between the IKEA Tradfri app and the Tradfri gateway! Of course, it's not THAT exciting, if you use the coap-client utility on the CLI, because you can see all in- and output directly, but even with that tool, you may get additional information about the CoAP protocol itself.

You can get a recording, for example, via port mirroring on your own network switch or a Fritzbox and calling it up via the URL /html/capture.html. At first you only see encrypted packets, because the IKEA Tradfri Gateway only speaks COAPS. However, there is the option of storing the pre-shared key in Wireshark in order to be able to read the data traffic in plain text.

To do this, the pre-shared key that Node-RED uses, for example, must first be read out of the config-node. You can call up the embedded tradfri-config node via a tradfri-monitor node (click on the edit symbol) and read out the pre-shared key very easily.

<p align="center"><img src="/images/Read&#32;Identity&#32;and&#32;Pre-shared-key&#32;from&#32;config-node&#32;in&#32;Node-RED.png" alt="Read Identity and Pre-shared-key from config-node in Node-RED" width="50%" ></p>

The key still has to be converted from ASCII to HEX, which can be done using various tools. The key is then entered in hex format in Wireshark in the "*Settings*", "*Protocols*", "*DTLS*" menu:

<p align="center"><img src="/images/Wireshark&#32;-&#32;Preferences&#32;-&#32;Protocols.png" alt="Wireshark, Preferences, Protocols" width="75%" ></p>

**After** you have started a packet capture in Wireshark, you have to tell your client to start a new session, e.g. by triggering a restart of the flows in Node-RED or restarting the IKEA Smart Home app. On the CLI a new session is automatically created each time you send a URL, so you don't have to care about this.

<p align="center"><img src="/images/Node-RED&#32;-&#32;Restart&#32;Flows.png" alt="Node-RED - Restart Flows" width="30%" ></p>

From this point on, the CoAP communication is displayed in plain text.

**NOTE**: after each new capture you have to restart the flows in Node-RED or restart the IKEA Smart Home app, since a dynamic key is generated for each "*session*" from the pre-shared key and identity name, which Wireshark unfortunately does not remember when you start a new capture. But you can certainly live with that. It might be easiest to keep capturing while looking into the packets.

The following is an example of the PUT command that Node-RED uses to set the values of a lamp with instanceID 65550 (Spot A1):

<p align="center"><img src="/images/Wireshark&#32;-&#32;COAP&#32;decrypted.png" alt="Wireshark - COAP decrypted" width="100%" ></p>

The parameters and values are passed in JSON format. In the example below, the brightness is set to 25% and the light color to "warm":

<p align="center"><img src="/images/CoAP&#32;decrypted.png" alt="CoAP decrypted in detail" width="50%" ></p>

This message is acknowledged by the IKEA Tradfri Gateway as most messages. By using ACKs on the application layer the CoAP protocol compensates for the build-in reliability that is provided by TCP.

I find it very interesting to read the CoAP communication, for example to check that events are being sent between Node-RED and the Tradfri Gateway and the values don't have to be polled at a regular short interval. It is not clear to me why Node-RED is constantly sending empty *CON* messages. Maybe it is used as a health-check. However, these messages are all acknowledged with an RST (reset) by the gateway.

If you examine a complete "*session*" more closely, you can see that Node-RED initially queries the device list and all devices from it in order to be able to select the devices by user-friendly name - at least for the "control" nodes. In addition, all instanceIDs with an Observe: Register option are subscribed with the queries and therefore the CoAP client is able to get an event when values for these instance IDs have changed. A publisher-subscriber communication is used here, which avoids constant polling.

<p align="center"><img src="/images/CoAP&#32;Observe&#32;-&#32;Register.png" alt="CoAP Observe - Register" width="50%" ></p>

If you change the lamps via the IKEA remote control, for example, an event is sent to Node-RED, as can be seen in the example below:

<p align="center"><img src="/images/Event&#32;from&#32;Tradfri&#32;Gateway&#32;to&#32;CoAP.png" alt="Event from Tradfri Gateway to CoAP" width="100%" ></p>

## Sniffing IKEA smart home app

Since the IKEA Smart Home app is the primary reference for communication with the Tradfri Gateway, it would of course be great if you could also read this communication.

My first ideas: *If you can read the pre-shared key from the IKEA Smart Home App or initially capture the coupling via the security code, then maybe you could also read the communication between the IKEA Smart Home App and the Tradfri Gateway?*

I didn't take a closer look at the first option, but tried the second option right away. So I deleted the IKEA Smart Home app from my mobile phone and reinstalled it. This will delete locally stored information such as pre-shared keys. Then I reconnected to the gateway via the app and was prompted -  as expected - to scan the QR code or alternatively to enter the security code manually. After entering this information, I was able to see my installed lamps in the app again.

Then I examined the packet capture with Wireshark. Initially I only saw DTLS communication and Hello packets from both sides. Upon closer examination, I found that encrypted communication is negotiated, similar to what happens with TLS, but with fewer ciphers. The authentication phase, in which the client requests a pre-shared key from the server, has already been described [here](https://github.com/glenndehaan/ikea-tradfri-coap-docs#authenticate) in the guide mentioned. This was already encrypted, so that nothing could be read in plain text.

I didn't even have to search the COAPS RFC, but saw that the security code '*coincidentally*' had the same length as the pre-shared keys and assumed that this key might have been used for the initial encryption. So I saved the packet capture, converted the security code from ASCII to hex and entered it in Wireshark. **Bingo!** Now I was able to read the authentication in clear text:

<p align="center"><img src="/images/COAPS&#32;request&#32;from&#32;client&#32;to&#32;authenticate.png" alt="COAPS request from client to authenticate" width="100%" ></p>

At the beginning you can see several "*Client Hello*" and "*Server Hello*" packets and the negotiation of the ciphers for symmetric encryption. This is followed by authentication, during which the CoAP client requests a pre-shared key for a given user name. To do this, the client sends the property "*9090*" and the given (typically dynamically selected) username in JSON format to the CoAP server.

The response from the CoAP server is explained in more detail below:

<p align="center"><img src="/images/COAPS&#32;server&#32;response&#32;with&#32;pre-shared&#32;key.png" alt="COAPS server response with pre-shared key" width="100%" ></p>

The response in JSON format contains the property "*9091*" and the pre-shared key that is randomly chosen by the server. You can easily copy this key to the clipboard and save it using the menu items "*Copy*", "*...as Printable Text*". After converting this pre-shared key back from ASCII to hex and entering it into Wireshark, you can finally read the communication between the app and the gateway.

## Security of COAPS
Even if this guide shows how to read the encrypted communication, for example to better understand the communication for new features, existing or new devices, the CoAPS protocol is very secure in my opinion. Decryption is only possible if you can read the initial coupling between the app and the gateway, which takes place via the security code. Since communication between the IKEA Tradfri gateway and the Smart Home app only takes place in the local LAN and (hopefully encrypted) WLAN at home, security is not compromised in practice. I really like that secure 'local' approach from IKEA without using a public cloud for communication! No personal data or keys are forwarded to any cloud. 

Have fun decoding the CoAP communication!
