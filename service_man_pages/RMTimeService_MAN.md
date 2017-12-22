Copyright 2017 Dell, Inc. All rights reserved

# RMTimeService -- RackManager Time Service

---

## About
The ***Rackmanager Time Service*** is a systemd integrated service that keeps the DSS9000 MC's local time in sync with the RackManager's local time.

***Rackmanager Time Service*** specific logs can be found on the RackManager: `/var/log/rackmanager/RMTimeService.log`

### Description
* On start up, ***Rackmanager Time Service*** will first try to establish communications with the DSS9000 MC.
* In the event that the RackManager has lost persistance of its local time (possibly following a poweroff), ***Rackmanager Time Service*** will then re-initialize the RackManager's local time with that of the DSS9000 MC's local time.
* After this setup, a periodic sync will begin at a default rate of 15 minutes.
  * This will sync the local time of the RackManager to the DSS9000 MC, so that the MC's local time stays valid over time.
* ***Rackmanager Time Service*** will log all events.

### Current Limitation:
* ***Rackmanager Time Service*** does not sync or even check timezone's, but the timezone on the DSS9000 MC is not generally visable.

## Usage
* `systemctl {restart, start, stop, status} RMTimeService`
