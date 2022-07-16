# NTLM relay attack

The NTLM relay attack is actually a man-in-the-middle attack. An attacker manages to be in a man-in-the-middle position between a client and a server, and simply relays information from one to the other.

However, the key of the attack is how to get the Net-NTLM hash of the victim. There are several ways.

## LLMNR/NBNS protocols spoofing

The LLMNR and NBNS protocols are name resolution protocols. When the user lookup a hostname that does not exist, the windows will broadcast LLMNR/NBNS packets to request the resolution of the hostname in the LAN.

Therefore, we can pretend to be the host that the victim wants to access and respond to him. Then we can get the Net-NTLM hash.

## WPAD protocol hijack

The WPAD protocol is used to locate proxy auto-config (PAC) file through DHCP, DNS, LLMNR, NBNS protocols. And the PAC file specifies which proxy server the browser chooses to access the URL.

So we can hijack the WPAD protocol to return the constructed PAC file through LLMNR/NBNS protocols spoofing. In this way, the victim will access our host as a proxy server.

## Note

The authentication protocols are almost independent of the application protocols. The application protocols usually support multiple authentication protocol types.
