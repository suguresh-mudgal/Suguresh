/* # Suguresh
TASK */
#include <stdint.h>
#include <stdbool.h>
#include "inc/hw_types.h"
#include "inc/hw_memmap.h"
#include "inc/hw_uart.h"
#include "inc/hw_gpio.h"
#include "inc/hw_sysctl.h"
#include "driverlib/sysctl.h"
#include "driverlib/gpio.h"
#include "driverlib/pin_map.h"
#include "driverlib/pwm.h"
#include "driverlib/uart.h"
#include "driverlib/eeprom.h"
#include "driverlib/pin_map.h"
#include "utils/uartstdio.h"

#define DATA_SIZE 1000  // The data size in bytes

// Function to initialize UART0
void init_UART0(void) {
    // Enable the peripherals for UART0 and GPIO
    SysCtlPeripheralEnable(SYSCTL_PERIPH_GPIOA);
    SysCtlPeripheralEnable(SYSCTL_PERIPH_UART0);

    // Configure UART0 pins: PA0 -> U0RX, PA1 -> U0TX
    GPIOPinConfigure(GPIO_PA0_U0RX);
    GPIOPinConfigure(GPIO_PA1_U0TX);
    GPIOPinTypeUART(GPIO_PORTA_BASE, GPIO_PIN_0 | GPIO_PIN_1);

    // Set the UART parameters: 2400 baud, 8 data bits, 1 stop bit
    UARTConfigSetExpClk(UART0_BASE, SysCtlClockGet(), 2400, (UART_CONFIG_WLEN_8 | UART_CONFIG_STOP_ONE | UART_CONFIG_PAR_NONE));
}

// Function to send data via UART
void uart_send_data(const char *data) {
    while (*data) {
        UARTCharPut(UART0_BASE, *data);  // Send each character
        data++;
    }
}

// Function to receive data via UART
void uart_receive_data(char *buffer, uint32_t length) {
    uint32_t i = 0;
    while (i < length) {
        if (UARTCharsAvail(UART0_BASE)) {
            buffer[i++] = UARTCharGet(UART0_BASE);  // Read a character
        }
    }
}

// Function to store received data in EEPROM
void store_data_in_eeprom(const char *data, uint32_t length) {
    uint32_t i;
    for (i = 0; i < length; i++) {
        EEPROMProgram(&data[i], i, 1);  // Store one byte at a time
    }
}

// Function to retrieve data from EEPROM
void retrieve_data_from_eeprom(char *buffer, uint32_t length) {
    uint32_t i;
    for (i = 0; i < length; i++) {
        buffer[i] = *(char *)(EEPROM_BASE + i);  // Retrieve one byte at a time
    }
}

// Main function to handle UART communication
int main(void) {
    char received_data[DATA_SIZE];
    char send_back_data[DATA_SIZE];

    // Set the system clock to 50 MHz
    SysCtlClockSet(SYSCTL_SYSDIV_4 | SYSCTL_USE_PLL | SYSCTL_OSC_MAIN);

    // Initialize UART0 for communication
    init_UART0();

    // Receive data from PC
    uart_receive_data(received_data, DATA_SIZE);

    // Store the received data in EEPROM
    store_data_in_eeprom(received_data, DATA_SIZE);

    // Delay to simulate EEPROM write
    SysCtlDelay(100000);

    // Retrieve the data from EEPROM
    retrieve_data_from_eeprom(send_back_data, DATA_SIZE);

    // Send the retrieved data back to the PC
    uart_send_data(send_back_data);

    return 0;
}
