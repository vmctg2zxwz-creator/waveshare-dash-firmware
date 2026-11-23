# KIT-B 0.5.3-can-trace — ακριβής καταγραφή σφάλματος εκπομπής

Δοκιμαστική έκδοση για ESP32-P4-WIFI6-KIT-B / 10.1-DSI-TOUCH-A.
Βάση: 0.5.2-ecu-test. Ο χρήστης ενέκρινε ρητά αυτή τη διαγνωστική προσθήκη.
Η 0.5.2 στη φυσική δοκιμή σταμάτησε με `ID=200 TEC=48 ARB=0 BUS=2`.
Αυτό αποδεικνύει αναφορά αποτυχίας εκπομπής από τον driver, όχι την ακριβή αιτία.

## Εγκατάσταση και δοκιμή

1. Το **dash-kitb-usb.bin** γράφεται μέσω USB στη διεύθυνση **0x2000**,
   με **Erase All OFF**, και ακολουθεί RESET. Μόνο για την παραπάνω P4/KIT-B.
2. Το Autel/VCI αποσυνδεδεμένο, διακόπτης αυτοκινήτου ανοιχτός, Wi-Fi κλειστό.
3. Δεύτερη σελίδα → χειροκίνητο **READ ECU ON** μία φορά.
4. Αν σταματήσει, φωτογράφισε **ολόκληρη τη δεύτερη σελίδα**. Κάτω από την
   κάρτα λάμδα υπάρχουν δύο γραμμές `PRE TX call` και `POST TX call`.
   Δεν χρειάζεται να ενεργοποιηθεί Wi-Fi ή να πατηθεί επανειλημμένα ON.

Το σταθερό κανάλι ενημερώσεων παραμένει 0.5.1-passive65. Η προαιρετική δοκιμή
εγκαθίσταται από το ξεχωριστό αρχείο USB, χωρίς αντικατάσταση του kitb/latest.json.
Το ανεπίλυτο πρόβλημα Wi-Fi/C6/γραφικών δεν διορθώνεται από αυτή την έκδοση.

## Ενδείξεις και όρια ερμηνείας

- `PRE TX call`: σφάλματα κατά το NORMAL mode, πριν από την πρώτη κλήση
  `twai_transmit` αυτού του χειροκίνητου κύκλου. Το NORMAL mode μπορεί ήδη
  να στέλνει τα αυτόματα ACK του CAN· PRE δεν σημαίνει πλήρη ηλεκτρική σιωπή.
- `POST TX call`: σφάλματα που παρατηρήθηκαν μετά το συγκεκριμένο σημείο
  κλήσης. Δεν είναι σήμανση της πραγματικής έναρξης SOF στο καλώδιο ούτε
  απόδειξη ότι κάθε σφάλμα προκλήθηκε από το request. Η κατεύθυνση TX/RX
  και το hardware error segment δίνουν επιπλέον στοιχεία.
- `first`: πρώτο παρατηρημένο σφάλμα σε κάθε φάση, με **τύπο, κατεύθυνση,
  segment (όνομα/αριθμό), TEC και REC**. `+... us` είναι η χρονική απόσταση
  της εξυπηρέτησης του πρώτου POST interrupt από την πρώτη κλήση TX.
- ACK αναγράφεται μόνο για hardware type OTHER / segment ACK_SLOT(25).
  OTHER σε διαφορετικό segment δεν βαφτίζεται ACK.
- Αποθηκεύονται επίσης τελευταίο PRE/POST και συνολικοί παρατηρημένοι
  μετρητές, διαθέσιμα στο υπάρχον status JSON ως `can_error_trace`.
  Δεν χρειάζεται Wi-Fi για τη φωτογραφική δοκιμή. Οι μετρητές είναι events
  που παρέλαβε ο driver, όχι ποσοστό χαμένων πακέτων ή μετρητής εφαρμογών.
- Η καταγραφή αρχίζει πριν από το twai_start του NORMAL mode. Σταματάει
  στην αναφερόμενη αποτυχία TX ή στο κλείσιμο του driver, διατηρείται στην
  επιστροφή listen-only και μηδενίζεται μόνο με νέο χειροκίνητο κύκλο.
- Χρόνοι modulo uint32 microseconds· η ερμηνεία σχετικών χρόνων αφορά
  τη σύντομη δοκιμή σύνδεσης, όχι πολύωρη καταγραφή (>71 λεπτά).

## Υλοποίηση χωρίς αλλαγή αιτημάτων

Δεν αλλάζουν **500kbit/s, SP65%, SJW3, TSEG1=12, TSEG2=7, TX2/RX3**,
single-shot/fail-stop, απαιτήσεις χειροκίνητου ON, timeout ή ρυθμός αιτημάτων.
Το καντράν παραμένει παθητικό. Μόνο η δεύτερη σελίδα μετά από ON ζητά
115.4,106.1,103.1,004.2,031.1. OFF/έξοδος/ρυθμίσεις/OTA αφοπλίζουν τον reader.
Δεν προστίθενται επαναλήψεις, probing, auto restart ή νέο CAN frame.

Παράγεται hash-guarded build copy του ακριβούς `hal/twai_hal_v1.c` της
ESP-IDF 5.5.3, SHA256 `1eafb46f8c2b58cbdcf0ce2e182e32d48a6da10669daff1fd4c8a0931fd6c8ac`.
Η μοναδική προσθήκη παίρνει τα type/direction/segment **μετά την υπάρχουσα
ανάγνωση ECC**, μαζί με TEC/REC. Δεν διαβάζει δεύτερη φορά το ECC (η ανάγνωση
ξεκλειδώνει το latch), δεν αλλάζει registers/interrupts και δεν πειράζει το
εγκατεστημένο SDK. Διαφορετικό SDK ή μη μοναδικό anchor απορρίπτεται στο build.

Το ISR hook και τα δεδομένα είναι IRAM/DRAM, με μικρό σταθερό critical section,
χωρίς allocation, logging, LVGL ή κλήσεις CAN από ISR. Αποθηκεύονται τέσσερα
σταθερά event slots (πρώτο/τελευταίο PRE/POST), όχι δυναμική ουρά. Η αρχική
χαρτογράφηση HAL σε IRAM διατηρείται και ελέγχεται στο τελικό ELF.
Η παρατήρηση προσθέτει μικρό κόστος εκτέλεσης μόνο κατά τα error interrupts.

## Build

```sh
export DASH_FIRMWARE_VERSION=0.5.3-can-trace
idf.py -B ../../build/kitb-can-trace \
  -DSDKCONFIG=../../build/kitb-can-trace-sdkconfig \
  '-DSDKCONFIG_DEFAULTS=sdkconfig.defaults;sdkconfig.ecu_trace' build
python scripts/package_kitb_ecu_trial.py \
  --build build/kitb-can-trace --out build/kitb-can-trace-download
```

Το idf.py εκτελείται από firmware/kitb, το packaging από τη ρίζα του repository.
Απαραίτητη φυσική δοκιμή στην P4· οι host tests δεν επιβεβαιώνουν επικοινωνία ECU.
Η αποκωδικοποίηση των πέντε τιμών παραμένει όπως στην προηγούμενη δοκιμή,
με επιβεβαιωμένα πακέτα Autel· δείτε KITB_ECU_CAPTURED_TRIAL.md.


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

