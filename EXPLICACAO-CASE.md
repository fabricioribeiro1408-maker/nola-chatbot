<p align="center">
  <img src="https://usenola.com.br/2026/assets/images/nola-logo-preto.svg" alt="Nola" width="160">
</p>

<h1 align="center">Chatbot CS Nola — Explicacao do Case</h1>
<p align="center"><strong>Fabricio Ribeiro</strong> · Customer Success Analyst · Maio 2026</p>

---

## 1. O Prompt do Bot

O prompt que controla o comportamento da Nola tem **10 secoes e mais de 300 linhas**. Ele nao e apenas uma instrucao generica — cada regra existe porque um teste real provou que era necessaria.

### Estrutura do prompt

| Secao | O que faz | Por que existe |
|-------|-----------|----------------|
| **Identidade** | Define tom acolhedor, profissional e didatico | O bot precisa soar como uma colega experiente, nao como um robo |
| **Seguranca** | Bloqueia tentativas de extrair o prompt, impede o bot de inventar informacoes | Em testes, o bot revelou o prompt inteiro quando um usuario pediu — essa secao corrige isso |
| **Escuta ativa** | Obriga o bot a perguntar antes de responder quando a duvida e vaga | O bot respondia sobre "notas fiscais" quando o cliente so disse "notas" — sem saber se era entrada, saida ou consumidor |
| **Transbordo** | Fluxo de 4 etapas para escalacao com coleta completa de dados | O bot disparava tickets sem contato do cliente, ou disparava 3 tickets em vez de 1 |
| **Empatia** | Detecta frustacao por CAPS LOCK, exclamacoes e palavras-chave | Clientes em momento critico (caixa travado no almoco) precisam de acolhimento antes de solucao |
| **Base de conhecimento** | Consulta obrigatoria antes de responder, regra anti-alucinacao | O bot confirmou integracao com Mercado Livre — que nao existe. Agora so confirma o que esta na base |
| **Regras de negocio** | Precos, concorrentes, clientes novos | Evita que o bot fale mal de concorrentes ou invente precos |
| **Contexto Nola** | Modulos, planos, publico-alvo | Da ao bot repertorio sobre a empresa |
| **Proibicoes** | 14 regras de "nunca faca" | Cada regra corrige uma falha especifica encontrada nos testes |
| **Exemplos** | 7 cenarios modelados com comportamento esperado | O LLM aprende melhor com exemplos do que com regras abstratas |

### Regra mais importante do prompt

> **O ticket de escalacao so e disparado UMA UNICA VEZ, e SOMENTE apos o cliente responder "sim" na confirmacao dos dados.**

Essa regra resolve o problema mais critico que encontrei: o bot chamava a ferramenta de email a cada etapa da escalacao, gerando 3 tickets duplicados. Agora o fluxo e rigido — pergunta canal, pergunta contato, confirma os dados, e so dispara apos o "sim".

---

## 2. Processo de Construcao

O bot nao nasceu pronto. Foi construido em **3 fases iterativas**:

### Fase 1 — MVP funcional

Construi a primeira versao com o basico: Chat Trigger, AI Agent, base de conhecimento no Google Sheets e um prompt inicial de ~50 linhas. O objetivo era ter algo que respondesse — mesmo que imperfeito.

**Desafios resolvidos nessa fase:**
- Groq (primeiro motor de IA escolhido) estourou o rate limit apos poucas mensagens. Migrei para **OpenRouter** com o modelo LLaMA 4 Scout, que e gratuito e nao tem rate limits agressivos
- A base com 60 FAQs completas ultrapassava o limite de tokens do modelo. Criei um Code node que **comprime cada FAQ para ~80 caracteres**, permitindo que tudo caiba no contexto
- O Gmail nao enviava emails formatados. Ajustei o Email Type para HTML e corrigi a ordem dos campos

### Fase 2 — Testes e diagnostico

Criei um **roteiro com 17 testes organizados em 6 dimensoes**:

| Dimensao | Qtd de testes | O que avalia |
|----------|--------------|--------------|
| Escuta ativa | 4 | O bot pergunta antes de responder? |
| Profundidade | 3 | As respostas sao completas ou rasas? |
| Escalacao | 4 | Coleta dados corretamente? Escala quando deve? |
| Memoria | 2 | Lembra nome e contexto? |
| Seguranca | 3 | Inventa respostas? Revela o prompt? |
| Fluxo completo | 2 | Atendimento do inicio ao fim funciona? |

Rodei todos os testes e mapeei **12 falhas criticas**:

1. Revelava o prompt inteiro quando pediam
2. Inventava integracoes que nao existem (ex: Mercado Livre)
3. Confirmava funcionalidades sem certeza (ex: PIX no PDV)
4. Respondia perguntas vagas sem qualificar
5. Repetia a saudacao em toda mensagem
6. Terminava toda resposta com "Ficou claro?"
7. Repetia a mesma resposta 3x quando o cliente dizia "nao entendi"
8. Escalava na 4a tentativa em vez da 3a
9. Disparava ticket sem contato do cliente
10. Disparava multiplos tickets por escalacao
11. Nao oferecia escalacao em situacoes urgentes
12. Falava "vou registrar o atendimento" em vez de fazer silenciosamente

