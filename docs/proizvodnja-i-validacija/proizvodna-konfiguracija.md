# Proizvodna konfiguracija i kalibracija

Proizvodni proces treba da koristi jednu proizvodnu firmware sliku za sve uređaje. Individualni podaci, fleet konfiguracija i kalibracija upisuju se naknadno preko USB proizvodnog protokola ili generisane NVS particije.

## Klase konfiguracije

| Klasa | Opis |
| --- | --- |
| Fleet konfiguracija | Zajednička podešavanja za seriju uređaja: backend host, APN, vremenska zona, servisni AP, timeout-i i model. |
| Unit konfiguracija | Per-device podaci: serijski broj, claim code, fabrički metapodaci i eventualno per-unit kredencijali. |
| Kalibracija | Koeficijenti i registri dobijeni merenjem referentnih tačaka. |
| Runtime stanje | Checkpoint, uplink stanje, tamper latch i trenutni operativni podaci. |

## Preporučeni proizvodni tok

1. Flash proizvodne firmware slike.
2. Pokretanje USB fabričkog protokola.
3. Upis fleet konfiguracije.
4. Generisanje i upis unit konfiguracije.
5. Restart uređaja.
6. Kalibracija kroz referentna opterećenja.
7. Verifikacija zapisa i CRC vrednosti.
8. Upis proizvodnog izveštaja.
9. Zaključavanje proizvodnog stanja ako je uređaj spreman za isporuku.

## Kalibracija

Kalibracija treba da koristi najmanje jednu poznatu referentnu tačku, a poželjno dve različite tačke opterećenja. Za svaku tačku treba zabeležiti:

- referentni napon,
- referentnu struju,
- referentnu aktivnu snagu,
- frekvenciju,
- uslove testiranja,
- firmware verziju,
- serijski broj uređaja.

## Izveštaj

Proizvodni izveštaj treba da sadrži:

- serijski broj,
- batch ID,
- operatora ili stanicu,
- firmware commit,
- korišćenu fleet konfiguraciju,
- kalibracione rezultate,
- validacione tolerancije,
- vreme završetka procesa.
