# Setup UniFi Network

## Devices

- [UniFi Security Gateway (USG)](https://www.ui.com/unifi-routing/usg/)
- [UniFi Switch US-8-60W](https://www.ui.com/unifi-switching/unifi-switch-8/)
- [UniFi AP AC Pro](https://www.ui.com/unifi/unifi-ap-ac-pro/)
- UniFi Controller installed on Raspberry Pi 4
- CenturyLink Fiber Gigabit Optical Network Terminal (ONT)
- Computer used to setup the network

## Setup UniFi Controller on Raspberry Pi 4

## Connect the devices

Connect the devices according to Figure XXX. The status LED of all the UniFi
devices should turn solid white, which means that the devices are ready for
adoption.

## Initial configuration of the USG

At this point, the computer used to configure the network may not have access
to internet, at least if CenturyLink is the internet provider.

1. Go to the USG web interface: https://192.168.1.1
2. By pass the browser warning about `Potential Security Risk Ahead` (FireFox)
   by clicking on `Advanced...` and `Accept the Risk and Continue`
3. Configure PPPoe under `Settings` > `Configuration`
    - Connection Type: `PPPoE`
    - Username: `<username from CenturyLink>`
    - Password: `<password from CenturyLink>`
    - Preferred DNS: `8.8.8.8` (Google Public DNS)
    - Alternate DNS: `8.8.4.4` (Google Public DNS)
    - Use VLAN ID: `<VLAN ID from CenturyLink>`
4. Change the subnet (optional). It is recommended to change the subnet instead
   of using the default one for increased security. Under `LAN Settings`:
   - IP Address: `192.168.<subnet ID>.1`
   - DHCP Range: `192.168.<subnet ID>.6` to `192.168.<subnet ID>.254`
5. Click on `Apply Changes`

Upon changing the subnet, this message should appear:

> The LAN IP has been changed. You may need to reopen this application at the
new IP. You may also need to release/renew DHCP on your client machine.

After 1-2 minutes, navigate to the address `192.168.<subnet ID>.1` and you
should be back to the USG web interface. From now on, the instructions will
be given for the default subnet `192.168.1.1` to be more concise. If the subnet
has been changed, update the instructions accordingly.

The USG web interface should list the following devices and provide their IP
addresses.

- USG
- UniFi controller
- UniFi AP
- Computer

The message `Congratulations! The Gateway is connected to the internet.` should
now be visible at the top of the page. The USG should now have access to
internet. To check that it is the case, ssh to the the USG (`ssh ubnt@192.168.1.1`,
password: `ubnt`) and run the command `ping 1.1.1.1`.

At this point of the setup, the computer and UniFi controller never had access
to internet, even after trying to reconnect them and renewing the DHCP lease.
The solution suggested on the UniFi forum is to move on to configuring the UniFi
controller and set PPPoE using the controller dashboard.

## Initial configuration of the UniFi controller

Before moving on to the web interface of the controller, set the field `Inform URL`
in the USG dashboard to the value `http://<controller ip address>:8080/inform`
and click on `Save Changes`. Leave the dialog titled `Waiting for adoption`
open. Open a new tab in the browser and navigate to the address of the controller
web interface.

1. Go to the web interface of the controller: `https://<ip address>:8443`
2. On the setup page 1:
    - Controller Name: `unifi-controller`
    - Check the checkbox and click on `Next`
3. On the setup page 2: We are given the choice to login with an Ubiquiti account
or with a local account. The later is considered more secure. Also the controller
may not have access to internet at this point of the setup (see above) so creating
a local account would be the only solution. If internet is available, the Ubiquiti
option can be selected if you intend to access your controller remotely.
    - Click on `Switch to Advanced Setup`
    - Disable `Enable Remote Access` and `Use your Ubiquiti account for local access`.
    - Fill in the form to create a local account
    - Click on `Next`
4. On the setup page 3:
    - Leave both options turned on (`Automatically optimize my network` and
    `Enable Auto Backup`).
    - Click on `Next`
5. On the setup page 4:
    - The three UniFi devices should be listed (USG, Switch and AP)
    - Do not select any devices
    - Click on `Next`
6. On page Step 5 of 6:
    - Set the wifi name and password
    - Leave the option `Combine 2 GHz and 5 GHz WiFi Network Names into one`
    unchecked (preferred)
    - Click on `Next`
7. On page Step 6 of 6:
    - Verify the information and click on `Finish`

After a brief loading screen, you should now be presented with the controller
dashboard.

## Adopt devices

Navigate to the page `Devices`. This page should list the three UniFi devices
with the status `Pending Adoption`. We will first adopt the USG, then the Switch
and finally the AP.

### Adopt the USG

1. Click on the USG item to open a side menu on the right of the page.
2. Click on `Adopt`
3. Go back to the USG web interface and close the open dialog by clicking on
   the button `Confirm`.
4. In the controller dashboard, the status of the USG should now be set to
   `Provisioning`. After a couple of minutes, the USG should be adopted its
   status set to `Connected`.

The principal LED of the USG should have turned blue upon successful adoption.

### Get access to internet

All the devices can get access to internet now that the controller has adopted
the USB.

- Computer: Simply turn disable/enable the Ethernet connection
- AP: Power off/on
- Controller: Ssh to it then `sudo reboot`

IMPORTANT: Never power off/on the controller by removing the power cable,
otherwise the controller configuration may get corrupted! This is why a systemd
service has been created on the Raspberry Pi to automatically and gracefully
stop the Docker container `unifi-controller` when shutting down the Pi.

### Silence "Controller software update x.y.z is now available."

Now that the controller has access to internet, its dashboard may show a message
like `Controller software update x.y.z is now available.`. From the message options,
it's preferrable to silence this type of message. Instead, you should update the
Docker image used to run the controller on the Raspberry Pi regularly.

### Adopt the Switch

Click on the Switch item, then click on `Adopt`. The status should change to
`Adopting`, then

`Disconnected`


Status:      Unknown[11] (http://172.17.0.2:8080/inform)



UBNT-US.v4.3.20# set-inform http://192.168.XXX.9:8080/inform

Adoption request sent to 'http://192.168.1.9:8080/inform'.  Use the controller to complete the adopt process.




USGL Forget > device busy > power off > wait 1-2 min > forget > success > power on USG > controller says that USG is `Managed by Other`



## Manual update of the USG firmware

Ssh to the USG and run `upgrade https://dl.ui.com/unifi/firmware/UGW3/<version>.tar`.
The value of `<version>` can be found on XXX.

## FAQ

### The browser shows `Warning: Potential Security Risk Ahead`

In Firefox, click on the button `Advanced...`, then click on `Accept the Risk and Continue`.


set-inform http://192.168.XXX.10:8080/inform

Adopting from controller first, then setting inform on the device.

Reset USG, Reset Switch, restart USG

The LAN IP has been changed. You may need to reopen this application at the new IP. You may also need to release/renew DHCP on your client machine.

Set 27 on USB, save and all devices will get a new IP (but PC that needs renew lease?)