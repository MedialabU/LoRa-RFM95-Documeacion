# DOCUMENTACIÓN LoRa-RFM95W

## Introducción

Esta PCB sirve para integrar fácilmente el módulo transceptor RFM95W en proyectos de LoRa. Permite conectar dos antenas con conectores SMA hembra a macho, de tipo borde de placa o 90º. Se selecciona que conector se utiliza estañando la conexión en la cara opuesta de la placa.

![Diseño3D_1](/Fotos/Diseño3D_1.png)

![Diseño3D_2](/Fotos/Diseño3D_2.png)

---
## Prototipo:

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
## RFM95W
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
