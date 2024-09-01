# Panoramica del Progetto

Questo progetto è un sito web locale sviluppato con Python (Flask), HTML, CSS e JavaScript, che permette di monitorare in tempo reale una serra automatizzata. Il sistema è progettato per fornire una panoramica completa delle condizioni interne della serra, inclusi i valori di temperatura, umidità, luminosità, livello dell'acqua nel serbatoio e umidità del terreno per tre piante: basilico, prezzemolo e menta.

## Componenti

<details>
<summary>Sistema di Monitoraggio Ambientale Basato su Arduino</summary>
Utilizza un Arduino per monitorare e controllare vari fattori ambientali con sensori e attuatori:
- Lettura della temperatura e dell'umidità con un sensore DHT11.
- Misurazione del livello dell'acqua, intensità della luce e umidità del suolo con sensori analogici.
- Controllo di un motore passo-passo, LED e relè in base alle letture dei sensori.
</details>

<details>
<summary>Sistema di Raccolta Dati Basato su Python</summary>
Si connette a un server OPC UA per recuperare dati e memorizzarli in un database MySQL:
- Connessione a un server OPC UA.
- Recupero di vari punti dati dal server.
- Inserimento dei dati in un database MySQL.
- Esecuzione continua con raccolta periodica dei dati e inserimento nel database.
</details>

## Descrizione dei File

<details>
<summary>File 1: Codice Arduino</summary>
Il codice Arduino gestisce il monitoraggio ambientale. Le sezioni chiave includono:

**Librerie e Definizioni**:
```cpp
#include <LiquidCrystal.h>
#include <Stepper.h>
#include <DHT.h>

#define LED_PIN 9
#define RELAY1_PIN 5
#define RELAY2_PIN 6
#define RELAY3_PIN1 3
#define RELAY3_PIN2 7
#define RELAY3_PIN3 8
#define RELAY3_PIN4 10
#define RELAY3_PIN_CONTROL 11
#define DHT_PIN 2
#define DHT_TYPE DHT11
#define WATER_SENSOR_PIN A1
#define PHOTORESISTOR_PIN A0
#define HYGROMETER_PIN A2
#define LED_PIN2 4

#define WATER_THRESHOLD 150
#define LIGHT_THRESHOLD 100
#define TEMP_THRESHOLD 28
#define HUMIDITY_THRESHOLD 850
```

**Funzione di Setup**:
```cpp
void setup() {
  // Inizializzare la comunicazione seriale, i pin e i sensori
}
```

**Funzione di Loop**:
```cpp
void loop() {
  // Leggere i valori dei sensori e controllare gli attuatori in base alle soglie
}
```
</details>

<details>
<summary>File 2: Codice Python</summary>
Il codice Python gestisce la raccolta dati dal server OPC UA e il loro inserimento nel database MySQL. Le sezioni chiave includono:

**Importazioni e Inizializzazione**:
```python
from opcua import Client
import datetime
import mysql.connector
from mysql.connector import Error
import time
```

**Ciclo Principale di Raccolta Dati**:
```python
try:
    client_prova = Client("opc.tcp://192.168.0.12:4840")
    client_prova.connect()

    cnx = mysql.connector.connect(user="root", password="password", host="127.0.0.1", database="OPCUA")

    if cnx.is_connected():
        cursor = cnx.cursor()

        crea_tabella_query = """
        CREATE TABLE IF NOT EXISTS dati (
            timestamp DATETIME PRIMARY KEY,
            pezzi_prodotti INT,
            stato_setup INT,
            stato_stop INT,
            stato_auto INT,
            tempo_setup FLOAT
        )
        """
        cursor.execute(crea_tabella_query)
        cnx.commit()

        while True:
            for i in range(10):
                var2 = client_prova.get_node("ns=4;i=2")
                var3 = client_prova.get_node("ns=4;i=3")
                var4 = client_prova.get_node("ns=4;i=4")
                var5 = client_prova.get_node("ns=4;i=5")
                var6 = client_prova.get_node("ns=4;i=6")

                Pezzi_Prodotti = var6.get_value()
                Stato_Setup = var2.get_value()
                Stato_Stop = var3.get_value()
                Stato_Auto = var4.get_value()
                Tempo_Setup = var5.get_value()
                Timestamp = datetime.datetime.now()

                inserisci_query = """
                INSERT INTO dati (timestamp, pezzi_prodotti, stato_setup, stato_stop, stato_auto, tempo_setup)
                VALUES (%s, %s, %s, %s, %s, %s)
                """
                cursor.execute(inserisci_query, (Timestamp, Pezzi_Prodotti, Stato_Setup, Stato_Stop, Stato_Auto, Tempo_Setup))
                cnx.commit()

                print("DATI INSERITI CON SUCCESSO")

                time.sleep(1)

            print("PAUSA PRIMA DELL'ALTRO CICLO")
            time.sleep(10)

except Error as e:
    print(f"Error: {e}")

finally:
    if cnx is not None and cnx.is_connected():
        cursor.close()
        cnx.close()
        print("CONNESSIONE MYSQL CHIUSA")

    if client_prova is not None:
        try:
            client_prova.disconnect()
            print("CLIENT OPC UA DISCONNESSO")
        except Exception as e:
            print(f"Errore nella disconnessione del client OPC UA: {e}")
```
</details>

## Istruzioni di Configurazione

<details>
<summary>Sistema Basato su Arduino</summary>

1. **Configurazione Hardware**:
   - Collegare il sensore DHT11 al DHT_PIN (pin 2).
   - Collegare il sensore dell'acqua al WATER_SENSOR_PIN (A1).
   - Collegare il fotoresistore al PHOTORESISTOR_PIN (A0).
   - Collegare l'igrometro al HYGROMETER_PIN (A2).
   - Collegare il motore passo-passo ai pin dei relè (RELAY3_PIN1, RELAY3_PIN2, RELAY3_PIN3, RELAY3_PIN4).
   - Collegare i relè ai pin RELAY1_PIN, RELAY2_PIN e RELAY3_PIN_CONTROL.

2. **Configurazione Software**:
   - Caricare il codice Arduino sulla scheda Arduino.
</details>

<details>
<summary>Sistema Basato su Python</summary>

1. **Installare le Dipendenze**:
   - Installare le librerie Python richieste usando pip:
     ```bash
     pip install opcua mysql-connector-python
     ```

2. **Configurazione MySQL**:
   - Assicurarsi di avere MySQL installato e in esecuzione.
   - Creare un database chiamato `OPCUA`.

3. **Eseguire lo Script Python**:
   - Eseguire lo script Python per iniziare la raccolta dei dati:
     ```bash
     python your_script.py
     ```
</details>

## Esecuzione del Progetto

<details>
<summary>Guida alla Esecuzione</summary>
- Accendere il sistema Arduino per iniziare a monitorare le condizioni ambientali.
- Eseguire lo script Python per iniziare a raccogliere dati dal server OPC UA e memorizzarli nel database MySQL.
</details>
