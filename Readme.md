# m2m-api
## Table of contents
1. [Connecting to server](#connecting-to-server)
2. [Create Access Token for Browser Client](#create-an-access-token-for-browser-client)
3. [Channel Data Resources](#channel-data-resources)
   * [Set Channel Data Resources on a Device](#set-channel-data-resources-on-a-device)
   * [Capture Channel Data from a Client](#capture-channel-data-from-a-client)
   * [Watch Channel Data from a Client](#watch-channel-data-from-a-client)
   * [Send Data from a Client](#sending-data-from-a-client)
   * [Example - Using MCP 9808 Temperature Sensor](#using-mcp-9808-temperature-sensor)
4. [GPIO Resources for Raspberry Pi](#gpio-resources-for-raspberry-pi)  
   * [Set GPIO Input Resources on a Device](#set-gpio-input-resources-on-a-device)
   * [Set GPIO Output Resources on a Device](#set-gpio-output-resources-on-a-device)
   * [Capture/Watch GPIO Input Resources from a Client](#capture-and-watch-gpio-input-resources-from-a-client)
   * [Control (On/Off) GPIO Output Resources from a Client](#control-gpio-output-resources-from-a-client)
   * [Using Channel Data API for GPIO Input/Output Resources](#using-channel-data-api-for-gpio-resources)
   * [Example - GPIO Input Monitoring and Output Control](#gpio-input-monitoring-and-output-control)
5. [HTTP API Resources](#http-resources)
    * [Set GET and POST Resources on a Device](#device-get-and-post-method-setup)
    * [Client GET and POST Request](#client-get-and-post-request)
6. [Get all available remote devices](#server-query-to-get-all-available-remote-devices-per-user)
7. [Get the available resources from a specific device](#client-request-to-get-the-available-resources-from-a-specific-device)
8. [Connecting to other server](#connecting-to-other-m2m-server)

## Connecting to Server

Before your clients and devices start communicating with each other, they must be authenticated with the server.

### Node.js Application

### Client
```js
const { Client } = require('m2m');

let Client = new Client();

client.connect(() => { your app logic ...});
```
### Device/Server
```js
const { Device } = require('m2m');

let device = new Device(deviceId);

device.connect(() => { your app logic ...});
```    

### Browser Client

```js
// User access token
let tkn = 'jc7734138116139a6ad9a4234e71de810a1087fa9e7fbfda74503d9f5216fc5';

let client = new NodeM2M.Client();

client.connect(tkn, () => { your app logic ...});
```

Click [here](#create-an-access-token-for-browser-client) on how to create an access token.   

## Channel Data Resources

### Set Channel Data Resources on a Device

One unique feature of using a channel data resource is that you only need to set one data source for each unique channel name in the device. 

You can then access it from any client by capturing its value, monitor it by watching its value for changes or update it by sending a new value in real-time.  

```js
const { Device } = require('m2m');

let device = new Device(deviceId);

device.connect(() => {
  ...
  /*
   * Set a name for your channel data. You can use any name you want.
   */
  device.setChannelData(channel, (data) => {
    /*
     * Provide a data source for your channel data.
     * Your data source can be of type string, number or object.
     *
     * It could be a value from a sensor device, database, inventory,
     * metrics, machine status or any performance data from a business application  
     *
     * Below is a pseudocode DataSource() function that returns the value of a data source.
     */
    let ds = DataSource();
    data.send(ds);
  });
  
  // or use the shorthand api below which is the same as above
  
  device.setData(channel, (data) => {
    let ds = DataSource();
    data.send(ds);
  });
  
});
```
### Capture Channel Data from a Client
```js
const { Client } = require('m2m');

let client = new Client();

client.connect(() => {
  ...

  // Use the alias method if you are accessing only one device from your client

  /**********************************************
   * 
   *  Capture channel data using a device alias
   * 
   **********************************************/
  // Access the remote device you want to access by creating an alias object
  let device = client.accessDevice(deviceId);

  device.getChannelData(channel-name, (data) => {
    // data is the value of channel data source
    console.log(data);
  });
  
  // or use the shorthand api below which is the same as above
  
  device.getData(channel-name, (data) => {
    console.log(data);
  });


  // Use the client object method if you are accessing multiple devices from your client

  /*********************************************************
   * 
   *  Capture channel data directly from the client object
   * 
   *********************************************************/

  // Provide the deviceId of the remote device you want to access
  client.getChannelData({id:deviceId, channel:channel-name}, (data) => {
    // data is the value of channel data source
    console.log(data);
  });
  
  // or use the shorthand api below which is the same as above
  
  client.getData({id:deviceId, channel:channel-name}, (data) => {
    console.log(data);
  });
  
});
```

### Watch Channel Data from a Client
```js
const { Client } = require('m2m');

let client = new Client();

client.connect(() => {
  ...

 /********************************************
  * 
  *  Watch channel data using a device alias
  * 
  ********************************************/

  // Create an alias object for the remote device you want to access
  let device = client.accessDevice(deviceId);

  // watch using a default poll interval of 5 secs
  device.watchChannelData(channel-name, (data) => {
    // data is the value of 'channel' data source
    console.log(data);
  });
  
  // or use the shorthand api below which is the same as above
  
  device.watchData(channel-name, (data) => {
    console.log(data);
  });

  // watch using a 1 minute poll interval
  device.watchData(channel-name, 60000, (data) => {
    console.log(data);
  });

  // unwatch channel data
  setTimeout(() => {
    device.unwatchData(channel-name, (data) => {
      console.log(data); // outputs true if channel-name is valid
    });
    // or w/o callback
    device.unwatchData(channel-name);    
  }, 5*60000);

  /*******************************************************
   * 
   *  Watch channel data directly from the client object
   * 
   *******************************************************/

  // Provide the device id of the remote device you want to access
  // as 1st argument of watch method

  // watch using a default poll interval of 5 secs
  client.watchChannelData({id:deviceId, channel:channel-name}, (data) => {
    // data is the value of 'channel' data source
    console.log(data);
  });
  
  // or use the shorthand api below which is the same as above

  client.watchData({id:deviceId, channel:channel-name}, (data) => {
    console.log(data);
  });
  
  // watch using 30000 ms or 30 secs poll interval
  client.watchData({id:deviceId, channel:channel-name, interval:30000}, (data) => {
    console.log(data);
  });

  // unwatch channel data
  setTimeout(() => {
    client.unwatchData({id:deviceId, channel:channel-name}, (data) => {
      console.log(data); // outputs true if channel-name is valid
    });
    // or w/o callback
    client.unwatchData({id:deviceId, channel:channel-name});
  }, 5*60000);
});
```

### Example
#### Device setup
```js
const m2m = require('m2m');

let device = new m2m.Device(100);

let tempData = 'node-m2m';

device.connect(function(){
  device.setData('channel-data', function(data){
    if(data.payload){
      tempData = data.payload;
    }  
    data.send(tempData);
  });
});
```
#### Client setup
```js
const m2m = require('m2m');

let client = new m2m.Client();

client.connect(function(){

  // Note: Use only one of the method below, not both of them

  /***********************************************
   * 
   *  Method 1: access data using a device alias
   * 
   ***********************************************/

  let device = client.accessDevice(100);

  device.getData('channel-data', function(data){
    console.log('device.getData result', data); // 'node-m2m'
  });

  device.watchData('channel-data', function(data){
    // outputs 'node-m2m', after  update it will show 'hello server using an alias'
    console.log('device.watchData result', data); 
  });

  // sending a simple string payload data to 'channel-data' channel
  let payload = 'hello server using an alias';

  device.sendData('channel-data', payload , function(data){
    console.log('device.sendData result', data); // 'hello server using an alias'
  });

  /********************************************************
   * 
   *  Method 2: send data directly from the client object
   * 
   ********************************************************/

  client.getData({id:100, channel:'channel-data'}, function(data){
    console.log('client.getData', data); // 'node-m2m'
  });

  client.watchData({id:100, channel:'channel-data'}, function(data){
    console.log('client.watchData', data); 
  });

  let payload = 'hello server from client object';

  client.sendData({id:100, channel:'channel-data', payload:payload}, function(data){
    console.log('client.sendData', data); // 'hello server from client object'
  });
});
```

### Sending Data from a Client

Instead of capturing or receiving data, you can send data to your remote devices.  

### Set Channel Data on Your Device
```js
const m2m = require('m2m');

let server = new m2m.Device(deviceId);

server.connect(function(){

  server.setData(channel, function(data){

    // you will receive the data from any client from the payload property
    // which you can use for whatever purposes in your application logic
    data.payload;

    // send a response
    data.send(response);
  });

});  
```
### Send Channel Data from a Client
```js
const m2m = require('m2m');

let client = new m2m.Client();

client.connect(function(){

  /*****************************
   * 
   *  Send data using an alias
   * 
   *****************************/
  let server = client.accessDevice(deviceId);

  // the payload can be a string, number or object
  let payload = 'hello server';

  server.sendChannelData(channel-name, payload , function(data){
    console.log('response', data);
  });
  
  // or use the shorthand api below which is the same as above
  
  server.sendData(channel-name, payload , function(data){
    console.log('response', data);
  });

  /**********************************************
   * 
   *  Send data directly from the client object
   * 
   **********************************************/
  // Provide the device id of the remote device you want to access
  client.sendChannelData({id:deviceId, channel:channel-name, payload:payload-data}, (data) => {
    console.log('response', data);
  });
  
  // or use the shorthand api below which is the same as above
  
  client.sendData({id:deviceId, channel:channel-name, payload:payload-data}, (data) => {
    console.log('response', data);
  });

});
```

### Example
#### Device setup
```js
const m2m = require('m2m');
const fs = require('fs');

let server = new m2m.Device(200);

server.connect(function(){

  server.setData('echo-server', function(data){
    // send back the payload to client
    data.send(data.payload);
  });

  server.setData('send-file', function(data){
    let file = data.payload;
    fs.writeFile('myFile.txt', file, function (err) {
      if (err) throw err;
      console.log('file has been saved!');
      // send a response
      data.send({result 'success'});
    });
  });

  server.setData('send-data', function(data){
    console.log('data.payload', data.payload);
    // data.payload  [{name:'Ed'}, {name:'Jim', age:30}, {name:'Kim', age:42, address:'Seoul, South Korea'}];
    // send a response
    if(Array.isArray(data.payload)){
      data.send({data: 'valid'});
    }
    else{
      data.send({data: 'invalid'});
    }
  });

  // server just receiving the payload w/o response
  server.setData('number', function(data){
    console.log('data.payload', data.payload); // 1.2456
  });
});
```
#### Client setup
```js
const m2m = require('m2m');
const fs = require('fs');

let client = new m2m.Client();

client.connect(function(){

  /***********************************
   * 
   *  send data using a device alias
   * 
   ***********************************/

  let server = client.accessDevice(200);

  // sending a simple string payload data to 'echo-server' channel
  let payload = 'hello server';

  server.sendData('echo-server', payload , function(data){
    console.log('echo-server', data); // 'hello server'
  });

  // sending a text file
  let myfile = fs.readFileSync('myFile.txt', 'utf8');

  server.sendData('send-file', myfile , function(data){
    console.log('send-file', data); // {result: 'success'}
  });

  // sending a json data
  let mydata = [{name:'Ed'}, {name:'Jim', age:30}, {name:'Kim', age:42, address:'Seoul, South Korea'}];

  server.sendData('send-data', mydata , function(data){
    console.log('send-data', data); // {data: 'valid'}
  });

  // sending data w/o a response
  let num = 1.2456;

  server.sendData('number', num);

  /**********************************************
   * 
   *  send data directly from the client object
   * 
   **********************************************/
  client.sendData({id:200, channel:'echo-server', payload:payload}, function(data){
    console.log('echo-server', data); // 'hello server'
  });

  client.sendData({id:200, channel:'send-file', payload:myfile}, function(data){
    console.log('send-file', data); // {result: 'success'}
  });

  client.sendData({id:200, channel:'number', payload:num});
});
```

### Using MCP 9808 Temperature Sensor

![](https://raw.githubusercontent.com/EdoLabs/src2/master/example1.svg?sanitize=true)
[](example1.svg)
#### Remote Device Setup in Tokyo

Using a built-in MCP9808 i2c library from array-gpio
```js
$ npm install array-gpio
```
```js
const m2m = require('m2m');
const i2c = require('./node_modules/m2m/examples/i2c/9808.js');

let device = new m2m.Device(110);

device.connect(function(){

  device.setData('sensor-temperature', function(data){
    // temperature data
    let td =  i2c.getTemp();
    data.send(td);
  });
});
```
Using *i2c-bus* npm module to setup MCP 9808 temperature sensor
```js
$ npm install i2c-bus
```
[Configure I2C on Raspberry Pi.](https://github.com/fivdi/i2c-bus/blob/HEAD/doc/raspberry-pi-i2c.md)
\
\
After configuration, setup your device using the following code.
```js
const m2m = require('m2m');
const i2c = require('i2c-bus');

const MCP9808_ADDR = 0x18;
const TEMP_REG = 0x05;

const toCelsius = rawData => {
  rawData = (rawData >> 8) + ((rawData & 0xff) << 8);
  let celsius = (rawData & 0x0fff) / 16;
  if (rawData & 0x1000) {
    celsius -= 256;
  }
  return celsius;
};

const i2c1 = i2c.openSync(1);
const rawData = i2c1.readWordSync(MCP9808_ADDR, TEMP_REG);

let device = new m2m.Device(110);

device.connect(function(){

  device.setData('sensor-temperature', function(data){
    // temperature data
    let td =  toCelsius(rawData);
    data.send(td);
  });
});
```
#### Client application in Boston
```js
$ npm install m2m
```
```js
const m2m = require('m2m');

let client = new m2m.Client();

client.connect(function(){

  let device = client.accessDevice(110);

  device.watch('sensor-temperature', function(data){
    console.log('sensor temperature data', data); // 23.51, 23.49, 23.11
  });
});
```
#### Client application in London
```js
$ npm install m2m
```

```js
const m2m = require('m2m');

let client = new m2m.Client();

client.connect(function(){

  let device = client.accessDevice(110);

  // scan/poll the data every 15 secs instead of the default 5 secs
  device.watch('sensor-temperature', 15000, function(data){
    console.log('sensor temperature data', data); // 23.51, 23.49, 23.11
  });

  // unwatch temperature data after 5 minutes
  setTimeout(function(){
    device.unwatch('sensor-temperature');
  }, 5*60000);

  // watch temperature data again using the default poll interval
  setTimeout(function(){
    device.watch('sensor-temperature');
  }, 10*60000);
});
```

## GPIO Resources for Raspberry Pi

### Set GPIO Input Resources on a Device

Install array-gpio on your Raspberry Pi device
```js
$ npm install array-gpio
```

GPIO input object resources are *read-only*. Clients can read/capture and watch its current state in real-time but they *cannot set or change it*.

```js
const { Device }  = require('m2m');

let device = new Device(deviceId);

device.connect(() => {
  ...

  // Set a GPIO input resource using a single pin - pin1
  device.setGpio({mode:'input', pin:pin1});

  // Set GPIO input resources using multiple pins - pin1, pin2, pin3 and pin4
  device.setGpio({mode:'input', pin:[pin1, pin2, pin3, pin4]});

  // Set GPIO input resources w/ a callback argument
  device.setGpio({mode:'input', pin:[pin1, pin2]}, (gpio) => {
    /*
     * If there is no error, the callback will return a gpio object
     * with a pin and a state property that you can use for additional
     * data processing/filtering with a custom logic
     */
    console.log('pin', gpio.pin, 'state', gpio.state);

    // add custom logic here
  });
});
```

#### Set Simulated GPIO Input Resources on Non-Raspberry Device

You can set GPIO input objects in simulation on Windows or Linux computers for trial. It behaves similarly as if you are using a Raspberry Pi but only in simulation. Set the GPIO input object  resources as usual with a callback and add a *type* property with a value of `sim` or `simulation` from the object argument.  

```js
const { Device }  = require('m2m');

let device = new Device(deviceId);

device.connect(() => {
  ...
  device.setGpio({mode:'input', pin:[15, 19], type:'sim'}, (gpio) => {
    console.log('input pin', gpio.pin, 'state', gpio.state);
  });
});
```

### Set GPIO Output Resources on a Device

Install array-gpio on your Raspberry Pi device
```js
$ npm install array-gpio
```

GPIO output object resources are both *readable* and *writable*. Clients can read/capture and control (on/off) its current state in real-time. At present, you *cannot watch* the state of GPIO output objects.
```js
const { Device }  = require('m2m');

let device = new Device(deviceId);

device.connect(() => {
  ...

  // Set a GPIO output resource using a single pin - pin1
  device.setGpio({mode:'output', pin:pin1});

  // Set GPIO output resources using multiple pins - pin1, pin2, pin3 and pin4  device.setGpio({mode:'output', pin:[pin1, pin2, pin3, pin4]});

  // Set GPIO output resources w/ a callback argument
  device.setGpio({mode:'output', pin:[pin1, pin2]}, (gpio) => {
    /*
     * If there is no error, the callback will return a gpio object
     * with a pin and a state property that you can use for additional
     * data processing/filtering with a custom logic
     */
    console.log('pin', gpio.pin, 'state', gpio.state);

    // add custom logic here
  });
});
```
#### Set Simulated GPIO Output Resources on Non-Raspberry Device

Similar with input objects, you can set GPIO output objects in simulation for Windows or Linux computers for trial. Set the GPIO output objects as usual with a callback and add a *type* property with a value of `sim` or `simulation` from the object argument.    

```js
const { Device }  = require('m2m');

let device = new Device(deviceId);

device.connect(() => {
  ...

  device.setGpio({mode:'output', pin:[33, 35], type:'sim'}, (gpio) => {
    console.log('output pin', gpio.pin, 'state', gpio.state);
  });
});
```

### Capture and Watch GPIO Input Resources from a Client

There are two ways we can capture and watch GPIO input resources from remote devices.


```js
const { Client } = require('m2m');

let client = new Client();

client.connect(() => {
  ...

  let device = client.accessDevice(deviceId);

  /*********************************************
   * 
   *  Using .gpio() method from a device alias
   * 
   *********************************************/

  // get current state of input pin1
  device.gpio({mode:'in', pin:pin1}).getState((state) => {
    // returns the state of pin1
    console.log(state);
  });

  // watch input pin1, default scan interval is 100 ms
  device.gpio({mode:'in', pin:pin1}).watchState((state) => {
    console.log(state);
  });

  /*******************************************************
   * 
   *  Using .gpio() method directly from a client object
   * 
   *******************************************************/

  // get current state of device (id:deviceId) input pin1
  client.gpio({id:deviceId, mode:'in', pin:pin1}).getState((state) => {
    console.log(state);
  });

  // watch state of device (id:deviceId) input pin1, default scan interval is 100 ms
  client.gpio({id:deviceId, mode:'in', pin:pin1}).watchState((state) => {
    console.log(state);
  });


  /*****************************************************
   * 
   *  Using .input()/output() method from device alias
   * 
   *****************************************************/

  // get current state of input pin1
  device.input(pin1).getState((state) => {
    // returns the state of pin1
    console.log(state);
  });

  // watch input pin1, default scan interval is 100 ms
  device.input(pin1).watchState((state) => {
    console.log(state);
  });

  /*******************************************************************
   * 
   *  Using .input()/output() method directly from the client object
   * 
   *******************************************************************/

  // get current state of device (id:deviceId) input pin1
  client.input({id:deviceId, pin:pin1}).getState((state) => {
    // returns the state of pin1
    console.log(state);
  });

  // watch state of device (id:deviceId) input pin1, default scan interval is 100 ms
  client.input({id:deviceId, pin:pin1}).watchState((state) => {
    console.log(state);
  });

});
```

### Control GPIO Output Resources from a Client

Similar with GPIO input access, there are two ways we can set or control (on/off) the GPIO output state from remote devices.

```js
const { Client } = require('m2m');

let client = new Client();

client.connect(() => {
  ...

  let device = client.accessDevice(deviceId);

  /************************************************
   * 
   *  Using .gpio() method to control gpio output
   *  from device alias
   * 
   ************************************************/

  // Applies both for on/off methods

  // turn ON output pin1 w/o callback
  device.gpio({mode:'out', pin:pin1}).on();

  // turn OFF output pin1 w/o callback
  device.gpio({mode:'out', pin:pin1}).off();

  // turn ON output pin1 w/ a callback for state confirmation and
  // for additional data processing/filtering with a custom logic
  device.gpio({mode:'out', pin:pin1}).on((state) => {
    console.log(state);
    // add custom logic here
  });

  // turn OFF output pin1 w/ a callback for state confirmation and
  // for additional data processing/filtering with a custom logic
  device.gpio({mode:'out', pin:pin1}).off((state) => {
    console.log(state);
    // add custom logic here
  });

  /***********************************************************
   * 
   *  Using .input()/output() method to control gpio output
   *  from device alias
   * 
   ***********************************************************/

  // Applies both for on/off methods

  // turn ON output pin1 w/o callback
  device.output(pin1).on();

  // turn OFF output pin1 w/o callback
  device.output(pin1).off();

  // turn ON output pin1 w/ a callback for state confirmation and
  // for additional data processing/filtering with a custom logic
  device.output(pin1).on((state) => {
    console.log(state);
    // add custom logic here
  });  

  // turn OFF output pin1 w/ a callback for state confirmation and
  // for additional data processing/filtering with a custom logic
  device.output(pin1).off((state) => {
    console.log(state);
    // add custom logic here
  });  

  /**********************************************************
   * 
   *  Using .input()/output() method to control gpio output
   *  directly from the client object
   * 
   **********************************************************/

  // get current state of output pin1  
  client.output({id:deviceId, pin:pin1}).getState((state) => {
    // returns the state of pin1
    console.log(state);
  });

  // turn ON output pin1 w/ a callback
  client.output({id:deviceId, pin:pin1}).on((state) => {
    console.log(state);
  });

  // turn OFF output pin1 w/ a callback
  client.output({id:deviceId, pin:pin1}).off((state) => {
    console.log(state);
  });

  // turn ON output pin1 w/o a callback
  client.output({id:deviceId, pin:pin1}).on();

  // turn OFF output pin1 w/o a callback
  client.output({id:deviceId, pin:pin1}).off();

});
```
### GPIO Input Monitoring and Output Control

![](https://raw.githubusercontent.com/EdoLabs/src2/master/example2.svg?sanitize=true)
Install array-gpio both on device1 and device2
```js
$ npm install array-gpio
```
#### Configure GPIO input resources on device1
```js
const m2m = require('m2m');

let device = new m2m.Device(120);

device.connect(function(){
  // Set GPIO input resources using pin 11 and 13
  device.setGpio({mode:'input', pin:[11, 13]});
});
```

#### Configure GPIO output resources on device2

```js
const m2m = require('m2m');

let device = new m2m.Device(130);

device.connect(function(){
  // Set GPIO output resources using pin 33 and 35
  device.setGpio({mode:'output', pin:[33, 35]});
});
```
#### Access GPIO input/output resources from device1 and device2

```js
const m2m = require('m2m');

let client = new m2m.Client();

client.connect(function(){

  let device1 = client.accessDevice(120);
  let device2 = client.accessDevice(130);

  // get current state of input pin 13
  device1.input(13).getState(function(state){
    // show current state of pin 13
    console.log(state);
  });

  // watch input pin 13
  device1.input(13).watchState(function(state){
    if(state){
      // turn OFF output pin 35
      device2.output(35).off();
    }
    else{
      // turn ON output pin 35
      device2.output(35).on();
    }
  });

  // get current state of input pin 11
  device1.input(11).getState(function(state){
    // show current state of pin 11
    console.log(state);
  });

  // watch input pin 11
  device1.input(11).watchState(function(state){
    if(state){
      // turn ON output pin 33
      device2.output(33).on();
    }
    else{
      // turn OFF output pin 33
      device2.output(33).off();
    }
  });

});
```
### Using Channel Data API for GPIO Resources

If the standard API for setting GPIO resources does not meet your requirements, you can use the channel data API to set GPIO input/output resources.

In this example, we will use the API of the *array-gpio* module to watch an input object and control (on/off) an output object. If you are familar with other *npm* GPIO library, you can use it instead.     

#### Device setup
```js
const m2m = require('m2m');
const { setInput, setOutput } = require('array-gpio');

// set pin 11 as input sw1
const sw1 = setInput(11);

// set pin 33 as output led
const led = setOutput(33);

const server = new m2m.Device(200);

server.connect(function(){

  // set 'sw1-state' resource
  server.setData('sw1-state', function(data){
    data.send(sw1.state);  
  });

  // set 'led-state' resource
  server.setData('led-state', function(data){
    data.send(led.state);
  });

  // set 'led-control' resource
  server.setData('led-control', function(data){
    let ledState = null;

    if(data.payload === 'on'){
      ledState = led.on();
    }
    else{
      ledState = led.off();
    }
    data.send(ledState);
  });
});
```

#### Client setup
```js
const m2m = require('m2m');

let client = new m2m.Client();

client.connect(function(){

  let device = client.accessDevice(200);

  // monitor sw1 state transitions every 5 secs
  device.watch('sw1-state', function(data){
    console.log('sw1-state value', data);

    if(data === true){
      device.sendData('led-control', 'on', function(data){
        console.log('led-control on', data); // true
      });
    }
    else{
      device.sendData('led-control', 'off', function(data){
        console.log('led-control off', data); // false
      });
    }
  });

  // monitor led state transitions every 5 secs
  device.watch('led-state', function(data){
    console.log('led-state value', data);
  });
});
```
## HTTP Resources

### Device GET and POST method setup
```js
const m2m = require('m2m');

const server = new m2m.Device(deviceId);

server.connect(() => {

  // set a GET method resource
  server.setHttpGet(path, (data) => {
    // add application logic for the current http GET path
    // send a response
    data.send(response);
  });
  
  // or use the shorthand api below which is the same as above
  
  server.get(path, (data) => {
    data.send(response);
  });

  // set a POST method resource
  server.setHttpPost(path, (data) => {
    // add application logic for the current http POST path

    // the data.body property is the payload from the client
    data.body;
    // send a response
    data.send(response);
  });
  
  // or use the shorthand api below which is the same as above
  
  server.post(path, (data) => {
    // the data.body property is the payload from client
    data.body;
    // send a response
    data.send(response);
  });
  
});
```
### Client GET and POST request
```js
const m2m = require('m2m');

const client = new m2m.Client();

client.connect(() => {

  /******************************************
   * 
   *  Access http resources using an alias
   * 
   ******************************************/
  let server = client.accessServer(deviceId);

  // GET method request
  server.getRequest(path, (data) => {    
    console.log('response', data);
  });
  
  // or use the shorthand api below which is the same as above
  
  server.get(path, (data) => {    
    console.log('response', data);
  });

  // POST method request
  server.postRequest(path, body, (data) => {   
    console.log('response', data);
  });
  
  // or use the shorthand api below which is the same as above

  server.post(path, body, (data) => {   
    console.log('response', data);
  });

  /**********************************************************
   * 
   *  Access http resources directly from the client object
   * 
   **********************************************************/
  
  // GET method request
  client.getRequest({id:deviceId , path:path-name}, (data) => {    
    console.log('response', data);
  });
  
  // or use the shorthand api below which is the same as above
  
  client.get({id:deviceId , path:path-name}, (data) => {    
    console.log('response', data);
  });

  // POST method request
  client.postRequest({id:deviceId, path:path-name, body:body-data}, (data) => {   
    console.log('response', data);
  });
  
  // or use the shorthand api below which is the same as above
  
  client.post({id:deviceId, path:path-name, body:body-data}, (data) => {   
    console.log('response', data);
  });
});
```

### Example
#### Device setup
```js
const { Device } = require('m2m');

const server = new Device(300);

let currentData = {name:'Jim', age:34};

server.connect(() => {
  server.get('current/data', (data) => {
    // send current data as response
    data.send(currentData);
  });

  server.post('data/update', (data) => {
    if(data.body){
      currentData = data.body;
      // send a 'success' response
      data.send('success');
    }
    else{
      // send a 'invalid body' response
      data.send('invalid body');
    }
  });
});
```
#### Client request
```js
const { Client } = require('m2m');

const client = new Client();

client.connect(() => {

  // Note: Use only one method below, but not both

  /*************************************
   * 
   *  GET/POST request using an alias
   * 
   *************************************/
  let server = client.accessDevice(300);

  server.get('current/data', (data) => {    
    console.log('current/data', data); // {name:'Jim', age:34}
  });

  // send {name:'Ed', age:45} as http post body
  server.post('data/update', {name:'Ed', age:45}, (data) => {   
    console.log('data/update', data); // 'success'
  });

  // request after update for path 'current/data'
  server.get('current/data'); // {name:'Ed', age:45}

  /*******************************************************
   * 
   *   GET/POST request directly from the client object
   * 
   *******************************************************/
  client.get({id:300, path:'current/data'}, (data) => {    
   console.log('current/data', data); // {name:'Jim', age:34}
  });

  client.post({id:300, path:'data/update', body:{name:'Ed', age:45}}, (data) => {   
    console.log('data/update', data); // 'success'
  });
});
```
## Device Orchestration

### Remote Machine Monitoring

Install array-gpio for each remote machine.
```js
$ npm install array-gpio
```
#### Server setup
Configure each remote machine's rpi microcontroller with the following GPIO input/output and channel data resources
```js
const { Device } = require('m2m');
const { setInput, setOutput, watchInput } = require('array-gpio');

const sensor1 = setInput(11); // connected to switch sensor1
const sensor2 = setInput(13); // connected to switch sensor2

const actuator1 = setOutput(33); // connected to alarm actuator1
const actuator2 = setOutput(35); // connected to alarm actuator2

// assign 1001, 1002 and 1003 respectively for each remote machine
const device = new Device(1001);

let status = {};

// Local I/O machine control process
watchInput(() => {
  // monitor sensor1
  if(sensor1.isOn){
    actuator1.on();
  }
  else{
    actuator1.off();
  }
  // monitor sensor2
  if(sensor2.isOn){
    actuator2.on();
  }
  else{
    actuator2.off();
  }
});

// m2m device application
device.connect(() => {

  device.setData('machine-status', function(data){

    status.sensor1 = sensor1.state;
    status.sensor2 = sensor2.state;

    status.actuator1 = actuator1.state;
    status.actuator2 = actuator2.state;

    console.log('status', status);

    data.send(JSON.stringify(status));
  });
});
```
#### Client application to monitor the remote machines
In this example, the client will iterate over the remote machines once and start watching each machine's sensor and actuactor status. If one of the sensors and actuator's state changes, the status will be pushed to the client.     
```js
const { Client } = require('m2m');

const client = new Client();

client.connect(() => {

  client.accessDevice([100, 200, 300], function(devices){
    let t = 0;
    devices.forEach((device) => {
      t = t + 100;
      setTimeout(() => {
        // device watch interval is every 10 secs
        device.watch('machine-status', 10000, (data) => {
          console.log(device.id, 'machine-status', data); // 200 machine-status {"sensor1":false,"sensor2":false,"actuator1":false,"actuator2":true}
          /* If one of the machine's status has changed,
           * it will receive only the status from the affected machine
           *
           * Add logic to process the 'machine-status' channel data
           */
        });
      }, t);
    });
  });
});
```
## Using the Browser Interface

### Remote Application Code Editing

Using the browser interface, you can download, edit and upload your application code from your remote clients and devices from anywhere.

To allow the browser to communicate with your application, add the following *m2mConfig* property to your project's package.json.

```js
"m2mConfig": {
  "code": {
    "allow": true,
    "filename": "device.js"
  }
}
```
Set the property *allow* to true and provide the *filename* of your application.

From the example above, the filename of the application is *device.js*. Replace it with the actual filename of your application.


### Application Auto Restart
Using the browser interface, you may need to restart your application after a module update, code edit/update, or a remote restart command.

Node-m2m uses **nodemon** to restart your application.
```js
$ npm install nodemon
```

You can add the following *nodemonConfig* and *scripts* properties in your project's npm package.json as *auto-restart configuration*.
```js
"nodemonConfig": {
  "delay":"2000",
  "verbose": true,
  "restartable": "rs",
  "ignore": [".git", "public"],
  "ignoreRoot": [".git", "public"],
  "execMap": {"js": "node"},
  "watch": ["node_modules/m2m/mon"],
  "ext": "js,json"
}
"scripts": {
  "start": "nodemon device.js"
},
```
From the example above, the filename of the application is *device.js*. Replace it with the actual filename of your application when adding the scripts property. Then restart your node process using *npm start* command as shown below.
```js
$ npm start
```
For other custom nodemon configuration, please read the nodemon documentation.

## Code Edit and Auto Restart Automatic Configuration
Install nodemon.
```js
$ npm install nodemon
```
To configure your package.json for code editing and auto-restart without manual editing of package.json, start your node process with *-config* flag.

*m2m* will attempt to configure your package.json by adding/creating the *m2mConfig*, *nodemonConfig*, and *scripts* properties to your existing project's package.json. If your m2m project does not have an existing package.json, it will create a new one.  

Assuming your application filename is *device.js*, start your node application as shown below.
```js
$ node device -config
```

Stop your node process using *ctrl-c*. Check and verify your package.json if it was properly configured.

If the configuration is correct, you can now run your node process using *npm start* command.
```js
$ npm start
```
Your application should restart automatically after a remote *code update*, an *npm module update*, a remote *restart command* from the browser interface.

### Naming your Client Application for Tracking Purposes

Unlike the *device/server* applications, users can create *client* applications without registering it with the server.

Node-m2m tracks all client applications through a dynamic *client id* from the browser.
If you have multiple clients, tracking all your clients by its *client id* is not easy.

You can add a *name*, *location* and a *description* properties to your clients as shown below.

This will make it easy to track all your clients from the browser interface.
```js
const m2m = require('m2m');

const client = new m2m.Client({name:'Main client', location:'Boston, MA', description:'Test client app'});

client.connect(() => {
  ...
});
```

### Server query to get all available remote devices per user
```js
const m2m = require('m2m');

const client = new m2m.Client();

client.connect(() => {

  // server request to get all registered devices
  client.getDevices((devices) => {
    console.log('devices', devices);
    // devices output
    /*[
      { id: 100, name: 'Machine1', location: 'Seoul, South Korea' },
      { id: 200, name: 'Machine2', location: 'New York, US' },
      { id: 300, name: 'Machine3', location: 'London, UK' }
    ]*/
  });
});
```
### Client request to get the available resources from a specific device

```js
const m2m = require('m2m');

const client = new m2m.Client();

client.connect(() => {

  let device = client.accessDevice(300);

  // request to get device 300 resources
  // GPIO input/output objects, available channels and HTTP url paths, system information etc.
  device.resourcesInfo(function(data){
    console.log('device1 setup data', data);
    // data output
    /*{
      id: 300,
      systemInfo: {
        cpu: 'arm',
        os: 'linux',
        m2mv: '1.2.5',
        totalmem: '4096 MB',
        freemem: '3160 MB'
      },
      gpio: {
        input: { pin: [11, 13], type: 'simulation' },
        output: { pin: [33, 35], type: 'rpi' }
      },
      channel: { name: [ 'voltage', 'gateway1', 'tcp' ] }
    }*/
  });  
});
```
## Create an Access Token for Browser Client
##### 1. Login to [node-m2m](https://www.node-m2m.com/m2m/account/login).
From the main dashboard, go to manage security section at the right side. Select create access token for browser client. You will receive an email link to get your access token.

##### 2. Install m2m.

Copy the minimized file `node-m2m.min.js` from `node_modules/m2m/dist` directory to your server's public javascript directory.

Include the minimized file `node-m2m.min.js` into your HTML file `<script src="YOUR_SCRIPT_PATH/node-m2m.min.js"></script>`. This will create a global **NodeM2M** object.

##### 3. Create a client object instance from NodeM2M global object.

You can now access the resources from your remote devices from the various methods available from the client instance. The API is similar to Node.js client applications as shown below.   

```js
<script>

// Protect your access token at all times  
var tkn = 'fce454138116159a6ad9a4234e7de810a1087fa9e7fbfda74503d9f52616fc5';

var client = new NodeM2M.Client();

client.connect(tkn, () => {

  // capture 'random-number' data using a pull method
  client.getData({id:100, channel:'random-number'}, (data) => {
    console.log('getData random-number', data); // 97
  });

  // capture 'random-number' data using a push method
  client.watch({id:100, channel:'random-number'}, (data) => {
    console.log('watch random-number', data); // 81, 68, 115 ...
  });

  // update test-data
  client.sendData({id:100, channel:'test-data', payload:'node-m2m is awesome'}, (data) => {
    console.log('sendData test-data', data);
  });

  // capture updated test-data
  client.getData({id:100, channel:'test-data'}, (data) => {
    console.log('getData test-data', data); // node-m2m is awesome
  });

});

</script>
```

## Connecting to other m2m server
### You can connect to a different server by providing the url of the server you want to use
By default without a url argument, the connect method will use the 'https://www.node-m2m.com' server
```js
const m2m = require('m2m');

const client = new m2m.Client();

client.connect(() => {
  ...
});
```
To connect to a different server, provide the url of the new server

e.g. using the 'https://www.my-m2m-server.com' as the url of the new server
```js
const m2m = require('m2m');

const client = new m2m.Client();

client.connect('https://www.my-m2m-server.com', () => {
  ...
});
```
