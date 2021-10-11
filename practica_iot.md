# Práctica: enviar métricas a Elasticsearch

## Configuración de Elasticsearch

#### Descargar
 - Ubicarnos mediante una terminal en el directorio donde queramos trabajar
 - Descargar el paquete completo de elasticsearch: `wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.9.1-linux-x86_64.tar.gz`
 - Descargar el checksum necesario para comprobar que el paquete es válido: `wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.9.1-linux-x86_64.tar.gz.sha512`
 - Comprobar la validez del paquete: `shasum -a 512 -c elasticsearch-7.9.1-linux-x86_64.tar.gz.sha512`

#### Descomprimir
 - Descomprimimos el contenido del paquete: `tar -xzf elasticsearch-7.9.1-linux-x86_64.tar.gz`
 - Entramos en la carpeta de elasticsearch: `cd elasticsearch-7.9.1/`

#### Ejecutar
 - Ejecutamos elasticsearch : `./bin/elasticsearch`
 - Elasticsearch ya viene con una configuración base por defecto, pero según nuestras necesidades, deberemos encontrar su punto óptimo
 - Para comprobar que funciona: `curl -X GET "localhost:9200/?pretty"`

#### Configurar
 - Fichero de configuración: `elasticsearch-7.9.1/config/elasticsearch.yml`
 ```
 elasticsearch-7.4.2/
├── bin/
├── config/
│     ├── elasticsearch.yml
│     └── ...
├── data/
├── jdk/
├── lib/
├── logs/
├── modules/
└── plugins/
```
 - Retirar el “#” de una línea para que esa configuración tenga efecto
 - Por defecto no acepta llamadas de otras máquinas. Para ello, modificar la línea: `network.host: 0.0.0.0`
 - También necesitamos configurar el discovery con la línea: `cluster.initial_master_nodes: node-1`
 - Damos más memoria virtual de Java al sistema: `sudo sysctl -w vm.max_map_count=262144`
 - Reiniciamos el servicio de Elasticsearch: Ctrl+C en la consola en la que está corriendo y volvemos a lanzar `./bin/elasticsearch`

## Configuración de Logstash

#### Java
 - Para utilizar logstash nuestro sistema debe contar con Java 8 o Java 11, ya sea de oracle o de OpenJDK. En una terminal comprobamos si nuestro sistema cumple el requisito: `java -version`
 - Si el resultado es “Command 'java' not found” o una salida no correspondiente a Java 8 u 11, debemos realizar el paso previo de integrar Java en el sistema:
```
sudo add-apt-repository ppa:openjdk-r/ppa
sudo apt update
sudo apt install openjdk-11-jre-headless
```

#### Instalación
 - Descargamos el paquete tar.gz desde la web de elastic a una ubicación a elegir de nuestro sistema de ficheros: `wget https://artifacts.elastic.co/downloads/logstash/logstash-7.9.1.tar.gz`
 - Descomprimimos el paquete: `tar -xvzf logstash-7.9.1.tar.gz`
 - Entramos en la carpeta de logstash: `cd logstash-7.9.1/`
 - Ejecutamos logstash consultando la versión: `./bin/logstash --version`

#### Configuración y ejecución
 - Para ejecutar: `./bin/logstash -f logstash.conf --config.reload.automatic`
 - Pero para ello necesitamos hacer un fichero de configuración primero
 - En este punto de la præctica no es necesaria esta configuración, más adelante se indicarán los pasos a seguir

#### Estructura de fichero .conf

```
input {

}
filter {

}
output {

}
```
 - Ejemplos de cláusula `input`:

```
Entrada de teclado:
input {
    stdin {}
}

Entrada de fichero:
input {
    file {
        path => "/var/log/apache.log"
        type => "apache-access"    #Asignar un tipo (útil más adelante)
        start_position => "beginning" # Comenzar siempre desde el principio
    }
}

```

 - Ejemplos de cláusula `filter`:

```
filter {
  grok {
    match => { "message" => "%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration}" }
  }
}
```
Links útiles de filtro grok:
 - Patrones preconfigurados: https://github.com/elastic/logstash/blob/v1.4.2/patterns/grok-patterns
 - Introducción al uso de GROK: https://logz.io/blog/logstash-grok/
 - GROK debugger: https://grokdebug.herokuapp.com/

 - Ejemplos de cláusula `output`:

```
Imprimir por consola:
output {
     stdout {}
}

Enviarlos a elasticsearch e imprimirlos por consola a la vez:
output {
    elasticsearch { hosts => ["localhost:9200"] }
    stdout { codec => rubydebug } #Codificarlo de una manera más estructurada y legible en consola
}
```

## Configuración de Kibana

#### Descargar
 - Ubicarnos mediante una terminal en el directorio donde queramos trabajar
 - Descargar el paquete completo de kibana: `wget https://artifacts.elastic.co/downloads/kibana/kibana-7.9.1-linux-x86_64.tar.gz`
 - Descargar el checksum necesario para comprobar que el paquete es válido: `wget https://artifacts.elastic.co/downloads/kibana/kibana-7.9.1-linux-x86_64.tar.gz.sha512`
 - Comprobar la validez del paquete: `shasum -a 512 -c kibana-7.9.1-linux-x86_64.tar.gz.sha512`

