### Generic triggers
Generic triggers listed below are applicable for all policies flags

#### Registered app is not listed on the LPT
1. 
In case

an `appID` of a registered app is not listed on the LPT

SDL must 

request an update to its Local Policy Table(LPT)

#### The stored status of PTU is `UPDATE_NEEDED`
2. 
In case

upon any Ign_On the stored status of PTUpdate is `UPDATE_NEEDED`

SDL must 

trigger a PolicyTableUpdate sequence

#### Absent/invalid `certificate`
3.  
In case

a secure service is starting [app sends StartService (any_serviceType, encypted=true)]

there is NO `certificate` on a file system or the certificate is invalid

SDL must

trigger a PolicyTableUpdate sequence on sending SDL.OnStatusUpdate(UPDATE_NEEDED) to HMI to get `certificate`

Note: In case `certificate` exists on a file system (files location are defined by `CertificatePath` and `KeyPath` parameters in .INI file) -> SDL must NOT trigger PTU but start TLS/DTLSHandshake (_see also [GetSystemTime](https://github.ford.com/SmartDeviceLinkMirror/sdl_requirements/blob/develop/detailed_docs/SDL-HMI_API/GetSystemTime/GetSystemTime_TRS.md)_)

#### The current date is 24 hours prior to module's certificate expiration
4. 
In case

the current date is 24 hours prior to module's certificate expiration date

SDL must 

trigger a PolicyTable Update sequence

_Information:_  

a. The triggers for checking the cert expiration status are:  
-> ignition on  
-> TLS handshake  
b. current model of backend implementation: `certificate` is always present in PTU (meaning, each successfull request for Policies Update would bring a certificate in updated PT)

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


### EXTERNAL_PROPRIETARY Specific

#### Registered app is not listed on the LPT - EXTERNAL_PROPRIETARY - new app won't be started until device is consented
1. 
In case

SDL is built with "DEXTENDED_POLICY:EXTERNAL_PROPRIETARY" flag

 `appID` of a registered app is not listed in the LPT

and the device the application is running on is **consented** (_see also **User-consent in terms of Policies (EXTERNAL_PROPRIETARY flow**_)

SDL must 

request an update to its Local Policy Table 

2. 

In case  

the app that has never received the updated policies registers from non-consented device

SDL must 

initiate the 'device consent prompt' sequence on HMI

3. 

In case  

the app that has never received the updated policies registers from non-consented device

and then the User consents this device

SDL must 

trigger a PolicyTableUpdate sequence

#### Manual PTU start from HMI

4. 
In case

SDL is built with "DEXTENDED_POLICY:EXTERNAL_PROPRIETARY" flag

user requests PTU (e.g. from Settings Menu on HMI)

and HMI sends SDL.UpdateSDL

SDL must 
- respond SDL.OnStatusUpdate() to HMI 
- process a PolicyTableUpdate sequence

5. 

In case

PTU was triggered by user  

and SDL generates PoliciesSnapshot  
and no consented devices OR one and/or more un-consented devices with registered apps connected were found

SDL must 

- log the corresponding error internally
- NOT send the PoliciesSnapshot over OnSystemRequest to any of the apps
