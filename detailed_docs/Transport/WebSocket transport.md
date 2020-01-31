## WebSocket transport implementation of SDL Core

1. 
SDL must 

support a WebSocket Server as a transport for running local web applications

_Info:_

a. A web app is a single page application developed with web technology, containing at least 
- sdl.js file  
- manifest.js file - must be included in the HTML file as a **script source**
- an HTML file

b. Web applications and services can be installed directly into the vehicle. These applications should be independent from a mobile phone.  
They connect to the locally running SDL Core and use the in-vehicle modem to communicate to the internet. 

c. Web apps must be run in a WebEngine (or Browsers) and connect to SDL using WebSockets

d. The WebEngine should be on the same host as SDL (in the infotainment system)


2. 
WebSocket server must 

- be permanently available while SDL Core is operating 
- listen to a port specified in the smartDeviceLink.ini file ()

Info:
Another ini configuration should allow binding the socket to the localloop address or any address.

3. 
WebSocket server transport must be 

available as a build configuration called BUILD_WEBSOCKET_SERVER_SUPPORT (definition -DWEBSOCKET_SERVER_TRANSPORT_SUPPORT)  
so that SDL Core can be compiled with or without this additional transport type.

4. 
For the WebSocket transport

the transport role would be WebSocket Server (ws-server) or WebSocket Secure Server (wss-server) where

* sdl-host points to the host that runs SDL Core (in production that would be the IVI system so local host) and 

* sdl-port is the port number that SDL Core bound the WebSocket to.

Example: file://somewhere/HelloSDL/index.html?sdl-host=localhost&sdl-port=12345&sdl-transport-role=wss-server


## WebSocket transport security
5. 
In case
`CertificatePath`, `KeyPath`, `CACertificatePath` params are present ini file settings 

SDL must
start the WebSocket Secure Server

6. 
In case
`CertificatePath`, `KeyPath`, `CACertificatePath` params are absent in ini file settings   
(i.e. certificate and and private key are not present)

SDL must
start the WebSocket Server

7.  
In case
some of security params (but not all) are present ini file settings

SDL must

NOT start ws/wss server at all

### Motivation 
Using the in-vehicle infotainment system as a platform and runtime for SDL applications.

In-vehicle application store (OEM store) provides applications and services to the user.  
The user should be able to install or uninstall apps from the store.  
5. If a user installs an app, the OEM store should download the app package from the OEM backend and decompress/install the app to the system.  
6.	After installation, the OEM store should make the app visible and available on the HMI.
7.	Core should support a WebSocket Server as a transport.
8.	If a user activates a local app through the HMI, the HMI should launch the app by opening the entrypoint HTML file.
9.	HMI should launch the app including SDL Core's hostname and port as GET parameters.
10.	The app should connect to Core using the SDL library using hostname and port specified.
11.	If Core sends UpdateAppList, the HMI should compare matching SDL app IDs and avoid showing an app twice. If SetAppProperties added a "new" app with enabled=true then yes. Also if SetAppProperties "modified" an existing app from enabled=false to enabled=true.
I think, upon SDL start, 
Core should also send UpdateAppList for every app which is enabled = true.








### Resumption
param `isCloudApp=false`

sdl ТРЕБУЕТ concent для web engine apps

