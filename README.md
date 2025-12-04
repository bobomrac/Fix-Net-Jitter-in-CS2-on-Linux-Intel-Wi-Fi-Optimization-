# Fix Net Jitter in CS2 on Linux (Intel Wi-Fi Optimization)

Overview: Competitive CS2 gameplay on Linux can suffer from ping spikes, micro-stuttering, or erratic VAR readings, even on strong 5 GHz Wi-Fi. These issues are usually local to the system, caused by the interaction between the Wi-Fi supplicant, kernel driver, firmware, and network management services. Intel Wi-Fi hardware is capable, but default configurations prioritize compatibility and power saving over low-latency performance. Optimizing the stack eliminates most causes of network jitter in CS2.

## Technical Cause of Net Jitter

### Wi-Fi Supplicant

The supplicant is the daemon responsible for authentication, encryption key management, and roaming.

Linux defaults to wpa_supplicant, which is broadly compatible but occasionally blocks networking during key refresh or background scans. On Intel Wi-Fi hardware, these pauses show up as micro-latency or jitter in CS2.

iwd (Intel Wireless Daemon) is a modern, Intel-optimized alternative that manages Wi-Fi in a non-blocking, low-latency way.

### Wi-Fi Driver and Firmware

The kernel driver (iwlwifi) and firmware control packet transmission, aggregation, and power management.

Default power-saving introduces micro-sleep cycles or delayed packet processing, which increase latency variability.

Disabling power-saving and forcing performance mode ensures immediate packet handling, stabilizing ping.

### Service Conflicts

Running multiple supplicants or letting NetworkManager automatically spawn wpa_supplicant can cause connection events, duplicated scans, and interface contention, worsening jitter. Only one active supplicant should manage the Wi-Fi interface.

## Distro-Agnostic Conceptual Fix

Check which supplicant is running with<br> `systemctl status wpa_supplicant`<br> `systemctl status iwd`

The active service shows which supplicant is currently managing Wi-Fi.

Stop wpa_supplicant and prevent it from restarting `sudo systemctl stop wpa_supplicant sudo systemctl disable wpa_supplicant`

## Important: NetworkManager may attempt to restart wpa_supplicant. To prevent this, you need to configure NetworkManager to use iwd as the backend.

Configure NetworkManager to use iwd

NetworkManager supports iwd natively on most modern distributions. Conceptually:

Set `wifi.backend=iwd` in NetworkManager’s configuration file, usually at /etc/NetworkManager/NetworkManager.conf:

`[device]`
`wifi.backend=iwd`

Then restart NetworkManager to apply the change:

`sudo systemctl restart NetworkManager`

This ensures NetworkManager will delegate Wi-Fi management to iwd and will not restart wpa_supplicant.

Enable and verify iwd `sudo systemctl enable iwd` `sudo systemctl start iwd`

`systemctl status iwd`<br>
`systemctl status wpa_supplicant`

Only iwd should be active.

Optimize Intel Wi-Fi driver and firmware

Recommended driver options (conceptual explanation):

`options iwlwifi power_save=0 # disable Wi-Fi power saving`<br>
`options iwlwifi uapsd_disable=1 # prevent packet aggregation pauses`<br>
`options iwlwifi disable_11ax=1 # optional: disable Wi-Fi 6 if unstable`<br>
`options iwlmvm power_scheme=1 # set firmware to high-performance mode`

How to apply (distro-independent approach):

Place the options in a file in /etc/modprobe.d/, e.g., iwlwifi-gaming.conf.

Reload the driver to apply the settings (Wi-Fi will temporarily disconnect):

sudo modprobe -r iwlwifi && sudo modprobe iwlwifi

## Expected Outcomes

Elimination of recurring micro-jitter in CS2.

Stable ping and VAR metrics in net_graph.

Minimal packet loss and choke.

Works on any Linux distribution with Intel Wi-Fi cards.

Slight increase in power consumption (acceptable for desktops or gaming laptops).

# Summary

Net jitter in CS2 on Linux is primarily a local system issue: default Wi-Fi stacks (wpa_supplicant + iwlwifi defaults) introduce micro-latency via authentication pauses and power-saving behaviors. By:

Switching to iwd

Ensuring only one supplicant is active

Configuring NetworkManager to delegate Wi-Fi to iwd

Disabling driver/firmware power-saving features

Intel Wi-Fi hardware can achieve stable, low-latency networking ideal for competitive CS2.
