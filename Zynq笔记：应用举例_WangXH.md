
<h1 style="text-align:center">Xilinx Zynq 7000：应用举例</h1>

--------------------------------------------------------------------------------
# 1. Ethernet

## 1.1. LwIP在Zynq中的实现

### 1.1.1. Xilinx 的 LwIP 库
已经全部完成，必要时更改硬件相关代码：
* 代码位于 src\contrib\ports\xilinx\netif
* Hdr
    * xemacpsif.h
* Src
    * xemacpsif.c
    * xemacpsif_dma.c
    * xemacpsif_hw.c
    * xemacpsif_hw.h
    * xemacpsif_physpeed.c

### 1.1.2. 使用模板
```c
struct netif *netif, server_netif;
struct ip_addr ipaddr, netmask, gw;

/* the MAC address of the board. This should be unique per board/PHY */
unsigned char mac_ethernet_address[] = {0x00, 0x0a, 0x35, 0x00, 0x01, 0x02};

lwip_init();

/* Add network interface to the netif_list, and set it as default */
if (!xemac_add(netif, &ipaddr, &netmask,
    &gw, mac_ethernet_address,
    EMAC_BASEADDR)) {
    printf(“Error adding N/W interface\n\r”);
    return -1;
}
netif_set_default(netif);

/* now enable interrupts */
platform_enable_interrupts();

/* specify that the network if is up */
netif_set_up(netif);

/* start the application, setup callbacks */
start_application();

/* receive and process packets */
while (1) {
    xemacif_input(netif);
    /* application specific functionality */
    transfer_data();
}"
```