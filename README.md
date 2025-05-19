| Supported Targets | ESP32-H2 | ESP32-C3 |
| ----------------- | -------- | -------- |

# Genaral Procedure

to run first apply the patches and update patch code
```shell
#first load your paths of IDF, Rainmaker and ESP-Matter
idf.py build
```




flash app bin 
```shell
esptool.py --chip esp32h2 --port /dev/cu.usbmodem21101 --baud 460800 write_flash 0x20000 build/rezti_occupancy_v1.bin 
```

```shell
#no need since bootloader is probably loaded in the firmware
esptool.py --chip esp32h2 write_flash 0x0 build/bootloader/bootloader.bin
```

```shell
#might need to sign the partition table 
esptool.py --chip esp32h2 write_flash 0xc000 build/partition_table/partition-table.bin
```

```shell
esptool.py --chip esp32h2 --port /dev/cu.usbmodem21101 --baud 460800 write_flash 0xd000 out/15cb_1234/*.esp_secure_cert.bin  
```

```shell
esptool.py --chip esp32h2 --port /dev/cu.usbmodem21101 --baud 460800 write_flash 0x3E0000  out/15cb_1234/*.partition.bin
```

private server: 
a3fnlslsyuks6d-ats.iot.ap-southeast-1.amazonaws.com

to check if bin is signed 

```shell
espsecure.py signature_info_v2 build/bootloader/bootloader.bin
```

to see chip's efuse status -- check secure boot and flash encryption
```shell
espefuse.py --port /dev/tty.usbmodem21101 summary
```

# Add chip-cert and esp-matter-mfg-tool to PATH
```shell
export PATH=$PATH:$HOME/.espressif/esp-matter/connectedhomeip/connectedhomeip/out/host/
```

# Generate Certification Declaration (CD)
```shell

chip-cert gen-cd \
  -K ../../.espressif/esp-matter/connectedhomeip/connectedhomeip/credentials/test/certification-declaration/Chip-Test-CD-Signing-Key.pem \
  -C ../../.espressif/esp-matter/connectedhomeip/connectedhomeip/credentials/test/certification-declaration/Chip-Test-CD-Signing-Cert.pem \
  -O esp_dac_15CB_1234.der \
  -f 1 \
  -V 0x15CB \
  -p 0x1234 \
  -d 0x0107 \
  -c "ZIG20141ZB330001-24" \
  -l 0 \
  -i 0 \
  -n 1 \
  -t 1 \
  --dac-origin-vendor-id 0x131B \
  --dac-origin-product-id 0x8045
```
## Example certification ID: ZIG20141ZB330001-24, CSA00000SWC00000-01

# Run esp-matter manufacturing tool
```shell
esp-matter-mfg-tool \
  -v 0x15CB \
  -p 0x1234 \
  --pai \
  -k ../../.espressif/esp-matter/connectedhomeip/connectedhomeip/credentials/test/attestation/Chip-Test-PAI-FFF2-8001-Key.pem \
  -c ../../.espressif/esp-matter/connectedhomeip/connectedhomeip/credentials/test/attestation/Chip-Test-PAI-FFF2-8001-Cert.pem \
  --product-name "Occupancy Sensor MoT" \
  --vendor-name "Rez-TI" \
  --hw-ver 1 \
  --hw-ver-str "v1.0" \
  --serial-num 0001 \
  -cd esp_dac_15CB_1234.der \
  --csv rainmaker_creds/keys.csv \
  --mcsv rainmaker_creds/master.csv
  ```

# for chip tool 
```shell
./chip-tool pairing ble-thread 0x1234 hex:<TLV hex value copy from home assistant-- the long hex string > <pincode-passcode> <discriminator>
```
