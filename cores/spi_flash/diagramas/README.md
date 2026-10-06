# Protocolo SPI

## 1. Explicación general del protocolo

### 1.1 ¿Qué es SPI?

SPI (**Serial Peripheral Interface**) es un protocolo de comunicación **sincrónico, serie y full-duplex**. Un dispositivo **maestro** (en nuestro caso la FPGA) controla a uno o varios **esclavos** (en este caso, la memoria Flash).

### 1.2 Señales del bus

El protocolo cuenta con cuatro señales principales de conexión:

| Señal       | Nombre completo            | Función                               |
| ----------- | -------------------------- | ------------------------------------- |
| `SCLK`      | Serial Clock               | Reloj generado por el maestro         |
| `MOSI`      | Master Out Slave In        | Datos del maestro hacia el esclavo    |
| `MISO`      | Master In Slave Out        | Datos del esclavo hacia el maestro    |
| `CS` / `SS` | Chip Select / Slave Select | Activo en bajo. Selecciona el esclavo |

### 1.3 Características principales

* **Protocolo sincrónico:** los datos se leen en los flancos del reloj. La configuración del flanco de lectura y el estado base del reloj determinan los cuatro modos de operación del protocolo.
* **Comunicación full-duplex:** el protocolo soporta comunicación bidireccional simultánea mediante las líneas independientes `MOSI` y `MISO`.
* **Generación del reloj:** la señal de reloj es generada por el maestro.
* **Estabilidad de los datos:** cuando ocurre el flanco utilizado para leer un dato, la información debe encontrarse estable.
* **Velocidad de transmisión:** la frecuencia utilizada debe respetar los tiempos de operación establecidos para la memoria Flash.

### 1.4 Modos SPI (CPOL / CPHA)

La comunicación SPI puede utilizar cuatro configuraciones determinadas por **CPOL** y **CPHA**:

| Modo | CPOL | CPHA | Flanco de muestreo |
| ---- | ---- | ---- | ------------------ |
| 0    | 0    | 0    | Subida             |
| 1    | 0    | 1    | Bajada             |
| 2    | 1    | 0    | Bajada             |
| 3    | 1    | 1    | Subida             |

En memorias Flash se utiliza normalmente **Modo 0** o **Modo 3**, dependiendo de la memoria específica.

El dato cambia en un flanco del reloj y se lee en el otro. Por lo tanto, en el momento del flanco de lectura, el dato debe permanecer estable.

### 1.5 Estructura general de la comunicación

Toda transacción con la Flash sigue, de manera general, el siguiente patrón:

```text
CS baja → [Comando] → [Dirección] → [Dummy, si aplica] → [Datos] → CS sube
```

Los elementos principales de la transacción son:

* **Comando:** 1 byte que indica la operación que se desea realizar.
* **Dirección:** 3 bytes (24 bits) que indican la posición de memoria.
* **Dummy:** 0 o más bytes de relleno utilizados por algunos comandos.
* **Datos:** información que se transmite o recibe.

La secuencia general de la comunicación es:

1. `CS` baja al inicio de la transacción.
2. El maestro genera el reloj.
3. Se transmite el comando.
4. Se transmite la dirección cuando la operación la requiere.
5. Se transmiten ciclos dummy cuando el comando los requiere.
6. Se transmiten o reciben los datos.
7. `CS` sube al finalizar la transacción.


### 1.6 Estructura interna de la memoria 

Como informacion extra, es necesario comprender como se divide la estructura interna de la memoria flash, ya que esto es importante para entender la funcionalidad de las direcciones que se envia en multiples comandos.



![Estructura de memoria](Estructura_memoria%20.png)


Típicamente, estas memorias se dividen en 3 niveles, tal como se muestra en la imagen anterior. La memoria completa tiene una capacidad de 16 MiB, la cual se divide en 256 bloques, cada uno de 64 KB. Cada bloque se divide en 16 sectores, cada uno de 4 KB. Cada sector se vuelve a dividir en 16 paginas, cada una de 256 Bytes y estas ultimas se dividen en los ya mencionados 256 Bytes. Una imagen mas representativa se muestra a continuación.

![Estructura detallada de memoria](Estructura_detallada_de_memoria.png)


Las direcciones de memoria que se envian desde el maestro hacia el esclavo se dividen en 4 partes (esto se refiere únicamente a la distribución de la información, las direcciones se envian de forma continua bit tras bit).

![Estrcutura de la direccion enviada](Estructura_direcciones.png)




---

# 2. Comandos de estado y control

