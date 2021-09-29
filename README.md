# Node-Red-OPCuaServer-UR
 This flow brings data from a Universal robot via Modbus TCP to a Dashboard. This data is then send to a Node-Red OPC UA server and can be access via an OPC UA client.

**Tested Node-red Version**
- 2.0.6

**Requied Node-red nodes:**
- node-red-contrib-opcua-server
- node-red-contrib-bitunloader
- node-red-dashboard
- node-red-contrib-modbus
- node-red-contrib-ui-artless-gauge

**A overview of the Node-red flow - Modbus**

![Modbus Flow overview](https://github.com/glinvad/Node-Red-OPCuaServer-UR/blob/main/Pictures/OverviewFlowModbus.jpg)

The data is send from this flow to the OPC-UA flow

![Sending data to OPC UA flow](https://github.com/glinvad/Node-Red-OPCuaServer-UR/blob/main/Pictures/SendingDataToOPCUAflow.jpg)
![Sending data to OPC UA flow](https://github.com/glinvad/Node-Red-OPCuaServer-UR/blob/main/Pictures/SendingDataToOPCUAflow2.jpg)

**A overview of the Node-red flow - OPC UA**

![Modbus Flow overview](https://github.com/glinvad/Node-Red-OPCuaServer-UR/blob/main/Pictures/OverviewFlowOPCUA.jpg)

**How to setup**

OPC UA is a machine to machine communication protocol for industrial automation - It gives a posiblility to communicate both vertically and horizontally in an industrial setting.

Modbus can be used for a wide variety of equitment. Here it is used as a data provider. For simpel testing, I used URSim, an virtual mashine with the UR Polyscope software build in (It can be downloaded from Universal robots homepage) that runs a Modbus TCP Server.

REMEMBER TO CHANGE THE IP TO A MODBUS SERVER THAT HAVE RELEVANT DATA AVALIABLE

![Modbus server](https://github.com/glinvad/Node-Red-OPCuaServer-UR/blob/main/Pictures/ModbusTCPserver.jpg)

**How to import the code**
- Install Node-Red on your platform
- Install the Nodes
- Go to Import in the top corner 

![Import Node-red](https://github.com/glinvad/Node-Red-OPCuaServer-UR/blob/main/Pictures/NodeRedImport.jpg)

- Inset the code from the **Node-Red Code**
- Setup the Modbus server, this setup is made for the adresses of the FC3: Holding register that fits a Universal Robot.
- Setup the OPC UA server for your use case.
- Have some fun with the data :)

**Change the pull from holding register**

![FC3 holding register](https://github.com/glinvad/Node-Red-OPCuaServer-UR/blob/main/Pictures/SettingUpGetHOLDING.jpg)

**Making data ready for the server**

The two function are sorting the data and putting it into an array, that are set to be a global variable. 

**Setting up the server**

The OPC UA Compact server can be setup by dobble clicking on it.

![OPC UA Compact server](https://github.com/glinvad/Node-Red-OPCuaServer-UR/blob/main/Pictures/OPCUACompactserver.jpg)

When the data are avalible globally via the function blocks the servers adresse space can configurede. *See the file: Node-Red Code\OPCUA-Compact-server-adress-space.json*.

**How to add a new data point to the server**
1. Put the data as a global variable, here the variable are "isoOutput" and "IsoInput"

![Ready data for Server](https://github.com/glinvad/Node-Red-OPCuaServer-UR/blob/main/Pictures/FunctionSetGlobalDataToOPCUAserver.jpg)

2. Open the server and edit the address space (From here the focus will be en the variable "isoInput"). 

- Set the variable to 0: line 27

`this.sandboxFlowContext.set("isoInputs", 0);`

- Create the device: line 45 - 47 

`const myDevice = namespace.addFolder(rootFolder.objects, {`

`"browseName": "Universal Robot"`

`});`

- Create the namespace: line 49

`const DigitalInputOutput = namespace.addFolder(myDevice, { "browseName": "Digital InputOutput" });`

- place the variable on the namespace and name it : line 50

`const isoInputs = namespace.addFolder(DigitalInputOutput, {"browseName": "Inputs"});`

- configere the variable as a OPC UA object: line 61 - 82

`//Digital input`
`  const DigitalInput = namespace.addVariable({`
`    "organizedBy": isoInputs,`
`    "browseName": "Input",`
`    "nodeId": "ns=1;s=Isolated_Input",`
`    "dataType": "Double",`
`    "value": {`
`      "get": function() {`
`        return new Variant({`
`          "dataType": DataType.Double,`
`          "value": flexServerInternals.sandboxFlowContext.get("isoInput")`
`        });`
`      },`
`      "set": function(variant) {`
`        flexServerInternals.sandboxFlowContext.set(`
`          "isoInput",`
`          parseFloat(variant.value)`
`        );`
`        return opcua.StatusCodes.Good;`
`      }`
`    }`
`  });`

- configere a view on the server: line 241 - 244
`  const viewDIO = namespace.addView({

`    "organizedBy": rootFolder.views,`
	
`    "browseName": "UR-Digital-Input-Output"`
 
` });`
  
- add a reference to the view: line 252 - 255

`  viewDIO.addReference({`

`    "referenceType": "Organizes",`

`    "nodeId": DigitalInput.nodeId`

`  });`
  
- deploy the Node-Red and wait for the server to light up green 

![OPC UA Server active](https://github.com/glinvad/Node-Red-OPCuaServer-UR/blob/main/Pictures/OPCServeractive.jpg)

- connect to the server via a client



**Access data via OPC UA Client**

To access the data on the server, use a OPC UA Client, here I use Unified Autoamtion UAExpert Client (It can be downloaded for free on they homepage)

![OPC UA Client](https://github.com/glinvad/Node-Red-OPCuaServer-UR/blob/main/Pictures/OPCUAclient.jpg)


