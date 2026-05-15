# Pinout i komponente

Ovaj dokument je radna referenca za povezivanje glavnih hardverskih blokova. Konačne GPIO brojeve treba proveriti u aktuelnom firmware repozitorijumu pre izrade šeme, PCB-a ili test plana.

## Grupe signala

| Grupa | Tipični signali |
| --- | --- |
| ATM90E26 | SPI `MOSI`, `MISO`, `SCLK`, `CS`, prekid ili status signal ako se koristi. |
| LCD | I2C `SDA`, `SCL`, napajanje i backlight kontrola ako postoji. |
| RTC | I2C `SDA`, `SCL`, baterijsko napajanje ako je predviđeno. |
| Modem | UART `TX`, `RX`, `PWRKEY`, reset, status i napajanje modema. |
| Tamper | Digitalni ulaz sa debouncing logikom u firmware-u. |
| Power-fail | Digitalni ulaz iz komparatora ili nadzornog kola napajanja. |
| Taster | Digitalni ulaz za promenu ekrana i servisni režim. |
| USB | USB Serial/JTAG za flash, logove i fabrički protokol. |

## Pravila za šemu

- Merno kolo i analogni front-end treba fizički odvojiti od bučnih digitalnih i modem sekcija.
- Modem mora imati napajanje projektovano za strujne pikove pri GSM transmisiji.
- I2C magistralu treba završiti odgovarajućim pull-up otpornicima.
- Tamper i power-fail ulazi treba da imaju definisano stanje pri resetu.
- USB fabrički interfejs mora biti dostupan u proizvodnom i servisnom procesu.

## Šta dodati kada stignu slike

Kada budu dostupne slike ili šeme, ovde treba dodati:

- blok dijagram uređaja,
- pinout tabelu,
- šemu mernog front-end-a,
- raspored konektora,
- fotografije prototipa,
- fotografije testne postavke.
