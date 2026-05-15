# Pregled hardvera

Hardver Strujomerko uređaja je organizovan oko ESP32-C6 mikrokontrolera, ATM90E26 mernog kola, GSM/GPRS modema, lokalnog LCD prikaza, RTC kola, tamper ulaza i power-fail zaštite.

## Glavne komponente

| Komponenta | Uloga |
| --- | --- |
| ESP32-C6 | Glavni mikrokontroler, FreeRTOS aplikacija, Wi-Fi, Matter, NVS/SPIFFS, USB fabrički interfejs. |
| ATM90E26 | Merni IC za napon, struju, frekvenciju, aktivnu snagu i energiju. |
| Quectel M65 | GSM/GPRS modem za uplink komunikaciju ka backend sistemu. |
| LCD 2x16 preko I2C | Lokalni prikaz merenja, statusa, alarma i servisnog režima. |
| RTC DS1307 | Lokalno vreme za tarife kada mrežna sinhronizacija nije dostupna. |
| Tamper prekidač | Detekcija otvaranja kućišta ili neovlašćenog pristupa. |
| Power-fail signal | Detekcija gubitka glavnog napajanja. |
| Superkondenzator | Kratko rezervno napajanje za hitno čuvanje stanja. |

## Hardverski blokovi

| Blok | Namena |
| --- | --- |
| Merni front-end | Prilagođavanje naponskog i strujnog signala za ATM90E26. |
| Merni procesor | Digitalna obrada električnih veličina i akumulacija energije. |
| Glavna logika | Firmware, lokalna obrada, komunikacija i skladištenje. |
| Provajderska veza | GSM/GPRS prenos telemetrije. |
| Lokalni UI | LCD ekran, taster i osnovne statusne indikacije. |
| Vreme i tarife | RTC, mrežna sinhronizacija vremena i izbor aktivne tarife. |
| Zaštita i događaji | Tamper ulaz, power-fail ulaz i hitno čuvanje podataka. |
| Napajanje | Stabilne naponske grane za logiku, merno kolo i modem. |

## Merni front-end

ATM90E26 prima naponski i strujni signal preko odgovarajućeg front-end kola. Naponski signal dolazi preko naponskog delitelja, a strujni signal preko strujnog transformatora. Kalibracija se radi kroz merno kolo i firmware sloj kako bi se sirove vrednosti pretvorile u inženjerske jedinice.

## Napajanje i power-fail režim

Uređaj mora stabilno raditi iz glavnog napajanja, ali mora imati i kontrolisano ponašanje pri gubitku napona. Power-fail signal aktivira hitnu putanju u firmware-u, a superkondenzator obezbeđuje kratak vremenski prozor za upis kritičnih podataka u NVS.

## Tamper zaštita

Tamper ulaz detektuje otvaranje kućišta. Kada se aktivira, firmware postavlja latched stanje, čuva ga u perzistentnoj memoriji i prikazuje ga kroz lokalni UI, servisni API i telemetriju.

## Validacija hardvera

Hardverska validacija treba da obuhvati:

- proveru naponskih grana pod opterećenjem,
- proveru LCD i I2C komunikacije,
- proveru SPI komunikacije sa ATM90E26,
- proveru UART komunikacije sa modemom,
- proveru tamper ulaza,
- proveru power-fail signala,
- proveru ponašanja pri kratkotrajnom i potpunom gubitku napajanja.
