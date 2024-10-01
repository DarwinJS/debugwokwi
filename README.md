
1. In the same execution environment the command `wokwi-cli . --scenario blink.test.yaml --timeout 30000` works in the subdirectory 'pico', but not in 'picow'
2. The difference in test files is intentional, picow_blink.uf2 outputs all upper case
3. When I upload my firmware to the interactive simulator - it does not blink the light with either of the picow/*diagram.json files
4. When the below code is "built" in the simulator - it works with both of the picow/*diagram.json files
5. Both blink.uf2 and picow_blink.uf2 were built with the same build tools container
    - build container source for both: https://gitlab.com/guided-explorations/embedded/ci-components/arm-gnu-embdd-rpi-pico-sdk/-/blob/main/Dockerfile?ref_type=heads
    - blink pico build script: https://gitlab.com/guided-explorations/embedded/ci-components/working-code-examples/wokwi-for-raspberry-pi/-/blob/main/.gitlab-ci.yml?ref_type=heads
    - blink picow build script: https://gitlab.com/dsanoy-demo/experiments/pico-examples-5/-/blob/4-add-wokwi-ci/.gitlab-ci.yml?ref_type=heads#L70-75


## Code

The below is also in https://gitlab.com/dsanoy-demo/experiments/pico-examples-5/-/tree/4-add-wokwi-ci/pico_w/wifi/blink?ref_type=heads

#### CMakeLists.txt

add_executable(picow_blink
        picow_blink.c
        )

target_link_libraries(picow_blink
        pico_stdlib              # for core functionality
        pico_stdio_usb
        pico_cyw43_arch_none     # we need Wifi to access the GPIO, but we don't need anything else
        )

# create map/bin/hex file etc.
pico_add_extra_outputs(picow_blink)

# add url via pico_set_program_url
example_auto_set_url(picow_blink)
pico_enable_stdio_usb(picow_blink 1)
pico_enable_stdio_uart(picow_blink 0)


#### main.c

#include "pico/stdlib.h"
#include "pico/cyw43_arch.h"
#include <stdio.h>

int main() {
    stdio_init_all();
    if (cyw43_arch_init()) {
        printf("Wi-Fi init failed");
        return -1;
    }
    while (true) {
        cyw43_arch_gpio_put(CYW43_WL_GPIO_LED_PIN, 1);
        printf("LED ON\n");
        sleep_ms(444);
        cyw43_arch_gpio_put(CYW43_WL_GPIO_LED_PIN, 0);
        printf("LED OFF\n");
        sleep_ms(444);
    }
}



## Test Results

root@480058cb2434:~/wokwi/picow# wokwi-cli . --scenario blink.test.yaml --timeout 30000
Wokwi CLI v0.14.2 (94a2f9cec3b5)
Connected to Wokwi Simulation API 1.0.0-20241001-gb8f5c5e1
Starting simulation...
Timeout: simulation did not finish in 30000ms

root@480058cb2434:~/wokwi/picow# cd ../pico
root@480058cb2434:~/wokwi/pico# wokwi-cli . --scenario blink.test.yaml --timeout 30000
Wokwi CLI v0.14.2 (94a2f9cec3b5)
Connected to Wokwi Simulation API 1.0.0-20241001-gb8f5c5e1
Starting simulation...
LED on
[Blinky test] Expected text matched: "LED on"
[Blinky test] expect-pin led1:A == 1
LED off
[Blinky test] Expected text matched: "LED off"
[Blinky test] expect-pin led1:A == 0
[Blinky test] Scenario completed successfully
