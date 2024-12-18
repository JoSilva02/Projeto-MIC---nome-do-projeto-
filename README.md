*Disciplina:* MIC014 – Hands-On Basic Desenvolvimento Orientado a Testes  
*Atividade:* Maker Aula 2   
*Nome dos participantes:* Andreza Oliveira Gonçalves, Fábio Aurélio Barros Alexandre, Jonathan Emerson Braga da Silva  
 
## **Projeto de botão de pânico com delay reduzido**
```c
#include <WiFi.h>
#include <HTTPClient.h>
#include <UrlEncode.h>

// Definição das credenciais do WiFi
const char* ssid = "MinhaRede"; // Nome da rede WiFi
const char* password = "Rede12345"; // Senha do WiFi

// Definição dos pinos do botão e LEDs
#define botao 21
#define led1 23
#define led2 2

bool flag = 1; // Variável de controle para evitar múltiplos envios da mensagem

// Número de telefone e chave da API para CallMeBot
String phoneNumber1 = "559584123456"; 
String apiKey1 = "xxx"; 
String phoneNumber2 = "xxx";
String apiKey2 = "6364292";

// Função para enviar a mensagem via CallMeBot API
bool sendMessage(String phoneNumber, String apiKey, String message) {
  String url = "https://api.callmebot.com/whatsapp.php?phone=" + phoneNumber +
               "&apikey=" + apiKey + "&text=" + urlEncode(message);
  
  HTTPClient http;
  http.begin(url); // Inicia a conexão com a URL
  http.addHeader("Content-Type", "application/x-www-form-urlencoded");
  
  int httpResponseCode = http.GET(); // Utiliza GET em vez de POST, o que pode ser mais rápido
  if (httpResponseCode == 200) {
    Serial.println("Mensagem enviada com sucesso");
    http.end();
    return true; // Retorna true se a mensagem foi enviada com sucesso
  } else {
    Serial.println("Erro no envio da mensagem");
    Serial.print("HTTP response code: ");
    Serial.println(httpResponseCode);
    http.end();
    return false; // Retorna false se houve erro no envio
  }
}

void setup() {
  Serial.begin(115200); 
  pinMode(botao, INPUT_PULLUP); 
  pinMode(led1, OUTPUT); 
  pinMode(led2, OUTPUT); 
  digitalWrite(led1, LOW); 
  digitalWrite(led2, LOW); 
  
  WiFi.begin(ssid, password); 
  Serial.println("Conectando");

  // Tenta conectar ao Wi-Fi sem delay longo
  while (WiFi.status() != WL_CONNECTED) {
    Serial.print(".");
    digitalWrite(led2, !digitalRead(led2)); 
    delay(100); // Reduzido de 500 para 100 ms
  }
  
  Serial.println("");
  Serial.print("Conectado ao WiFi neste IP ");
  digitalWrite(led2, HIGH); 
  Serial.println(WiFi.localIP()); 
}

void loop() {
  int estado_botao = digitalRead(botao); // Lê o estado do botão
  if (estado_botao == 0) { // Se o botão estiver pressionado
    Serial.println("Botão Pressionado, enviando mensagem");
    digitalWrite(led1, LOW); // Certifica-se de que o LED1 está apagado antes de tentar enviar a mensagem
    
    if (flag) { // Verifica se a flag está ativada
      // Tenta enviar a mensagem para o primeiro número
      if (sendMessage(phoneNumber1, apiKey1, "SOCORRO!!!!, ME AJUDE")) {
        digitalWrite(led1, HIGH); // Acende o LED1 se a mensagem foi enviada com sucesso
      }
      
      // Tenta enviar a mensagem para o segundo número
      if (sendMessage(phoneNumber2, apiKey2, "SOCORRO!!!!, Help me")) {
        digitalWrite(led1, HIGH); // Acende o LED1 se a mensagem foi enviada com sucesso
      }
      
      flag = 0; // Desativa a flag após o envio
    }
  } else {
    flag = 1; // Reseta a flag quando o botão não está pressionado
    digitalWrite(led1, LOW); // Apaga o LED1
  }
}
```

## Mudanças implementadas:  
**Redução do delay da verificação Wi-Fi:** O delay(500) foi reduzido para delay(100), diminuindo o tempo de espera enquanto o ESP32 tenta se conectar ao Wi-Fi.

**Uso de GET em vez de POST:** A mudança para o método GET pode ser mais rápida do que o POST, já que o GET não envolve o envio de um corpo de requisição, apenas os parâmetros na URL.

**Melhora na lógica do botão:** O código agora verifica de forma mais rápida o estado do botão para evitar que o processo de envio da mensagem seja interrompido por delays desnecessários.


