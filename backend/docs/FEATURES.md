# Funcionalidades do StoryPilot

Este documento descreve as principais funcionalidades previstas para o **StoryPilot**.

O StoryPilot é um sistema para conectar contas profissionais do Instagram via Meta, criar publicações, visualizar
preview, publicar imediatamente ou agendar posts e stories para horários futuros.

## Objetivo do sistema

O objetivo do StoryPilot é permitir que usuários gerenciem publicações do Instagram de forma organizada, com suporte
para:

* Autenticação de usuários
* Conexão com conta profissional do Instagram
* Criação de publicações
* Escolha entre post e story
* Upload de mídia
* Preview da publicação
* Publicação imediata
* Agendamento em múltiplos dias e horários
* Execução automática das publicações agendadas
* Acompanhamento de status das publicações

## Fluxo geral

```txt
Usuário cria uma conta ou faz login
  ↓
Usuário conecta sua conta do Instagram via Meta
  ↓
Sistema identifica as contas profissionais disponíveis
  ↓
Usuário escolhe uma conta Instagram
  ↓
Usuário cria uma publicação
  ↓
Usuário escolhe se será post ou story
  ↓
Usuário adiciona mídia
  ↓
Se for post, usuário pode adicionar descrição
  ↓
Usuário visualiza o preview
  ↓
Usuário escolhe publicar agora ou agendar
  ↓
Sistema salva e processa a publicação
  ↓
Sistema publica usando a API oficial da Meta
```

## Autenticação do usuário

O sistema deve permitir que o usuário acesse sua conta na aplicação.

Funcionalidades previstas:

* Cadastro de usuário
* Login
* Logout
* Recuperação de usuário autenticado
* Proteção de rotas autenticadas
* Armazenamento do timezone do usuário

Dados principais do usuário:

```txt
nome
email
senha
timezone
```

O timezone será usado para garantir que os agendamentos sejam executados de acordo com o fuso horário correto do
usuário.

## Conexão com Instagram via Meta

Após autenticar na aplicação, o usuário poderá conectar sua conta do Instagram através da Meta.

O sistema deverá permitir:

* Iniciar conexão com a Meta
* Redirecionar o usuário para autorização da Meta
* Receber callback de autorização
* Trocar código de autorização por token de acesso
* Buscar páginas vinculadas ao usuário
* Identificar contas profissionais do Instagram conectadas
* Salvar a conta Instagram no sistema
* Desconectar uma conta Instagram

O sistema trabalhará com contas profissionais do Instagram, como contas Business ou Creator, compatíveis com a Instagram
Graph API.

## Gerenciamento de contas Instagram

Depois da conexão com a Meta, o usuário poderá visualizar as contas Instagram disponíveis.

Funcionalidades previstas:

* Listar contas Instagram conectadas
* Visualizar detalhes de uma conta conectada
* Ativar ou desativar uma conta
* Desconectar uma conta
* Exibir username, nome e imagem de perfil da conta

Uma conta Instagram conectada será usada como destino das publicações.

## Criação de publicação

O usuário poderá criar uma nova publicação escolhendo uma conta Instagram conectada.

Uma publicação deverá conter:

* Conta Instagram de destino
* Tipo da publicação
* Mídia
* Descrição, quando aplicável
* Modo de publicação
* Agendamentos, quando aplicável

Tipos de publicação:

```txt
post
story
```

Modos de publicação:

```txt
publicar agora
agendar
```

## Publicação do tipo post

Quando o usuário escolher o tipo **post**, o sistema deverá permitir:

* Selecionar imagem ou vídeo
* Escrever uma descrição
* Visualizar preview no formato de post
* Publicar imediatamente
* Agendar para um ou vários horários

A descrição deve ser opcional, pois um post pode existir sem legenda.

## Publicação do tipo story

Quando o usuário escolher o tipo **story**, o sistema deverá permitir:

* Selecionar imagem ou vídeo
* Visualizar preview no formato vertical de story
* Publicar imediatamente
* Agendar para um ou vários horários

Inicialmente, stories não terão campo de descrição.

## Upload de mídia

O sistema deverá permitir o envio de mídias para publicação.

Funcionalidades previstas:

* Upload de imagem
* Upload de vídeo
* Validação de formato
* Validação de tamanho
* Armazenamento do arquivo
* Retorno de URL para preview
* Associação da mídia com uma publicação

A mídia enviada será usada tanto para preview quanto para publicação via Meta.

## Preview da publicação

Antes de publicar ou agendar, o usuário deverá visualizar uma prévia do conteúdo.

O preview será responsabilidade do frontend, usando os dados fornecidos pelo backend.

Para post, o preview deve exibir:

* Mídia
* Nome ou username da conta
* Imagem de perfil da conta
* Descrição da publicação

Para story, o preview deve exibir:

* Mídia em formato vertical
* Nome ou username da conta
* Elementos visuais simulando um story

## Publicar agora

O usuário poderá escolher publicar imediatamente.

Fluxo esperado:

```txt
Usuário cria publicação
  ↓
Usuário escolhe publicar agora
  ↓
Sistema salva a publicação
  ↓
Sistema cria uma execução imediata
  ↓
Sistema dispara um job de publicação
  ↓
Sistema envia o conteúdo para a Meta
  ↓
Sistema atualiza o status da publicação
```

