# PRACTICA 6: BUS SPI. LECTOR TARJETAS SD

## Ejercicio 1: Lectura/Escritura de archivo

El siguiente codigo emplea la escritura y lectura de una trajeta SD. 

Debemos incluir las librerías SPI y SD para ello. 
Tambien declaramos la variable myFile de tipo File para poder manipular archivos. 
```cpp
#include <Arduino.h>
#include <SPI.h>
#include <SD.h>

File myFile;
```
En el *setup()* debemos indicar los pines de conexión para el bus SPI. 

Hecho comporobamos la connexion con la tarjeta. Para ello pasamos por parametro el pin asignado a los datos (CS) y si esta connexión no se realizó enviamos al puerto serie un mensaje de error.  
Si es exitosa se procede y da los mensajes de confirmación. 

Tras esto, procedemos a abrir el ficher con nombre *archivo.txt*. En caso de que no exista el fichero, el parámetro *FILE_WRITE* permite que se cree el archivo y se abra immediatamente. 

Escribimos algo en el fichero y lo cerramos. 

Si el fichero que quisimos crear SÍ se creó correctamente entonces leemos el contenido y se envía este por el puerto serie. 
Si no se pudo crear, enviamos un mensaje de error. 
``` cpp

void setup(){

Serial.begin(9600);
Serial.print("Iniciando SD ...");

 SPI.begin(18,19,23,4);

if(! SD.begin(4)){
  Serial.println("No se pudo inicializar");
  while(1);
    }
    
  Serial.println("SD OK");
  
Serial.println("inicializacion exitosa");

  //Escribimos en el fichero
  myFile=SD.open("/archivo.txt",FILE_WRITE); // FILE_WIRTE: si no existe el archivo, lo crea. 
  // SD.remove("/archivo.txt");
  // myFile=SD.open("/archivo.txt");
  myFile.println("Me estoy comunicando con SD");
  myFile.println("Doble entrada");
  myFile.close();

  myFile=SD.open("/archivo.txt"); //Abrimos, mostramos y leemos
  if (myFile) {
    Serial.println("archivo.txt:");
    while (myFile.available()) {
    String line=myFile.readStringUntil('\n');
    Serial.println(line);
    }
    myFile.close(); 
  }
  
  else {
    Serial.println("Error al abrir el archivo");
  }
}

void loop(){}
```

Así pues por el puerto serie deberíamos ver lo siguiente: 
```
Iniciando SD ...sd init ok
SD OK
inicializacion exitosa
Me estoy comunicando con SD
Doble entrada
```
---------------------------------------------------------------------

Este programa es un simple ejemplo de como usar el lector de tarjetas. 
 
El lector de tarjetas puede ser útil sobretodo para almacenar datos si no tenemos una memoria RAM en nuestro microprocesador. 

## Ejercicio 2: Lectura de etiqueta RFID

En este ejercicio utilizaremos el módulo RC522 para obtener lecturas de tarjetas RFID
Al inicio del código incluímos la librería MFRC522
```cpp
#include <SPI.h>
#include <MFRC522.h>
```

Definimos los pins de reset y sda para el módulo RC522, y instanciamos el módulo con dichos pines
```cpp
#define RST_PIN	9    
#define SS_PIN	10  
MFRC522 mfrc522(SS_PIN, RST_PIN); 
```

El bus de comunicación y el módulo RC522 se necesitan iniciar solo una vez, así que lo hacemos con las siguientes funciones en el setup:
```cpp
	SPI.begin();        //Iniciamos el Bus SPI
	mfrc522.PCD_Init(); // Iniciamos  el MFRC522
	
```

Dentro del loop, seguimos el siguiente proceso:
- Comprobar si hay nuevas tarjetas
- Seleccionar la tarjeta presente
- Para leer el código de identificación de la tarjeta, recorremos con un for de tamaño uid.size cada uno de los dígitos de dicho código
- Finalizamos lectura
Las respectivas funciones utilizadas son las siguientes:
```cpp
	
	mfrc522.PICC_IsNewCardPresent()

  mfrc522.PICC_ReadCardSerial()

  for (byte i = 0; i < mfrc522.uid.size; i++) {
            Serial.print(mfrc522.uid.uidByte[i] < 0x10 ? " 0" : " ");
            Serial.print(mfrc522.uid.uidByte[i], HEX);   
      } 
  mfrc522.PICC_HaltA();         
  ```
  Este código permite obtener y mostrar por el monitor el código de identificación de la tarjeda RFD