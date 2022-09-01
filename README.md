# Результаты исследования

Итак, это пример который я залил на HW MyBox Master. 

Всё железо работает исправно:
- пины `GPIO 12` и `GPIO 14` в режиме USART отрабатывают корректно, но может вылазить ошибка: 

```
rst:0x10 (RTCWDT_RTC_RESET),boot:0x33 (SPI_FAST_FLASH_BOOT) flash read err, 1000
```

 когда подключаю сперва USB2SERIAL флешку, а потом кабель прошивки. Это связано с тем, что Gpio12 — это контакт начальной загрузки, он может установить (неправильное) напряжение флэш-памяти и включить внутренний ldo. На форуме много тем по этому поводу [тыц](https://www.esp32.com/viewtopic.php?t=9941) ;
- USART через MAX3485CSA отрабатывает исправно (правый терминал на [рис.1](img\photo_2022-09-01_02-06-19.jpg) );
- RS-485 отрабатывает корректно через 485ю флешку[рис.2](img\photo_2022-09-01_02-06-38.jpg), осциллограмма корректная [рис.3](img\photo_2022-09-01_01-55-49.jpg); 


В итоге, я внес фикс в ветку `modbus_test`, но еще не тестил его - [линк](https://github.com/PetrZapletal/ESP-EVSE3upgrade/commit/aaa94becef42dfe79100171848496d0885c0e41d).
Добавил в components папку soc с либами, но не чистил лишнее.
Дописал в `main/modbus_master.c` функцию `remap_uarts()` и заинитил ее после сета пинов, там видно.
Теоретически, даже если его так запустить то будет работать. Но я єще не до конца понимаю внутрянку настроек проекта, и не смог запустить ирт.

Важные инклуды:
```
#include "driver/uart.h"
#include "soc/uart_periph.h"
#include "esp_rom_gpio.h"
#include "driver/gpio.h"
#include "hal/gpio_hal.h"
```

Материалы: 
- [Линк](https://micro-pi.ru/terminal-1-9b-%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%B0%D0%B5%D0%BC-com-%D0%BF%D0%BE%D1%80%D1%82%D0%BE%D0%BC/) на терминал как у меня, настройки на [скрине](img\photo_2022-09-01_02-06-19.jpg);
- [Доки](https://www.espressif.com/sites/default/files/documentation/esp32_technical_reference_manual_en.pdf) референс мануала по которому работал (Глава 4, IO_MUX and GPIO Matrix (GPIO, IO_MUX), если что);
- Ниже, родной ридми примера репла...


# UART REPL Example

(See the README.md file in the upper level 'examples' directory for more information about examples.)

This example demonstrates how to use REPL console on a different UART than the default one.
It also shows how to connect these two UART together, either for testing or for sending commands
without any human interaction.

## How to use example

### Hardware Required

The example can be run on any ESP board that have at least 2 UARTs. The development board shall be connected to a
PC with a single USB cable for flashing and monitoring. If you are willing to monitor the console UART, you may use
a 3.3V compatible USB-to-Serial dongle on its GPIO pin.

### Setup the Hardware

No external connection is needed in order to run the example. However, as stated before, if you are willing to see what
is going on on the second UART (console UART), you can connect pins CONSOLE_UART_TX_PIN (5 by default) and
CONSOLE_UART_RX_PIN (4 by default) to a Serial-to-USB adapter.

### Configure the project

The default values, located at the top of `main/uart_repl_example_main.c` can be changed such as:
DEFAULT_UART_CHANNEL, CONSOLE_UART_CHANNEL, DEFAULT_UART_RX_PIN, DEFAULT_UART_TX_PIN, CONSOLE_UART_RX_PIN,
CONSOLE_UART_TX_PIN, UARTS_BAUD_RATE, TASK_STACK_SIZE, and READ_BUF_SIZE.

### Build and Flash

Build the project and flash it to the board, then run monitor tool to view default UART's serial output:

```
idf.py -p PORT flash monitor
```

(To exit the serial monitor, type ``Ctrl-]``.)

See the Getting Started Guide for full steps to configure and use ESP-IDF to build projects.

## Example Output

The example will set up the default UART to use DEFAULT_UART_RX_PIN and DEFAULT_UART_TX_PIN. Then, it will set up
the REPL console on the second UART. Finally, it will connect both UARTs together in order to let default UART
be able to send commands and receive replies to and from the console UART.

Here is a diagram of what UARTs will look like:

```
                  UART default      UART console

USB monitoring <------ TX -----------> RX----+
                                             v
                                       Parse command
                                     and output result
                                             |                 Optional 3.3V
                       RX <----------- TX<---+  (----------->) Serial-to-USB
                                                                  Adapter
```

If everything goes fine, the output on default UART should be "Result: Success". Else, it should be "Result: Failure".