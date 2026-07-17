# Agente de Atendimento e Agendamento via WhatsApp

## Sobre o projeto

Este projeto apresenta um agente de Inteligência Artificial desenvolvido no n8n para realizar o atendimento de clientes de uma clínica de estética pelo WhatsApp.

O agente é capaz de responder dúvidas sobre os serviços oferecidos, listar os procedimentos disponíveis, informar preços e duração dos atendimentos, consultar horários disponíveis e realizar agendamentos diretamente no Google Calendar da clínica.

O fluxo também permite o recebimento de mensagens de texto e áudio, utiliza memória conversacional e possibilita que um funcionário da clínica intervenha manualmente no atendimento quando necessário.

O projeto está funcional e foi desenvolvido por meio da integração de diferentes ferramentas, APIs e serviços.

---

## Principais funcionalidades

- Recebimento de mensagens pelo WhatsApp;
- Atendimento automatizado com Inteligência Artificial;
- Respostas a dúvidas sobre os procedimentos;
- Listagem dos procedimentos oferecidos pela clínica;
- Informação de preços;
- Informação sobre a duração de cada procedimento;
- Recebimento e interpretação de mensagens de texto;
- Recebimento e transcrição de mensagens de áudio;
- Consulta de disponibilidade no Google Calendar;
- Criação de agendamentos;
- Confirmação do agendamento ao cliente;
- Memória do contexto da conversa;
- Agrupamento de mensagens fragmentadas;
- Tempo de espera antes do envio da resposta;
- Possibilidade de intervenção humana no atendimento.

---

## Como o fluxo funciona

### 1. Recebimento da mensagem

O fluxo começa por meio de um Webhook no n8n.

Esse Webhook recebe as mensagens enviadas pelo cliente pelo WhatsApp. A comunicação entre o WhatsApp e o n8n é realizada por meio da Evolution API.

```text
Cliente
   ↓
WhatsApp
   ↓
Evolution API
   ↓
Webhook
   ↓
n8n
```

---

### 2. Identificação do tipo de mensagem

Depois que a mensagem chega ao fluxo, um nó Switch identifica se o conteúdo recebido é:

- mensagem de texto;
- mensagem de áudio.

Cada tipo de mensagem segue um caminho específico dentro do workflow.

---

### 3. Processamento das mensagens de áudio

Quando o cliente envia um áudio, o fluxo realiza o tratamento do arquivo para que o conteúdo possa ser compreendido pelo agente.

O processamento acontece da seguinte forma:

1. o áudio é recebido por meio do WhatsApp;
2. o arquivo é obtido em formato Base64;
3. o conteúdo é preparado para processamento;
4. a API da Groq realiza a transcrição do áudio;
5. o texto transcrito é encaminhado ao agente de Inteligência Artificial.

Dessa forma, o cliente pode conversar com o agente utilizando mensagens digitadas ou mensagens de voz.

---

### 4. Separação entre mensagens automáticas e humanas

Outro nó Switch é utilizado para distinguir as mensagens enviadas pelo agente das mensagens enviadas manualmente por um funcionário da clínica.

Essa separação evita que uma mensagem enviada por um atendente seja interpretada pelo fluxo como uma nova solicitação do cliente.

O agente não realiza uma transferência automática, mas o fluxo foi estruturado para permitir a intervenção humana quando necessário.

---

### 5. Agrupamento de mensagens fragmentadas

É comum que clientes enviem várias mensagens curtas em sequência.

Por exemplo:

```text
Olá
Gostaria de saber o preço
Do procedimento facial
E se há horário amanhã
```

Para evitar que o agente responda separadamente a cada mensagem, o fluxo utiliza o Redis para armazenar temporariamente e agrupar essas mensagens.

Depois de um pequeno intervalo, as mensagens são reunidas e enviadas ao agente como uma única solicitação.

Isso ajuda o agente a compreender melhor o contexto geral da conversa.

---

### 6. Tempo de espera para resposta

O fluxo possui um nó configurado para aguardar alguns segundos antes de enviar a resposta ao cliente.

Esse intervalo permite:

- aguardar o envio de novas mensagens;
- reunir mensagens fragmentadas;
- evitar respostas muito rápidas;
- tornar o atendimento mais próximo de uma conversa humana.

O tempo de espera pode ser configurado dentro do workflow.

---

### 7. Atendimento com Google Gemini

Depois que a mensagem é tratada, o conteúdo é encaminhado ao agente de Inteligência Artificial.

O Google Gemini é utilizado como modelo principal do agente.

