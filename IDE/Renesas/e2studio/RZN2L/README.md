wolfSSL for Renesas RZN2L Board
=================================================

## Description

This directory contains e2studio projects targeted at the Renesas RZ MCUs.
The example projects include a wolfSSL TLS client and server.
They also include benchmark and cryptography tests for the wolfCrypt library.

The example project contains both the wolfSSL and wolfCrypt libraries.
It is built as a `Renesas RZ/N C/C++ FSP Project` and contains the Renesas RZ
configuration. The wolfssl project uses `Renesas Secure IP on RZ`
as hardware acceleration for cyptography.

**Limitation**

Due to lacking of TLS related feature on RSIP driver version, TLS connection examples below use `SHA` and `Random generation` of RSIP driver.

The example project summary is listed below and is relevant for every project.

### Project Summary
|Item|Name/Version|
|:--|:--|
|Board|RZN2L|
|Device|R9A07G084M04GBG|
|Toolchain|GCC for Renesas RZ|
|Toolchain Version|10.3.1.20210824|
|FSP Version|1.2.0|

#### Selected software components

|Components|Version|Note|
|:--|:--|:--|
|Board Support Package Common Files|v1.20||
|I/O Port|v1.2.0||
|Arm CMSIS Version 5 - Core (M)|v5.7.0+renesas.1||
|Board support package for R9A07G084M04GBG|v1.2.0||
|Board support package for RZN2L|v1.2.0||
|Board support package for RZN2L - FSP Data|v1.2.0||
|RSK+RZN2L Board Support Files (RAM execution without flash memory)|v1.2.0||
|FreeRTOS - Buffer Allocation 2|v1.2.0||
|FreeRTOS - Memory Management - Heap 4|v1.2.0||
|FreeRTOS+TCP|v1.2.0||
|Ethernet PHY |v1.2.0||
|Ethernet Selector|v1.2.0||
|Ethernet|v1.2.0||
|Ethernet Switch|v1.2.0||
|SCI UART|v1.2.0||
|r_ether to FreeRTOS+TCP Wrapper|v1.2.0||
|Renesas Secure IP Driver|v1.3.0+fsp.1.2.0|Need to contact Renesas to get RSIP module|
|RSIP Engine for RZ/N2L|v1.3.0+fsp.1.2.0|Need to contact Renesas to get RSIP module|


## Setup Steps and Build wolfSSL Library

1.) Import  projects from [File]->[Open projects from File System]

+ Select folder at /path/to/wolfssl/IDE/Renesas/e2studio/RZN2L/test

2.) Create a `dummy_application` Renesas RZ/N C/C++ FSP Project.

+ Click File->New->`RZ/N C/C++ FSP Project`.
+ Enter project name `dummy_application`.
+ Select Board: to `RSK+RZN2L (RAM execution without flash memory)`.
+ Select Device: to `R9A07G084M04GBG`. Click Next.
+ Check to `Executable`
+ Select FreeRTOS from RTOS selection. Click Finish.
+ Check `FreeRTOS minimal - Static Allocation`. Click Finish.
+ Open FSP Configurator by clicking configuration.xml in the project -->
+ Go to `Stacks` tab
+ Add new thread by clicking `New Thread`, and set properties below

|Property|Value|
|:--|:--|
|Thread Symbol|rzn2l_tst_thread|
|Thread Name|rzn2l_tst_thread|
|Thread Stack size|increase depending on your environment<br> e.g. 0xA000|
|Thread MemoryAllocation|Dyamic|
|Common General Use Mutexes|Enabled|
|Common General Enable Backward Compatibility|Enabled|
|Common Memory Allocation Support Dynamic Allocation|Enabled|
|Common Memory Allocation Total Heap Size|increase depending on your environment<br> e.g. 0x20000|

+ Add `Heap 4` stack to rzn2l_tst_thread from `New Stack` -> `FreeRTOS` -> `FreeRTOS Heap 4`
+ Add `UART Driver` stack to rzn2l_tst_thread from `New Stack` -> `Connectivity` -> `UART Driver`
+ Add `FreeRTOS + TCP` stack to rzn2l_tst_thread from `New Stack` -> -> `FreeRTOS` -> `Libraries` -> `FreeRTOS+TCP` and set properties

+ Save `dummy_application` FSP configuration
+ Copy <u>configuration.xml</u> under `dummy_application` to `test_RZN2L`
+ Open FSP configuration by clicking copied configuration.xml at `test_RZN2L`
+ Click `Generate Project Content` on FSP configuration GUI

