# Check Point 05 - Edge Computing & Computer Systems

## Vinheria Agnello - IoT

Projeto desenvolvido para o Check Point 05 da disciplina **Edge Computing & Computer Systems**, com o objetivo de monitorar as condições de armazenamento da Vinheria Agnello utilizando sensores, ESP32, MQTT e a plataforma FIWARE.

---

## Links do projeto

### Projeto no Wokwi

https://wokwi.com/projects/478694558326385665

### Vídeo de demonstração

https://www.youtube.com/watch?v=o7-djNSjPR0

---

## Integrantes

- Joaquim Gaspardo - RM572208
- Gabriel Flausino - RM572486
- Gustavo Ducatti - RM570932
- Felipe Kenji - RM568739

---

## Objetivo

O projeto realiza o monitoramento de três informações importantes para o armazenamento dos vinhos:

- Temperatura
- Umidade
- Luminosidade

Os dados são coletados por sensores conectados a um ESP32 simulado no Wokwi e enviados pela internet utilizando o protocolo MQTT.

Além da comunicação MQTT, os dados são enviados para a plataforma FIWARE, permitindo consultar os valores atuais e também armazenar o histórico das medições.

---

## Tecnologias utilizadas

- ESP32
- Wokwi
- DHT22
- LDR
- Wi-Fi
- MQTT
- MyMQTT
- FIWARE
- IoT Agent MQTT
- Orion Context Broker
- STH-Comet
- Postman
- C++ / Arduino

---

## Sensores

### DHT22

Responsável pela leitura de:

- Temperatura
- Umidade

Ligação utilizada:

```text
VCC  -> 3V3
DATA -> GPIO 15
GND  -> GND
```

---

### LDR

Responsável pela leitura da luminosidade do ambiente.

Ligação utilizada:

```text
VCC -> 3V3
GND -> GND
AO  -> GPIO 34
```

O pino digital `DO` não foi utilizado.

---

## Arquitetura do projeto

```text
DHT22 ---------\
                \
                 ESP32
                /
LDR -----------/
      |
      | Wi-Fi
      v
     MQTT
      |
      v
Servidor IoT
3.135.234.101
      |
      +------------------+
      |                  |
      v                  v
   MyMQTT          IoT Agent MQTT
                         |
                         v
                  Orion Context Broker
                         |
                         v
                     STH-Comet
                         |
                         v
                  Histórico de dados
```

---

## Configuração Wi-Fi

O ESP32 utiliza a rede disponibilizada pelo Wokwi:

```text
SSID: Wokwi-GUEST
Senha: sem senha
```

---

## Servidor MQTT

O projeto utiliza o servidor disponibilizado para o Check Point 05:

```text
Broker MQTT: 3.135.234.101
Porta: 1883
```

---

## Tópicos MQTT

### Envio dos sensores

```text
/TEF/vinheria001joaquim/attrs
```

Os dados são enviados utilizando o formato UltraLight:

```text
t|16.30|u|29.00|l|488
```

Onde:

```text
t = temperatura
u = umidade
l = luminosidade
```

---

### Envio de comandos para o ESP32

```text
/TEF/vinheria001joaquim/cmd
```

Esse tópico foi utilizado para testar a comunicação no sentido contrário, permitindo enviar uma mensagem pelo aplicativo MyMQTT e recebê-la no ESP32.

---

## Comunicação com MyMQTT

O aplicativo MyMQTT foi configurado utilizando:

```text
Host: 3.135.234.101
Porta: 1883
```

Para visualizar os dados enviados pelo ESP32 foi realizado o Subscribe no tópico:

```text
/TEF/vinheria001joaquim/attrs
```

Para testar a escrita pelo aplicativo foi utilizado o tópico:

```text
/TEF/vinheria001joaquim/cmd
```

Assim, o sistema permite comunicação nos dois sentidos:

```text
ESP32 -> MQTT -> MyMQTT
MyMQTT -> MQTT -> ESP32
```

---

## Configuração do FIWARE

### IoT Agent

O IoT Agent está disponível na porta:

```text
4041
```

Teste realizado:

```http
GET http://3.135.234.101:4041/iot/about
```

---

## FIWARE Service

Foi utilizado:

```text
fiware-service: vinheriajoaquim
fiware-servicepath: /
```

API Key:

```text
TEF
```

---

## Dispositivo cadastrado

Device ID:

```text
vinheria001joaquim
```

Entity ID:

```text
urn:ngsi-ld:Vinheria:001joaquim
```

