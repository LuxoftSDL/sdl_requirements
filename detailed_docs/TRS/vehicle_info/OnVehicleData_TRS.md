## Functional Requirements

1. 
In case mobile application has already subscribed on some vehicle data via SubscribeVehicleData_request  

and this subscribed data was updated  

SDL must:  

transfer OnVehicleData_notification (with updated vehicle data value) to subscribed mobile applications  

2.  
In case HMI sends invalid notification that SDL should transfer to mobile app  

SDL must:  

- log the issue  
- ignore this notification  


3. 
In case SDL received valid OnVehicleData notification from HMI

and this notification is NOT allowed by Policies

SDL must:

- log corresponding error internally
- ignore this notification and not transfer it to subscribed mobile applications  

4.  
In case SDL receives OnVehicleData notification from HMI  

and this notification is allowed by Policies for this mobile app  

and "parameters" field is **empty** at PolicyTable for OnVehicleData notification  

SDL must:
- log corresponding error internally
- NOT transfer this notification to mobile app

5.
In case SDL receives OnVehicleData notification from HMI  

and this notification is allowed by Policies for this mobile app  

and "parameters" field is **omitted** at PolicyTable for this notification  

SDL must:  
- transfer received notification with all parameters as is to mobile app  
- respond with `<received_resultCode_from_HMI>` to mobile app

6.  
In case SDL receives OnVehicleData notification from HMI 
and this notification is contains `shifted` item in `gps` parameter  

SDL must  
transfer received notification to mobile app with the same value of `shifted` item in `gps` parameter as those from HMI

## Non-Functional Requirements  
Additions to mobile API, HMI_API  
Changes to enums and functions should be applied  

1.
```
<function name="OnVehicleData" functionID="OnVehicleDataID" messagetype="notification">
            :
    <param name="fuelRange" type="FuelRange" minsize="0" maxsize="100" array="true" mandatory="false">
        <description>The estimate range in KM the vehicle can travel based on fuel level and consumption</description>
    </param>
</function>
```
```xml
<enum name="VehicleDataType">
            :
    <element name="VEHICLEDATA_ENGINEOILLIFE" />
</enum>
```
2.
```
<function name="OnVehicleData" functionID="OnVehicleDataID" messagetype="notification">
            :
    <param name="engineOilLife" type="Float" minvalue="0" maxvalue="100" mandatory="false">
        <description>The estimated percentage of remaining oil life of the engine.</description>
    </param>
</function>
```


## Diagram

OnVehicleData

![OnVehicleData](https://github.com/smartdevicelink/sdl_requirements/tree/feature/FuelRange/detailed_docs/accessories/OnVehicleData.png)


