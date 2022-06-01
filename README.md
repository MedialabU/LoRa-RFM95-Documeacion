# DOCUMENTACIÓN LoRa-RFM95W

### Introducción

Esta PCB sirve para integrar fácilmente el módulo transceptor RFM95W en proyectos de LoRa. Permite conectar dos antenas con conectores SMA hembra a macho, de tipo borde de placa o 90º. Se selecciona que conector se utiliza estañando la conexión en la cara opuesta de la placa.

![Diseño3D_1](/Fotos/Diseño3D_1.png)

![Diseño3D_2](/Fotos/Diseño3D_2.png)
---
### Prototipo:

Las primeras pruebas se realizaron con una PCB de prototipado y una antena helicoidal. Con este primer prototipo se hicieron las pruebas de conexión y librerías de cara a diseñas la PCB final.

![Prototipo](/Fotos/Prototipo.png)

### Diseño:

![Diseño1](/Fotos/Diseño1.png)

![Diseño2](/Fotos/Diseño2.png)

### Resultado final:

![ResultadoFinal3](/Fotos/ResultadoFinal3.png)

![ResultadoFinal1](/Fotos/ResultadoFinal1.png)

![ResultadoFinal2](/Fotos/ResultadoFinal2.png)

---
### RFM95W
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

### Pin Map para Arduino Mini Pro:

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
### Tabla de conexionado para Arduino Mini Pro:

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

### Pin Map para Black Pill STM32:
```
const lmic_pinmap lmic_pins = {
    .nss = PB12,
    .rxtx = LMIC_UNUSED_PIN,
    .rst = PA3,
    .dio = {PA0, PA1, LMIC_UNUSED_PIN},
};
```

### Tabla de conexionado para STM32:

| RFM95 | STM32|
| ------ | ------ |
| NSS | B12 |
| SCK | A5 |
| MOSI | A7 |
| MISO | A6 |
| ------ | ------ |
| 3.3V | Vcc|
| G | GND |
| ------ | ------ |
| DIO0 | A0 |
| DIO1 | A1 |
| RESET   | A3   |

---
### Componentes del montaje completo:

[RFM95 - Link de Compra](https://es.aliexpress.com/item/1005003980988859.html)

[Soporte Antena SMA - Link de Compra](https://www.amazon.es/sourcing-map-Conector-Soldadura-Montaje/dp/B01ISOJE1K/ref=sr_1_17)

[Soporte de Antena SMA 2 - Link de Compra](https://www.amazon.es/Fydun-Conectores-conectar-Soldadura-enchufes/dp/B07YCPSF66/ref=pd_sbs_sccl_2/262-8953205-3606815)

---
### Datasheet:

![Datasheet](/Fotos/Datasheet.png)

---

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

---

### Ahora que ya tenemos nuestro dispositivo, llegarán los datos:
PARA EL PROGRAMA DE LA ESTACIÓN METEOROLÓGICA
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

---
