---
title: Logitech Teams Rooms Repair
author: Gudszent Otto
date: 2020-09-01 14:10:00 +0800
categories: [O365, Teams]
tags: [O365, Teams]
---

# Source: Logitech Support

1. Running the following as detailed resolves the vast majority of customers that have this type of issue with Teams Room:

    <https://support.microsoft.com/en-us/help/4565998/teams-rooms-application-does-not-start-after-update>

2. If the above does not resolve the issue then here is the link to the Teams Room Recovery Tool (run it in Repair Mode):

    Run the recovery script in repair mode (instructions at <https://docs.microsoft.com/en-us/microsoftteams/rooms/recovery-tool> )

3. If this does not resolve the Teams Room Auto Launch and functionality issue you may have to re-image the PC if this is necessary you will need to provide a photo of the bottom of the Intel NUC PC showing the AIS model label.

4. If the above does resolve the auto launch issues you should also apply the following to the NUC PC:

On the NUC PC perform the following steps: (We recommend that the NUC power settings be set to always On never shutdown due to Tap Screen going to sleep and not waking due to Intel Power management of the internal motherboard USB/HDMI ports.

A. Boot into the Windows Admin account and change the Power setting (see attached) Note the power settings may be different depending on the version of Windows 10 your are using. The Intel Motherboard has power save options enabled that reduce power to sections of the circuits which cause issues with the displays and USB connections. So all power save options must be disabled.

In addition to the power setting changes on the attached document Microsoft has recently recommended going into the Device Manager and  then disable the "turn this device off to save power" feature on all USB Root Hubs on a given device. Select the USB Controllers - Root Hub device then right click then select "Properties" then select the "Power management" tab and un-check " Allow computer to turn this device off to save power".

Boot into Windows:
To boot into Windows connect a Keyboard to the NUC then reboot it while it is coming up tap the Windows Key over and over until the Windows Login comes up on the screen. The select the Teams Admin account - password is sfb once completed with changes logout then login to the User account same password.

B.. Install the intel driver and support assistant (Intel DSA) in the Windows Admin account and run the Intel Device Driver Assistant to ensure that the NUC is running the most updated versions of all required drivers.

Link - <https://www.intel.com/content/www/us/en/support/detect.html>

C. Update the NUC HDMI driver: (Alleviates Teams Display Related issues)


<https://downloadcenter.intel.com/download/29472/HDMI-Firmware-Update-Tool-for-NUC8i3BE-NUC8i5BE-NUC8i7BE?v=t>

Note - Please ensure ALL instructions for followed when updating the HDMI drivers. Specifically, an HDMI device must be connected to the NUC HDMI port and powered on. These instructions can be found in the extracted driver folder or at this link.