3.) Prepare UART to logging

+ Download Sample package from [BACnet Start-Up](https://www.renesas.com/us/en/products/microcontrollers-microprocessors/rz-mpus/bacnet-start-rzn2l-rsk)
+ Copy the following C source files from the project to src/serial_io folder of `test_RZN2L`
      + um_serial_io_uart.c
      + um_serial_io_task_writer.c
      + um_serial_io_cfg.h
      + um_common_api.h
      + um_common_cfg.h
      + um_serial_io.c
      + um_serial_io.h
      + um_serial_io_api.h
      + um_serial_io_internal.h
+ Open um_serial_io_task_writer.c and re-name printf to uart_printf

3.) Build `test_RZN2L` project

## Run `test_RZN2L`

1). Right click the project and Select menu `Debug` -> `Renesas GDB Hardware debugging`

2). Select J-Link ARM and R9A07G084M04

3). Break at Entory point. Change `cpsr` register value from 0xXXXXX1yy to 0xXXXXX1da

## Run TLS 1.3 Client
1.) Enable `WOLFSSL_TLS13` macro in `user_settings.h`

2.) Enable `TLS_CLIENT` macro in `wolfssl_demo.h` of test_RZN2L project

3.) Client IP address and Server IP address

+ Client IP address can be changed by the following line in `rzn2l_tst_thread_entry.c`.
```
static const byte ucIPAddress[4]          = { 192, 168, 11, 241 };
```
+ Server IP address can be changed by the following line in wolf_client.c.
```
#define SERVER_IP    "192.168.11.65"
```

3.) Build test_RZN2L project

4.) Prepare peer wolfssl server

+ On Linux
+ Clone wolfssl from [github repository](https://github.com/wolfssl/wolfssl.git)
```
$ ./autogen.sh
$ ./configure
$ make
```

+ Run peer wolfSSL server

+ RSA sign and verify use, launch server with the following option
```
$./examples/server/server -b -d -i -v 4
```

+ For ECDSA sign and verify use,
Enable the `USE_CERT_BUFFER_256` macro in `wolfssl_demo.h`
Disble the `USE_CERT_BUFFER_2048` macro in `wolfssl_demo.h`

+ launch server with the following option.
```
$./examples/server/server -b -d -i -v 4 -c ./certs/server-ecc.pem -k ./certs/ecc-key.pem
```

5.) Run the example Client

You will see the following message on a UART terminal when using RSA sign and verify or ECDSA sign and verify.
```
 Started Serial I/O interface.
 Start TLS Connection to 192.168.11.65 port(11111)
 Error [-116]: FreeRTOS_connect. <-- A number of messages will be showed by depending on number of connection failures.
 Start to connect to the server.
  Cipher : TLS13-AES128-GCM-SHA256
 Received: I hear you fa shizzle!

 Start to connect to the server.
  Cipher : TLS13-AES256-GCM-SHA384
 Received: I hear you fa shizzle!

 End of Client Example
```

## Run TLS 1.2 Client
1.) Disable `WOLFSSL_TLS13` macro in `user_settings.h`

2.) Enable `TLS_CLIENT` macro in `wolfssl_demo.h` of test_RZN2L project

3.) Client IP address and Server IP address

+ Client IP address can be changed by the following line in `rzn2l_tst_thread_entry.c`.
```
static const byte ucIPAddress[4]          = { 192, 168, 11, 241 };
```
+ Server IP address can be changed by the following line in wolf_client.c.
```
#define SERVER_IP    "192.168.11.65"
```

3.) Build test_RZN2L project

4.) Prepare peer wolfssl server

+ On Linux
+ Clone wolfssl from [github repository](https://github.com/wolfssl/wolfssl.git)
```
$ ./autogen.sh
$ ./configure
$ make
```

+ Run peer wolfSSL server

+ RSA sign and verify use, launch server with the following option
```
$./examples/server/server -b -d -i -v 3
```

+ For ECDSA sign and verify use,
Enable the `USE_CERT_BUFFER_256` macro in `wolfssl_demo.h`
Disble the `USE_CERT_BUFFER_2048` macro in `wolfssl_demo.h`

+ launch server with the following option.
```
$./examples/server/server -b -d -i -v 3 -c ./certs/server-ecc.pem -k ./certs/ecc-key.pem
```

5.) Run the example Client

