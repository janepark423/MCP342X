# MCP342x Multi-Channel, 16-Bit, Analog to Digital Converter
[![codecov](https://codecov.io/gh/glassboard-dev/MCP342x/branch/master/graph/badge.svg?token=O8GFCPCA2J)](https://codecov.io/gh/glassboard-dev/MCP342x)

C Driver for the MCP342xMulti-Channel, 16-Bit, Analog to Digital Converter IC. This driver can be included directly into a developers source or a static library can be created and then linked against. Development is done on the **develop** branch, and official releases can be found on the **master** branch.

## Retrieving the Source
The source is located on Github and can be either downloaded and included directly into a developers source OR the developer can add this repo as a submodule into their project directory (The latter is the preferred method).

To include the driver as a git submodule
```bash
$ cd ./${DIR_WHERE_YOU_WANT_TO_ADD_THE_MODULE}
$ git submodule add https://github.com/glassboard-dev/MCP342x
```

## Integration
#### Creating & Linking against a static library
To create a static library to link against, you must first source your build environment and ensure the **CC** environment variable is set to your desired toolchain.

Once your cross-compiler is properly setup. Execute the following commands:
```bash
$ mkdir build && cd build
$ cmake ..
$ make
```
The output library (libmcp342x.a) can be found in the **lib/** folder. Link against this file, and include the mcp342x.h header file into your source include directories.
```c
#include "mcp342x.h"
```

#### Adding to your own source/project
The other option for integrating the source into your project, is to include everything directly into your project
* Set your include directories to include the driver inc/ folder.
* Add the mcp342x.c to your source list to be compiled.
* Include the API header file wherever you intended to implement the driver source.
```c
#include "mcp342x.h"
```

## Implementing the driver
After following the integration steps above, you are ready to implement the driver and start retrieving ADC data. An example [***main.c***](./template/main.c) can be found in the templates folder that shows how to implement the settings, and data retrieval API. Note that you will need to fill out your own ***usr_*** functions for reading, writing, and a uS delay. You can build the example ***main.c*** by first compiling the static lib following the steps in the ***"Creating & Linking against a static library"*** and then executing the following commands.
```bash
$ cd template
$ mkdir build && cd build
$ cmake ..
$ make
```

## Testing
To test the source this submodule uses Ceedling, Unity and Gcov to run unit tests and generate HTML reports. You will need to install the following to run Unit Tests.
- [Ruby](https://www.ruby-lang.org/en/)
- [Ceedling](http://www.throwtheswitch.org/ceedling)
- [Gcovr](https://gcovr.com/en/stable/installation.html)

To execute the tests:
```bash
$ mkdir build && cd build
$ cmake ..
$ make tests
```

## Example application
Example application and main can be found below:
```C
#include <stdint.h>
#include <stdio.h>
#include "mcp342x.h"

#define MCP342x_ADDR    (0x6E)

int8_t usr_i2c_write(const uint8_t busAddr, const uint8_t *data, const uint32_t len) {
    mcp342x_return_code_t ret = MCP342x_RET_OK;

    // Transmit the data to the specified device from the provided
    // data buffer.

    return ret;
}

int8_t usr_i2c_read(const uint8_t busAddr, uint8_t *data, const uint32_t len) {
    mcp342x_return_code_t ret = MCP342x_RET_OK;

    // Received the specified amount of data from the device and store it
    // in the data buffer

    return ret;
}

void usr_delay_us(uint32_t period) {
    // Delay for the requested period
}

int main(void) {
    mcp342x_return_code_t ret = MCP342x_RET_OK;

    // Create an instance of our mcp342x device
    mcp342x_dev_t dev;

    // Provide the hardware abstraction functions for
    // I2c Read/Write and a micro-second delay function
    dev.intf.i2c_addr = MCP342x_ADDR;
    dev.intf.write = usr_i2c_write;
    dev.intf.read = usr_i2c_read;
    dev.intf.delay_us = usr_delay_us;

    // Init our desired config
    dev.registers.bits.config.bits.conv_mode = MCP342x_CM_CONT;
    dev.registers.bits.config.bits.gain = MCP342x_GAIN_x1;
    dev.registers.bits.config.bits.sample_rate = MCP342x_SR_60SPS;

    ret = mcp342x_writeConfig(&dev);

    if( MCP342x_RET_OK != ret ) {
        return 0;
    }

    while(1) {
        ret = mcp342x_sampleChannel(&dev, MCP342x_CH_1);

        if( MCP342x_RET_OK == ret ) {
            printf("CH1_raw: %d \t CH1_V: %0.2fV", dev.results->outputCode, dev.results->voltage);
        }
    }

    return 0;
}
```
