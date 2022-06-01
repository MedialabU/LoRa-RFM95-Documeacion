# DOCUMENTACIÓN LoRa-RFM95W

### Introducción

Esta PCB sirve para integrar fácilmente el módulo transceptor RFM95W en proyectos de LoRa. Permite conectar dos antenas con conectores SMA hembra a macho, de tipo borde de placa o 90º. Se selecciona que conector se utiliza estañando la conexión en la cara opuesta de la placa.

![Diseño3D_1](Celia%20PRIVADA%2077de56e9b721404fa09241322e39240c/Untitled%203.png)

![Diseño3D_2](Celia%20PRIVADA%2077de56e9b721404fa09241322e39240c/Untitled%204.png)

Es un transceptor basado en LoRa SX1276 con interfaz SPI. Se alimenta con 3.3V. Para un funcionamiento adecuado es necesario conectar como mínimo los pines **SPI**:

- NSS
- RESET
- DIO0
- DIO1
- MOSI
- MISO
- SCK
- 3V3
- GND

**Definición de pines Arduino Mini Pro según librería:**

```arduino
const int NSS_GPIO = 10;
const int RESET_GPIO = 5;
const int DIO0_GPIO = 2;
const int DIO1_GPIO = 3;
const int DIO2_GPIO = 4;

const lmic_pinmap lmic_pins = {
    .nss = NSS_GPIO,
    .rxtx = LMIC_UNUSED_PIN,
    .rst = RESET_GPIO,
    .dio = {DIO0_GPIO, DIO1_GPIO, DIO2_GPIO},
};
```


### Componentes:

