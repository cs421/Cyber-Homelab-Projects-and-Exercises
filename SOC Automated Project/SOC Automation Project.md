**Goal:** The goal of this project is to create an automated SOC analyst lab with **Wazuh** as the SIEM tool, **TheHive** as the case management tool, and **Shuffle** for SOAR capabilities. This lab will also utilize **Mimikatz** as the sample credential harvesting attack.

There will be **two (2)** virtual machines to be used in this project, namely:
1. A Windows 10 Client which will serve as the Wazuh agent, to be configured via VirtualBox.
2. An Ubuntu machine which will serve as another Wazuh agent later on, to be configured on the cloud.

**Wazuh, TheHive, and Shuffle** will be configured on the cloud.

I will be following **MyDFIR's** videos on creating this lab, as found in: https://www.youtube.com/watch?v=Lb_ukgtYK_U

## Part 1: Diagram

![[SOC Automated Homelab Diagram.png]]

**1.** The client (**Windows 10 Wazuh Agent**) will send events to the internet