# KIT-B 0.5.1-passive65 — προσωρινό παθητικό καντράν

Στις22/11/2026 ο χρήστης ζήτησε επαναφορά γραφικών και ενδείξεων, με την
ενεργή διάγνωση αναβλημένη. Η νέα πλακέτα CAN έδειξε AUTO LOCKED_MONITOR
SP65%/SJW3 με μηδενικά errors/missed/overrun/invalid/API errors. Μετά την
αφαίρεση της επιπλέον αντίστασης και μέτρηση60Ω, η φωτογραφία έδειξε επίσης
14s VERIFY SP65/SJW3 χωρίς errors. Σύντομο άλλο τεστ SP75/SJW4 ήταν καθαρό·
δεν τεκμηριώνεται υπεροχή του από μία φωτογραφία. Επιλέγεται65/SJW3 για
συνέχεια με την ελεγχθείσα λήψη και τον ξεχωριστό καταγραφέα SavvyCAN.

## Συμπεριφορά

- Ίδιο κυκλικό καντράν, BSP, τοποθέτηση αφής, startup photo και renderer.
- Πραγματικές RPM, ταχύτητα, θερμοκρασία ψυκτικού και εισαγωγής από το
  υπάρχον `pq35::Model`, με την αποκωδικοποίηση της S3 που επιβεβαίωσε ο
  χρήστης. Δεν αλλάζουν οι τύποι ή τα validity/freshness checks.
- Λάδι, στάθμη καυσίμου και boost μένουν `--` όταν δεν υπάρχει επαληθευμένη
  πηγή. Τάση μόνο αν φτάσει έγκυρο ήδη υποστηριζόμενο frame· δεν υπόσχεται
  ανάγνωση004.2 ή τιμή από frame με διαφορετικό DLC.
- Νέο build profile `CONFIG_DASH_CAN_PASSIVE_ONLY=y`, που επιλέγει το
  `can_source_passive.cpp` αντί του ενεργού worker. TWAI πάντα LISTEN_ONLY,
  TXqueue0, fixed500kbit/s, SP65/TSEG1=12/TSEG2=7/SJW3/single sampling,
  quanta10MHz. Δεν υπάρχει CAN transmit ή ενεργός KWP reader στον worker.
- Το API απορρίπτει κάθε READ ECU ON ανεξάρτητα από σελίδα. Ο διακόπτης
  στη δεύτερη σελίδα είναι ανενεργός και γράφει `ECU requests disabled`.
  Δεν ξεκινά διαγνωστικό κανάλι, session, ACK, request ή disconnect.
- Παραμένουν START/PAUSE CAN, RX/drops/errors, latched error protection,
  και generation-ack παύση πριν από εγγραφή OTA. Η παύση δεν επανενεργοποιεί
  ECU. Η επανεκκίνηση ξεκινά μόνο παθητική λήψη.
- Wi-Fi/C6 και η αναφερθείσα αλλοίωση γραφικών παραμένουν εκκρεμή. Δεν
  αλλάζει ο Wi-Fi startup κώδικας και δεν δηλώνεται φυσική επίλυση του
  προβλήματος. Για την προσωρινή χρήση εγκατάσταση USB και κλειστό Wi-Fi.
- Το αποτυχημένο Autel session δεν αποδεικνύει βλάβη gateway. Δεν αλλάζονται
  gateway, ECU, καλωδίωση ή ενεργές διαγνωστικές υπηρεσίες.

## Εγκατάσταση

`dash-kitb-usb.bin` από τη συγκεκριμένη release, **0x2000**, Erase All OFF,
έπειτα RESET. Μόνο KIT-B10.1/P4rev3.x/32MB. Η συγχωνευμένη εγκατάσταση
αντικαθιστά το προσωρινό CAN-only/SavvyCAN firmware και επαναφέρει κανονικό
OTA layout. Περιλαμβάνει κενά NVS/otadata και μπορεί να απαιτηθούν εκ νέου
οι αποθηκευμένες ρυθμίσεις. Εμφανίζεται **0.5.1-passive65** στις ρυθμίσεις
και **SP65/SJW3 LISTEN ONLY** στο status.

Το OTA BIN είναι μόνο για φόρμα/updater, όχι συγχωνευμένο USB αρχείο.
Δεν αλλάζουν S3, SavvyCAN/capture manifests ή `leon-1p-can`.

## Build και έλεγχοι

Με ESP-IDF5.5.3, από `firmware/kitb`:

```sh
export DASH_FIRMWARE_VERSION=0.5.1-passive65
idf.py -B ../../build/kitb-ota -DSDKCONFIG=../../build/kitb-passive-sdkconfig \
  '-DSDKCONFIG_DEFAULTS=sdkconfig.defaults;sdkconfig.passive' build
```

Συσκευασία: `python scripts/package_kitb_passive.py --build build/kitb-ota
--out build/kitb-passive-download`. Ελέγχονται target/config/descriptor,
συγχωνευμένες θέσεις, hashes και ZIP πριν από τη διανομή. Επιπλέον έλεγχος
τελικού ELF ότι δεν έχει `twai_transmit`/`twai_transmit_v2` ή KWP Reader TX.

Host tests του πραγματικού passive worker: φόρτος, freshness/μη έγκυρα,
task failure, pause/resume, OTA generation ack, error latch και αποτυχία
stop. Ελέγχεται ότι κάθε προσπάθεια ενεργοποίησης ECU απορρίπτεται.
Native LVGL tests: ίδιο dashboard/pages, πραγματικά δείγματα από το baseline
CSV του χρήστη, disabled switch και παύλες για μη διαθέσιμη διάγνωση.
Οι έλεγχοι λογισμικού δεν αποτελούν φυσική δοκιμή γραφικών/CAN/Wi-Fi.


Copyright (c) 2020, Comma.ai, Inc.

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
