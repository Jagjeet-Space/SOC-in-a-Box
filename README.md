# SOC-in-a-Box

## Project Vision
A comprehensive hands-on cybersecurity detection and response lab that simulates real-world enterprise security operations.

It consists of Debian-Slim docker for attacking Windows-VM to generate real adversary logs for analysing and testing detecion rules these log the forward with the help of FulentBit log collection software and send it to our host machine where we analyse logs through Elastic & Kibana. 

We are also hoping to add Wazuh to generating alerts and testing threat detection rules.

Here is Visual representation of Detection and Response SIEM Lab 









For installing this lab into your machine you have first download Attacking Simulaation architect where you run atomic red using debian-slim docker to simulate an attack on Windows-VM.
Download Debian-docker and snap-shot of VM to use into your VM 


Now for analysing and response you have to download Host docker files for runing Elastic & Kibana with Fluentd Aggregrator.