Esta sección presenta los comandos utilizados para controlar la memoria y consultar su estado. Para cada comando se documentará posteriormente su forma de uso y su correspondiente forma de onda.

### 2.1 Índice de comandos

| Código | Comando                | Función                                      |
| ------ | ---------------------- | -------------------------------------------- |
| `06h`  | Write Enable           | Habilita escritura/programación              |
| `04h`  | Write Disable          | Deshabilita escritura/programación           |
| `05h`  | Read Status Register 1 | Consulta el registro de estado               |
| `35h`  | Read Status Register 2 | Consulta información adicional de estado     |
| `01h`  | Write Status Register  | Escribe información en el registro de estado |

### 2.2 Documentación

Para cada comando se incluirá:

* Secuencia de utilización.
* Información transmitida y recibida.
* Forma de onda correspondiente.

---

# 3. Comandos de lectura

Esta sección presenta los comandos utilizados para obtener información almacenada en la memoria Flash. Se documentará posteriormente la estructura de cada comando y su respectiva forma de onda.

### 3.1 Índice de comandos

| Código | Comando               | Descripción                     |
| ------ | --------------------- | ------------------------------- |
| `03h`  | READ                  | Lectura de datos                |
| `0Bh`  | FAST READ             | Lectura rápida con ciclos dummy |
| `3Bh`  | FAST READ DUAL OUTPUT | Lectura mediante dos líneas     |
| `BBh`  | FAST READ DUAL I/O    | Lectura Dual I/O                |
| `6Bh`  | FAST READ QUAD OUTPUT | Lectura mediante cuatro líneas  |
| `EBh`  | FAST READ QUAD I/O    | Lectura Quad I/O                |

### 3.2 Documentación

Para cada comando se incluirá:

* Secuencia de utilización.
* Comando, dirección y ciclos dummy cuando correspondan.
* Datos obtenidos mediante la lectura.
* Forma de onda correspondiente.

---

# 4. Comandos de escritura y borrado

Esta sección presenta los comandos utilizados para modificar el contenido de la memoria Flash, incluyendo las operaciones de programación y borrado.

### 4.1 Índice de comandos

| Código      | Comando      | Función                  |
| ----------- | ------------ | ------------------------ |
| `0x02`       | Page Program | Programa hasta 256 bytes |
| `0x20`       | Sector Erase | Borra un sector de 4 KB  |
| `0xD8`       | Block Erase  | Borra un bloque de 64 KB |
| `0xC7` | Chip Erase   | Borra toda la memoria    |

### 4.2 Documentación

#### 4.2.1 Page Program

El comando 0x02 permite escribir en la memoria hasta un total de 256 Bytes, estos se empezaran a escribir a parir de la direccion dada por el maestro, si los datos enviados hacia el esclavo, llegan al final de una pagina, los siguientes es escribiran al inicio de la misma, y no pasaran a la siguiente pagina.

![Formas de onda para el comando Page program](Comando_page_program.png)



#### 4.2.2 Sector Erase

El comando `0x20` permite borrar un sector completo de la memoria, correspondiente a un tamaño de 4 KB. El borrado se realiza a partir de la dirección indicada por el maestro, tomando como referencia el sector al que pertenece dicha dirección.

![Formas de onda para el comando Sector Erase](Comando_sector_erase.png)

#### 4.2.3 Block Erase

El comando `0xD8` permite borrar un bloque completo de la memoria, correspondiente a un tamaño de 64 KB. El borrado se realiza a partir de la dirección indicada por el maestro, tomando como referencia el bloque al que pertenece dicha dirección.

![Formas de onda para el comando Block Erase](Comando_block_erase.png)

#### 4.2.4 Chip Erase

El comando `0xC7` permite borrar completamente el contenido de la memoria, eliminando los datos almacenados en todos sus sectores y bloques. A diferencia de los comandos anteriores, no requiere una dirección específica para determinar la zona que será borrada.

![Formas de onda para el comando Chip Erase](Comando_chip_erase.png)


Para cada comando se incluirá:

* Secuencia de utilización.
* Dirección y datos cuando correspondan.
* Forma de onda correspondiente.
* Secuencia de control necesaria antes y después de la operación.

---

# 5. Diagrama de flujo general del protocolo

Esta sección presenta el **diagrama de flujo general del protocolo SPI para la memoria Flash**, mostrando de manera global la secuencia que sigue una transacción desde la activación de `CS` hasta su finalización.

El diagrama de flujo utilizado para representar esta secuencia se encuentra en los archivos gráficos incluidos en este directorio.


---


