## Setting up Chirpstack.io LoRaWAN Nanogateway using LoPy and RPi3

LoRa Gateway in a LoRaWAN network is a device/hardware which connects an end sensing/client LoRa node to a wide network such as the internet or your local network. Though the full fledged LoRa gateways available on the market proivide advance features and capabilities, they are costly and and bulky. Today, I will share the steps and my experience in setting up a LoRa gateway using two palm-sized embedded systems - a LoPy and Raspberry Pi 3.

<p>The already available instructions on manufacturer site are designed to be used for The Things Network. This blog provide a minimum steps to setup a Chirpstack LoRaWAN server to which one have more control and is not connected to the internet.</p>

<p>The whole process has four major steps:</p>

<ol>
<li>Install Chirpstack LoRaWAN stack on RPi3.</li>
<li>Create a wireless access point on the RPi3.</li>
<li>Setup the LoPy as nanogateway and adding it to Chirpstack server.</li>
<li>Registering the Nanogateway and client LoPy node with Chirpstack server.</li>
</ol>

<p><em>Pre-requisites</em></p>

<ol>
<li>RPi3 with Raspbian installed and connected to a monitor. We need the screen at least during all the stages. However, once the whole setup is finished, you can use the system headless.</li>
<li>RPi3 connected to internet for update and downloading required packages.</li>
<li>Two LoPys with latest firmware. I am using 1.16.</li>
<li>Atom editor with pycom plugin to upload code to LoPy.</li>
</ol>

<p><em>Step-1</em></p>

<ol>
<li><p><code>sudo apt update</code> &amp; <code>sudo apt upgrade</code> &ndash; first update and then upgrade the Raspbian OS.</p></li>

<li><p>Follow the steps from <a href="https://www.chirpstack.io/guides/debian-ubuntu/" target="_blank">here</a> and install the chirpstack.</p></li>
</ol>

<p><em>Step-2</em></p>

<p>The RPi3 has an in-built WiFi chipset which can be converted into an access point to which other WiFi-enabled devices can connect (for e.g., LoPy). Follow the steps in this tutorial:</p>

<p><a href="https://www.raspberrypi.org/documentation/configuration/wireless/access-point.md" target="_blank">Setting up a Raspberry Pi as an access point in a standalone network (NAT)</a>. No need to follow the steps for sharing internet, we dont need it.</p>

<p><em>Step-3</em></p>

<p>I am using a LoPy4.0 with expansion board 3.0 updated to firmware 1.16. <em>Please update the LoPy before proceeding.</em> We will use the nano-gateway code provided by pycom. There are a total of 3 files - main.py, config.py, and nanogateway.py.</p>

<ol>
<li><p>We need a Gateway ID which is unique to each LoPy device. Just run the following lines of code on the device through REPL command line interface (use ATOM editor with pycom plugin).</p>

<pre><code>from network import WLAN
import ubinascii
wl = WLAN()
ubinascii.hexlify(wl.mac())[:6] + 'FFFE' + ubinascii.hexlify(wl.mac())[6:]
</code></pre>

<p>You will get output similar to <code>b'240ac4FFFE008d88'</code>. Copy the <code>240ac4FFFE008d88</code> part and save it somewhere for later use.</p></li>

<li><p>Create a project folder and copy these three files - <a href="https://github.com/pycom/pycom-libraries/blob/master/examples/lorawan-nano-gateway/main.py" target="_blank">main.py</a>, <a href="https://github.com/pycom/pycom-libraries/blob/master/examples/lorawan-nano-gateway/config.py" target="_blank">config.py</a>, and <a href="https://github.com/pycom/pycom-libraries/blob/master/examples/lorawan-nano-gateway/nanogateway.py" target="_blank">nanogateway.py</a> inside the folder.</p></li>

<li><p>Open the project folder in ATOM and edit the <code>nanogateway.py</code> file. Since I am on an eduroam network, it is difficult to setup internet. Thus, we will remove any lines of code in the <code>nanogateway.py</code> file which requires internet. Fortunately, there are not many lines. Specifically, just comment out lines <code>145--149</code> which have <code>ntp_sync</code> function calls.</p></li>

<li><p><code>main.py</code>: Keep the file untouched.</p></li>

<li><p><code>config.py</code>: Enter the required fields,</p>

<ul>
<li>SERVER = &lsquo;IP of RPi access point entered in Step-2&rsquo;</li>
<li>PORT = 1700</li>
<li>WIFI_SSID = &lsquo;WIFI name created in Step-2&rsquo;</li>
<li>WIFI_PASS = &lsquo;WIFI password created in Step-2&rsquo;</li>
<li>I am in Europe region and thus uncomment the lines for EU868 and comment for US915.</li>
</ul></li>

<li><p>Upload the project into the LoPy using Atom.</p></li>
</ol>

<p><em>Step-4</em></p>

<p>Now we need to register both the Nanogateway and the client LoPy node on the Chirpstack server. This step is important in order to create and join the LoRaWAN network.</p>

<ol>
<li>Follow this youtube <a href="https://youtu.be/mkuS5QUj5Js?t=285tea" target="_blank">video</a> from time=04:45.</li>
<li>The gateway, service, device and application profiles should be created by now.</li>
<li>Now keep the RPi and the LoPy with the nanogateway code together.</li>
<li>The nanogateway should now be able to connect to the RPi hotspot and thus they are on the same network. In principle, whatever the data packet nanogateway receives from the client LoPy node, will be forwarded to the Chirpstack server running on the RPi.</li>
<li>At the client node side: Upload the code from <a href="https://github.com/pycom/pycom-libraries/blob/master/examples/lorawan-nano-gateway/otaa_node.py" target="_blank">here</a> into another LoPy which will act as a clinet node.</li>
</ol>

<p><strong>Conclusions</strong></p>

<p>If everything is setup fine, you should be able to see the packets received at the RPi side. I hope this tutorial helps any enthusiast and hacker out there.</p>

<p>Thank you.</p>
