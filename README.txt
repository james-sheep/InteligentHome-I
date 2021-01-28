# InteligentHome-I
Primeiras Camadas de Inteligência no nosso Apartamento

   Utilizamos duas placas Esp32Cam para realizar o reconhecimento facial, através do Sketch de exemplo "camerawebserver". É possível encontrar diversos tutoriais na internet ensinando a utilizar o esp32cam na IDE Arduino para carregar o código na placa. Sugerimos o sguinte tutorial https://www.filipeflop.com/blog/esp-32-camera-ip/ , sendo que a única diferança é que não utilizamos o sketch proposto, mas o exemplo "camerawebserver" constante na ide arduino com as alterações que passaremos a relatar:

  Nosso desafio foi fazer com que a assistente Alexa reagisse de maneiras diferentes às faces detectadas pelas cãmeras IP. Como então enviar os gatilhos para que nossa querida Assistente Virtual pudesse: a) cumprimentar os visitantes; b) avisar que nosso filho de 3 anos entrou na cozinha do apartamento, c) e tentar me dissuadir de assaltar a geladeira. Vamos lá!! 

1. Fizemos algumas alterações no exemplo "camera webserver" para que ele pudesse capturar o id da pessoa identificada ou do "invasor" e encaminhar para o servidor MQTT da Adafruit (https://io.adafruit.com/)
