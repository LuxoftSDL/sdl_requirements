## The triggers of PTU


1. 
The PoliciesManager must request an update to its local policy table when an `appID` of a registered app is not listed on the Local Policy Table.

2. 
PoliciesManager must check the stored status of PTUpdate upon every Ign_On and in case it is UPDATE_NEEDED

PoliciesManager must 

initiate the PTUpdate sequence by sending SDL.PolicyUpdate request to HMI right after the first application has registered on SDL

3. 
PoliciesManager must start a PolicyTable Update sequence 

IN CASE the current date is 24 hours prior to module's certificate expiration date

_Information:_  

a. The triggers for checking the cert expiration status are:  
-> ignition on  
-> TLS handshake  
b. in case module's certificate in policies is expired or invalid, the TLS handshake will fail  
c. in case TLS handshake fails because module's certificate is expired or invalid, the `count_of_TLS_errors` must not be incremented  
d. current model of backend implementation: `certificate` is always present in PTU (meaning, each successfull request for Policies Update would bring a certificate in updated PT)

4. 
	
In case

SDL is built with any value of "DEXTENDED_POLICY" flag or without this flag at all

and the amount of ignition cycles notified by HMI via BasicCommunication.OnIgnitionCycleOver gets equal to the value of `exchange_after_x_ignition_cycles` field (`module_config` section) of policies database

SDL must

trigger a PolicyTableUpdate sequence (and the flow will depend on the flag)

5. 
	
In case

SDL is built with any value of "DEXTENDED_POLICY" flag or without this flag at all

and subscribed via SubscribeVehicleData(odometer) right after ignition on  
SDL gets OnVehcileData (`odometer`) notification from HMI
and the difference between current `odometer value_2` and `odometer value_1` when the previous UpdatedPollicyTable was applied is equal or greater than to the value of `exchange_after_x_kilometers` field (`module_config` section) of policies database

SDL must

trigger a PolicyTableUpdate sequence 

6. 
In case

SDL is built with any value of "DEXTENDED_POLICY" flag or without this flag at all

and the difference between current `system time value_2` and `system time value_1` when the previous UpdatedPollicyTable was applied is equal or greater than to the value of `exchange_after_x_days` field (`module_config` section) of policies database

SDL must

trigger a PolicyTableUpdate sequence

7. 
In case

there is no `certificate` at `module_config` section at LocalPT

SDL must

trigger a PolicyTableUpdate sequence on sending SDL.OnStatusUpdate(UPDATE_NEEDED) to HMI to get `certificate`
