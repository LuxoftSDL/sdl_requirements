## Functional requirements

### Subscription for RPC spec and custom vehicle data after PTU with VehicleDataItems
1. 
In case
preloaded file contains VehicleDataItems for all RPC spec VehicleData (allowed by Policies)
and app is registered and activated

and PTU is performed with VehicleDataItems in update file (`VehicleData` from `VehicleDataItems` is defined in parameters of `functional group` for the application)

and app sends SubscribeVehicleData request

SDL must
- transfer SubscribeVehicleData request (with `VD_name` for RPC spec data and with `VD_key` for custom data) from mobile app to HMI
- convert HMI response (`VD_keys` to -> `VD_names`)
- send the response with SubscribeVehicleData(VD_name) to mobile app
- transfer OnVehicleData notification with subscribed data from HMI to mobile app


### Unsubscription from RPC spec and custom data after PTU with VehicleDataItems
2. 
In case
preloaded file contains VehicleDataItems for all RPC spec VehicleData (allowed by Policies)
and app is registered and activated

and PTU is performed with VehicleDataItems in update file (`VehicleData` from `VehicleDataItems` is defined in parameters of `functional group` for the application)

and app sends UnsubscribeVehicleData request

SDL must

- transfer UnsubscribeVehicleData request (with `VD_name` for RPC spec data and with `VD_key` for custom data) from mobile app to HMI
- convert HMI response (`VD_keys` to -> `VD_names`)
- send the response with SubscribeVehicleData(VD_name) to mobile app
- NOT transfer OnVehicleData notification with subscribed data from HMI to the mobile app


### Policy prohibition for VehicleDataItems in case VD is not allowed by Policies
3. 
In case
preloaded file contains VehicleDataItems for all RPC spec VehicleData (allowed by Policies)
and app is registered and activated

and PTU is performed with custom VD items 
and custom VD is not included in 'parameters' of `functional group`

SDL must 

respond with DISALLOWED resultCode to SubscribeVD/GetVD/UnsubscribeVD app requests

### Processing PTU with unknown types of custom VD parameters
4. 
In case
preloaded file contains VehicleDataItems for all RPC spec VehicleData (allowed by Policies)
and app is registered and activated

and PTU is performed and contains the custom VD items with an unknown enum data type of parameter

SDL must
- successfully processes mobile app requests with RPC spec data
- successfully processes mobile app requests with custom data from PTU allowing arbitrary string values for the modified parameter

### Processing second PTU with empty vehicle_data
5. 
In case
preloaded file contains contains VehicleDataItems for all RPC spec VD
PTU1 is successfully performed with VehicleDataItems in update file
PTU2 is triggered from HMI and PTU2 is performed with empty `vehicle_data` structure in update file

and mobile app sends a request with custom data from PTU1

SDL must

reject request with INVALID_DATA resultCode