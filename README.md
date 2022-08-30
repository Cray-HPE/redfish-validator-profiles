# Redfish Validation Profiles

## About
The Shasta software stack requires certain properties of a devices's Redfish
service to exist so the device can be properly managed. To assist in determining
compatibility, a set of profiles have been created for this purpose. These
profiles are conformant to the schematics specified by the DMTF Redfish Profile
schema and are used with the [Redfish Interop
Validator](https://github.com/DMTF/Redfish-Interop-Validator).

## Pre-requisites

* python3
* pip3
* Power capping is enabled on the device, if applicable

## Installation for use with the Redfish Interop Validator
1. Create and activate python3 virtual environment.
   ```
   # python3 -m venv riv
   # cd riv
   # . bin/activate
   ```
2. Clone validator and profile repos.
   ```
   # git clone git@github.com:DMTF/Redfish-Interop-Validator.git
   # git clone git@github.com:Cray-HPE/redfish-validator-profiles.git
   ```
   or
   ```
   # git clone https://github.com/DMTF/Redfish-Interop-Validator.git
   # git clone https://github.com/Cray-HPE/redfish-validator-profiles.git
   ```
3. Set up local directories.
   ```
   # cd Redfish-Interop-Validator
   # mkdir -p config
   # mkdir -p logs
   ```
4. Install local requirements.
   ```
   # pip3 install -r requirements.txt
   ```
5. Create configuration ini file.
   ```
   # cat > config/CSM.ini << EOF
   [Tool]
   Version = 2
   Copyright = Redfish DMTF (c) 2021
   verbose = 0

   [Host]
   username = root
   description = "Redfish Endpoint"
   forceauth = False
   authtype = Session

   [Validator]
   payload = Tree /redfish/v1/
   logdir = ./logs
   oemcheck = False
   debugging = False
   EOF
   ```
## Execute the Redfish Interop Validator
1. Set environment variables.
   ```
   # read -s PASSWD
   <enter redfish root password>
   # ENDPOINT=<hostname or IP>
   ```
1. Select which profile to use
   * Standard node
      ```
      # PROFILE="../redfish-validator-profiles/profiles/CSMRedfishProfile.v1_2_0.json"
      ```
   * Node with GPUs
      ```
      # PROFILE="../redfish-validator-profiles/profiles/CSMRedfishProfile-GPU.v1_0_0.json"
      ```
      * This profile will indicate a ProcessorType failure with non-GPUs. These
      errors can safely be ignored. 
2. Execute the Redfish Interop Validator.
   ```
   # python3 RedfishInteropValidator.py -c config/CSM.ini -i https://$ENDPOINT $PROFILE -p $PASSWD
   ```
   Execution time is hardware dependent
3. A summary is displayed at the end of the execution. A **.txt** and **.html**
file are also created in the **logs** directory for further analysis.