You will see the following message on a UART terminal when using RSA sign and verify.
```
Started Serial I/O interface.
 Start TLS Connection to 192.168.11.65 port(11111)
 Error [-116]: FreeRTOS_connect.
 Start to connect to the server.
  Cipher : ECDHE-RSA-AES128-GCM-SHA256
 Received: I hear you fa shizzle!

 Start to connect to the server.
  Cipher : ECDHE-RSA-AES256-SHA
 Received: I hear you fa shizzle!

 Start to connect to the server.
  Cipher : ECDHE-RSA-AES128-SHA256
 Received: I hear you fa shizzle!


 End of Client Example
```

You will see the following message on a UART terminal when using ECDSA sign and verify.
```
Started Serial I/O interface.
 Start TLS Connection to 192.168.11.65 port(11111)
 Error [-116]: FreeRTOS_connect.
 Start to connect to the server.
  Cipher : ECDHE-ECDSA-AES128-GCM-SHA256
 Received: I hear you fa shizzle!

 Start to connect to the server.
  Cipher : ECDHE-ECDSA-AES256-SHA
 Received: I hear you fa shizzle!

 Start to connect to the server.
  Cipher : ECDHE-ECDSA-AES128-SHA256
 Received: I hear you fa shizzle!


 End of Client Example
```

## Run TLS 1.3 Server
1.) Enable `WOLFSSL_TLS13` macro in `user_settings.h`

2.) Enable `TLS_SERVER` macro in `wolfssl_demo.h` of test_RZN2L project

3.) Client IP address and Server IP address

+ Server IP address can be changed by the following line in `rzn2l_tst_thread_entry.c`.
```
static const byte ucIPAddress[4]          = { 192, 168, 11, 241 };
```

3.) Build test_RZN2L project

+ For ECDSA sign and verify use,
Enable the `USE_CERT_BUFFER_256` macro in `wolfssl_demo.h`
Disble the `USE_CERT_BUFFER_2048` macro in `wolfssl_demo.h`

+ launch server from e2studio

4.) Prepare peer wolfssl client

+ On Linux
+ Clone wolfssl from [github repository](https://github.com/wolfssl/wolfssl.git)
```
$ ./autogen.sh
$ ./configure
$ make
```

5.) Run peer wolfSSL client

+ RSA sign and verify use, run peer client with the following option
```
$./examples/client/client -h 192.168.11.241 -p 11111 -v 4
```

You will see the following message on a UART terminal when using RSA sign and verify.
```
Started Serial I/O interface.
 Start TLS Accept at 192.168.011.241 port(11111)
Received: hello wolfssl!
Cleaning up socket and wolfSSL objects.
Waiting connection....
```

You will see the follwoing message on Linux terminal.
```
$ ./examples/client/client -h 192.168.11.241 -p 11111 -v 4
SSL version is TLSv1.3
SSL cipher suite is TLS_AES_128_GCM_SHA256
SSL curve name is SECP256R1
I hear ya fa s
```

+ ECDSA sign and verify use, run peer client with the following option
```
$./examples/client/client -h 192.168.11.241 -p 11111 -v 3 -A ./certs/ca-ecc-cert.pem -c ./certs/client-ecc-cert.pem  -k ./certs/ecc-client-key.pem
```

You will see the following message on a UART terminal when using ECDSA sign and verify.
```
Started Serial I/O interface.
 Start TLS Accept at 192.168.011.241 port(11111)
Received: hello wolfssl!
Cleaning up socket and wolfSSL objects.
Waiting connection....
```
You will see the follwoing message on Linux terminal.
```
$ ./examples/client/client -h 192.168.11.241 -p 11111 -v 4 -A ./certs/ca-ecc-cert.pem -c ./certs/client-ecc-cert.pem  -k ./cert
s/ecc-client-key.pem
SSL version is TLSv1.3
SSL cipher suite is TLS_AES_128_GCM_SHA256
SSL curve name is SECP256R1
I hear ya fa s
```

## Run TLS 1.2 Server
1.) Disable `WOLFSSL_TLS13` macro in `user_settings.h`

2.) Enable `TLS_SERVER` macro in `wolfssl_demo.h` of test_RZN2L project

3.) Client IP address and Server IP address

+ Server IP address can be changed by the following line in `rzn2l_tst_thread_entry.c`.
```
static const byte ucIPAddress[4]          = { 192, 168, 11, 241 };
```

