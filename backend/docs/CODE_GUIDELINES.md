# Code Guidelines

Este documento define boas práticas de código para o backend do **StoryPilot**.

O objetivo é manter o projeto organizado, fácil de evoluir e compatível com uma aplicação Laravel legada, sem abrir mão
de princípios sólidos de orientação a objetos.

## Objetivo

O backend deve ser escrito pensando em manutenção, clareza e evolução.

Cada classe deve ter uma responsabilidade clara, depender o mínimo possível de implementações concretas e expor
comportamentos que representem o domínio da aplicação.

O foco principal é evitar código rígido, frágil, altamente acoplado e difícil de testar.

## Versões do projeto

O backend utiliza:

```txt
PHP 7.2
Laravel 5.6
```

Por isso, o código deve respeitar limitações da stack legada.

Não utilizar recursos de versões modernas do PHP, como:

```txt
typed properties
attributes
match
constructor property promotion
native enums
union types
readonly properties
```

Sempre escrever código compatível com PHP 7.2.

## Princípios gerais

O código deve seguir estes princípios:

```txt
Alta coesão
Baixo acoplamento
Injeção de dependência
Dependência de abstrações
Responsabilidade única
Clareza de intenção
Classes pequenas
Métodos pequenos
Regras de negócio centralizadas
Baixa repetição
```

## Coesão

Uma classe deve ter membros relacionados ao mesmo propósito.

Evite classes que misturam responsabilidades diferentes.

Ruim:

```txt
PublicationService
  cria publicação
  faz upload
  chama API da Meta
  calcula timezone
  envia email
  monta resposta HTTP
```

Melhor:

```txt
CreatePublicationAction
MediaUploadService
MetaInstagramPublishingService
TimezoneConverter
```

Cada classe deve representar uma responsabilidade clara.

## Acoplamento

O acoplamento deve ser controlado.

Classes não devem instanciar diretamente dependências importantes usando `new` quando essas dependências representam
serviços, integrações, repositories ou regras externas.

Evite:

```php
$publisher = new MetaInstagramPublishingService();
$publisher->publish($publication);
```

Prefira receber a dependência pelo construtor:

```php
class PublishPublicationAction
{
    private $publisher;

    public function __construct(InstagramPublisherInterface $publisher)
    {
        $this->publisher = $publisher;
    }

    public function execute($publication)
    {
        return $this->publisher->publish($publication);
    }
}
```

## Injeção de dependência

Serviços, repositories e integrações devem ser injetados.

Isso facilita:

```txt
testes
manutenção
troca de implementação
mock de dependências
redução de acoplamento
```

Use injeção de dependência principalmente em:

```txt
Actions
Services
Jobs
Controllers
Repositories
```

## Dependência de abstrações

Classes de alto nível não devem depender diretamente de implementações concretas de baixo nível.

Uma Action não deve depender diretamente de uma classe concreta da Meta se puder depender de um contrato.

Evite:

```php
class PublishPublicationAction
{
    private $metaPublisher;

    public function __construct(MetaInstagramPublishingService $metaPublisher)
    {
        $this->metaPublisher = $metaPublisher;
    }
}
```

Prefira:

```php
class PublishPublicationAction
{
    private $publisher;

    public function __construct(InstagramPublisherInterface $publisher)
    {
        $this->publisher = $publisher;
    }
}
```

A implementação concreta deve ser registrada em um Service Provider.

Exemplo:

```php
$this->app->bind(
    InstagramPublisherInterface::class,
    MetaInstagramPublishingService::class
);
```

## Direção correta das dependências

Classes de regra de negócio não devem depender de detalhes externos.

A regra correta é:

```txt
Camadas de alto nível dependem de contratos
Implementações concretas dependem desses contratos
```

Exemplo no StoryPilot:

```txt
PublishPublicationAction
  depende de InstagramPublisherInterface

MetaInstagramPublishingService
  implementa InstagramPublisherInterface
```

