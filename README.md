# CMP2

## Description
use the Pmod CMPS2 to read the local magnetic field. When facing north, turn on the tri-led in green. Else turn on the tri-led in Red. 
1. using Jb pmod 
2. viatas file location  C:\vitis_project\vitis_midterm_COPM2v2
<img width="121" alt="image" src="https://github.com/Fall-2023-Classes/fa23-midterm-exam-2-XiangFei89213/assets/143454666/f002d03a-38a3-4709-bcae-e29238d2ff42">

## Hardware Configuration
- instantiate a I2C core in slot 4 of the MMIO system.  
- Modified pins in Jb for inout and out
```bash
set_property -dict {PACKAGE_PIN G13 IOSTANDARD LVCMOS33} [get_ports cmp_i2c_scl]
set_property -dict {PACKAGE_PIN H16 IOSTANDARD LVCMOS33} [get_ports cmp_i2c_sda]
```

## Software function
- cmp_init:
Initializes the CMP (magnetometer) sensor using I2C communication. Configures the sensor for continuous mode and sets measurement parameters.
```c++
// device address for CMP2
   const uint8_t DEV_ADDR = 0x30; 
// sent to reg(0x07) command 0x20 (enable continuous mode)
   wbytes[0] = 0x07;
   wbytes[1] = 0x20;
// if sucess, ack =0, failed ack =-1
   ack = cmp_p->write_transaction(DEV_ADDR, wbytes, 2, 0);
   sleep_ms(10);
```
- cmp_reset:
Resets or sets the internal control registers of the CMP sensor. 
- cmp_read:
Reads the X, Y, and Z components of the magnetic field from the CMP sensor. Converts raw data to milliGauss..
```c++
   // sent to reg(0x07) command (take measurement(0x01))
   wbytes[0] = 0x07;
   wbytes[1] = 0x01;
   cmp_p->write_transaction(DEV_ADDR, wbytes, 2, 0);
   sleep_ms(8);

   // read the status of CMP reg(0x06)
   // if bit0 =1 measure is done
   wbytes[0] = 0x06;
   cmp_p->write_transaction(DEV_ADDR, wbytes, 1, 1);
   cmp_p->read_transaction(DEV_ADDR, rbytes, 1, 0);
   for (int i = 0; i <= 10; i++)
   {
      cmp_p->write_transaction(DEV_ADDR, wbytes, 1, 1);
      cmp_p->read_transaction(DEV_ADDR, rbytes, 1, 0);
// if not done than wait
      if ((rbytes[0] & 0x01) != 1)
      {
         sleep_ms(10);
      }
      else
         break;
   }

   // read 6 byte
   wbytes[0] = 0x00;
   cmp_p->write_transaction(DEV_ADDR, wbytes, 1, 1);
   cmp_p->read_transaction(DEV_ADDR, rbytes, 6, 0);

```
- CMPS2_decodeHeading:
Decodes the heading angle based on the measured angle from the magnetometer sensor. Determines the cardinal direction (North, South, East, West, etc.) and controls the tri-color LED accordingly.
- CMPS2_getHeading:
Gets the heading angle from the CMP sensor. Reads X and Y components, eliminates offsets, and calculates the heading in degrees. Corrects the heading based on the declination.

## reference 
Pmod CMP2: https://digilent.com/reference/pmod/pmodcmps2/reference-manual?redirect=1