3.) Build test_RZN2L project

+ For ECDSA sign and verify use,
Enable the `USE_CERT_BUFFER_256` macro in `wolfssl_demo.h`
Disble the `USE_CERT_BUFFER_2048` macro in `wolfssl_demo.h`

+ launch server from e2studio

4.) Prepare peer wolfssl client

+ On Linux
+ Clone wolfssl from [github repository](https://github.com/wolfssl/wolfssl.git)
```
$ ./autogen.sh
$ ./configure
$ make
```

5.) Run peer wolfSSL client

+ RSA sign and verify use, run peer client with the following option
```
$./examples/client/client -h 192.168.11.241 -p 11111 -v 4
```

You will see the following message on a UART terminal when using RSA sign and verify.
```
Started Serial I/O interface.
 Start TLS Accept at 192.168.011.241 port(11111)
Received: hello wolfssl!
Cleaning up socket and wolfSSL objects.
Waiting connection....
```

You will see the follwoing message on Linux terminal.
```
$ ./examples/client/client -h 192.168.11.241 -p 11111 -v 3
SSL version is TLSv1.2
SSL cipher suite is TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
SSL curve name is SECP256R1
I hear ya fa s
```

+ ECDSA sign and verify use, run peer client with the following option
```
$./examples/client/client -h 192.168.11.241 -p 11111 -v 3 -A ./certs/ca-ecc-cert.pem -c ./certs/client-ecc-cert.pem  -k ./certs/ecc-client-key.pem
```

You will see the following message on a UART terminal when using ECDSA sign and verify.
```
Started Serial I/O interface.
 Start TLS Accept at 192.168.011.241 port(11111)
Received: hello wolfssl!
Cleaning up socket and wolfSSL objects.
Waiting connection....
```
You will see the follwoing message on Linux terminal.
```
$ ./examples/client/client -h 192.168.11.241 -p 11111 -v 3 -A ./certs/ca-ecc-cert.pem -c ./certs/client-ecc-cert.pem  -k ./certs/ecc-client-key.pem
SSL version is TLSv1.2
SSL cipher suite is TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
SSL curve name is SECP256R1
I hear ya fa s
```
## Run Crypt test
1.) Enable `CRYPT_TEST` macro in `wolfssl_demo.h`

2.) Run `test_RZN2L` from e2studio

Sample Output

```
------------------------------------------------------------------------------
 wolfSSL version 5.6.3
------------------------------------------------------------------------------
error    test passed!
MEMORY   test passed!
base64   test passed!
asn      test passed!
RANDOM   test passed!
MD5      test passed!
MD4      test passed!
SHA      test passed!
SHA-224  test passed!
SHA-256  test passed!
SHA-384  test passed!
SHA-512  test passed!
SHA-512/224  test passed!
SHA-512/256  test passed!
Hash     test passed!
HMAC-MD5 test passed!
HMAC-SHA test passed!
HMAC-SHA224 test passed!
HMAC-SHA256 test passed!
HMAC-SHA384 test passed!
HMAC-SHA512 test passed!
HMAC-KDF    test passed!
TLSv1.3 KDF test passed!
DES      test passed!
DES3     test passed!
AES      test passed!
AES256   test passed!
AES-GCM  test passed!
RSA      test passed!
PWDBASED test passed!
ECC      test passed!
ECC buffer test passed!
CURVE25519 test passed!
logging  test passed!
time test passed!
mutex    test passed!
crypto callback test passed!
Test complete
 End wolfCrypt Test
```

**Note**
`SHA1/224/256/384/512` and `Random generation` of RSIP driver are enabled at the sampele output above while running wolfCrypt test.

## Run Benchmark

1.) Enable `BENCHMARK` macro in `wolfssl_demo.h`

2.) Run `test_RZN2L` from e2studio

