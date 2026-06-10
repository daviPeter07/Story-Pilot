# StoryPilot

**StoryPilot** é um sistema de agendamento, gerenciamento e simulação de publicação de stories, desenvolvido como projeto prático para estudo de uma stack legada com **Laravel 5.6** e **Vue 2.7**.

O objetivo do projeto é simular um ambiente próximo ao de sistemas reais e legados, trabalhando com backend em Laravel como API REST, frontend em Vue como SPA, Docker, filas, scheduler, arquitetura em camadas e integração externa simulada.

## Sobre o projeto

A proposta do StoryPilot é permitir que usuários cadastrem contas, criem stories, agendem publicações e acompanhem o status de cada postagem.

O projeto não tem como foco criar um bot de Instagram. A ideia é construir uma base profissional para estudar conceitos como:

* API REST com Laravel 5.6
* Vue 2.7 com TypeScript
* Organização de monorepo
* Arquitetura backend em camadas
* Jobs e filas
* Scheduler do Laravel
* Validações com Form Requests
* Repositories, Services e Actions
* Simulação de integração com API externa
* Organização de código em sistema legado

## Stack utilizada

### Backend

* PHP 7.2
* Laravel 5.6
* Composer 2
* Docker
* Apache
* API REST

### Frontend

* Vue 2.7.16
* TypeScript
* Vue Router 3
* Axios
* Tailwind CSS
* Vue CLI
* ESLint
* Prettier
* PNPM

## Arquitetura do backend

O backend foi pensado para não ficar preso em um MVC puro. A ideia é praticar uma organização mais próxima de projetos reais, separando responsabilidades em camadas.

### Responsabilidade de cada camada

| Camada       | Responsabilidade                           |
| ------------ | ------------------------------------------ |
| Controller   | Receber a requisição e devolver a resposta |
| Form Request | Validar os dados de entrada                |
| Action       | Executar um caso de uso da aplicação       |
| Repository   | Buscar e persistir dados                   |
| Model        | Representar tabelas e relacionamentos      |
| Service      | Concentrar regras de integração externa    |
| Job          | Executar tarefas em background             |
| Policy       | Controlar autorização                      |
| Exception    | Representar erros de domínio ou integração |
| Trait        | Reaproveitar comportamentos pequenos       |

## Funcionalidades planejadas

* Cadastro de contas de Instagram simuladas
* Cadastro de stories
* Upload ou referência de mídia
* Agendamento de publicação
* Listagem de stories agendados
* Cancelamento de agendamento
* Publicação simulada
* Status da publicação
* Histórico de publicações
* Tratamento de falhas na publicação
* Execução via fila
* Execução via scheduler

## Status dos stories

Os stories podem possuir os seguintes status:

```ts
pending
published
failed
cancelled
```

## Requisitos

Antes de rodar o projeto, é necessário ter instalado:

* Docker
* Docker Compose
* Node.js
* PNPM
* Git

### Atenção com a versão do PHP

O backend usa **Laravel 5.6** e exige **PHP 7.1.3+**.

Por isso, para rodar comandos como `php artisan` e `composer install`, você precisa usar uma destas opções:

* subir o ambiente Docker do projeto, que já usa uma versão compatível do PHP
* ou ter **PHP 7.1.3+** instalado localmente na sua máquina

Se você tentar executar esses comandos com uma versão mais nova e incompatível do PHP da máquina, o projeto pode apresentar erros de bootstrap, classes ou rotas.

## Como rodar o projeto

### 1. Clone o repositório

```bash
git clone https://github.com/daviPeter07/Story-Pilot.git
cd Story-Pilot
```

### 2. Suba o ambiente Docker

```bash
docker compose up -d
```

### 3. Acesse o backend

```bash
cd backend
```

### 4. Instale as dependências do Laravel

Se for rodar esse passo fora do Docker, confirme antes que sua máquina está com **PHP 7.1.3+**.

```bash
composer install
```

### 5. Configure o ambiente

Crie o arquivo `.env` com base no `.env.example`:

```bash
cp .env.example .env
```

Depois gere a chave da aplicação:

```bash
php artisan key:generate
```

### 6. Rode as migrations

```bash
php artisan migrate
```

### 7. Acesse o frontend

```bash
cd ../frontend
```

### 8. Instale as dependências

```bash
pnpm install
```

### 9. Rode o frontend

```bash
pnpm run serve
```

## Backend

O backend é uma API REST construída com Laravel 5.6.

Exemplo de rotas planejadas:

```txt
GET /api/story-posts
POST /api/story-posts
GET /api/story-posts/{id}
PUT /api/story-posts/{id}
DELETE /api/story-posts/{id}
POST /api/story-posts/{id}/publish
POST /api/story-posts/{id}/cancel
```

## Frontend

O frontend é uma SPA em Vue 2.7 com TypeScript.

A ideia é manter o frontend separado do Laravel, consumindo a API através do Axios.

## Integração externa

Inicialmente, o projeto utiliza uma integração falsa para simular a publicação dos stories.

```txt
FakeInstagramPublisher
```

Futuramente, a estrutura permite trocar essa implementação por uma integração real.

```txt
GraphApiInstagramPublisher
```

Essa separação é feita através de contrato:

```txt
InstagramPublisherInterface
```

## Objetivo de estudo

Este projeto foi criado para treinar uma stack próxima de sistemas legados encontrados no mercado, principalmente com:

* Laravel 5.6
* PHP 7.2
* Vue 2.7
* TypeScript em Vue legado
* Docker com versões antigas
* Organização de código em camadas
* Boas práticas em projetos monorepo