Tipo:

```text
Vinheria
```

Transporte:

```text
MQTT
```

Protocolo:

```text
PDI-IoTA-UltraLight
```

---

## Mapeamento dos atributos

Os valores enviados pelo ESP32 são convertidos pelo IoT Agent da seguinte forma:

```text
t -> temperature
u -> humidity
l -> luminosity
```

---

## Orion Context Broker

O Orion é responsável por armazenar o estado atual da entidade.

Consulta utilizada:

```http
GET http://3.135.234.101:1026/v2/entities/urn:ngsi-ld:Vinheria:001joaquim
```

Headers:

```text
fiware-service: vinheriajoaquim
fiware-servicepath: /
```

Exemplo de resposta:

```json
{
  "id": "urn:ngsi-ld:Vinheria:001joaquim",
  "type": "Vinheria",
  "humidity": {
    "type": "Number",
    "value": 29
  },
  "luminosity": {
    "type": "Number",
    "value": 488
  },
  "temperature": {
    "type": "Number",
    "value": 16.3
  }
}
```

---

## Timestamp

Cada leitura recebida pelo FIWARE possui um `TimeInstant`, permitindo identificar quando o dado foi coletado.

Exemplo:

```json
"TimeInstant": {
  "type": "DateTime",
  "value": "2026-10-23T17:26:40.568Z"
}
```

---

## Histórico de dados

O armazenamento histórico é realizado através do STH-Comet.

Teste do serviço:

```http
GET http://3.135.234.101:8666/version
```

### Histórico de temperatura

```http
GET http://3.135.234.101:8666/STH/v2/entities/urn:ngsi-ld:Vinheria:001joaquim/attrs/temperature?type=Vinheria&lastN=10
```

### Histórico de umidade

```http
GET http://3.135.234.101:8666/STH/v2/entities/urn:ngsi-ld:Vinheria:001joaquim/attrs/humidity?type=Vinheria&lastN=10
```

### Histórico de luminosidade

```http
GET http://3.135.234.101:8666/STH/v2/entities/urn:ngsi-ld:Vinheria:001joaquim/attrs/luminosity?type=Vinheria&lastN=10
```

Headers utilizados nas consultas:

```text
fiware-service: vinheriajoaquim
fiware-servicepath: /
```

As respostas apresentam os valores históricos juntamente com o horário em que cada dado foi recebido.

---

## Como executar

### 1. Abrir o projeto no Wokwi

Acesse o projeto do ESP32 e verifique se estão presentes:

- ESP32
- DHT22
- LDR

### 2. Verificar as bibliotecas

Bibliotecas utilizadas:

```text
DHTesp
PubSubClient
WiFi
```

### 3. Executar a simulação

Clique em:

```text
Start Simulation
```

O ESP32 irá:

1. Conectar ao Wi-Fi.
2. Conectar ao servidor MQTT.
3. Ler temperatura e umidade do DHT22.
4. Ler luminosidade do LDR.
5. Publicar os valores no MQTT.
6. Enviar os valores para o FIWARE através do IoT Agent.

### 4. Testar pelo MyMQTT

Configure:

```text
Host: 3.135.234.101
Porta: 1883
```

Subscribe:

```text
/TEF/vinheria001joaquim/attrs
```

Publish:

```text
/TEF/vinheria001joaquim/cmd
```

### 5. Consultar o FIWARE

Utilize o Postman para consultar o Orion:

```http
GET http://3.135.234.101:1026/v2/entities/urn:ngsi-ld:Vinheria:001joaquim
```

### 6. Consultar o histórico

Utilize o STH-Comet para consultar os históricos de temperatura, umidade e luminosidade.

---

## Estrutura do repositório

```text
CP05-Edge-Computing/
|
|-- README.md
|-- sketch.ino
|-- diagram.json
|-- libraries.txt
|
`-- evidencias/
    |-- wokwi.png
    |-- mymqtt.png
    |-- orion.png
    `-- historico.png
```

---

## Arquivos necessários

Para executar o projeto no Wokwi devem estar disponíveis:

```text
sketch.ino
diagram.json
libraries.txt
```

---

## Resultado

O projeto permite monitorar remotamente as condições da Vinheria Agnello, enviando temperatura, umidade e luminosidade através de MQTT.

Os dados podem ser visualizados em tempo real pelo MyMQTT e pelo Orion Context Broker, além de serem armazenados historicamente pelo STH-Comet para futuras consultas e análises.