# InteligentHome-I
Primeiras Camadas de Inteligência no nosso Apartamento

   Utilizamos duas placas Esp32Cam para realizar o reconhecimento facial, através do Sketch de exemplo "camerawebserver". É possível encontrar diversos tutoriais na internet ensinando a utilizar o esp32cam na IDE Arduino para carregar o código na placa. Sugerimos o sguinte tutorial https://www.filipeflop.com/blog/esp-32-camera-ip/ , sendo que a única diferança é que não utilizamos o sketch proposto, mas o exemplo "camerawebserver" constante na ide arduino com as alterações que passaremos a relatar:

  Nosso desafio foi fazer com que a assistente Alexa reagisse de maneiras diferentes às faces detectadas pelas cãmeras IP. Como então enviar os gatilhos para que nossa querida Assistente Virtual pudesse: a) cumprimentar os visitantes; b) avisar que nosso filho de 3 anos entrou na cozinha do apartamento, c) e tentar me dissuadir de assaltar a geladeira. Vamos lá!! 

 PASSO 1: 
 
 Fizemos algumas alterações no exemplo "camera webserver" para que ele pudesse capturar o id da pessoa identificada ou do "invasor" e encaminhar para o servidor MQTT da Adafruit (https://io.adafruit.com/). Aqui o segredo foi adicionar o código necessário no arquivo  "app_httpd.cpp" para que, quando a função de reconhecimento facial fosse ativada ("static int run_face_recognition"), também ocorresse um "publish" no tópico correspondente (feed). O código com as alterações se sencontram no repositório.  Basicamente: 
      a. adicionamos as bibliotecas: <WiFi.h>; "Adafruit_MQTT.h"; dafruit_MQTT_Client.h";
      b. criamos as instâncias necessárias:
      
                  #define AIO_SERVER      "io.adafruit.com"
                  #define AIO_SERVERPORT  1883
                  #define IO_USERNAME  ".........."
                  #define IO_KEY       "................sua chave de acesso"

                  WiFiClient espClient32; // Cria o objeto espClient
                  Adafruit_MQTT_Client ada_mqtt(&espClient32, AIO_SERVER, AIO_SERVERPORT, IO_USERNAME, IO_KEY);
                  Adafruit_MQTT_Publish cameracozinha = Adafruit_MQTT_Publish(&ada_mqtt, IO_USERNAME "/feeds/cameracozinha");

      
      c.  Alteramos o método run_face_recognition, o qual ficou assim: 
      
                     static int run_face_recognition(dl_matrix3du_t *image_matrix, box_array_t *net_boxes){

               //funcionamento da camera;

                   dl_matrix3du_t *aligned_face = NULL;
                   int matched_id = 0;

                   aligned_face = dl_matrix3du_alloc(1, FACE_WIDTH, FACE_HEIGHT, 3);

                   if(!aligned_face){
                       Serial.println("Could not allocate face recognition buffer");
                       return matched_id;
                   }
        AQUI ->    if (align_face(net_boxes, image_matrix, aligned_face) == ESP_OK){

                                   //conecta ao mqtt broker quando o reconhecimento facial estiver ativo 

                                Serial.print("Connecting to Ada_MQTT... ");
                                while ((ada_mqtt.connect()) != 0)
                                { 
                                Serial.println("Retrying MQTT connection in 5 seconds...");
                                ada_mqtt.disconnect();
                                delay(5000);  // wait 5 seconds   
                                }
                                Serial.println("ADAFRUIT MQTT Connected!");  


                       if (is_enrolling == 1){
                           int8_t left_sample_face = enroll_face(&id_list, aligned_face);

                           if(left_sample_face == (ENROLL_CONFIRM_TIMES - 1)){
                               Serial.printf("Enrolling Face ID: %d\n", id_list.tail);
                           }
                           Serial.printf("Enrolling Face ID: %d sample %d\n", id_list.tail, ENROLL_CONFIRM_TIMES - left_sample_face);
                           rgb_printf(image_matrix, FACE_COLOR_CYAN, "ID[%u] Sample[%u]", id_list.tail, ENROLL_CONFIRM_TIMES - left_sample_face);
                           if (left_sample_face == 0){
                               is_enrolling = 0;
                               Serial.printf("Enrolled Face ID: %d\n", id_list.tail);
                           }
                       } else {
                           matched_id = recognize_face(&id_list, aligned_face);
                           if (matched_id >= 0) {
                               Serial.printf("Match Face ID: %u\n", matched_id);
                               cameracozinha.publish(String(matched_id).c_str());
                               rgb_printf(image_matrix, FACE_COLOR_GREEN, " Oi morador n. %u", matched_id);
                           } else {
                               Serial.println("No Match Found");
                               rgb_print(image_matrix, FACE_COLOR_RED, "Olá Visitante");
                               matched_id = -1;
                               cameracozinha.publish(String(matched_id).c_str());
                           }
                       }
                   } else {
                       Serial.println("Face Not Aligned");
                       //rgb_print(image_matrix, FACE_COLOR_YELLOW, "Human Detected");
                   }

                   dl_matrix3du_free(aligned_face);

                   return matched_id;
               }
               
               
 PASSO 2:
 
 Após isso sua espcam irá encaminhar os id's para o broker mqtt da adafruit (lembre-se de ter criado o feed (tópico) antes do passo 1. Nesse momento você terá o seguinte cenário: 
 - o feed receberá = -1 (invasor), de 0 em diante serão os sujeitos cadastrados.  Mas como fazê-lo chegar até a Alexa? A boa notícia é que o monstro mais feio já morreu e agora você precisará apenas usar o IFTTT (https://ifttt.com/) para interligar a sua conta adafruit com a sua conta alexa. Feito isso vamos passar aos ultimos dois passos
 
 
 PASSO3: 
 
 Entre na sua conta ifttt e cria um novo app :
 
 

 
 
 
 
 
 
 
 
 
 
