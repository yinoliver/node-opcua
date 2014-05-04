Creating a Simple Server
=========================

[sample_server.js](#Structure "save: | jshint ")

In this example, we want to create a OPCUA Server that exposes 2 read/write variables

The server will expose the variable under a new folder named "MyDevice".

     + RootFolder
         + MyDevice
             + MyVariable1
             + MyVariable2

## Structure

    _"declaration"

    _"server instantiation"

    _"setting server info"

    _"server initialisation"


### declaration

The node-opcua sdk is made available to the application by this 'require' statement:

    var opcua = require("node-opcua");

### server instantiation

A OPCUAServer instance need to be created.
Options can be passed to the OPCUAServer to customize the behavior.
For a simple server, you just need to specify a tcp port.

    // Let create an instance of OPCUAServer
    var server = new opcua.OPCUAServer({
        port: 4334 // the port of the listening socket of the server
    });

### setting server info

additional information can be set at this stage such as the server *buildInfo*.

    // we can set the buildInfo
    server.buildInfo.productName = "MySampleServer1";
    server.buildInfo.buildNumber = "7658";
    server.buildInfo.buildDate = new Date(2014,5,2);

### server initialisation

Once created the server shall be initialised.
During initialisation, the server will load its default **nodeset** and prepare the binding of all standard OPCUA variables.
The **initialize** method is a asynchronous operation that requires a 'callback' function that will get executed
when the initialization process is completed. the *callback* is function that contains the post_initialisation
steps that we want to execute.

    function post_initialize() {
        console.log("initialized");
        _"post initialisation"
    }
    server.initialize(post_initialize);


### post initialisation

Once the server has been initialized, it is a good idea to extend the default server namespace with our variables.

Lets create a function that will extend the server default address space with some
variables that we want to expose. This function will be called inside the initialize callback.


    function construct_my_address_space(server) {

        // declare some folders
        _"declare a new folder"

        // add variables in folders
        _"add a variable"

    }

    construct_my_address_space(server);

    server.start(function() {
        console.log("Server is now listening ... ( press CTRL+C to stop)");
        console.log("port ", server.endpoints[0].port);
        _"display endpoint url"
    });


#### declare a new folder

     server.engine.createFolder("RootFolder",{ browseName: "MyDevice"});

#### add a variable

Adding a read variable inside the server namespace requires only a getter function that returns a opcua.Variant
containing the value of the variable to scan.

    // add a variable named MyVariable1 to the newly created folder "MyDevice"
    var variable1 = 10.0;
    server.nodeVariable1 = server.engine.addVariableInFolder("MyDevice",{
        browseName: "MyVariable1",
        value: {
            get: function () {return new opcua.Variant({dataType: opcua.DataType.Double, value: variable1 });}
        }
    });

Note that we haven't specified a nodeid for the variable.The server will automatically assign a new nodeId for us.

Let's create a more comprehensive Read-Write variable with a fancy nodeId

    // add a variable named MyVariable1 to the newly created folder "MyDevice"
    var variable2 = "some text";
    server.nodeVariable2 = server.engine.addVariableInFolder("MyDevice",{
            browseName: "MyVariable1",
            value: {
                nodeId: "ns=4;b=1020FFAA", // some opaque NodeId in namespace 4
                get: function () {
                    return new opcua.Variant({dataType: opcua.DataType.Double, value: variable2 });
                },
                set: function (variant) {
                    variable1 = parseFloat(variant.value);
                    return opcua.StatusCodes.Good;
                }
            }
    });

Lets create a variable that expose the percentage of free memory on the running machine.

Let's write a small utility function that calculate this value.

    var os = require("os");
    /**
     * returns the percentage of free memory on the running machine
     * @returns {double}
     */
    function available_memory() {
        // var value = process.memoryUsage().heapUsed / 1000000;
        var percentageMemUsed = os.freemem() / os.totalmem() * 100.0;
        return percentageMemUsed;
    }

Now let's expose our OPUC Variable

    server.nodeVariable3 = server.engine.addVariableInFolder("MyDevice", {
        nodeId: "ns=4;s=free_memory", // a string nodeID
        browseName: "FreeMemory",
        value: {
            get: function () {return new opcua.Variant({dataType: opcua.DataType.Double, value: available_memory() });}
        }
    });


### display endpoint url

Once the server has been created and configured, it is possible to retrieve the endpoint url.

    var endpointUrl = server.endpoints[0].endpointDescription().endpointUrl;
    console.log(" the primary server endpoint url is ", endpointUrl );


### starting a server

Once the server has been created and initialised, we use the **start** asynchronous method to let the server
initiate all its endpoints and start listening to clients.

```javascript
    server.start(function(){
        console.log("  server now waiting for connections. CTRL+C to stop");
    });
```
