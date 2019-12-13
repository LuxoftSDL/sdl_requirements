## Functional requirements

### General info
1. 
SDL must

notify HMI via SDL.OnStatusUpdate() notification  

right after one of the statuses of `UPDATING`, `UPDATE_NEEDED` and `UP_TO_DATE` is changed from one to another.

2. 

In case  

any PolicyTableUpdate trigger happens (_see also [PTU triggers](./Policies/PTU/PTU_Triggers.md)_)

SDL must

- SDL.OnStatusUpdate(UPDATE_NEEDED) to HMI
- create PTSnaphot

3. 

In case  
SDL is built with PROPRIETARY or EXTERNAL_PROPRIETARY flag 

SDL sent BC.PolicyUpdate(path_to_PTS)

and **HMI responds** BC.PolicyUpdate(SUCCESS)

SDL must

send SDL.OnStatusUpdate(UPDATING) to HMI

3.1

In case  

SDL is built with HTTP flag

Upon creating PTSnaphot

SDL must

- send SDL.OnStatusUpdate(UPDATING) to HMI
- send OnSystemRequest(PTS_in_binary) to mob app

4. 

In case

SDL is built with PROPRIETARY or EXTERNAL_PROPRIETARY flag 

PTU passes successfully 

and **HMI sends** valid updated PT via SDL.OnReceivedPolicyUpdate()

SDL must
- merge PTU into Local PT
- send SDL.OnStatusUpdate(UP_TO_DATE) to HMI

4.1

In case
 
SDL is built with HTTP flag

PTU passes successfully 

and **mob app sends** valid updated PT via SystemRequest(PTU_in_binary_data)

SDL must
- merge PTU into Local PT
- send SDL.OnStatusUpdate(UP_TO_DATE) to HMI

### Statuses update during/after PTU retry sequence with multiple apps

#### Status after unsuccessful retry sequence
5. 

In case 

an app (App_1) registers in the first ign cycle from consented device (-> PTU is triggered)

and PTU retry sequence is finished unsuccessfully with UPDATE_NEEDED status

and another app (App_2) registers (-> new PTU trigger)

SDL must

- send SDL.OnStatusUpdate(UPDATE_NEEDED) to HMI
- create PTSnaphot and run PTU
- send SDL.OnStatusUpdate(UP_TO_DATE) to HMI once PTU is successfully completed

#### Status when 2nd PTU is started after unsuccessful retry
6. 

In case 
 
an app (App_1) registers in the first ign cycle from consented device, 

PTU retry sequence is in progress

and a new application (App_2) registers during the retry sequence

SDL must

 - send SDL.OnStatusUpdate(UPDATE_NEEDED, UPDATING) to HMI
 - send request* to start the new PTU sequence after retry

 _Info: 
 a. * if SDL is built with PROPRIETARY or EXTERNAL_PROPRIETARY flag -> SDL sends BC.PolicyUpdate to HMI  
 b. * if SDL is built with HTTP flag -> SDL sends OnSystemRequest to mob app_

## Non-functional requirements
HMI API

```
    <function name="OnStatusUpdate" messagetype="notification">
      <description>Notification from SDL to HMI when current status of PT exchange changed (i.e. it Succeded or Failed etc)</description>
      <param name="status" type="Common.UpdateResult" mandatory="true" />
    </function>
```

## Diagram
[OnStatusUpdate](./accessories/OnStatusUpdate_in_Proprietary_PTU_flow.png)
