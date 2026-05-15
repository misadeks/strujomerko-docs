# Pinout

Ovaj dokument je referenca za opis pinova glavnog mikrokontrolera 

## ESP32 pinout

| PIN      | Deskrpipcija signala               |
| -------- | ---------------------------------- |
| `pin 5`  | Dugme, aktivan u '0'               |
| `pin 6`  | I2C SDA, za ekran i RTC            |
| `pin 7`  | I2C SCL, za ekran i RTC            |
| `pin 0`  | UART RX, komunikacija sa modemom   |
| `pin 1`  | UART TX, komunikacija sa modemom   |
| `pin 10` | SPI MOSI, komunikacija sa ATM90E26 |
| `pin 11` | SPI MISO, komunikacija sa ATM90E26 |
| `pin 2`  | SPI CLK, komunikacija sa ATM90E26  |
| `pin 3`  | SPI CS, komunikacija sa ATM90E26   |
| `pin 23` | PWRKEY, paljenje modema            |
| `pin 21` | Power Loss, aktivan u '1'          |
| `pin 22` | Tampering, akrivan u '1'           |
| `pin 18` | IRQ                                |
| `pin 19` | CF1                                |
| `pin 20` | Wout                               |

# 
