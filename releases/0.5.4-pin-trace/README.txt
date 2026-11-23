# KIT-B 0.5.4-pin-trace — ψηφιακή καταγραφή GPIO2/3

Ρητά εγκεκριμένη διαγνωστική έκδοση για ESP32-P4-WIFI6-KIT-B / 10.1-DSI-TOUCH-A.
Βάση0.5.3-can-trace. Η φυσική προηγούμενη δοκιμή έδειξε PRE0 και πρώτο POST
`TX BIT SOF(3), TEC24, REC0`, τελικό `ID200 TEC48 ARB0 BUS2`.
Ο χρήστης επιβεβαίωσε μπλε SN65HVD230, τροφοδοσία3.3V, Rs72mV και
συνέχεια GPIO2→CTX0Ω. Δεν συνάγεται από αυτά βλάβη του module ή της P4.

## Εγκατάσταση / επόμενη μέτρηση

1. Το `dash-kitb-usb.bin` γράφεται στη **0x2000**, **Erase All OFF**, μετά RESET.
2. Κράτησε την ίδια μπλε πλακέτα, καλώδια και αντιστάσεις. Δεν χρειάζονται
   εξωτερικός παλμογράφος, επιπλέον σύνδεση ή δοκιμαστική γεφύρωση.
3. Διακόπτης αυτοκινήτου ανοιχτός, Autel/VCI αποσυνδεδεμένο, Wi-Fi κλειστό.
4. Δεύτερη σελίδα → **READ ECU ON μία φορά**. Όταν σταματήσει, φωτογράφισε
   ολόκληρη τη δεύτερη σελίδα: επάνω error, νέο πλαίσιο δεξιά από τη λάμδα
   και τις δύο γραμμές PRE/POST στο κάτω μέρος.

Δεν υπόσχεται διόρθωση της επικοινωνίας ECU. Προσθέτει στοιχεία για τη
διαδρομή εκπομπής. Το πρόβλημα Wi-Fi/C6/γραφικών παραμένει εκκρεμές.
Το σταθερό `kitb/latest.json` παραμένει0.5.1-passive65· εγκατάσταση trial μέσω USB.

## Τι μετράει

Δύο **receive-only RMT** κανάλια της P4, στα υπάρχοντα GPIO2/TX καιGPIO3/RX,
καταγράφουν ψηφιακές στάθμες και διάρκειες με resolution10MHz (0.1us/tick).
Το input buffer του GPIO2 ενεργοποιείται προσωρινά, ενώ η έξοδος παραμένει
συνδεδεμένη στον TWAI. Δεν χρησιμοποιείται RMT TX και δεν παράγονται test frames.

- `TX2/RX3`: παρατηρημένες εναλλαγές στάθμης, συνολικός χρόνος LOW και
  διάρκεια πρώτου LOW. Δεν είναι μετρητές επιτυχημένων CAN frames/απαντήσεων.
- `OE/OUT/INV/OD`: πραγματική ρύθμιση GPIO2 πριν από την πρώτη TX API.
  Για τη συγκεκριμένη διαδρομή αναμένονται OE1, OUT80 (TWAI0), INV0, OD0.
  Το `kept YES` σημαίνει διατήρηση της ρύθμισης εξόδου κατά την προσθήκη
  της παρακολούθησης· δεν σημαίνει επιτυχημένη εκπομπή ή σωστό hardware.
- `RX input/INV`: πραγματικό GPIO matrix input του TWAI0 κατά την κλήση,
  αναμενόμενα3/0. Το matrix-enable καταγράφεται στο JSON.
- `cleanup`:0 όταν οι ενέργειες διακοπής/αποδέσμευσης της καταγραφής επέτυχαν.

Καταστάσεις:

- `DONE`: ο driver ολοκλήρωσε καταγραφή με σύμβολα, χωρίς πλήρες user buffer.
- `LIMIT`: γέμισαν οι96θέσεις συμβόλων· δεν θεωρείται ολόκληρο το σήμα.
- `INCOMPLETE`: η λήψη διακόπηκε πριν παραληφθεί callback ολοκλήρωσης. Δεν
  διαβάζεται buffer που μπορεί ακόμη να γράφεται και δεν παρουσιάζεται σαν
  «TX χωρίς παλμούς». Χρειάζεται νέα ερμηνεία/μέτρηση, όχι συμπέρασμα βλάβης.
- `EARLY`: το callback καταγραφής ήρθε πριν από το σημείο κλήσης TX· δεν
  χρησιμοποιείται ως απόδειξη απουσίας παλμών της συγκεκριμένης αποστολής.
- `EMPTY`: ο driver ολοκλήρωσε με μηδέν σύμβολα· δεν αποδεικνύει σταθερό pin.
- `WAIT`: δεν έχει ολοκληρωθεί ακόμη η καταγραφή.

