# Main flow
```c

 retVal = wizchip_init(bufSize, bufSize);
  wiz_NetInfo netInfo = { .mac 	= {0x00, 0x08, 0xdc, 0xab, 0xcd, 0xef},	// Mac address
  	                          .ip 	= {192, 168, 2, 3},					    // IP address
  	                          .sn 	= {255, 255, 255, 0},					// Subnet mask
  	                          .gw 	= {192, 168, 2, 1}                      // Gateway address
  	  	  	  	  	  	  	};
  wiz_NetInfo netInfoGet = {0};
  do
  {
	  wizchip_setnetinfo(&netInfo);
	  wizchip_getnetinfo(&netInfoGet);
	  HAL_Delay(1000);
  } while(netInfoGet.ip[0] == 0 && netInfoGet.ip[1] == 0 && netInfoGet.ip[2] == 0 && netInfoGet.ip[3] == 0);
  
  while(1)
  {
	  if((retVal = socket(0, Sn_MR_TCP, 5000, 0)) == 0) {
		  uint8_t destip[4] = 	{192, 168, 2, 2};
	  	retVal = connect(0, destip, 1234);
	  	if(retVal == SOCK_OK)
	  	{
	  		uint8_t msg[] = "hello server";
	  		retVal = send(0, msg, sizeof(msg)-1);
        }
    }
  }
  
```
## 1. wizchip_init
- reset w5500
- check size and clear tx, rx buffer
### 1.1. wizchip_sw_reset
    ```c
    getSHAR(mac);
    getGAR(gw);  getSUBR(sn);  getSIPR(sip);
    setMR(MR_RST);
    getMR();
    setSHAR(mac);
    setGAR(gw);
    setSUBR(sn);
    setSIPR(sip);
    ```
- get mac:
```log
WIZCHIP_READ_BUF: addr 0x0900 |  val: [0x00, 0x00, 0x00, 0x00, 0x00, 0x1f]
```
- get gw:
```log
WIZCHIP_READ_BUF: addr 0x0100 |  val: [0xc0, 0xa8, 0x02, 0x01]
```
- get sn:
```log
WIZCHIP_READ_BUF: addr 0x0500 |  val: [0xff, 0xff, 0xff, 0x00]
```
- get sip:
```log
WIZCHIP_READ_BUF: addr 0x0f00 |  val: [0xc0, 0xa8, 0x02, 0x03]
```

- setMR(MR_RST) (Set Mode Register: If this bit is  All internal registers will be initialized. It will be automatically cleared as after S/W reset.):
```log
WIZCHIP_WRITE: addr 0x0004 | val: 0x80 
```
- getMR:
```log
WIZCHIP_READ: addr 0x0000 | val: 0x00
```
- get: mac, gw, sn, sip
```log
WIZCHIP_WRITE_BUF: addr 0x0904 |  val: [0x00, 0x00, 0x00, 0x00, 0x00, 0x1f]
WIZCHIP_WRITE_BUF: addr 0x0104 |  val: [0xc0, 0xa8, 0x02, 0x01]
WIZCHIP_WRITE_BUF: addr 0x0504 |  val: [0xff, 0xff, 0xff, 0x00]
WIZCHIP_WRITE_BUF: addr 0x0f04 |  val: [0xc0, 0xa8, 0x02, 0x03]
```
## 2. wizchip_setnetinfo
```c
void wizchip_setnetinfo(wiz_NetInfo* pnetinfo)
{
	LOG_DEBUG("%s:%d begin \n", __FUNCTION__, __LINE__);
   setSHAR(pnetinfo->mac);
   setGAR(pnetinfo->gw);
   setSUBR(pnetinfo->sn);
   setSIPR(pnetinfo->ip);
   _DNS_[0] = pnetinfo->dns[0];
   _DNS_[1] = pnetinfo->dns[1];
   _DNS_[2] = pnetinfo->dns[2];
   _DNS_[3] = pnetinfo->dns[3];
   _DHCP_   = pnetinfo->dhcp;
   LOG_DEBUG("%s:%d end \n", __FUNCTION__, __LINE__);
}
```
- set mac, gwm sn, ip
```log
WIZCHIP_WRITE_BUF: addr 0x0904 |  val: [0x00, 0x08, 0xdc, 0xab, 0xcd, 0xef]
WIZCHIP_WRITE_BUF: addr 0x0104 |  val: [0xc0, 0xa8, 0x02, 0x01]
WIZCHIP_WRITE_BUF: addr 0x0504 |  val: [0xff, 0xff, 0xff, 0x00]
WIZCHIP_READ_BUF: addr 0x0f00 |  val: [0xc0, 0xa8, 0x02, 0x03]
```

