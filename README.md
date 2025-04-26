## Pont de série vers midi - version out 
```
import rtmidi
import serial
import threading
import time
import sys

# Configuration des ports
SERIAL_PORT = "/dev/cu.usbmodem212201"  # Remplacez par votre port série
BAUDRATE = 115200
MIDI_OUT_NAME = "IAC Bus 2"  # Nom du port MIDI de sortie

# Initialisation du port série
try:
    ser = serial.Serial(SERIAL_PORT, BAUDRATE, timeout=0.1)
except serial.SerialException as e:
    print(f"Erreur lors de l'ouverture du port série : {e}")
    sys.exit(1)

# Initialisation des ports MIDI
midi_out = rtmidi.MidiOut()

# Ouverture du port MIDI de sortie
available_ports_out = midi_out.get_ports()
out_port_index = -1
for i, port in enumerate(available_ports_out):
    if MIDI_OUT_NAME in port:
        out_port_index = i
        break

if out_port_index == -1:
    print(f"Port MIDI de sortie '{MIDI_OUT_NAME}' non trouvé.")
    sys.exit(1)

midi_out.open_port(out_port_index)


# Fonction pour lire depuis le port série et envoyer aux sorties MIDI
def read_serial():
    while True:
        data = ser.read(3)  # Lire 3 octets (taille typique d'un message MIDI)
        if data:
            midi_out.send_message(data)


# Fonction pour lire depuis les entrées MIDI et envoyer au port série
def midi_callback(message, data=None):
    message, deltatime = message
    ser.write(bytearray(message))


# Démarrer le thread de lecture série
serial_thread = threading.Thread(target=read_serial, daemon=True)
serial_thread.start()

print("Pont MIDI série en cours d'exécution. Appuyez sur Ctrl+C pour quitter.")
try:
    while True:
        time.sleep(1)
except KeyboardInterrupt:
    print("Arrêt du pont MIDI série.")
    ser.close()
    midi_out.close_port()

```
## Pont de série vers midi - version in et out 
```
import rtmidi
import serial
import threading
import time
import sys

# Configuration des ports
SERIAL_PORT = "/dev/cu.usbmodem212201"  # Remplacez par votre port série
BAUDRATE = 115200
MIDI_IN_NAME = "IAC Bus 1"  # Nom du port MIDI d'entrée
MIDI_OUT_NAME = "IAC Bus 2"  # Nom du port MIDI de sortie

# Initialisation du port série
try:
    ser = serial.Serial(SERIAL_PORT, BAUDRATE, timeout=0.1)
except serial.SerialException as e:
    print(f"Erreur lors de l'ouverture du port série : {e}")
    sys.exit(1)

# Initialisation des ports MIDI
midi_in = rtmidi.MidiIn()
midi_out = rtmidi.MidiOut()

# Ouverture du port MIDI d'entrée
available_ports_in = midi_in.get_ports()
in_port_index = -1
for i, port in enumerate(available_ports_in):
    if MIDI_IN_NAME in port:
        in_port_index = i
        break

if in_port_index == -1:
    print(f"Port MIDI d'entrée '{MIDI_IN_NAME}' non trouvé.")
    sys.exit(1)

midi_in.open_port(in_port_index)

# Ouverture du port MIDI de sortie
available_ports_out = midi_out.get_ports()
out_port_index = -1
for i, port in enumerate(available_ports_out):
    if MIDI_OUT_NAME in port:
        out_port_index = i
        break

if out_port_index == -1:
    print(f"Port MIDI de sortie '{MIDI_OUT_NAME}' non trouvé.")
    sys.exit(1)

midi_out.open_port(out_port_index)


# Fonction pour lire depuis le port série et envoyer aux sorties MIDI
def read_serial():
    while True:
        data = ser.read(3)  # Lire 3 octets (taille typique d'un message MIDI)
        if data:
            midi_out.send_message(data)


# Fonction pour lire depuis les entrées MIDI et envoyer au port série
def midi_callback(message, data=None):
    message, deltatime = message
    ser.write(bytearray(message))


midi_in.set_callback(midi_callback)

# Démarrer le thread de lecture série
serial_thread = threading.Thread(target=read_serial, daemon=True)
serial_thread.start()

print("Pont MIDI série en cours d'exécution. Appuyez sur Ctrl+C pour quitter.")
try:
    while True:
        time.sleep(1)
except KeyboardInterrupt:
    print("Arrêt du pont MIDI série.")
    ser.close()
    midi_in.close_port()
    midi_out.close_port()
```