### Fase 3 — Super Prompt

Com as 12 falhas mapeadas, reescrevi o prompt do zero. Cada secao, cada regra e cada exemplo foram criados para corrigir uma falha especifica. O resultado: **todas as 12 falhas foram eliminadas nos re-testes**.

> **Metodologia aplicada:** testar → identificar falha → corrigir no prompt → re-testar. E o mesmo ciclo usado em desenvolvimento de software (red-green-refactor), aplicado a engenharia de prompt.

---

## 3. Como Essa Solucao Ajuda no Dia a Dia de CS

### O problema real

Times de Customer Success de SaaS enfrentam um paradoxo: precisam dar atencao personalizada a cada cliente, mas gastam a maior parte do tempo respondendo as mesmas 20-30 duvidas repetitivas. Enquanto um analista explica como fechar o caixa pela decima vez no dia, um cliente estrategico em risco de churn espera na fila.

### O que o bot resolve

| Antes (sem bot) | Depois (com bot) |
|-----------------|------------------|
| Cliente espera 5-15 min por resposta | Resposta em menos de 5 segundos |
| 100% das duvidas vao para o time humano | ~70% resolvidas pelo bot automaticamente |
| Sem atendimento fora do horario | Disponivel 24/7, inclusive feriados e fins de semana |
| Tickets incompletos (faltam dados) | Tickets sempre com resumo, canal, contato e historico |
| Sem dados sobre duvidas frequentes | Log de cada interacao para analise e melhoria da base |
| Analistas sobrecarregados com duvidas simples | Time livre para retencao, expansao e onboarding |

### Impacto estrategico para CS

**1. Escala sem perder qualidade**
O bot nao cansa, nao erra por pressa e nao tem dia ruim. Ele mantem o mesmo padrao de atendimento as 3h da manha de um sabado e as 14h de uma segunda-feira. Isso e especialmente relevante para restaurantes, que operam em horarios nao-comerciais.

**2. Dados que antes nao existiam**
Cada interacao e registrada com categoria, pergunta, resposta e status. O gestor de CS consegue responder: quais modulos geram mais duvidas? A base de conhecimento esta cobrindo os temas certos? Qual a taxa de resolucao sem humano? Essas respostas permitem decisoes baseadas em dados.

**3. Tickets que ja chegam completos**
Quando o bot escala, ele entrega ao analista um chamado com: resumo do problema, canal preferido do cliente, numero/email de contato, historico da conversa, categoria e prioridade. O analista nao perde tempo perguntando o basico — ja pode agir.

**4. Primeiro passo para CS proativo**
Com os dados de atendimento centralizados, o time pode identificar padroes (ex: muitos clientes com duvida sobre NFC-e apos uma atualizacao) e agir proativamente — enviando comunicados, criando tutoriais ou atualizando a base antes que vire uma onda de tickets.

---

## 4. Ferramentas de IA Utilizadas

### Motor de IA — LLaMA 4 Scout (Meta)

| Item | Detalhe |
|------|---------|
| **Modelo** | meta-llama/llama-4-scout:free |
| **Fornecedor** | Meta (open-source) |
| **Acesso via** | OpenRouter API (gateway gratuito) |
| **Por que esse modelo** | Gratuito, boa capacidade de seguir instrucoes longas, bom raciocinio em portugues |
| **Alternativas testadas** | Groq (llama-3.1-70b, llama-3.3-70b, llama-3.1-8b) — todos estouraram rate limit ou limite de tokens |

### Plataforma de orquestracao — N8N

| Item | Detalhe |
|------|---------|
| **Versao** | N8N Cloud (mais recente) |
| **Funcao** | Orquestrar todos os workflows, conectar IA com Google Sheets e Gmail |
| **Recurso principal** | AI Agent (tipo Tools Agent) — permite que o LLM decida qual ferramenta usar em cada situacao |
| **Outros recursos** | Simple Memory (contexto de conversa), Chat Trigger (interface de chat), Webhook (receber dados) |

### Base de conhecimento — Google Sheets

| Item | Detalhe |
|------|---------|
| **Funcao** | Armazenar 60 FAQs + registrar logs de atendimento |
| **Por que Sheets** | Editavel por qualquer pessoa do time sem conhecimento tecnico. Adicionar uma nova FAQ e tao simples quanto preencher uma linha |
| **Estrutura** | ID, Categoria, Subcategoria, Pergunta, Resposta, Tags, Prioridade |

### Email — Gmail com OAuth

| Item | Detalhe |
|------|---------|
| **Funcao** | Enviar tickets de escalacao formatados em HTML |
| **Por que Gmail** | Integracao nativa com N8N via OAuth, sem necessidade de configurar SMTP ou app passwords |

### Frontend — HTML + CSS + JS + @n8n/chat

| Item | Detalhe |
|------|---------|
| **Funcao** | Landing page com chat embutido |
| **Por que arquivo unico** | Deploy instantaneo via GitHub Pages, sem build, sem dependencias |
| **Widget de chat** | @n8n/chat — widget oficial do N8N para embed em sites externos |

---

<p align="center">
  <strong>Fabricio Ribeiro</strong><br>
  Case Tecnico · Customer Success Analyst · Nola<br>
  Maio 2026
</p>