Mesmo no modo “publicar agora”, o backend deve tratar a execução de forma controlada, usando job, para manter o mesmo
padrão das publicações agendadas.

## Agendamento de publicação

O usuário poderá agendar uma publicação para um ou vários dias e horários.

O sistema deverá permitir:

* Escolher uma data
* Escolher um horário
* Adicionar múltiplos horários
* Adicionar horários em dias diferentes
* Remover horários antes de salvar
* Cancelar horários pendentes
* Visualizar status de cada horário

Exemplo de agendamentos possíveis:

```txt
20/06/2026 09:00
20/06/2026 18:00
21/06/2026 10:30
22/06/2026 21:00
```

Cada horário será tratado como uma execução individual.

## Timezone

O sistema deverá respeitar o timezone do usuário.

Quando o usuário escolher uma data e hora, o sistema deverá salvar:

```txt
horário local escolhido pelo usuário
horário convertido para UTC
timezone utilizado
```

Exemplo:

```txt
Horário local: 20/06/2026 18:00
Timezone: America/Manaus
Horário UTC: convertido pelo backend
```

O horário UTC será usado para execução pelo scheduler.

O horário local será usado para exibição e auditoria.

## Execução automática

As publicações agendadas serão processadas automaticamente pelo backend.

Fluxo esperado:

```txt
Scheduler executa periodicamente
  ↓
Sistema busca agendamentos pendentes
  ↓
Sistema verifica quais já chegaram no horário correto
  ↓
Sistema dispara jobs de publicação
  ↓
Job chama serviço de publicação da Meta
  ↓
Sistema atualiza status conforme sucesso ou falha
```

## Status das publicações

Uma publicação poderá ter os seguintes status:

```txt
draft
scheduled
publishing
published
failed
cancelled
```

Descrição dos status:

```txt
draft       publicação criada, mas ainda não agendada ou publicada
scheduled   publicação possui horários pendentes
publishing  publicação está em processo de envio
published   publicação foi publicada com sucesso
failed      publicação falhou
cancelled   publicação foi cancelada
```

## Status dos agendamentos

Cada horário de publicação poderá ter seu próprio status.

Status previstos:

```txt
pending
processing
published
failed
cancelled
```

Descrição dos status:

```txt
pending     aguardando horário de execução
processing  publicação em andamento
published   horário publicado com sucesso
failed      tentativa de publicação falhou
cancelled   horário cancelado
```

## Cancelamento

O usuário poderá cancelar publicações ou agendamentos.

Regras previstas:

* Uma publicação pendente pode ser cancelada
* Um agendamento pendente pode ser cancelado
* Uma publicação já publicada não pode ser cancelada
* Uma publicação em processamento não deve ser cancelada diretamente
* Cancelamentos devem atualizar status e manter histórico

## Tratamento de falhas

O sistema deverá lidar com falhas de publicação.

Exemplos de falhas:

* Token da Meta expirado
* Conta Instagram desconectada
* Mídia inválida
* Erro de permissão
* Erro na API da Meta
* Erro de rede
* Agendamento vencido
* Publicação duplicada

Quando ocorrer falha, o sistema deverá:

* Atualizar o status do agendamento para failed
* Registrar mensagem de erro
* Manter a publicação disponível para consulta
* Permitir análise posterior do erro

## Histórico

O usuário deverá conseguir acompanhar suas publicações.

Funcionalidades previstas:

* Listar publicações
* Filtrar por status
* Filtrar por tipo
* Filtrar por conta Instagram
* Visualizar detalhes de uma publicação
* Visualizar horários associados
* Visualizar erros de publicação
* Visualizar publicações futuras

## Dashboard

O sistema poderá possuir um dashboard com resumo das publicações.

Informações previstas:

* Total de publicações
* Total de posts
* Total de stories
* Publicações agendadas
* Publicações publicadas
* Publicações com falha
* Publicações canceladas
* Próximas publicações
* Contas Instagram conectadas

## Integração com Meta

O sistema usará a API oficial da Meta para conexão e publicação.

Integrações previstas:

* Meta Graph API
* Instagram Graph API
* Instagram API with Facebook Login
* Instagram Content Publishing API

Responsabilidades da integração:

* Autenticar usuário com Meta
* Obter token de acesso
* Buscar páginas disponíveis
* Buscar conta Instagram conectada
* Obter Instagram Account ID
* Criar container de mídia
* Publicar container de mídia
* Capturar erros da Meta

## Escopo inicial

O escopo inicial do sistema será:

```txt
Autenticação do usuário
Conexão com Meta
Listagem de contas Instagram conectadas
Criação de publicação
Upload de mídia
Preview no frontend
Publicar agora
Agendamento simples e múltiplo
Execução via scheduler e jobs
Publicação via Meta
Histórico de publicações
```

## Fora do escopo inicial

Inicialmente, o sistema não terá:

```txt
Analytics avançado
Relatórios financeiros
Calendário visual completo
Edição avançada de imagem
Edição avançada de vídeo
Aprovação em equipe
Multiusuário por organização
Planos e pagamentos
Comentários e mensagens do Instagram
```

Essas funcionalidades poderão ser avaliadas em versões futuras.

## Observação

Este documento descreve as funcionalidades previstas do sistema. Detalhes técnicos de implementação, arquitetura,
contratos de API e modelagem de banco devem ser documentados em arquivos separados dentro da pasta `docs/`.
