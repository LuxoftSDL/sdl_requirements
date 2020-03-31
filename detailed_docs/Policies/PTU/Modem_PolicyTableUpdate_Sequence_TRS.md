## Functional Requirements

### PTU via in-vehicle modem - No registered apps

1. 
In case

SDL is build with PROPRIETARY or EXTERNAL_PROPRIETARY flag

there are no registered apps

and PTU is triggered (_see also triggers_)

SDL must

- send BC.PolicyUpdate (path to PTS) request to the HMI enable it to use the HMI vehicle modem to handle the PTU
- send SDL.OnStatusUpdate(UPDATE_NEEDED) notification to HMI

### HMI chooses to process PTU via in-vehicle modem - Happy path

2. 
In case 

SDL is build with PROPRIETARY or EXTERNAL_PROPRIETARY flag

an app is registred with SDL

and PTU was triggered

and SDL sent PTSnapshot to HMI

HMI is expected to 
- encrypt the PTS
- send the encrypted PTS file to the defined `url` using either SDL and mob app or in-vehicle modem

_HMI Note: In case HMI choses to use the modem, it should not use the OnSystemRequest itself and route the request through other channels to modem._

3. 
In case

an app is registred with SDL and PTU was triggered

and SDL sent PTS to HMI and started the timeout waiting for the PTU

and HMI chooses to process PTU using in-vehicle modem to reach Policy Server 
 
and sends SDL.OnReceivedPolicyUpdate (decrypted `policyFile`) to SDL within the timeout

SDL must

- stop `timeout_after_x_seconds` (value taken from Local PT)
- validate the received PTU and merge it
- send SDL.OnStatusUpdate (`UP_TO_DATE`) to HMI

### PTU via in-vehicle modem - Backend isn't reached by the modem

4. 
In case

SDL sent PTS to HMI and started the timeout waiting for the PTU

HMI processes PTU using in-vehicle modem to reach Policy Server and transferring PTU goes wrong by some reason 

and HMI sends BC.OnSystemRequest(`requestType=PROPRIETARY`,`url`,`appID`,`encryptedFile`) within the timeout

SDL must

- restart `timeout_after_x_seconds` (value taken from Local PT)
- send OnSystemRequest to mob app
- transfer the received SystemRequest response from the mob app
- stop `timeout_after_x_seconds` upon receiving SDL.OnReceivedPolicyUpdate (decrypted `policyFile`) from HMI

## Diagrams
### PTU using in-vehicle modem 

![PTU_Proprietary_modem_success](../accessories/PTU_PROPRIETARY_modem_success.png)

![PTU_Proprietary_modem_fail](../accessories/PTU_PROPRIETARY_modem_failure.png)
