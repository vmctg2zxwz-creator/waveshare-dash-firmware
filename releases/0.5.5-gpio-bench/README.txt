# KIT-B 0.5.5-gpio-bench — απομονωμένος έλεγχος πομπού/δέκτη

Ξεχωριστό, ρητά εγκεκριμένο firmware πάγκου για ESP32-P4-WIFI6-KIT-B /
10.1-DSI-TOUCH-A. Ο χρήστης θέλει βασικό έλεγχο της μπλε SN65HVD230 χωρίς
εξωτερικό παλμογράφο. Δεν είναι firmware καντράν ή ενεργού διαγνωστικού.

## Συνδέσεις και εγκατάσταση

- **CANH/CANL αποσυνδεδεμένα από το αυτοκίνητο.** Δεν εκτελείται σε ενεργό
  δίκτυο οχήματος. Η επιβεβαίωση στην οθόνη είναι δήλωση του χρήστη, όχι
  αυτόματος έλεγχος απομόνωσης.
- Στο απομονωμένο module, τοπική αντίσταση περίπου **120Ω μεταξύ CANH–CANL**.
  Στην προηγούμενη μέτρηση, χωρίς R2, ο χρήστης βρήκε50kΩ χωρίς φορτίο.
- Διατηρούνται3V3, κοινόGND, GPIO2→CTX και CRX→GPIO3 της P4.
  Ο χρήστης επιβεβαίωσε3.3V, Rs72mV και συνέχεια GPIO2→CTX0Ω.
- USB merged image **dash-kitb-usb.bin στη0x2000**, **Erase All OFF**, RESET.
- Πριν επιστρέψει στο αυτοκίνητο: αφαίρεση της προσωρινής αντίστασης και
  επαναφορά κανονικού firmware. Αυτό το firmware προορίζεται μόνο για πάγκο.

## Χειρισμός

1. Εμφανίζεται η οθόνη **P4 / TRANSCEIVER BENCH**, χωρίς φωτογραφία εκκίνησης,
   καντράν, READ ECU, Wi-Fi ή μενού update.
2. Με ολοκληρωμένη απομόνωση/φορτίο, επίλεξε την επιβεβαίωση
   **I confirm: isolated module, 120 ohm load fitted**.
3. Πάτησε **TEST MODULE μία φορά**. Η επιβεβαίωση καθαρίζεται μετά την
   αποδοχή του αιτήματος. Η νέα δοκιμή απαιτεί νέα χειροκίνητη επιβεβαίωση.
4. Φωτογράφισε ολόκληρη την οθόνη με τις γραμμές HIGH/LOW και το αποτέλεσμα.
   Τα αποτελέσματα διατηρούνται μέχρι νέο χειροκίνητο τεστ ή RESET.
   Τα ίδια καταγράφονται και στο USB serial115200.

Δεν ξεκινά τεστ σε εκκίνηση, RESET, επανεμφάνιση της οθόνης ή αποτυχία.
Δεν υπάρχει αυτόματη επανάληψη. Τα8δείγματα ανά φάση συνοψίζονται σε
HIGH/LOW/MIXED. Οι πλήρεις μετρητές HIGH φαίνονται στο serial.

## Τι κάνει η P4

Ο TWAI και ο reader ECU δεν αρχικοποιούνται ούτε καλούνται. Δεν υπάρχουν
CAN frames, KWP/TP2.0, RMT capture ή εκκίνηση Wi-Fi. Το GPIO2 λειτουργεί
προσωρινά ως απλή έξοδος με ενεργή ανάγνωση pad (INPUT_OUTPUT), όχι ως
TWAI TX. Το GPIO3 είναι είσοδος με ασθενές εσωτερικό pull-up. Αυτό δίνει
καθορισμένο HIGH σε ανοικτή γραμμή RX, η οποία κατόπιν αποτυγχάνει στο LOW.

Η έξοδος προφορτώνεται HIGH πριν ενεργοποιηθεί. Ο worker ξεκινά αφού
ολοκληρωθεί η επιβεβαίωση OTA/τυχόν boot flash write. Κατά την αναμονή
δεν δίνει LOW. Η φωτεινότητα διατηρείται από τον υπάρχοντα μηχανισμό NVS.

Μετά από χειροκίνητο αίτημα:
- Χονδρικός έλεγχος ότι το RX διαβάζεται HIGH σε10διαδοχικά δείγματα,
  με απόσταση ενός RTOS tick. Αν παρατηρηθεί LOW, δεν στέλνεται LOW παλμός.
  Αυτό δεν αποδεικνύει ηλεκτρική απομόνωση ή την τιμή της αντίστασης.
