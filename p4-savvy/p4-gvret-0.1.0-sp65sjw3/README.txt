# P4 KIT-B — καταγραφέας SavvyCAN / Autel

Ξεχωριστό προσωρινό firmware **p4-gvret-0.1.0-sp65sjw3** για την επιβεβαιωμένη
**ESP32-P4-WIFI6-KIT-B / P4 rev3.x / flash32MB**. Δεν είναι για την S3 7″.

Στη φωτογραφία του χρήστη στις22/11/2026, το AUTO LOCKED_MONITOR με τη νέα
πλακέτα CAN έδωσε **SP65% / SJW3**, περίπου1250frames/s και μηδενικά errors,
missed, overrun, invalid και API errors στα εμφανιζόμενα διαστήματα. Αυτό
τεκμηριώνει εκείνη τη λήψη, όχι εκπομπή ή φυσική δοκιμή αυτού του νέου build.

## Εγκατάσταση USB

1. Στο ESP Tool / Chrome ή Edge επίλεξε **kitb-savvycan-usb.bin**.
2. **Flash Address:0x2000**, Erase All κλειστό. Program και μετά RESET.
3. Κλείσε ή αποσύνδεσε το ESP Tool για να ελευθερωθεί η COM.

Η οθόνη μένει μαύρη. Το ESP32-C6 κρατιέται σε reset. Δεν φορτώνονται LCD,
LVGL, PSRAM ή Wi-Fi. Το πρόγραμμα αντικαθιστά το καντράν και δεν έχει runtime
OTA· επιστρέφουμε στο καντράν μέσω του δικού του αρχείου USB στη0x2000.
Το merged image περιλαμβάνει bootloader/partitions/application και κενά
περιοχών NVS/otadata, όπως ο υπάρχων CAN-only installer. Οι αποθηκευμένες
ρυθμίσεις του καντράν μπορεί να χρειαστούν επανεισαγωγή μετά την επιστροφή.

## Σύνδεση SavvyCAN

Χρησιμοποίησε την **ίδια θύρα UART USB / COM** που έδειχνε το AUTO scan.
Όχι δεύτερη σύνδεση στο web console παράλληλα με το SavvyCAN.

1. Άνοιξε SavvyCAN.
2. **Connection → Open Connection Window → Add New Device Connection**.
3. **Serial Connection (GVRET)**, επίλεξε τη σωστή COM και **Create New Connection**.
4. CAN bus0, **500000bit/s**. Το SavvyCAN ρυθμίζει τη σειριακή στα **1000000baud**.
5. Έλεγξε ότι εμφανίζονται πραγματικά CAN frames. Καμία απάντηση ή κενή
   λίστα δεν θεωρείται επιτυχής καταγραφή.
6. **File → Start Continuous Logging**, ή **File → Save Log File** για
   αποθήκευση των frames που βρίσκονται ήδη στη λίστα. Μορφή SavvyCAN CSV.

