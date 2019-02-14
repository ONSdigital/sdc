# Monitoring Performance - What should we monitor?
## Introduction
This spike report follows on from work and discussions in the OAT meeting to identify areas where we want to be monitoring the system. This outlines what I see as an idealised monitoring setup - there may be blockers to overcome to achieve some of this. I've also provided guidance in the document to how I see the monitoring being provided and used.
## Production metrics
All of these metrics should be radiated on monitors
### Performance
* **99% call time** - This can be pulled from Google Analytics, CloudWatch or Zipkin headers in Splunk logs, potentially further broken down by microservice
* **Current concurrent users** - This can be pulled most easily from Google Analytics
* **Peak concurrent users** - As above, but the maximum for the last 24h/72h/7d
### Physical
* **Diego cell usage** - This can be pulled from Cloudwatch. It's a candidate for alerting into Slack (and does currently alert), but note that Diego Cell balancing can often be poor (consider alerting if a significant number of DCs are maxed or the average load exceeds x%) and alerts are currently somewhat ignored due to volume.
* **Database CPU%** - This can be pulled from Cloudwatch or by passing DB physical metrics into Splunk. This is a very good candidate for alerting into Slack, as going above 80% is usually a precursor to a collapse in system performance
### Platform 