- Έως3κύκλοι HIGH-before → LOW → HIGH-after.
- Κάθε φάση θέτει το GPIO2, περιμένει10us και παίρνει8ζεύγη αναγνώσεων
  GPIO2/3 με2usανάμεσά τους. Υπάρχει πρόσθετος χρόνος εκτέλεσης/API· δεν
  δηλώνεται ακριβές hard-real-time πλάτος26us.
- Το μικρό critical section προστατεύει την κάθε φάση από τοπική
  προεκτόπιση. Δεν ισχυριζόμαστε μέτρηση αναλογικού σήματος ή καθυστέρησηςns.
- Στην πρώτη ασυμφωνία σταματούν οι επόμενοι κύκλοι. Μετά από αποτυχημένο
  LOW ελέγχεται πρώτα η επιστροφή HIGH. Δεν δοκιμάζεται ξανά το LOW.
- Πάντα εκτελείται τελική εντολή HIGH και έλεγχος pad. Αποτυχία
  επαναφοράς εμφανίζεται χωριστά.
- UI callbacks υποβάλλουν μόνο αίτημα. Όλες οι εγγραφές στα pins γίνονται
  στον μοναδικό worker. Δεν υπάρχουν GPIO pulses από LVGL/ISR/network.

## Ερμηνεία

- **RX followed TX in all 3 cycles**: επιτυχής βασικός ψηφιακός έλεγχος
  στα pins της P4 υπό το συγκεκριμένο φορτίο και συνθήκες.
- **TX pad did not follow the command**: το διαβασμένο GPIO2 δεν συμφωνεί
  με την εντολή· δεν αποδίδεται αυτόματα στο module.
- **RX did not follow TX / MIXED**: η βασική διαδρομή επιστροφής δεν
  ακολούθησε σταθερά. Πιθανές αιτίες περιλαμβάνουν module, γραμμή RX,
  κοινόGND/τροφοδοσία ή φόρτιση. Δεν δηλώνεται συγκεκριμένη βλάβη.
- **RX LOW/active before test**: το τεστ δεν έστειλε LOW.
- GPIO/setup/restore errors παρουσιάζονται ως errors, όχι ως πραγματικές
  μετρήσεις LOW ή ως επιτυχία.

Η επιτυχία δεν πιστοποιεί500kbit/s, propagation delay/sample point,
αναλογικά επίπεδα CANH/L, κανονικό frame ή πρωτόκολλο ECU. Απομονώνει έναν
βασικό έλεγχο της διαδρομής από τον ελεγκτή/χρονισμό CAN.

## Build

Από firmware/kitb, με το κλειδωμένο ESP-IDF5.5.3/BSP της KIT-B:

~~~sh
export DASH_FIRMWARE_VERSION=0.5.5-gpio-bench
idf.py -B ../../build/kitb-gpio-bench \
  -DSDKCONFIG=../../build/kitb-gpio-bench-sdkconfig \
  '-DSDKCONFIG_DEFAULTS=sdkconfig.defaults;sdkconfig.gpio_bench' build
~~~

Από τη ρίζα:

~~~sh
python scripts/package_kitb_gpio_bench.py --build build/kitb-gpio-bench \
  --out build/kitb-gpio-bench-download
~~~

Ο packager επιβεβαιώνει board/config/version/offsets και ότι το ELF
περιέχει τον bench worker χωρίς TWAI/RMT/network APIs. Το δημόσιο trial
είναι ξεχωριστό immutable release directory. Το stable kitb/latest.json
παραμένει0.5.1-passive65. Απαιτείται φυσική δοκιμή στην P4.



### Αυτόματη εκκίνηση ESP-Hosted

Η βιβλιοθήκη ESP-Hosted1.4.7 είχε constructor που καλούσε esp_hosted_init
πριν το app_main, ακόμη χωρίς Wi-Fi API στην εφαρμογή. Μόνο στο bench
profile εξαιρείται από το target η συγκεκριμένη startup compilation unit,
με SHA256 guard db7841e0da11b48e5999f218139c28f3807240ea66dde7c16ba5a3049768b6e8.
Δεν τροποποιούνται managed αρχεία ή η συμπεριφορά των κανονικών profiles.
Ο packager απορρίπτει build όπου παραμένει η startup unit ή το esp_hosted_init
στο ELF. Έτσι αποτρέπεται και η αυτόματη ενεργοποίηση του C6 transport.


Copyright (c) 2020, Comma.ai, Inc.

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
