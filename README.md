# Lab 02 - Estudos do ESP-32 WROOM.

## Objetivo

Avaliar e analisar a arquitetura do dispositivo ESP32-WROOM.

## 1 - Estudo de Portas Digitais

### 1 Parte 1 - Portas de entrada:

A tensão máxima suportada pelas entradas do ESP-32 wroom é de 3,3V. As portas de entrada do ESP-32 wroom podem ter resistores de pullup e pulldown. Isso pode ser configurado no código do programa que estiver sendo executado. Além disso, as portas de entrada também podem ser configuradas para não ter resistores de pullup ou pulldown.

### 1 Parte 2 - Porta de saída:

A corrente máxima das portas de saída do ESP-32 wroom é de cerca de 12mA..

### 1 Parte 3 - Frequência máxima:

De acordo com as especificações do fabricante, a frequência máxima de saída das portas digitais do ESP-32 wroom é de 40MHz.
A frequência máxima de entrada nas portas do ESP-32 wroom dependerá do circuito externo conectado a elas e da capacidade desse circuito em responder às mudanças de sinal. Portanto, não há uma frequência máxima definida para as portas de entrada.

```C

int i = i;

void setup*(){
 pinMode(22, OUTPUT);
 Serial.begin(115200);
 }

void loop(){
 while(1)
 {
  i = i - i;
  digiralWrite(22,i);
 }
}

```
![image](https://user-images.githubusercontent.com/50549048/227588903-1693bc1f-202a-452a-a762-17efb9bc650f.png)

Segundo imagem, foi conseguido testar até 1.356Mhz, inserindo um código de loop com o intuito de colocar uma saída como alta e baixa repetidamente e então analisada pelo osciloscópio. 
### Referências:

Datasheet do ESP-32 wroom: https://www.espressif.com/sites/default/files/documentation/esp32-wroom-32d_esp32-wroom-32u_datasheet_en.pdf
Artigo sobre as especificações do ESP-32: https://randomnerdtutorials.com/esp32-specs/

## 2 - Interrupções

### 2 Parte 1 - Interrupção básica

Foi implementada uma função que ativa a interrupção, toda vez que o botão for pressionado, é gerado uma mensagem no console que confirma o funcionamento da interrupção. A seguir uma imagem que confirma o funcionamento do circuito.

```C
#define GPIO_BOTAO 5
#define BOUNCE 10
 
int CONT = 0;
unsigned long TIME_LAST = 0;

void IRAM_ATTR funcao_ISR(){
if ( (millis() - TIME_LAST) >= BOUNCE ){
CONT++;
TIME_LAST = millis();
}}
 
 
void setup(){
Serial.begin(115200);
pinMode(GPIO_BOTAO, INPUT);
attachInterrupt(GPIO_BOTAO, funcao_ISR, RISING);
}
 
void loop(){
Serial.print("Acionamentos: ");
Serial.println(CONT);
delay(1000);
}

```

https://user-images.githubusercontent.com/50549048/227590057-9f9ea423-baee-4beb-8cec-fbe02b263acd.mp4

### 2 Parte 2 - Medidor de Frequências

O código da parte 1 foi alterado para que agora funcione como medidor de frequência usando o módulo PCNT do ESP32, então foi conectado o gerador de função com uma onda quadrada de 100kHz.


```C
volatile unsigned long pulse_count = 0;
unsigned long last_interrupt_time = 0;

#define PIN_USING 5

void countPulse() {
  unsigned long interrupt_time = millis();
  if (interrupt_time - last_interrupt_time > 10) { // debounce the input
    pulse_count++;
    last_interrupt_time = interrupt_time;
  }
}

void setup() {
  Serial.begin(115200);
  pinMode(PIN_USING, INPUT_PULLUP);
  attachInterrupt(digitalPinToInterrupt(PIN_USING), countPulse, RISING);
}

void loop() {
  unsigned long interval = millis() - last_interrupt_time;
  if (interval > 1000) { // print the frequency every second
    unsigned long pulse_count_copy = pulse_count;
    pulse_count = 0;
    float frequency = (pulse_count_copy / 2.0) * (1000.0 / interval);
    Serial.print("Frequency: ");
    Serial.print(frequency);
    Serial.println(" Hz");
    last_interrupt_time = millis();
  }
}

```

Um gerador de sinal de entrada foi conectado com uma onda configurada em 1Mhz, conforme imagem a seguir: 

![image](https://user-images.githubusercontent.com/50549048/227591684-24d591e8-a2d2-40a2-be45-f65a93e34fc8.png)

E o valor obtido via terminal se aproximou do esperado, sendo de 0.94Mhz.

![image](https://user-images.githubusercontent.com/50549048/227591778-2771f6c7-2fc7-4000-80e9-68cc8ae8ce7c.png)

## 3 - Temporizadores e Conversão A/D

### 2 Parte 1 - Interrupção básica


