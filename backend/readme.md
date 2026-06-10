# StoryPilot Backend

Backend API do **StoryPilot**, desenvolvido com **Laravel 5.6** e **PHP 7.2**.

O backend é responsável por autenticação, gerenciamento de usuários, conexão com a Meta/Instagram, criação de
publicações, agendamento e execução de posts e stories.

## Tecnologias

* PHP 7.2
* Laravel 5.6
* Composer 2
* Docker
* MySQL
* API REST
* Laravel Queue
* Laravel Scheduler
* Meta Graph API
* Instagram Graph API

## Requisitos

Antes de começar, é necessário ter instalado:

* Docker
* Docker Compose
* Composer
* Git

## Instalação

Acesse a pasta do backend:

```bash
cd backend
```

Instale as dependências:

```bash
composer install
```

Crie o arquivo de ambiente:

```bash
cp .env.example .env
```

Gere a chave da aplicação:

```bash
php artisan key:generate
```

Execute as migrations:

```bash
php artisan migrate
```

## Executando o projeto

A partir da raiz do projeto, suba os containers:

```bash
docker compose up -d
```

A API ficará disponível em:

```txt
http://localhost:8000
```

## Estrutura básica

```txt
app/
  Actions/        Casos de uso da aplicação
  Exceptions/     Exceções de domínio e integração
  Http/           Controllers, middlewares e requests
  Jobs/           Processos executados em background
  Models/         Models Eloquent
  Policies/       Regras de autorização
  Providers/      Service providers e bindings
  Repositories/   Camada de acesso a dados
  Services/       Serviços de domínio e integrações externas
  Support/        Constantes e helpers do domínio
  Traits/         Comportamentos reutilizáveis
```

## Fluxo principal

```txt
Usuário autentica na aplicação
  ↓
Usuário conecta uma conta do Instagram via Meta
  ↓
Backend obtém e salva a conta Instagram conectada
  ↓
Usuário cria uma publicação do tipo post ou story
  ↓
Usuário escolhe publicar agora ou agendar
  ↓
Backend salva os dados da publicação
  ↓
Laravel Queue/Scheduler processa a publicação
  ↓
Backend publica o conteúdo usando a API oficial da Meta
```

## Documentação

Documentações mais detalhadas sobre funcionalidades, padrões de código, arquitetura, integrações, padrões de API e
fluxos internos devem ficar na pasta `docs/`.

Documentos disponíveis:

* [Funcionalidades do sistema](./docs/FEATURES.md)
* [Regras de código](./docs/CODE_GUIDELINES.md)

## Status

Projeto em desenvolvimento.