Isso permite trocar a implementação futuramente.

Exemplo:

```txt
MetaInstagramPublishingService
FakeInstagramPublishingService
```

## Controllers

Controllers devem ser finos.

Responsabilidades permitidas:

```txt
receber request
chamar Action
retornar response JSON
```

Controllers não devem conter:

```txt
regra de negócio
query complexa
chamada direta para API da Meta
processamento de arquivo pesado
cálculo de timezone
decisão de status
```

Exemplo correto:

```php
public function store(StorePublicationRequest $request, CreatePublicationAction $action)
{
    $publication = $action->execute($request->validated());

    return response()->json($publication, 201);
}
```

## Form Requests

Validações de entrada devem ficar em Form Requests.

Exemplo:

```txt
StorePublicationRequest
UpdatePublicationRequest
SchedulePublicationRequest
ConnectMetaAccountRequest
```

Requests devem validar formato, obrigatoriedade e tipos básicos.

Regras de negócio complexas devem ficar em Actions, Services ou objetos de domínio.

## Actions

Actions representam casos de uso.

Exemplos:

```txt
CreatePublicationAction
SchedulePublicationAction
PublishPublicationAction
CancelPublicationAction
ConnectMetaAccountAction
```

Uma Action deve coordenar o fluxo de negócio.

Ela pode usar:

```txt
Repositories
Services
Support classes
Value Objects
Exceptions
```

Uma Action não deve ser uma classe gigante. Se ela crescer demais, parte da lógica deve ser extraída para Services ou
classes auxiliares.

## Services

Services devem concentrar regras que não pertencem diretamente a Controller, Model ou Repository.

Use Services para:

```txt
integrações externas
processamento de mídia
conversão de timezone
publicação via Meta
OAuth da Meta
descoberta de contas Instagram
```

Exemplos:

```txt
MetaOAuthService
MetaGraphApiClient
MetaInstagramPublishingService
TimezoneConverter
MediaStorageService
```

## Repositories

Repositories devem centralizar acesso ao banco.

Use repositories para:

```txt
buscar registros
criar registros
atualizar registros
consultas específicas
queries reutilizáveis
```

Evite queries espalhadas por Controllers, Jobs e Services.

Exemplo:

```php
$pendingSchedules = $this->publicationSchedules->findPendingToPublish($now);
```

Em vez de espalhar:

```php
PublicationSchedule::where(...)->where(...)->get();
```

## Models

Models devem representar entidades persistidas.

Eles podem conter:

```txt
relacionamentos
fillable
casts
scopes simples
métodos pequenos de domínio
```

Evite transformar Models em classes enormes com toda a regra da aplicação.

Exemplo aceitável:

```php
public function isPublished()
{
    return $this->status === PublicationStatus::PUBLISHED;
}
```

Exemplo perigoso:

```php
public function publishToInstagram()
{
    // chama API da Meta
    // cria container
    // publica container
    // atualiza status
}
```

Publicação via Meta deve ficar em Service/Action/Job, não diretamente no Model.

## Jobs

Jobs devem executar tarefas em background.

Use Jobs para:

```txt
publicar conteúdo agendado
processar mídia
renovar token
executar chamadas externas demoradas
```

Jobs podem chamar Actions.

Exemplo:

```txt
PublishScheduledPublicationJob
  chama PublishPublicationAction
```

O Job não deve concentrar toda a regra de publicação.

## Exceptions

Use Exceptions específicas para erros de domínio e integração.

Exemplos:

```txt
MetaAuthenticationFailedException
MetaPublishingFailedException
InstagramAccountNotConnectedException
PublicationAlreadyPublishedException
PublicationScheduleExpiredException
```

Evite lançar `Exception` genérica para tudo.

Prefira erros que expliquem o problema real.

## Support classes

Use `Support/` para constantes, validadores pequenos e helpers específicos do domínio.

Exemplos:

