============================================================
PROJETO: CHATBOT TELEGRAM - TEMPERATURA OPENWEATHER
============================================================

1. DESCRIÇÃO DO PROJETO
-----------------------

Este projeto implementa um chatbot no Telegram utilizando o n8n para consultar a temperatura atual de uma cidade por meio da API OpenWeather.

O usuário envia uma mensagem ao bot com o nome de uma cidade. O workflow recebe a mensagem, consulta a API meteorológica, valida a resposta e retorna a temperatura em graus Celsius.

Caso a consulta não seja bem-sucedida, o chatbot envia uma mensagem informando que não conseguiu localizar a cidade.

Tecnologias utilizadas:
- n8n: automação e execução do workflow.
- Telegram Bot API: recebimento e envio de mensagens.
- OpenWeather API: consulta de condições meteorológicas.
- JavaScript: tratamento dos dados e formatação das mensagens.

Nome do workflow:
Chatbot Telegram - Temperatura OpenWeather


2. FUNCIONAMENTO DO FLUXO
-------------------------

O workflow possui oito nós principais, organizados na seguinte sequência:

1. Telegram Trigger
2. Preparar entrada
3. Consultar OpenWeather
4. Validar resposta
5. Formatar temperatura
6. Preparar mensagem de erro
7. Enviar temperatura
8. Enviar erro

O fluxo funciona da seguinte maneira:

ETAPA 1 - RECEBER MENSAGEM

Nó: Telegram Trigger

Recebe as mensagens enviadas pelos usuários ao chatbot do Telegram.

O fluxo está configurado para receber atualizações do tipo "message".

ETAPA 2 - PREPARAR ENTRADA

Nó: Preparar entrada

Responsável por preparar as informações recebidas.

São criados três campos:

- queue: mensagem original, com espaços desnecessários removidos.
- queue_normalizada: nome da cidade normalizado, sem acentos e em letras minúsculas, acrescido de ",br".
- chatId: identificador da conversa no Telegram, utilizado para responder ao usuário.

Exemplo de normalização:

Entrada: São Paulo

Valor normalizado: sao paulo,br

ETAPA 3 - CONSULTAR A API OPENWEATHER

Nó: Consultar OpenWeather

Realiza uma requisição HTTP GET para o seguinte endpoint:

https://api.openweathermap.org/data/2.5/weather

Parâmetros utilizados:

- q: cidade normalizada.
- appid: chave de acesso da API OpenWeather.
- units: metric, para retornar a temperatura em graus Celsius.
- lang: pt_br, para solicitar a descrição meteorológica em português.

A chave da API é obtida por meio da variável do n8n:

$vars.OPENWEATHER_API_KEY

A configuração também define o cabeçalho Accept como application/json.

O nó está configurado para permitir o tratamento das respostas HTTP sem interromper automaticamente o fluxo em caso de erro.

ETAPA 4 - VALIDAR RESPOSTA

Nó: Validar resposta

Verifica se a consulta retornou informações suficientes para gerar uma resposta válida.

São verificadas as seguintes condições:

- O código de resposta indica sucesso, com status 200 ou código equivalente.
- A temperatura foi retornada como um valor numérico.
- O nome da cidade está disponível na resposta.

Se a resposta for válida, o fluxo segue para Formatar temperatura.

Caso contrário, segue para Preparar mensagem de erro.

ETAPA 5 - FORMATAR TEMPERATURA

Nó: Formatar temperatura

Extrai o nome da cidade e a temperatura retornados pela API.

A temperatura é arredondada para o número inteiro mais próximo.

A mensagem é montada no seguinte formato:

🌤️ A temperatura em São Paulo é de 25°C.



ETAPA 6 - PREPARAR MENSAGEM DE ERRO

Nó: Preparar mensagem de erro

Quando a consulta não retorna os dados esperados, prepara uma mensagem de erro para o usuário.

Mensagem configurada:

❌ Cidade não encontrada. Use o formato Cidade,UF (ex.: São Paulo,SP).

ATENÇÃO:

A mensagem de erro sugere o formato Cidade,UF, mas a lógica atual acrescenta automaticamente ",br" ao texto recebido.

Portanto, a mensagem de erro e a normalização devem ser ajustadas para trabalhar com o mesmo formato de entrada.

Para o comportamento atual, o ideal é testar inicialmente enviando somente o nome da cidade, por exemplo: São Paulo.

ETAPA 7 - ENVIAR TEMPERATURA

Nó: Enviar temperatura

Envia a mensagem formatada ao usuário por meio do Telegram.

Utiliza os campos:

- chatId: identificador da conversa.
- message: texto com a temperatura.

ETAPA 8 - ENVIAR ERRO

Nó: Enviar erro

Envia a mensagem de erro ao usuário quando a consulta não retorna uma resposta válida.

O objetivo é informar o problema sem interromper desnecessariamente o atendimento do chatbot.


3. PRÉ-REQUISITOS
-----------------

Antes de executar o projeto, você precisa ter:

- Uma instância do n8n funcionando.
- Uma conta no Telegram.
- Um bot criado no Telegram por meio do BotFather.
- Uma credencial do Telegram configurada no n8n.
- Uma conta na plataforma OpenWeather.
- Uma chave de API válida da OpenWeather.

A chave da OpenWeather deve permitir o acesso ao endpoint utilizado pelo workflow.

Documentação oficial:

