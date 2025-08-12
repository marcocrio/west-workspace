# Notes on Zephyr:


## Building tips:

## What Is west build Building?
When you run this command:
`west build -b esp32_devkitc/esp32/procpu -s apps/blinky_esp32 -d apps/blinky_esp32/build`
You are building Everything needed to flash the ESP32, specifically:
- Your app (main.c and friends)
- prj.conf settings (enabling drivers, features)
- Your device tree overlay (esp32_devkitc_esp32_procpu.overlay)
- The Zephyr kernel + drivers for your board
- Toolchain-specific build artifacts (ELF, bin, hex)

So it's not just your app — it's your app + Zephyr RTOS kernel + hardware configuration + board support = a complete firmware image.


### Board Core Qualifier
If west is returns issues building, maybe check if your board needs to specify which baord to build for. bor example:

`$ west build -b esp32_devkitc/esp32/procpu -p always apps/blinky_esp32/`
This is instructing west to compile the esp32_devkitc dts (a merged device tree, .dts, that Zephyr uses) for a ESP32 board to run on its procpu core (primary applicaiton processor which runs zephir) taking the .overlay file in consideration.

This will create the file `your_app/build/zephyr/zephyr.dts`




## Device Tree
### What Is a Devicetree?
A Devicetree (DT) is a hierarchical data structure used to describe the hardware configuration of a board or SoC to the operating system (in our case, Zephyr RTOS). It tells Zephyr:
`“Here’s what hardware exists, where it’s connected, and how to talk to it.”`

This allows you to write portable application code that doesn’t need to hardcode GPIO numbers, I2C addresses, or SPI configurations.

### Why Use a Devicetree?
- Decouples application logic from hardware
- Supports multiple boards or variants with the same app
- Centralized hardware configuration
- Compile-time error checking of device definitions

### Devicetree Files
When building a Zephyr app, your devicetree is automatically assembled from 3 layers:

| Source File	|	Role														|
|:--------------|---------------------------------------------------------------|
| `.dtsi`		|	SoC-level definitions (shared across boards)				|
| `.dts`		|	Board-specific configuration (pin mappings, memory, etc.)	|
| `.overlay`	|	Your project-specific overrides and additions				|

The resulting devicetree is compiled into a C header file used by your application.



### Key Syntax Elements:

|	Keywords					|	Meaning																|
|:------------------------------|:----------------------------------------------------------------------|
|	`/ { ... }`					|	Root node of the device tree										|
|	`aliases { ... }` 			|	Logical names that map to real hardware nodes 						|
|	`&<label>`					|	Reference an existing node label (like &gpio0)						|
|	`<label>: <name> { ... }` 	|	Create a new node with a label you can reference later				|
|	`gpios = <...>;`			|	Bind a GPIO pin to the node (requires a GPIO controller reference)	|
|	`compatible = "..."` 		|	Tells Zephyr what kind of device this node represents				|
|	`reg = <...>`				| 	Address or index on a bus (e.g., I2C address)						|
|	`status = "okay"` 			|	Enables a peripheral (default may be "disabled")					|
|	`label = "..."`				| 	Human-readable name; useful for debugging or device shell			|


### Devicetree Building Blocks:



#### 1 Node:
A node represents a physical hardware component (e.g., GPIO controller, I2C peripheral, LED, sensor).

Example:
``` dts
gpio0: gpio@3FF44000 {
    compatible = "espressif,esp32-gpio";
    reg = <0x3FF44000 0x1000>;
    status = "okay";
};
```
| 	Part			|	Meaning											|
|:------------------|:--------------------------------------------------|
|	`gpio0:`		|	Label (used in overlays or C code: &gpio0)		|
|	`gpio@3FF44000`	|	Node name and memory-mapped address				|
|	`compatible`	|	Tells Zephyr which driver to use				|
|	`reg`			|	Base address and size of the register region	|
|	`status`		|	"okay" to enable, "disabled" to ignore			|


#### Root Node:
This refers to the root node of the devicetree. 
All nodes are declared inside it, or inside other nodes like `&i2c0`, `&uart0`, etc. <br>
`/ { ... }`

#### 2. Property
A property is a key-value pair inside a node.
``` dts
reg = <0x3FF44000 0x1000>;
interrupts = <5>;
clock-frequency = <80000000>;
label = "GPIO Controller 0";
```
Properties configure how the peripheral behaves.



#### 3. Labels
Defined like:

``` dts
led0: led_node {
  gpios = <&gpio0 2 GPIO_ACTIVE_LOW>;
};
```
- `led0:` is the label — used in C like `DT_NODELABEL(led0)`
- `led_node` is the node name — not used much directly


#### Aliases
Provide shortcuts like:
``` dts
aliases {
  led0 = &led0;
  sw0 = &user_button;
};
```
Used in C like:
``` c
#define LED DT_ALIAS(led0)
#define BUTTON DT_ALIAS(sw0)
```


<br><br>

## .Overlays

### Basic Overlay Example:
``` dts
/ {
    aliases {
        led0 = &my_led;
    };

    my_led: my_led {
        gpios = <&gpio0 2 GPIO_ACTIVE_LOW>;
    };
};
```

### Overlay File Structure
A typical overlay file is placed at `apps/your_app/boards/your_board.overlay`.

For example: ```apps/blinky_esp32/boards/esp32_devkitc.overlay```.


## To find dts peripheral definitions:
(esp32 example):
```
- zephyr/boards/espressif/esp32_devkitc/ 
- zephyr/dts/xtensa/espressif/ 
- zephyr/soc/espressif/esp32/
```


## Overlay Snippets (Copy & Paste):

### 1. GPIO LED Node:
``` dts
/ {
    aliases {
        led0 = &user_led;
    };

    user_led: user_led {
        gpios = <&gpio0 2 GPIO_ACTIVE_LOW>;
        label = "User LED";
    };
};
``` 

### 2. Button Node
``` dts
/ {
    aliases {
        sw0 = &user_button;
    };

    user_button: user_button {
        gpios = <&gpio0 0 GPIO_ACTIVE_LOW>;
        label = "User Button";
    };
};
```

### 3. I2C Sensor Node (e.g., BME280)
``` dts
&i2c0 {
    status = "okay";
    clock-frequency = <I2C_BITRATE_STANDARD>;

    bme280@76 {
        compatible = "bosch,bme280";
        reg = <0x76>;
        label = "BME280 Sensor";
    };
};
```

### 4. SPI Display Node
``` dts
&spi2 {
    status = "okay";

    my_display@0 {
        compatible = "ilitek,ili9340";
        reg = <0>;
        spi-max-frequency = <8000000>;
        label = "ILI9340 Display";
        cs-gpios = <&gpio0 5 GPIO_ACTIVE_LOW>;
        reset-gpios = <&gpio0 4 GPIO_ACTIVE_LOW>;
    };
};
```

### 5. UART Console Redirect
``` dts
&uart1 {
    status = "okay";
    current-speed = <115200>;
    pinctrl-0 = <&uart1_tx_gpio &uart1_rx_gpio>;
    pinctrl-names = "default";
};
``` 