```txt
PublicationStatus
PublicationType
PublishMode
PublicationScheduleStatus
TimezoneConverter
```

Como o projeto usa PHP 7.2, não existem enums nativos. Valores fixos devem ser representados por classes com constantes.

Exemplo:

```php
class PublicationType
{
    const POST = 'post';
    const STORY = 'story';

    public static function all()
    {
        return [
            self::POST,
            self::STORY,
        ];
    }
}
```

## Value Objects

Use Value Objects quando um valor tiver regra, validação ou comportamento próprio.

Exemplos possíveis:

```txt
Email
Timezone
MetaAccessToken
PublicationCaption
ScheduleDateTime
```

Um Value Object deve:

```txt
validar seu próprio estado
evitar valor inválido
ser pequeno
representar uma ideia clara do domínio
```

Exemplo:

```php
class Timezone
{
    private $value;

    public function __construct($value)
    {
        if (!in_array($value, timezone_identifiers_list())) {
            throw new InvalidArgumentException('Invalid timezone.');
        }

        $this->value = $value;
    }

    public function value()
    {
        return $this->value;
    }
}
```

Use Value Objects quando eles realmente reduzirem duplicação e melhorarem a clareza.

Não crie Value Object para tudo sem necessidade.

## Evitar Primitive Obsession

Evite espalhar strings soltas pelo sistema para representar conceitos importantes.

Ruim:

```php
$status = 'published';
$type = 'story';
$mode = 'scheduled';
```

Melhor:

```php
$status = PublicationStatus::PUBLISHED;
$type = PublicationType::STORY;
$mode = PublishMode::SCHEDULED;
```

Isso reduz erro de digitação e centraliza os valores aceitos.

## Tell, Don’t Ask

Evite pedir dados de um objeto para fazer o trabalho fora dele quando o próprio objeto poderia executar a ação.

Evite:

```php
if ($publication->getStatus() === PublicationStatus::DRAFT) {
    $publication->setStatus(PublicationStatus::SCHEDULED);
}
```

Prefira algo mais expressivo:

```php
$publication->markAsScheduled();
```

A regra fica mais próxima do objeto que representa o conceito.

## Law of Demeter

Evite cadeias longas de chamadas.

Ruim:

```php
$user->getMetaConnection()->getInstagramAccount()->getPage()->getId();
```

Prefira métodos mais diretos ou serviços que encapsulem a navegação:

```php
$instagramAccount->metaPageId();
```

ou:

```php
$metaAccountService->getConnectedPageId($instagramAccount);
```

Isso reduz acoplamento entre estruturas internas.

## Métodos pequenos

Métodos devem ser curtos e objetivos.

Evite métodos com muitas etapas misturadas.

Se um método faz muitas coisas, extraia partes para métodos privados, Services ou Actions.

Sinais de alerta:

```txt
muitos ifs
muitos níveis de indentação
muitas variáveis temporárias
muitas responsabilidades no mesmo método
```

## Evitar else desnecessário

Sempre que possível, use retornos antecipados.

Evite:

```php
if (!$publication->canBeCancelled()) {
    throw new PublicationCannotBeCancelledException();
} else {
    $publication->cancel();
}
```

Prefira:

```php
if (!$publication->canBeCancelled()) {
    throw new PublicationCannotBeCancelledException();
}

$publication->cancel();
```

Isso reduz indentação e melhora legibilidade.

## Nomes claros

Não abreviar nomes importantes.

Evite:

```txt
PubSvc
MetaCli
SchJob
AccRepo
```

Prefira:

```txt
PublicationService
MetaGraphApiClient
SchedulePublicationJob
InstagramAccountRepository
```

Nomes devem explicar intenção.

## Classes pequenas

Classes devem ser pequenas e coesas.

Se uma classe começa a crescer demais, verifique se ela está acumulando responsabilidades.

Sinais de alerta:

```txt
classe com muitos métodos públicos
classe com muitas dependências no construtor
classe com muitos ifs de tipo/status
classe que conhece detalhes de muitas partes do sistema
```

## Comentários

Comentários devem explicar decisões, não repetir o que o código já mostra.

Evite:

```php
// cria publicação
$publication = Publication::create($data);
```

Prefira comentários apenas quando houver contexto relevante:

```php
// Meta requires the media container to be created before publishing.
```

## Padrão para integrações externas

Chamadas para serviços externos devem ficar isoladas.

No StoryPilot, chamadas para a Meta devem passar por:

```txt
Services/Meta
```

Nenhuma Action, Controller ou Job deve montar manualmente URLs da Graph API se isso puder ficar centralizado no
`MetaGraphApiClient`.

## Tratamento de erros externos

Erros da Meta devem ser normalizados.

O sistema não deve depender diretamente do formato bruto de erro da Meta em todas as camadas.

A camada de integração deve converter erros externos em Exceptions internas.

Exemplo:

```txt
Meta API error
  ↓
MetaGraphApiClient
  ↓
MetaPublishingFailedException
  ↓
Action/Job trata o erro
```

## Timezone

Horários de agendamento devem respeitar o timezone do usuário.

O sistema deve armazenar:

```txt
scheduled_at_local
scheduled_at_utc
timezone
```

Regra:

```txt
horário local é usado para exibição
horário UTC é usado para execução
timezone é usado para auditoria e conversão
```

Conversões de timezone não devem ficar espalhadas pelo sistema. Devem ser centralizadas em uma classe específica.

## Status

Status devem ser centralizados em classes de constantes.

Exemplos:

```txt
PublicationStatus
PublicationScheduleStatus
MetaConnectionStatus
```

Não usar strings soltas espalhadas pelo código.

## Respostas da API

Controllers devem retornar respostas JSON previsíveis.

Sempre que possível, manter padrão consistente para:

```txt
sucesso
erro de validação
erro de domínio
erro de autenticação
erro de integração externa
```

Detalhes de contrato de API devem ficar em documentação específica.

## Compatibilidade com Laravel legado

Por ser Laravel 5.6, evitar soluções que dependam de recursos modernos do framework.

Antes de usar alguma funcionalidade, verificar se ela existe na versão 5.6.

Preferir soluções simples, explícitas e compatíveis com a versão do projeto.

## O que evitar

Evitar:

```txt
Controllers grandes
Models gigantes
Services que fazem tudo
queries espalhadas
strings mágicas
new de dependências importantes dentro de classes
regras de negócio no Controller
lógica de integração externa fora de Services
métodos longos
classes com muitas responsabilidades
cadeias longas de chamada
uso excessivo de Traits
abstrações desnecessárias
```

## O que buscar

Buscar:

```txt
clareza
baixo acoplamento
alta coesão
dependência de contratos
classes pequenas
métodos pequenos
nomes expressivos
regras centralizadas
testabilidade
separação de responsabilidades
compatibilidade com PHP 7.2
```

## Regra prática

Antes de criar ou alterar uma classe, perguntar:

```txt
Essa classe tem uma responsabilidade clara?
Ela depende de abstração ou implementação concreta?
Ela está fazendo mais do que deveria?
Essa lógica deveria estar em uma Action, Service, Repository ou Model?
Esse valor deveria ser uma constante ou Value Object?
Essa alteração vai dificultar testes?
Essa implementação está compatível com Laravel 5.6 e PHP 7.2?
```

## Referência conceitual

Este documento se baseia em princípios de orientação a objetos aplicados ao contexto do StoryPilot:

```txt
abstração
encapsulamento
coesão
acoplamento
injeção de dependência
inversão de dependência
value objects
primitive obsession
Tell Don’t Ask
Law of Demeter
Object Calisthenics
```

Esses princípios não devem ser aplicados de forma cega. A prioridade é manter o código claro, sustentável e adequado ao
tamanho real do projeto.
