# Bring-up i test plan

Bring-up plan opisuje proveru uređaja od firmware preflight-a do završne validacije. Cilj je da se problemi otkriju kontrolisano, korak po korak, bez mešanja hardverskih, firmware i mrežnih grešaka.

## Osnovna pravila

- Testirati jednu promenu po koraku.
- Beležiti firmware verziju, commit i korišćeni build artefakt.
- Pre rada sa mrežnim naponom koristiti bezbednu testnu postavku i odgovarajuću zaštitu.
- Prvo proveriti napajanje i komunikaciju, zatim merenje, pa tek onda uplink i Matter.

## Potrebna bench oprema

- PC sa PlatformIO okruženjem.
- USB kabl i poznat COM port.
- Referentni multimetar ili power analyzer.
- Najmanje dva poznata opterećenja.
- Bezbedna testna instalacija za mrežni napon.
- Pristup servisnom portalu.

## Faza 0: firmware preflight

1. Proveriti build.
2. Pokrenuti statičku analizu ako je dostupna.
3. Proveriti da nema kritičnih upozorenja.
4. Zabeležiti commit i build konfiguraciju.

## Faza 1: flash i prvi boot

1. Flash-ovati firmware.
2. Otvoriti serijski monitor.
3. Proveriti da nema reset petlje.
4. Proveriti osnovne logove servisa.
5. Proveriti da storage postaje spreman.

## Faza 2: servisni portal

1. Aktivirati servisni režim.
2. Povezati se na servisni Wi-Fi AP.
3. Otvoriti lokalni portal.
4. Proveriti statusne endpoint-e.
5. Proveriti najnoviji merni snapshot.

## Faza 3: merni hardver

1. Bez opterećenja proveriti da su struja i snaga blizu nule.
2. Sa poznatim opterećenjem proveriti smer i približnu vrednost struje.
3. Uporediti napon, struju, snagu i frekvenciju sa referentnim instrumentom.
4. Proveriti sirove ATM90E26 vrednosti ako su dostupne.

## Faza 4: kalibracija

1. Pokrenuti fabrički ili servisni kalibracioni tok.
2. Primijeniti referentnu tačku sa nominalnim opterećenjem.
3. Po mogućnosti primijeniti drugu tačku sa drugačijim opterećenjem.
4. Upisati kalibraciju.
5. Reboot-ovati uređaj.
6. Proveriti da su koeficijenti perzistentni.

## Faza 5: skladištenje i oporavak

1. Proveriti NVS status.
2. Proveriti checkpoint.
3. Izazvati kontrolisan restart.
4. Proveriti da se stanje oporavlja.
5. Testirati power-fail putanju ako je hardver spreman.

## Faza 6: tamper

1. Aktivirati tamper ulaz.
2. Proveriti LCD indikaciju.
3. Proveriti servisni API.
4. Proveriti da je latch perzistentan posle restarta.
5. Brisanje latch-a raditi samo kroz servisnu komandu.

## Faza 7: mreža, uplink i Matter

1. Proveriti modem init.
2. Proveriti mrežnu registraciju.
3. Proveriti slanje telemetrije.
4. Proveriti offline ponašanje.
5. Proveriti Matter status i commissioning kada je relevantno.

## Završna validaciona lista

- Firmware build je poznat i zabeležen.
- Uređaj se ne resetuje u petlji.
- LCD prikazuje osnovne vrednosti.
- Servisni portal radi.
- Merenje je u očekivanoj toleranciji.
- Kalibracija je perzistentna.
- Storage status je zdrav.
- Tamper latch radi.
- Power-fail putanja čuva emergency zapis.
- Uplink šalje očekivanu telemetriju.
