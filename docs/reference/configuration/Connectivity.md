<h2 id="configuration-connectivity">Connectivity</h2>

```
Configuration parameters
------------------------

Name: ppp-cell-iface.apn-lookup
    Defined by: library:ppp-cell-iface
Name: ppp-cell-iface.at-parser-buffer-size
    Defined by: library:ppp-cell-iface
    Value: 256 (set by library:ppp-cell-iface)
Name: ppp-cell-iface.at-parser-timeout
    Defined by: library:ppp-cell-iface
    Value: 8000 (set by library:ppp-cell-iface)
Name: ppp-cell-iface.baud-rate
    Defined by: library:ppp-cell-iface
    Value: 115200 (set by library:ppp-cell-iface)
Name: lwip.addr-timeout
    Defined by: library:lwip
    Value: 5 (set by library:lwip)
Name: lwip.addr-timeout-mode
    Defined by: library:lwip
    Value: 1 (set by library:lwip)
Name: lwip.debug-enabled
    Defined by: library:lwip
Name: lwip.default-thread-stacksize
    Description: Stack size for lwip system threads
    Defined by: library:lwip
    Value: 512 (set by library:lwip)
Name: lwip.enable-ppp-trace
    Defined by: library:lwip
Name: lwip.ethernet-enabled
    Defined by: library:lwip
    Value: 1 (set by library:lwip)
Name: lwip.ip-ver-pref
    Defined by: library:lwip
    Value: 4 (set by library:lwip)
Name: lwip.ipv4-enabled
    Defined by: library:lwip
    Value: 1 (set by library:lwip)
Name: lwip.ipv6-enabled
    Defined by: library:lwip
Name: lwip.mem-size
    Defined by: library:lwip
Name: lwip.pbuf-pool-bufsize
    Defined by: library:lwip
Name: lwip.pbuf-pool-size
    Description: Number of pbufs in pool - usually used for received packets, so this determines how much data can be buffered between reception and the application reading. If a driver uses PBUF_RAM for reception, less pool may be needed. Current default (used if null here) is set to 5 in lwipopts.h, unless overridden by target Ethernet drivers.
    Defined by: library:lwip
Name: lwip.ppp-enabled
    Defined by: library:lwip
Name: lwip.ppp-ipv4-enabled
    Defined by: library:lwip
    Value: 1 (set by library:lwip)
Name: lwip.ppp-ipv6-enabled
    Defined by: library:lwip
Name: lwip.ppp-thread-stacksize
    Defined by: library:lwip
    Value: 768 (set by library:lwip)
Name: lwip.socket-max
    Defined by: library:lwip
    Value: 4 (set by library:lwip)
Name: lwip.tcp-enabled
    Defined by: library:lwip
    Value: 1 (set by library:lwip)
Name: lwip.tcp-server-max
    Defined by: library:lwip
    Value: 4 (set by library:lwip)
Name: lwip.tcp-socket-max
    Defined by: library:lwip
    Value: 4 (set by library:lwip)
Name: lwip.tcpip-thread-stacksize
    Description: Stack size for lwip TCPIP thread
    Defined by: library:lwip
    Value: 1200 (set by library:lwip)
Name: lwip.udp-socket-max
    Defined by: library:lwip
    Value: 4 (set by library:lwip)
Name: lwip.use-mbed-trace
    Defined by: library:lwip
Name: nsapi.present
    Defined by: library:nsapi
    Value: 1 (set by library:nsapi)
```
