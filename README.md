🌤️ Chatbot de Clima no Telegram com N8N
Bot no Telegram que informa a temperatura atual de qualquer cidade do Brasil em tempo real, usando a API gratuita do OpenWeather e opcionalmente o Google Gemini para respostas mais naturais.
---
🐳 Como rodar o N8N com Docker
Copie o arquivo `docker-compose.yml` deste repositório para uma pasta local.
Abra o terminal nessa pasta e execute:
```bash
docker compose up -d
```
Acesse o N8N em: http://localhost:5678
Crie sua conta de administrador na primeira vez que acessar.
---
🔑 Variáveis de Ambiente
As seguintes variáveis devem ser configuradas no N8N antes de ativar o workflow.
Variável	Descrição	Obrigatória
`OPENWEATHER_API_KEY`	Chave da API do OpenWeather	✅ Sim
`TELEGRAM_BOT_TOKEN`	Token do bot gerado pelo BotFather	✅ Sim
`GEMINI_API_KEY`	Chave da API do Google Gemini	⬜ Opcional
Como configurar no N8N
No N8N, acesse Settings → n8n Environment Variables (ou edite o `docker-compose.yml`).
Adicione as variáveis conforme a tabela acima.
Nunca suba essas chaves para o repositório.
---
📥 Como importar o Workflow
No N8N, clique em Workflows no menu lateral.
Clique em Import from file.
Selecione o arquivo `workflow-chatbot-telegram.json` deste repositório.
O workflow será carregado com todos os nós conectados.
---
🔐 Como configurar as credenciais no N8N
Telegram
No N8N, vá em Credentials → New Credential.
Busque por Telegram API.
Cole o `TELEGRAM_BOT_TOKEN` no campo Access Token.
Salve com o nome "Telegram Bot".
No workflow, clique nos nós Telegram Trigger, Enviar Resposta e Enviar Erro e vincule essa credencial.
OpenWeather
A chave do OpenWeather é usada via variável de ambiente (`$env.OPENWEATHER_API_KEY`) diretamente no nó HTTP Request. Nenhuma credencial separada é necessária — basta que a variável esteja configurada no ambiente do N8N.
Google Gemini (Opcional)
O nó Google Gemini (Opcional) usa a variável `$env.GEMINI_API_KEY`. Se a variável não estiver configurada, o nó Fallback / Processar Gemini detecta a falha e gera a mensagem de forma determinística automaticamente — sem interromper o fluxo.
> **Onde o Gemini foi inserido:** o nó fica entre "Extrair Temperatura" e "Fallback / Processar Gemini". Ele tenta reescrever a mensagem de forma mais amigável em JSON. Se falhar (sem credencial ou erro de API), o Code node seguinte usa a mensagem padrão gerada localmente.
---
▶️ Como executar e testar o chatbot
Com o workflow importado e as credenciais configuradas, ative o workflow clicando no toggle no canto superior direito.
Abra o Telegram e envie uma mensagem para o seu bot no formato:
```
Cidade,UF,BR
```
Exemplos de teste
Mensagem enviada	Resposta esperada
`São Paulo,SP,BR`	🌤️ A temperatura em São Paulo é de 28°C.
`Belo Horizonte,MG,BR`	🌤️ A temperatura em Belo Horizonte é de 25°C.
`Florianópolis,SC,BR`	🌤️ A temperatura em Florianópolis é de 22°C.
`CidadeInexistente,XX,BR`	❌ Cidade não encontrada. Use o formato Cidade,UF,BR (ex.: São Paulo,SP,BR).
---
🗺️ Estrutura do Workflow
```
Telegram Trigger
    └─► Capturar Mensagem (Set)
            └─► Formatar Cidade (Set - normalize/lowercase)
                    └─► OpenWeather API (HTTP Request)
                            └─► Resposta OK? (IF)
                                    ├─► [SIM] Extrair Temperatura (Set)
                                    │           └─► Google Gemini (HTTP - Opcional)
                                    │                       └─► Fallback / Processar Gemini (Code)
                                    │                                   └─► Enviar Resposta (Telegram)
                                    └─► [NÃO] Preparar Erro (Set)
                                                └─► Enviar Erro (Telegram)