Η παρακολούθηση οπλίζεται μία φορά ακριβώς πριν από την πρώτη υπάρχουσα
`twai_transmit`, μετά τη χειροκίνητη ενεργοποίηση. Το RMT σταματά σε500us
σταθερής στάθμης. Οι πόροι αποδεσμεύονται όταν ολοκληρωθούν και τα δύο
κανάλια, στο επόμενο worker poll μετά5ms από την κλήση ή κατά την έξοδο.
Η προθεσμία είναι συνεργατική, όχι εγγυημένο deadline hard real-time.
Εάν αποτύχει αποδέσμευση, ο πόρος δεν ελευθερώνεται ενώ είναι ενεργός·
καταγράφεται cleanup error και εμποδίζεται νέα κατανομή όσο παραμένει ενεργός.

## Περιορισμοί

Η μέτρηση βλέπει ψηφιακά pads της P4, όχι αναλογικά CANH/CANL ή τάση επάνω
στο pin του απομακρυσμένου module. Η επιβεβαιωμένη συνέχεια βοηθά, αλλά
δεν αποκλείει κάθε διακοπτόμενη/υπό φορτίο ηλεκτρική βλάβη.
Η ύπαρξη παλμών στο GPIO3 μπορεί να προέρχεται από άλλο node. Οι παλμοί TX
μπορεί να περιλαμβάνουν αυτόματα ACK/error flags του CAN. Δεν βαφτίζονται
αυτόματα σωστό request/επιτυχής επιστροφή του συγκεκριμένου frame.

Τα δύο RMT RX ξεκινούν διαδοχικά, με timestamp brackets πριν/μετά από κάθε
έναρξη. Τα GPIO2/3 δεν είναι δύο συγχρονισμένα κανάλια παλμογράφου. Οι
χρόνοι callback περιλαμβάνουν interrupt latency. Δεν υπολογίζεται από αυτά
με βεβαιότητα καθυστέρηση transceiver σεns. Χρόνοι modulo uint32us, για
τη σύντομη δοκιμή. Διατηρούνται πρώτες16σειρές στάθμης/διάρκειας και
aggregate stats από μέχρι96σύμβολα ανά pin στο υπάρχον status JSON.

## Προστασία υπάρχουσας λειτουργίας

Ίδια500kbit/s, SP65%, SJW3, TX2/RX3, single-shot/fail-stop και requests
TP2.0/KWP2000 των0.5.2/0.5.3. Όργανα μόνο ακρόαση, READ ECU μόνο χειροκίνητα
στη δεύτερη σελίδα για115.4/106.1/103.1/004.2/031.1. Χωρίς νέο test frame,
αυτόματο restart ή αλλαγή timeout του ECU. Υπάρχει μικρό πρόσθετο κόστος
ενεργοποίησης/interrupts μόνο στη διαγνωστική προσπάθεια.

RMT resources δημιουργούνται μόνο μετά από manual ON, πριν το NORMAL
`twai_start`. Αν αποτύχει η δημιουργία/όπλιση ή αλλάξει η ρύθμιση TX,
σταματά ο reader χωρίς αποστολή του αρχικού request.
Το SDK RMT RX ενεργοποιεί input και pull-up· επαναφέρεται αμέσως η αρχική
pull-up, επιβεβαιώνονται output signal/OE/inversions/drive/open-drain/
function/pull/sleep και στην έξοδο επανέρχονται οι αρχικές input/pull-up.
Δεν καλούνται `gpio_set_direction`, output-enable, GPIO reset ή matrix-out
από τον καταγραφέα. Τα RMT flags `io_loop_back`, `invert_in`, `with_dma`
παραμένουνfalse. Οι callbacks γράφουν μόνο lock-free atomic completion
metadata σε στατικά DRAM buffers, με κώδικα IRAM και χωρίς rendering/logging.

## Build

Με ESP-IDF5.5.3, από firmware/kitb:

```sh
export DASH_FIRMWARE_VERSION=0.5.4-pin-trace
idf.py -B ../../build/kitb-pin-trace \
  -DSDKCONFIG=../../build/kitb-pin-trace-sdkconfig \
  '-DSDKCONFIG_DEFAULTS=sdkconfig.defaults;sdkconfig.pin_trace' build
```

Από τη ρίζα:

```sh
python scripts/package_kitb_ecu_trial.py --build build/kitb-pin-trace \
  --out build/kitb-pin-trace-download
```

Προαπαιτεί τα επιβεβαιωμένα BSP/dependency lock και το ακριβές HAL observation
patch της0.5.3. Η ψηφιακή καταγραφή χρειάζεται ακόμη φυσική δοκιμή στην P4.


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