[SavvyCAN Windows V220](https://github.com/collin80/SavvyCAN/releases/download/V220/SavvyCAN-Windows_x64_CIBuild.zip).

Αρχική σύνδεση/handshake σταματούν ή επανεκκινούν τον δέκτη· ξεκίνησε τη
διαγνωστική συνεδρία Autel μόνο όταν έχει ήδη αρχίσει η καταγραφή στο PC.
Μην αλλάζεις τις ρυθμίσεις της σύνδεσης κατά τη διάρκεια της συνεδρίας.

## Καταγραφή Autel

Η P4 **μόνο ακούει**· το Autel είναι ο ενεργός tester. Το ίδιο powertrain
CAN μπορεί να μεταφέρει αιτήματα/αποκρίσεις εφόσον η διαδρομή του gateway
τα προωθεί εκεί. Η παρουσία frames πρέπει να ελεγχθεί από το αρχείο.

Ξεκίνα το log πριν επιλέξεις **01 Engine** στο Autel, ώστε να κρατήσουμε
channel setup, CAN IDs, parameters, session και αμφίδρομα δεδομένα, όχι
μόνο μεμονωμένες τιμές. Διάβασε διαδοχικά, περίπου10s το κάθε block:

- 106.1 — υψηλή πίεση καυσίμου
- 103.1 — χαμηλή πίεση καυσίμου
- 115.4 — boost / απόλυτη πίεση
- 004.2 — τάση
- 031.1 — πραγματικό λάμδα

Κράτησε σημείωση της σειράς και των πραγματικών τιμών Autel. Σταμάτα και
σώσε το log στο PC, μετά κλείσε τη συνεδρία. Στείλε το πραγματικό CSV/log.
Η οθόνη του ESP Tool και ένα snapshot τελευταίων bytes δεν είναι χρονικό log.

## Συμπεριφορά και όρια

- Fixed500kbit/s, TX2/RX3, XTAL40MHz / BRP4,20TQ,TSEG1=12,TSEG2=7,SJW3,
  single sampling. Δεν μεταβάλλεται από NVS ή αυτόματη σάρωση.
- TWAI_LISTEN_ONLY, TXqueue0, RXqueue1024, IRAM interrupt, bounded batches64.
  Απορρίπτει εντολές εκπομπής GVRET και κάθε bitrate εκτός500000. Αίτημα
  normal mode από PC δεν ενεργοποιεί εκπομπή. Δεν υπάρχει κλήση twai_transmit.
- Όλα τα classic CAN data IDs, standard και extended, χωρίς φίλτρο διάγνωσης.
  Το σταμάτημα/άνοιγμα δέκτη παραμένει μόνο ακρόαση.
- Χρησιμοποιεί τον ήδη ελεγμένο GVRET parser/encoder των άλλων καταγραφέων.
  UART0 στο ίδιο default console routing του CAN-only (P4 TX37/RX38),8N1,
  χωρίς RTS/CTS. Η αλλαγή baud από115200 σε1000000 αφορά μόνο USB serial.
- Δεν υπάρχουν περιοδικά ASCII logs μέσα στη δυαδική ροή. Η ROM μπορεί να
  γράψει boot bytes πριν από το handshake. Το SavvyCAN ανασυγχρονίζεται.
- RTR και DLC0 δεν υποστηρίζονται σωστά από αυτό το GVRET/SavvyCAN client:
  παραλείπονται και μετρώνται χωριστά, χωρίς κατασκευασμένο payload.
- Timestamps32bit σε microseconds από esp_timer όταν ο worker παραλαμβάνει
  το frame, όχι hardware timestamps του CAN interrupt. Αναδίπλωση~71.6min.
  Προτίμησε σύντομες καταγραφές. Δεν υποσχόμαστε ακριβές wire timing ή
  μηδενικές απώλειες σε κάθε φόρτο/USB/PC.
- Προαιρετική διάγνωση: εκτός SavvyCAN, terminal στην ίδια COM/1000000,
  στείλε **Q** για version, bitrate, timing, πραγματικούς counters και errors.
  Μόνο αυτή η ρητή εντολή παράγει το ASCII #P4GVRET. Μην τη στέλνεις μέσα
  στην καταγραφή SavvyCAN.

## Build

ESP-IDF5.5.3, χωρίς BSP/managed dependencies:

```sh
cd firmware/kitb_savvy_capture
idf.py -B ../../build/kitb-savvy-capture build
cd ../..
python scripts/package_kitb_savvy.py --build build/kitb-savvy-capture --out build/kitb-savvy-download
```

Διανέμεται στο ξεχωριστό `p4-savvy/<version>/`, χωρίς μεταβολή του
`kitb/latest.json`, των S3/capture manifests ή του `leon-1p-can`.

Έλεγχοι:7/7 σχετικοί host tests, πραγματικός recorder με mock UART/TWAI,
πάνω από6000 frames ανά επιτυχή διαδρομή, προστασία από εντολές εκπομπής,
αποτυχίες install/start/stop/backlight και bus-off. Πέρασαν επίσης
ASan/UBSan για κανονική ροή/start-failure/bus-off, build ESP-IDF5.5.3,
έλεγχοι εικόνας/ZIP/SHA256/offsets και linked-symbol audit χωρίς CAN TX,
Wi-Fi ή LVGL entry points. Εκκρεμεί φυσική δοκιμή με SavvyCAN/Autel.
