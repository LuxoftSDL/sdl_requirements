### PROPRIETARY & HTTP

#### Registered app is not listed on the LPT
1. 
In case

SDL is built with "DEXTENDED_POLICY:PROPRIETARY" or "DEXTENDED_POLICY:HTTP" flag

The PoliciesManager must 

request an update to its Local Policy Table(LPT) when an `appID` of a registered app is not listed on the LPT.

#### The stored status of PTU is `UPDATE_NEEDED`
2. 
In case

upon any Ign_On the stored status of PTUpdate is `UPDATE_NEEDED`

PoliciesManager must 

a. PROPORIETARY FLOW: initiate the PTUpdate sequence by sending SDL.PolicyUpdate request to HMI right after the first application has registered on SDL;
b. HTTP FLOW: initiate the PTUpdate sequence by sending OnSystemRequest(url, PTS_in_binaryData) to mobile app

#### Absent/invalid `certificate`
3.  
In case

SDL is built with "DEXTENDED_POLICY:PROPRIETARY" flag

a secure service is starting [app sends StartService (any_serviceType, encypted=true)]

there is NO `certificate` at `module_config` section at LocalPT or the certificate is invalid

SDL must

trigger a PolicyTableUpdate sequence on sending SDL.OnStatusUpdate(UPDATE_NEEDED) to HMI to get `certificate`

Note: In case `certificate` exists at `module_config` section -> SDL must NOT trigger PTU but start  TLS/DTLSHandshake (_see also [GetSystemTime](https://github.ford.com/SmartDeviceLinkMirror/sdl_requirements/blob/develop/detailed_docs/SDL-HMI_API/GetSystemTime/GetSystemTime_TRS.md)_)

#### The current date is 24 hours prior to module's certificate expiration
4. 
In case

SDL is built with any value of "DEXTENDED_POLICY" flag or without this flag at all  

and the current date is 24 hours prior to module's certificate expiration date

PoliciesManager must 

trigger a PolicyTable Update sequence

_Information:_  

a. The triggers for checking the cert expiration status are:  
-> ignition on  
-> TLS handshake  
b. in case module's certificate in policies is expired or invalid, the TLS handshake will fail  
c. in case TLS handshake fails because module's certificate is expired or invalid, the `count_of_TLS_errors` must not be incremented  
d. current model of backend implementation: `certificate` is always present in PTU (meaning, each successfull request for Policies Update would bring a certificate in updated PT)

#### ignition cycles
5. 
	
In case

SDL is built with any value of "DEXTENDED_POLICY" flag or without this flag at all

and the amount of ignition cycles notified by HMI via BasicCommunication.OnIgnitionCycleOver gets equal to the value of `exchange_after_x_ignition_cycles` field (`module_config` section) of policies database

SDL must

trigger a PolicyTableUpdate sequence (and the flow will depend on the flag)

#### odometer
6. 
	
In case

SDL is built with any value of "DEXTENDED_POLICY" flag or without this flag at all

and right after ignition on SDL gets OnVehcileData (`odometer`) notification from HMI  
and the difference between current `odometer value_2` and `odometer value_1` when the previous UpdatedPollicyTable was applied is equal or greater than to the value of `exchange_after_x_kilometers` field (`module_config` section) of policies database

SDL must

trigger a PolicyTableUpdate sequence 

#### system time
7. 
In case

SDL is built with any value of "DEXTENDED_POLICY" flag or without this flag at all

and the difference between current `system time value_2` and `system time value_1` when the previous UpdatedPollicyTable was applied is equal or greater than to the value of `exchange_after_x_days` field (`module_config` section) of policies database

SDL must

trigger a PolicyTableUpdate sequence


### EXTERNAL_PROPRIETARY

#### device consent 
1. 
In case

SDL is built with "DEXTENDED_POLICY:EXTERNAL_PROPRIETARY" flag

 `appID` of a registered app is not listed in the LPT

and the device the application is running on is **consented** (_see also **User-consent in terms of Policies (EXTERNAL_PROPRIETARY flow**_)

PoliciesManager must 

request an update to its Local Policy Table 

2. 18965

In case  

the app that has never received the updated policies registers from non-consented device

PoliciesManager must 

initiate the 'device consent prompt' sequence on HMI

3. 18966

In case  

the app that has never received the updated policies registers from non-consented device

and then the User consents this device

PoliciesManager must 

trigger a PolicyTableUpdate sequence

#### User requests PTU

4. 
In case

SDL is built with "DEXTENDED_POLICY:EXTERNAL_PROPRIETARY" flag

user requests PTU (e.g. from Settings Menu on HMI)

and HMI sends SDL.UpdateSDL

PoliciesManager must 
- respond SDL.OnStatusUpdate() to HMI 
- process a PolicyTableUpdate sequence

5. 20831

In case

PTU was triggered by user  

and SDL generates PoliciesSnapshot  
and no consented devices OR one and/or more un-consented devices with registered apps connected were found

SDL must 

- log the corresponding error internally
- NOT send the PoliciesSnapshot over OnSystemRequest to any of the apps