As informações sobre a clínica, os procedimentos, os preços e a duração dos serviços são disponibilizadas por meio das instruções inseridas no prompt.

O agente interpreta a solicitação do cliente e gera uma resposta de acordo com o contexto da conversa.

---

### 8. Memória da conversa

O agente utiliza o Postgres Chat Memory para manter o histórico da conversa.

Essa memória permite que o agente considere as mensagens anteriores durante o atendimento.

Com isso, o cliente não precisa repetir informações que já foram enviadas anteriormente.

---

### 9. Consulta de horários

Para consultar a agenda da clínica, o fluxo utiliza uma integração com o Google Calendar.

Por meio do MCP, o agente tem acesso a uma ferramenta responsável por verificar se o horário solicitado pelo cliente está disponível.

```text
Cliente solicita um horário
        ↓
Agente de IA
        ↓
MCP
        ↓
Consulta ao Google Calendar
        ↓
Verificação de disponibilidade
```

---

### 10. Realização do agendamento

Quando o horário solicitado está disponível, outra ferramenta conectada por meio do MCP cria o evento no Google Calendar da clínica.

Depois da criação do evento, o agente informa ao cliente que o atendimento foi agendado.

---

## Arquitetura simplificada

```text
Cliente
   ↓
WhatsApp
   ↓
Evolution API
   ↓
Webhook no n8n
   ↓
Identificação de texto ou áudio
   ↓
Transcrição de áudio com Groq
   ↓
Redis
   ↓
Postgres Chat Memory
   ↓
Google Gemini
   ↓
MCP
   ↓
Google Calendar
   ↓
Resposta enviada ao cliente
```

---

## Tecnologias e ferramentas utilizadas

| Tecnologia | Utilização |
|---|---|
| n8n | Construção e organização do workflow |
| WhatsApp | Canal de atendimento |
| Evolution API | Integração entre WhatsApp e n8n |
| Webhook | Recebimento das mensagens |
| Google Gemini | Inteligência Artificial principal do agente |
| Groq API | Transcrição das mensagens de áudio |
| Base64 | Tratamento do arquivo de áudio |
| Redis | Agrupamento temporário de mensagens fragmentadas |
| Postgres | Banco de dados utilizado pela memória |
| Postgres Chat Memory | Armazenamento do contexto da conversa |
| MCP | Disponibilização das ferramentas de agendamento |
| Google Calendar | Consulta de horários e criação de eventos |
| Google API | Integração com os serviços do Google |
| Hostinger VPS | Hospedagem da infraestrutura |
| JSON | Formato utilizado na exportação do workflow |

---

## Estrutura do repositório

```text
AGENTEWHATSAPP_CLINICA_DE_ESTETICA
│
├── README.md
├── .gitattributes
│
└── workflow
    └── agentewhatsapp_clinica_de_estetica.json
```

---

## Como importar o workflow

1. Faça o download do arquivo JSON disponível na pasta `workflow`;
2. Acesse a sua instância do n8n;
3. Crie um novo workflow;
4. Selecione a opção para importar um arquivo;
5. Escolha o arquivo JSON do projeto;
6. Configure as credenciais necessárias;
7. Configure a Evolution API;
8. Configure as credenciais do Google;
9. Configure o Google Calendar;
10. Configure o Google Gemini;
11. Configure a Groq API;
12. Configure o Redis;
13. Configure o Postgres;
14. Revise os Webhooks;
15. Realize os testes antes de ativar o fluxo.

---

## Segurança

Por motivos de segurança, o arquivo disponibilizado no GitHub não deve conter:

- senhas;
- tokens;
- chaves de API;
- credenciais do Google;
- credenciais do banco de dados;
- URLs privadas;
- números de telefone;
- dados de clientes;
- identificadores privados da Evolution API;
- Webhooks de produção.

As credenciais devem ser configuradas diretamente no ambiente do n8n.

---

## Possíveis melhorias futuras

- Envio automático de lembretes;
- Cancelamento de agendamentos;
- Reagendamento pelo WhatsApp;
- Integração com CRM;
- Cadastro automático de clientes;
- Confirmação de presença;
- Pesquisa de satisfação;
- Relatórios de atendimento;
- Dashboard com métricas;
- Transferência automática para atendimento humano.

---

## Autor

**Pablo Custódio Ferreira**

Projeto desenvolvido para demonstração de conhecimentos em:

- Automação de processos;
- Inteligência Artificial;
- Agentes de IA;
- IA conversacional;
- Integração de sistemas;
- n8n.