#### Descomprimir
 - Descomprimimos el contenido del paquete: `tar -xzf kibana-7.9.1-linux-x86_64.tar.gz`
 - Entramos en la carpeta de kibana: `cd kibana-7.9.1-linux-x86_64/`

#### Ejecutar
 - Ejecutamos Kibana: `./bin/kibana`
 - En http://localhost:5601 deberíamos tener ya disponible el panel de Kibana

#### Prueba inicial
 - Para comprobar el correcto funcionamiento de kibana, nosotr@s vamos a cargar datos directamente de un fichero CSV: https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_week.csv
 - También podríamos usar datos de ejemplo para explorar las visualizaciones
 - Desde la portada de kibana podemos subir directamente ficheros para tratarlos y convertirlos en índices
 - Desde `Discover` podemos verificar los eventos registrados
 - Desde `Visualize` creamos nuestras gráficas y visualizaciones
 - Desde `Dashboard` podemos crear paneles que agrupen distintos gráficos y visualizaciones

## Conexión con la Raspberri Pi

 - Para mayor facilidad, conectar por HDMI a cualquier monitor
 - Conectar alimentación y cable ethernet a una de las tomas del laboratorio
 - Se puede usar el comando `ip a` para saber la IP de la Raspberri, y poder conectarse a ella por SSH si es necesario
 - Usuario:contraseña por defecto: `pi:raspberry`
 - Conexión con el DHT: ver diagrama de pines
 - Una vez conectado, probar el DHT:
```
cd DHT11/python
sudo python test.py {11 o 22 dependiendo del modelo} {GPIO pin conectado}
```
 - Una vez probado el sensor, podemos enviar sus datos a Logstash, para ello configuramos un fichero de configuración en el ordenador:
```
input {
  tcp {
    port => 12345
    codec => json
  }
}

filter {}

output {
  stdout {}
}
```
 - Esta configuración nos permitirá mostrar por pantalla los logs de la Raspberry en crudo. Activamos logstash: `./bin/logstash -f logstash.conf --config.reload.automatic`
 - En la raspberry, modificamos el fichero `start_sensor_logging.py` para que apunte a la IP de nuestro ordenador (podemos verla con `ip a`) y al puerto 12345
 - Si todo funciona bien, podemos empezar a procesar la entrada modificando la cláusula `filter`:
 ```
 filter {
   grok {
     match => { '@message' => '%{DATESTAMP:fecha_hora} Temperatura=%{NUMBER:temperatura}\*  Humedad=%{NUMBER:humedad}\% SensorType=%{WORD:tipo_sensor} SensorID=%{UUID:id_sensor}' }
   }
 }
 ```
 - Volvemos a lanzar tanto logstash como el script de la Raspberry (`sudo python test.py {11 o 22 dependiendo del modelo} {GPIO pin conectado}`) y visualizamos por la pantalla de logstash
 - Si se rellenan los nuevos campos de logstash, mandaremos como output a elastic, como se puede ver al principio del archivo

## Gestión de Kibana

 - Las métricas estarán llegando al Elasticsearch, por lo que podremos visualizarlas en Kibana. Para ello vamos a `Stack Management > Index Patterns` y creamos un nuevo index pattern que apunte a nuestro nuevo índice (por defecto se llamará `logstash-` seguido de la versión y la fecha, por lo que el index pattern deberá llamarse `logstash-*` o similar)
 - Puede que los campos que deberían ser tratados como números (temperatura, humedad) hayan sido captados por defecto como campos de texto. Para que sean tratados como los números que son, vamos a definir su tipo con un Index Template. En la versión propietaria de Elasticsearch dan muchas más facilidades para tratar estos templates, pero como estamos en la versión open source hay que crearse un json y llamar a la API. Para no liarnos mucho en este punto, aquí tenéis la llamada, que tendréis que ejeutar en el apartado `Dev Tools` de Kibana:
```
POST _template/practica_iot
{
  "order" : 1,
  "index_patterns" : [
    "logstash*"
  ],
  "settings" : {
    "index" : {
      "mapping" : {
        "total_fields" : {
          "limit" : "10000"
        }
      },
      "refresh_interval" : "5s",
      "number_of_routing_shards" : "30",
      "number_of_shards" : "1",
      "number_of_replicas" : "1"
    }
  },
  "mappings" : {
    "properties": {
      "temperatura" : {
        "type" : "float"
      },
      "humedad" : {
        "type": "float"
      }
    }
  },
  "aliases" : {}
}
```
 - Después de aplicar el cambio, iremos a `Index Management > Indices` y borraremos el índice creado antes de definir el mapping, para que al relanzar el script de la Raspberry tengamos datos nuevos con los campos bien puestos
 - Ahora podremos hacer visualizaciones y dashboards usando los parámetros de temperatura y humedad
