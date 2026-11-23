# KIT-B 0.5.2-ecu-test — χειροκίνητη διάγνωση με επαληθευμένα πακέτα

Δοκιμαστική έκδοση που εγκρίθηκε από τον χρήστη στις23/11/2026. Διατηρεί
το καντράν και τις πέντε κάρτες της δεύτερης σελίδας. Κρατά CAN500kbit/s,
TX2/RX3, SP65%/TSEG1=12/TSEG2=7/SJW3,20TQ/quanta10MHz, single sampling.
Το profile είναι `CONFIG_DASH_ECU_CAPTURED_TRIAL=y`, με ενεργό KWP reader,
και `CONFIG_DASH_CAN_PASSIVE_ONLY=n`. Η παθητική0.5.1 παραμένει διαθέσιμη.

## Πρώτη δοκιμή

1. Μόνο KIT-B10.1/P4rev3.x/32MB. Εγκατάσταση του **dash-kitb-usb.bin**
   από αυτή τη release στη **0x2000**, Erase All OFF, και RESET.
2. Η εφαρμογή γράφει **0.5.2-ecu-test**, με SP65/SJW3. Στο καντράν γίνεται
   μόνο ακρόαση. Η παθητική λήψη ξεκινά αυτόματα όπως στην0.5.1.
3. Κλείσε τη συνεδρία Engine του Autel και αποσύνδεσε το VCI από την OBD
   πριν ενεργοποιήσεις τον reader της P4. Δεν μοιραζόμαστε ενεργό κανάλι ECU.
4. Με ανοιχτό διακόπτη, πήγαινε στη δεύτερη σελίδα. Ο διακόπτης READ ECU
   παραμένει OFF. Πάτησε χειροκίνητα ON και περίμενε περίπου3s.
5. Αν επιτύχει η P4 στην ενεργή εκπομπή/σύνδεση, εμφανίζονται μόνο οι πέντε
   καθορισμένες τιμές. Αν σταματήσει, κράτησε το ακριβές μήνυμα της πάνω γραμμής.
6. OFF ή έναρξη swipe/άνοιγμα ρυθμίσεων/OTA αφοπλίζει το αίτημα και επιστρέφει
   σε listen-only. Κάθε νέα επίσκεψη χρειάζεται νέο χειροκίνητο ON.

Το θέμα Wi-Fi/C6 και της αλλοίωσης γραφικών παραμένει εκκρεμές. Για αυτή τη
δοκιμή κρατάμε κλειστό το Wi-Fi και εγκαθιστούμε μέσω USB. Το σταθερό
`kitb/latest.json` παραμένει στην0.5.1-passive65, όχι στην πειραματική0.5.2.
Το CHECK UPDATE αυτής της δοκιμής εξακολουθεί να κοιτά το σταθερό κανάλι.

## Τι επιβεβαιώθηκε από τα πραγματικά logs

Τα αρχεία παραλήφθηκαν από τον χρήστη και αναλύθηκαν χωρίς αλλαγή firmware.
Οι αριθμητικές τιμές επιβεβαιώθηκαν από τον ίδιο στο Autel:

| Block.field | KWP αίτημα | Αποκρίσεις/αιτήματα | Τιμή στιγμιότυπου |
|---|---|---|---|
|115.4|21 73|70/70|0.84bar ABS|
|106.1|21 6A|86/86|3.84 ή4.48bar|
|103.1|21 67|66/66|4.2075bar|
|004.2|21 04|29/29|12.032V|
|031.1|21 1F|115/115|0.9921875lambda, Autel0.99|

Σύνολο366/366 complete positive replies, χωρίς κενά sequence στη
συγκεκριμένη ανασύνθεση. Το031.2 έδωσε1.0, χωρίς να προστεθεί έκτη κάρτα.
Τα requests/decoded values δεν πιστοποιούν την ενεργή μετάδοση της P4 ή
μηδενικά ηλεκτρικά CAN errors. Το Autel ήταν ο ενεργός tester στα logs.

Είδαμε TP2.0 setup200→201 με negotiated tester→ECU740 και ECU→tester300
για εκείνες τις συνεδρίες, και `10 89`→`50 89`. Ο reader εξακολουθεί να
διαπραγματεύεται το TX ID και δεν το κλειδώνει αυθαίρετα στο740.
Δεν αντιγράφονται οι πρόσθετες υπηρεσίες31/1A/22 που χρησιμοποιούσε το
Autel για ταυτοποίηση. Ο δικός μας whitelist reader ζητά session10/89
και μόνο τα πέντε εγκεκριμένα21/group blocks.

