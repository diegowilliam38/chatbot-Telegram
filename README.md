# Chatbot de Clima no Telegram com n8n

Projeto desenvolvido para o desafio da pós-graduação: um chatbot no Telegram que recebe uma cidade no formato `Cidade,UF`, consulta a OpenWeather e responde com a temperatura atual. O workflow também inclui uma camada opcional com Google Gemini para melhorar a mensagem final, sem depender dele para funcionar, graças a um fallback determinístico em `Code node`.

## Funcionalidades

- Recebe mensagens de texto via **Telegram Trigger**
- Captura e normaliza a entrada do usuário em uma variável `queue`
- Aceita entradas como `Fortaleza,CE`, `São Paulo,SP` e `Belo Horizonte,MG`
- Consulta o clima atual na **OpenWeather**
- Retorna a temperatura atual em português
- Trata erros de cidade não encontrada
- Possui integração opcional com **Google Gemini**
- Possui **fallback obrigatório** sem IA

## Estrutura do workflow

1. **Telegram Trigger** recebe a mensagem
2. **Code node** extrai e normaliza a cidade, salvando em `queue`
3. **HTTP Request** chama a OpenWeather
4. **IF node** valida a resposta
5. Em sucesso:
   - **Code Fallback**
   - **Gemini (opcional)**
   - **Code Saída Final**
   - **Telegram Send Message**
6. Em erro:
   - **Code Mensagem Erro**
   - **Telegram Send Message**

## Exemplo de resposta

Sucesso:

```text
🌤️ A temperatura em Belo Horizonte é de 25°C.
```

Erro:

```text
❌ Cidade não encontrada. Use o formato Cidade,UF (ex.: São Paulo,SP).
```

## Arquivos da entrega

- `workflow-chatbot-telegram.json`
- `README.md`

Opcionalmente:
- `docker-compose.yml`

## Como importar o workflow no n8n

1. Acesse sua instância do n8n
2. Clique em **Workflows**
3. Clique em **Import from File**
4. Selecione o arquivo `workflow-chatbot-telegram.json`
5. Configure as credenciais necessárias
6. Salve e ative o workflow

## Credenciais necessárias

### Telegram

Crie um bot no Telegram com o **BotFather** e obtenha o token.

No n8n:
1. Abra um node do Telegram
2. Clique em **Create new credential**
3. Escolha **Telegram API**
4. Informe o token do bot

Variável esperada:

```env
TELEGRAM_BOT_TOKEN=seu_token_aqui
```

### OpenWeather

Crie uma conta na OpenWeather e gere sua API key.

Variável esperada:

```env
OPENWEATHER_API_KEY=sua_chave_aqui
```

No workflow, o node HTTP Request usa:

```text
{{$env.OPENWEATHER_API_KEY}}
```

### Google Gemini (opcional)

O Gemini é usado apenas para melhorar a escrita da resposta. O workflow continua funcionando sem ele porque existe um fallback determinístico em `Code node`.

No n8n:
1. Abra o node `Message a model`
2. Clique em **Create new credential**
3. Informe sua API key do Gemini
4. Salve a credencial

Fluxo do Gemini:

```text
IF sucesso -> Code Fallback -> Gemini -> Code Saída Final -> Telegram
```

## Como executar localmente com Docker / Portainer

1. Suba sua stack no Portainer usando o `docker-compose.yml`
2. Acesse o n8n no navegador
3. Configure as credenciais do Telegram e do Gemini
4. Defina a variável `OPENWEATHER_API_KEY`
5. Importe o workflow
6. Ative o workflow
7. Envie uma mensagem para o bot no Telegram

## Como testar o chatbot

Teste pelo menos estes cenários:

### Sucesso

```text
Fortaleza,CE
São Paulo,SP
Curitiba,PR
```

### Erro

```text
CidadeInexistente,ZZ
```

Resultado esperado:

```text
❌ Cidade não encontrada. Use o formato Cidade,UF (ex.: São Paulo,SP).
```

## Normalização da entrada

Exemplo:

```text
São Paulo,SP
```

vira:

```text
sao paulo,br
```

Isso mantém o formato pedido no desafio e melhora a compatibilidade com a API.

## Checklist antes de enviar

- [ ] Testei com pelo menos 3 cidades
- [ ] Testei uma cidade inexistente
- [ ] O workflow responde com mensagem amigável
- [ ] O arquivo JSON não contém tokens ou credenciais reais
- [ ] O `README.md` explica importação e configuração
- [ ] O repositório está público
- [ ] O `docker-compose.yml`, se incluído, está sem segredos reais
