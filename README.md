# Workshop #5 - Automatización y Control de Procesos
## Integrantes 

Tomas Candelo Montoya

Carlos Farouk Abdalá Rincón

# Dashboard de IoT usando Thing Speak
Como parte de los requerimientos solicitados, fue necesaria la creación de un Dashboard accesible desde internet que monitorizará a tiempo real el funcionamiento completo del sistema, tanto la temperatura que registrará el controlador esclavo con ayuda del sensor TMP36, como los momentos en los que se presentara alguna alerta por temperaturas altas encendiendo en el montaje físico un led.

Para cumplir dicho propósito, se hizo uso de la plataforma ThingSpeak desarrollada por MathWorks que está especialmente diseñada para recibir las señales enviadas por un controlador, en este caso el ESP32 por su módulo WiFi.

Tras iniciar sesión en la plataforma, es necesario crear un canal de comunicación. Para el desarrollo de este proyecto, la configuración dispuesta para dicho canal fue la siguiente:

![Configuración de canal en ThingSpeak](Images/THINGSPEAK_CHANNEL_CONFIG.jpg)

De esta manera se crea un canal en el que se presentan dos gráficas para mostrar primeramente el comportamiento de la temperatura, y segundo, las alertas por temperaturas mayores a 30°C registradas por el sistema.

Tras crear el canal, es necesario extraer ciertas credenciales del mismo para permitir que nuestro controlador maestro (ESP32) envíe datos al dashboard. Estas credenciales se establecerán como variable estáticas en el código dispuesto para el manejo del controlador.

Las credenciales que necesitamos serán el Identificado del canal de comunicación (Channel ID) y la llave de escritura del canal (Write API Key).

![Credenciales ThingSpeak](Images/THINGSPEAK_CREDENTIALS.jpg)
![Credenciales ThingSpeak en código](Images/THINGSPEAK_CODE_CREDENTIALS.jpg)

Igualmente es necesario asignar como variables estáticas en el código el identificador y contraseña de red que va a usar el ESP32 para conectarse a internet como se ve en la imagen anterior.

Será necesario importar las librerías pertinentes, en este caso la de ThingSpeak para poder realizar la conexión, esto se hace con el comando "***#include "ThingSpeak.h"***". De esta manera las funciones para poder enviar la información ya estarán disponibles.

Finalmente, será necesario en el código del ESP32 configurar el envío de información a la plataforma de ThingSpeak, esto se hace con los siguientes comandos:
```cpp
void setup() {
  Serial.begin(115200);
  Wire.begin(); // ESP32 como maestro
  pinMode(ledPin, OUTPUT);

  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi connected");

  ThingSpeak.begin(client); // Iniciar ThingSpeak
} 
```


Para inicializar la comunicación como cliente a la plataforma, y:

```cpp
ThingSpeak.setField(1, tempReceived); // Campo 1 para temperatura
ThingSpeak.setField(2, tempReceived > 30 ? 1 : 0); // Campo 2 para alerta (1 o 0)

int x = ThingSpeak.writeFields(myChannelNumber, myWriteAPIKey);
```
Para escribir la información recibida en los dashboard, en este caso, el campo o gráfica 1, que es la de temperatura, recibirá y mostrará gráficamente el comportamiento de dicha variable cada 15 segundos debido a la configuración de ThingSpeak, y el campo dos mostrará una gráfica en la que se mostrará con el mismo tiempo de actualización si la variable está o no superando el límite de los 30°C establecidos, siendo el valor de 1 la señal de que se ha superado el límite, y de 0 la señal de que la temperatura esta debajo del límite.
A continuación una muestra del resultado en ThingSpeak:
![Dashboards en ThingSpeak](Images/THINGSPEAK_DASHBOARDS.jpg)