[RFM95 - Link de Compra](https://es.aliexpress.com/item/1005003980988859.html)

[Soporte Antena SMA - Link de Compra](https://www.amazon.es/sourcing-map-Conector-Soldadura-Montaje/dp/B01ISOJE1K/ref=sr_1_17)

[Soporte de Antena SMA 2 - Link de Compra](https://www.amazon.es/Fydun-Conectores-conectar-Soldadura-enchufes/dp/B07YCPSF66/ref=pd_sbs_sccl_2/262-8953205-3606815)


### Prototipo:

Las primeras pruebas se realizaron con una PCB de prototipado y una antena helicoidal. Con este primer prototipo se hicieron las pruebas de conexión y librerías de cara a diseñas la PCB final.

![Prototipo](Celia%20PRIVADA%2077de56e9b721404fa09241322e39240c/Untitled%205.png)

### Diseño:

![Diseño1](Celia%20PRIVADA%2077de56e9b721404fa09241322e39240c/Untitled%206.png)

![Diseño2](Celia%20PRIVADA%2077de56e9b721404fa09241322e39240c/Untitled%207.png)

### Resultado final:
![ResultadoFinal3](Celia%20PRIVADA%2077de56e9b721404fa09241322e39240c/Untitled%2010.png)

![ResultadoFinal1](Celia%20PRIVADA%2077de56e9b721404fa09241322e39240c/Untitled%208.png)

![ResultadoFinal2](Celia%20PRIVADA%2077de56e9b721404fa09241322e39240c/Untitled%209.png)


### ****Conexionado RFM95 y Arduino Mini Pro****

Conectamos nuestro ordenador via USB al USB UART Converter* (modelo FTDI232). 

Conectamos USB UART Converter al Arduino Mini Pro. 

Conectamos RFM95 al Mini Pro siguiendo el siguiente conexionado:

IMP: SOLDAR LA ANTENA 2 + SU JUMPER  (La de 5 puntos)

*Sirve para adaptar las funciones Serial al Mini Pro, otros lo tienen incluido en su placa (son más grandes y caros).

### Tabla de conexionado:

| RFM95 | Mini Pro|
| ------ | ------ |
| NSS | D10 |
| SCK | D13 |
| MOSI | D11 |
| MISO | D12 |
| ------ | ------ |
| 3.3V | Vcc|
| G | GND |
| ------ | ------ |
| DIO0 | D2 |
| DIO1 | D3 |
| D5   | RESET   |

### Datasheet:

![Untitled](Celia%20PRIVADA%2077de56e9b721404fa09241322e39240c/Untitled%2011.png)

### Envío de datos al dispositivo TTN:

Gracias a nuestro programa recibiremos del microcontrolador los valores de lluvia, velocidad y dirección del viento, presión, temperatura y humedad. Enviamos estos datos a nuestro dispositivo TTN en los siguientes formatos:

lluvia: 2 bytes

viento: 2 bytes

dirección de la veleta: 1 byte

presión: 2 bytes

temperatura: 2 bytes

humedad: 2 bytes

### Crear dispositivo TTN en la web

Entramos en la página web (se da por hecho que tenemos un perfil creado):

[ ](https://eu1.cloud.thethings.network/console/ )

Go to applications -> Creamos aplicación “vp2-medialab” -> Creamos dispositivo -> Manually escogiendo:

- Europe 863-870MHz (recommended)
- LoRaWan 1.0.3 version
- Generate devEUI
- Use zeros
- Generate AppKey
- Device id: *test1-estacion*

### Conectar por código con nuestro dispositivo:

En *configuracion.hpp* debemos añadir el siguiente código con las claves correspondientes que encontraremos en nuestro dispositivo:

```arduino
#ifdef CONFIGURACIÓN_TEST1_ESTACION

static const u1_t PROGMEM APPEUI[8] = {*Copiamos datos en LSB de AppEUI*};   // LSB

static const u1_t PROGMEM DEVEUI[8] = {*Copiamos datos en LSB de DevEUI*};   // LSB

static const u1_t PROGMEM APPKEY[16] = {*Copiamos datos en MSB de AppKey*};  // MSB

#endif
```

Y para elegir esta configuración en *platformio.ini:*

```arduino
build_flags =
    ; -D ARDUINO_LMIC_PROJECT_CONFIG_H_SUPPRESS
    -D CFG_eu868=1
    -D CFG_sx1276_radio=1
    -D LMIC_LORAWAN_SPEC_VERSION=LMIC_LORAWAN_SPEC_VERSION_1_0_3
    -D DEBUG
    -D CONFIGURACIÓN_TEST1_ESTACION ; Aquí escojo mi configuración, definida en configuracion.hpp
```

### Ahora que ya tenemos nuestro dispositivo, llegarán los datos:

Los recuperamos y en el ‘Formatter type’ en JS lo imprimimos:

```jsx
// IMPRIMINOS LLUVIA-VIENTO-BRÚJULA

  function Decoder(bytes, port) {

  // LLUVIA viene en 2 bytes, recuperamos el valor
  var valor_lluvia = (bytes[0] << 8) | bytes[1];
  
  // VELOCIDAD DEL VIENTO viene en 2 bytes, recuperamos el valor
  var valor_viento = (bytes[2] << 8) | bytes[3];
  
  // DIRECCIÓN VIENTO viene en 1 byte, recuperamos el valor
  var valor_direccion_veleta = bytes[8];
  var enum_dir = ["S","SW", "W", "NW" , "N" , "NE" , "E", "SE"];
  var dir = enum_dir [valor_direccion_veleta];
  
  // BME280 (presión,temperatura y humedad ambiente) viene en 2 bytes, recuperamos el valor
  var pres = bytes[5] << 8 | bytes[6];
  var temp = bytes[7] << 8 | bytes[8];
  var hume = bytes[9] << 8 | bytes[10];

  return {
     
      viento : {
        velocidad: valor_viento,
        direccion: dir,
      },
      precipitacion: valor_lluvia,
      bme280 : {
        presion: pres/10,
        temperatura: temp/100,
        humedad: hume/100,
      }
    }
  }

function decodeUplink(input) {
  var data = input.bytes;
  var valid = true;

  if (typeof Decoder === "function") {
    data = Decoder(data, input.fPort);
  }

  if (typeof Converter === "function") {
    data = Converter(data, input.fPort);
  }

  if (typeof Validator === "function") {
    valid = Validator(data, input.fPort);
  }

  if (valid) {
    return {
      data: data
    };
  } else {
    return {
      data: {},
      errors: ["Invalid data received"]
    };
  }
}
```

También lo enviamos desde TTN a [NodeRed](http://192.168.1.15:1880/#flow/6a3c2b6f.51d754) con un protocolo mqtt para pasarlo a formato http y después cargarlo a nuestra base de datos MySQL. 

De ahí reenviamos los datos a nuestra [API](http://192.168.1.6:8000/) (programando en php) gracias a almacenarlos en la base de datos.
