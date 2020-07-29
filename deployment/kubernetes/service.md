# Service

Each Pod has a unique IP address - but the address is ephemeral.  The Pod IP addresses are not stable and it can change when Pods start and/or restart. Moreover, if you have a Deployment that starts multiple Pods, and you need to consume an API from the Pods, you do not want to connect using an ephemeral IP address either.

A Service provides a single access point to a set of pods matching some constraints. A Service IP address is stable.