Sample Output
```
 Started Serial I/O interface. Start wolfCrypt Benchmark
wolfCrypt Benchmark (block bytes 1024, min 1.0 sec each)
RNG                      525.0 KiB took 1.038 seconds, 505.780 KiB/s
AES-128-CBC-enc          2.0 MiB took 1.011 seconds, 2.028 MiB/s
AES-128-CBC-dec          1.1 MiB took 1.003 seconds, 1.972 MiB/s
AES-192-CBC-enc          1.1 MiB took 1.007 seconds, 1.721 MiB/s
AES-192-CBC-dec          1.1 MiB took 1.004 seconds, 1.678 MiB/s
AES-256-CBC-enc          1.1 MiB took 1.015 seconds, 1.515 MiB/s
AES-256-CBC-dec          1.0 MiB took 1.008 seconds, 1.477 MiB/s
AES-128-GCM-enc          575.0 KiB took 1.014 seconds, 567.061 KiB/s
AES-128-GCM-dec          575.0 KiB took 1.013 seconds, 567.621 KiB/s
AES-192-GCM-enc          550.0 KiB took 1.017 seconds, 540.806 KiB/s
AES-192-GCM-dec          550.0 KiB took 1.017 seconds, 540.806 KiB/s
AES-256-GCM-enc          525.0 KiB took 1.012 seconds, 518.775 KiB/s
AES-256-GCM-dec          525.0 KiB took 1.011 seconds, 519.288 KiB/s
AES-128-GCM-enc-no_AAD   575.0 KiB took 1.003 seconds, 573.280 KiB/s
AES-128-GCM-dec-no_AAD   575.0 KiB took 1.003 seconds, 573.280 KiB/s
AES-192-GCM-enc-no_AAD   550.0 KiB took 1.006 seconds, 546.720 KiB/s
AES-192-GCM-dec-no_AAD   550.0 KiB took 1.006 seconds, 546.720 KiB/s
AES-256-GCM-enc-no_AAD   525.0 KiB took 1.001 seconds, 524.476 KiB/s
AES-256-GCM-dec-no_AAD   525.0 KiB took 1.000 seconds, 525.000 KiB/s
GMAC Default             791.0 KiB took 1.001 seconds, 790.210 KiB/s
3DES                     425.0 KiB took 1.025 seconds, 414.634 KiB/s
MD5                      12.0 MiB took 1.001 seconds, 12.439 MiB/s
SHA                      21.1 MiB took 1.000 seconds, 21.558 MiB/s
SHA-224                  21.0 MiB took 1.000 seconds, 21.362 MiB/s
SHA-256                  21.0 MiB took 1.000 seconds, 21.216 MiB/s
SHA-384                  19.1 MiB took 1.000 seconds, 19.897 MiB/s
SHA-512                  19.1 MiB took 1.000 seconds, 19.824 MiB/s
SHA-512/224              20.0 MiB took 1.000 seconds, 20.117 MiB/s
SHA-512/256              19.1 MiB took 1.000 seconds, 19.946 MiB/s
HMAC-MD5                 12.0 MiB took 1.000 seconds, 12.329 MiB/s
HMAC-SHA                 20.0 MiB took 1.000 seconds, 20.093 MiB/s
HMAC-SHA224              19.1 MiB took 1.000 seconds, 19.922 MiB/s
HMAC-SHA256              19.1 MiB took 1.000 seconds, 19.751 MiB/s
HMAC-SHA384              18.0 MiB took 1.000 seconds, 18.457 MiB/s
HMAC-SHA512              18.0 MiB took 1.000 seconds, 18.384 MiB/s
PBKDF2                   288.0 bytes took 1.082 seconds, 266.174 bytes/s
RSA     2048   public        56 ops took 1.007 sec, avg 17.982 ms, 55.611 ops/sec
RSA     2048  private         2 ops took 2.391 sec, avg 1195.500 ms, 0.836 ops/sec
ECC   [      SECP256R1]   256  key gen         2 ops took 1.213 sec, avg 606.500 ms, 1.649 ops/sec
ECDHE [      SECP256R1]   256    agree         2 ops took 1.211 sec, avg 605.500 ms, 1.652 ops/sec
ECDSA [      SECP256R1]   256     sign         2 ops took 1.224 sec, avg 612.000 ms, 1.634 ops/sec
ECDSA [      SECP256R1]   256   verify         2 ops took 2.312 sec, avg 1155.1000 ms, 0.865 ops/sec
CURVE  25519  key gen         2 ops took 1.061 sec, avg 530.500 ms, 1.885 ops/sec
CURVE  25519    agree         2 ops took 1.060 sec, avg 530.000 ms, 1.887 ops/sec
Benchmark complete
 End wolfCrypt Benchmark
```
**Note**
`SHA1/224/256/384/512` and `Random generation` of RSIP driver are enabled at the sampele output above.
## Support

For support inquiries and questions, please email support@wolfssl.com. Feel free to reach out to info@wolfssl.jp as well.