OpenWeather API:
https://openweathermap.org/api

Telegram Bot API:
https://core.telegram.org/bots/api

n8n:
https://n8n.io/


4. CONFIGURAR A CREDENCIAL DO TELEGRAM
-------------------------------------

PASSO 1 - CRIAR O BOT

1. Abra o Telegram.
2. Procure pelo usuário oficial @BotFather.
3. Envie o comando /newbot.
4. Informe o nome do bot.
5. Defina um username disponível, terminado em "bot".
6. Copie o token fornecido pelo BotFather.



PASSO 2 - CONFIGURAR NO N8N

1. Acesse o n8n.
2. Abra a área de Credentials.
3. Crie uma nova credencial do tipo Telegram API.
4. Informe o token fornecido pelo BotFather.
5. Salve a credencial.
6. Abra o workflow e confira se os nós Telegram Trigger e Enviar temperatura e Enviar erro utilizam a credencial correta.

A credencial referenciada no arquivo JSON original é denominada:

Telegram account

Se essa credencial não existir na sua instância, selecione a credencial que você criou.

A referência de uma credencial no JSON não garante que os dados de autenticação estejam disponíveis em outra instância do n8n.


5. CONFIGURAR A API OPENWEATHER
-------------------------------

PASSO 1 - OBTER A CHAVE

1. Acesse https://openweathermap.org/.
2. Crie uma conta ou entre na sua conta existente.
3. Acesse a área de API keys.
4. Gere ou copie uma chave de API.
5. Verifique se a chave está habilitada para utilização no serviço Current Weather Data.

A ativação de uma chave nova pode levar algum tempo.

PASSO 2 - CRIAR A VARIÁVEL NO N8N

O workflow utiliza a seguinte expressão:

$vars.OPENWEATHER_API_KEY

Por isso, a variável precisa existir no ambiente do n8n.

Procedimento:

1. Acesse a área de configurações da sua instância do n8n.
2. Localize a funcionalidade Variables, caso esteja disponível na sua instalação.
3. Crie uma variável com o nome:

OPENWEATHER_API_KEY

4. Informe como valor a chave de API fornecida pela OpenWeather.
5. Salve a configuração.
6. Retorne ao workflow e confira se a variável pode ser utilizada.

O nome da variável diferencia maiúsculas e minúsculas. Digite exatamente:

OPENWEATHER_API_KEY

Não coloque o nome da variável entre aspas ao cadastrá-la.




6. IMPORTAR O WORKFLOW NO N8N
----------------------------

PASSO 1 - PREPARAR O ARQUIVO JSON

1. Copie o JSON completo do workflow.
2. Cole o conteúdo em um editor de texto.
3. Salve o arquivo com o nome:

chatbot-telegram-openweather.json

4. Selecione a codificação UTF-8, se essa opção estiver disponível.

Confira se o arquivo contém o JSON completo, desde a chave inicial até a chave final.

PASSO 2 - IMPORTAR

1. Acesse o n8n.
2. Abra a área de workflows.
3. Crie um workflow ou utilize a opção de importar workflow de arquivo, conforme a interface disponível na sua versão.
4. Selecione o arquivo chatbot-telegram-openweather.json.
5. Aguarde a importação.
6. Confira se todos os nós e conexões foram carregados corretamente.

PASSO 3 - CONFERIR AS CONFIGURAÇÕES

Antes de executar:

- Confira a credencial do Telegram.
- Confira a variável OPENWEATHER_API_KEY.
- Abra o nó Consultar OpenWeather e verifique os parâmetros da requisição.
- Confira as conexões entre os nós.
- Verifique se não há erros de configuração.

O JSON contém referências a credenciais e identificadores da instância original. Esses dados não substituem a configuração das credenciais no ambiente de destino.


7. COMO EXECUTAR O CHATBOT
--------------------------

PASSO 1 - ATIVAR O WORKFLOW

O arquivo JSON original indica que o workflow está ativo.

Entretanto, após a importação, confirme o estado do workflow na sua própria instância.

Se estiver inativo, ative-o depois de configurar as credenciais e as variáveis necessárias.

O Telegram Trigger precisa estar funcionando para receber as mensagens.

PASSO 2 - ABRIR O TELEGRAM

1. Abra o Telegram.
2. Procure pelo username do bot que você criou.
3. Abra a conversa.
4. Clique em Iniciar ou envie o comando /start, se necessário.

PASSO 3 - ENVIAR UMA CIDADE

Envie a seguinte mensagem:

São Paulo

O workflow normalizará o texto e tentará consultar a OpenWeather.

PASSO 4 - CONFERIR A RESPOSTA

Se a consulta funcionar, o bot deverá responder com uma mensagem semelhante a:

🌤️ A temperatura em São Paulo é de 25°C.

A temperatura é ilustrativa e não representa uma consulta real.

O resultado efetivo depende das condições meteorológicas retornadas pela API no momento da execução.





8. RESUMO DO FLUXO
-------------------

Telegram Trigger
        |
        v
Preparar entrada
        |
        v
Consultar OpenWeather
        |
        v
Validar resposta
        |
        +-----------------------+
        |                       |
        v                       v
Resposta válida          Resposta inválida
        |                       |
        v                       v
Formatar temperatura    Preparar mensagem de erro
        |                       |
        v                       v
Enviar temperatura      Enviar erro
        |                       |
        v                       v
Resposta no Telegram    Erro informado no Telegram