## Διορθώσεις

- Block responses14 ή26bytes: τέσσερις ή οκτώ τριπλέτες `formula,A,B`.
  Τα πραγματικά004/103 περιέχουν επιπλέον μη κενές θέσεις· αυτές πλέον
  αποθηκεύονται χωρίς να ακυρώνουν το δεσμευμένο πρώτο/δεύτερο πεδίο.
  Άγνωστο/μη συμβατό selected field παραμένει μη έγκυρο, με παύλες.
- Προστέθηκε ο unitless τύπος0x1F, `A*B/2560`, για το επαληθευμένο031.1.
  Πηγές αριθμητικού τύπου: kw1281test SensorValue commit
  6b3567f68db5aa8ce205d43694ad5decf3db787d, και KLineScan formula table
  commit251490c6ac425055eb323c2446c4722b00ecc184. Δεν ενσωματώθηκε κώδικας τους.
- Διατηρούνται typed-unit/freshness checks και καμία προσομοίωση/μηδενικό
  fallback. Το boost στη δεύτερη σελίδα μένει **bar ABS**, χωρίς αφαίρεση1bar.
- Το καντράν παραμένει παθητικό και το ECU boost δεν τροφοδοτεί το κεντρικό
  όργανο. Λάδι/στάθμη και άλλες μη διαθέσιμες τιμές μένουν `--`.
- Διατηρείται όριο5block requests/s συνολικά, μία εκκρεμής αίτηση, bounded
  timeouts, single-shot frames, προστασία πολλών bus errors και stop σε
  αποτυχία. Δεν προστέθηκαν επαναλήψεις μετάδοσης ή background restart.
- Wire failure εμφανίζεται ως **ECU TX failed ID=... TEC=... ARB=... BUS=...**.
  Οι ARB/BUS είναι μετρητές της τρέχουσας εγκατάστασης driver· δεν αποτελούν
  από μόνοι τους διάγνωση ACK error ή απόδειξη της αιτίας. Οι πραγματικοί
  SDK API error codes παραμένουν ξεχωριστοί από τη γενική αποτυχία εκπομπής.

Το A8 disconnect παραμένει best effort, με όριο20ms κατά την έξοδο. Δεν
συνεχίζονται περιοδικά requests ή keepalives στο καντράν. Δεν αλλάζουν
renderers, buffers, BSP, Wi-Fi startup, CAN GPIO ή χάρτες broadcast σημάτων.

## Build / αναπαραγωγή

Με ESP-IDF5.5.3 από `firmware/kitb`:

```sh
export DASH_FIRMWARE_VERSION=0.5.2-ecu-test
idf.py -B ../../build/kitb-ecu-trial \
  -DSDKCONFIG=../../build/kitb-ecu-trial-sdkconfig \
  '-DSDKCONFIG_DEFAULTS=sdkconfig.defaults;sdkconfig.ecu_trial' build
cd ../..
python scripts/package_kitb_ecu_trial.py --build build/kitb-ecu-trial \
  --out build/kitb-ecu-trial-download
```

Οι tests περιλαμβάνουν literal payloads από τα πέντε logs, πραγματικό
reader/worker με ανεξάρτητο test peer, manual gate/exit/no restart,
unsupported/timeouts/TX failures και πραγματικό LVGL rendering. Η φυσική
δοκιμή των ενεργών requests στην P4 παραμένει το επόμενο βήμα.

Τα αρχικά CSV με τυχόν στοιχεία ταυτότητας δεν δημοσιεύονται. Το source
snapshot αποθηκεύεται μόνο στο ιδιωτικό αποθετήριο, σε ξεχωριστό tag.


Copyright (c) 2020, Comma.ai, Inc.

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

Protocol and numeric formula references. The bounded C++ implementation is written for this project.

Reference: https://github.com/I-CAN-hack/pq-flasher/tree/95d283075714c9476cacc6ef041fd810abc86f8a

MIT License

Copyright (c) 2021 Willem Melching

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.


Reference: https://github.com/gmenounos/kw1281test/tree/6b3567f68db5aa8ce205d43694ad5decf3db787d

MIT License

Copyright © 2021 Greg Menounos

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

Reference: https://github.com/fazyamir/KLineScan/tree/251490c6ac425055eb323c2446c4722b00ecc184

MIT License

Copyright (c) 2026 fazyamir

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

