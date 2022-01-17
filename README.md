# Debugging COAPS with IKEA Tradfri gateway
These notes are an add-on to the excellent introduction [Ikea Tradfri CoAP Docs](https://github.com/glenndehaan/ikea-tradfri-coap-docs). With the linked, very well-described instructions and the coap-client tool, you can easily take the first steps yourself and query your own devices.

The Constrained Application Protocol (CoAP) was developed for the Internet of Things (IoT) and submitted to the Internet Engineering Task Force (IETF) in 2016. It is specially designed for the scarce memory and CPU resources of IoT devices and adopts the basics of REST, but also HTTP. Here is a link to the protocol definition: [Constrained Application Protocol (CoAP)](https://tools.ietf.org/id/draft-ietf-core-coap-09.html#rfc.section.3). Also of interest is the following link, which contains notes on integration into programming languages and tools (https://coap.technology/).

The protocol runs over UDP and is therefore connectionless. For IoT devices, this has the advantage that complex TCP communication with acknowledgments and buffers can be omitted. The IKEA Gateway only uses the encrypted variant COAPS, which uses DTLS as transport.

The encrypted communication is not negotiated via certificates as with SSL, but initially via a security code that is printed on the underside of the gateway. With this securtiy code, a CoAP client can, for example, request a pre-shared-key for a desired username when setting up the communication. This username / pres-shared-key are not used to encrypt each packet, but are used when a "session" ist established. A "session" is comparable to a TLS session where each side is sending a "hello" and dynamic keys are negotiated between both parties.

**TIP**: If you work in parallel with Node-RED and the node-red-contrib-ikea-tradfri library, you should adopt the user name and pre-shared key from Node-RED. At least I had problems when several clients with different keys accessed the gateway at the same time.

With a pipe to the tools * *sed* * and then to * *jq* * , the JSON data can be output in a more readable manner. This is shown below for one of my IKEA GU10 spots:

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
``` 

