# Simulador de Tráfego do Serviço de Usuários - Especificação Técnica

## Resumo Executivo

Este documento delineia a especificação técnica abrangente para implementar um comando Hyperf que simula tráfego realista de API para gerar métricas significativas para visualização no Grafana. O simulador criará interações diversas de usuários em todos os endpoints do serviço usando corrotinas para performance otimizada.

**Assinatura do Comando:**
```bash
php bin/hyperf.php user:populate --users=100 --min-request-count=10 --max-request-count=1000
```

## Análise do Contexto do Projeto

### Arquitetura Atual
- **Framework**: Hyperf com corrotinas Swoole
- **Autenticação**: Baseada em JWT com cache de tokens (Redis)
- **Sistema de Métricas**: Integração Prometheus com painéis Grafana
- **Estrutura de Domínio**: Arquitetura Limpa com padrão CQRS
- **Endpoints da API**: Interfaces REST (HTTP) e gRPC

### Infraestrutura de Métricas Existente
O projeto já possui coleta robusta de métricas:
- **Métricas de Auth**: Tentativas de login, geração de tokens, validação
- **Métricas de Usuário**: Contagens de usuários, rastreamento de atividade
- **Métricas de Performance**: Duração de requisições, performance do sistema
- **Métricas de Sistema**: Uso de memória, utilização de recursos

### Endpoints da API Disponíveis para Simulação

#### Endpoints de Autenticação (`/api/v1/auth`)
- `POST /login` - Autenticação de usuário
- `POST /logout` - Encerramento de sessão
- `POST /refresh` - Atualização de token
- `POST /me` - Informações do usuário atual

#### Gerenciamento de Usuários (`/api/v1/users`)
- `GET /` - Listar usuários (paginado)
- `GET /search` - Buscar usuários
- `GET /{id}` - Obter usuário por ID
- `GET /email/{email}` - Obter usuário por email
- `GET /username/{username}` - Obter usuário por nome de usuário
- `POST /` - Criar usuário
- `PUT /{id}` - Atualizar usuário
- `DELETE /{id}` - Excluir usuário
- `PUT /{id}/activate` - Ativar usuário
- `PUT /{id}/deactivate` - Desativar usuário
- `PUT /{id}/change-password` - Alterar senha
- `PUT /{id}/reset-password` - Resetar senha

#### Perfis de Usuário (`/api/v1/users/{userId}/profile`)
- `POST /` - Criar perfil
- `GET /` - Obter perfil
- `PUT /` - Atualizar perfil
- `DELETE /` - Excluir perfil

#### Gerenciamento de Roles (`/api/v1/roles`)
- `GET /` - Listar roles
- `POST /` - Criar role
- `GET /{id}` - Obter role
- `PUT /{id}` - Atualizar role
- `DELETE /{id}` - Excluir role

#### Gerenciamento de Permissões (`/api/v1/permissions`)
- `GET /` - Listar permissões
- `POST /` - Criar permissão
- `GET /{id}` - Obter permissão
- `PUT /{id}` - Atualizar permissão
- `DELETE /{id}` - Excluir permissão

#### Atribuições de Roles de Usuário (`/api/v1/users/{userId}/roles`)
- `GET /` - Listar roles do usuário
- `POST /{roleId}` - Atribuir role
- `DELETE /{roleId}` - Revogar role

## Arquitetura Técnica

### 1. Estrutura do Comando

```php
<?php

namespace App\Infrastructure\Command;

use Hyperf\Command\Annotation\Command;
use Hyperf\Command\Command as HyperfCommand;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Input\InputOption;
use Symfony\Component\Console\Output\OutputInterface;

#[Command]
class UserTrafficSimulatorCommand extends HyperfCommand
{
    protected ?string $name = 'user:populate';
    
    protected string $description = 'Simular tráfego realista de API para geração de métricas';
}
```

#### Parâmetros do Comando
- `--users` (obrigatório): Número de usuários virtuais concorrentes (padrão: 100)
- `--min-request-count` (obrigatório): Mínimo de requisições por usuário (padrão: 10)
- `--max-request-count` (obrigatório): Máximo de requisições por usuário (padrão: 1000)
- `--duration`: Limite de tempo opcional em segundos
- `--base-url`: URL base do serviço de destino (padrão: http://localhost:9501)
- `--delay-min`: Delay mínimo entre requisições em ms (padrão: 100)
- `--delay-max`: Delay máximo entre requisições em ms (padrão: 2000)
- `--scenario`: Cenário específico para executar (padrão: mixed)

### 2. Componentes Principais

#### A. Orquestrador de Tráfego
**Responsabilidade**: Coordenar todas as atividades de simulação
```php
class TrafficOrchestrator
{
    public function orchestrate(SimulationConfig $config): void;
    public function createUserPools(int $userCount): array;
    public function distributeWorkload(array $users, array $scenarios): void;
}
```

#### B. Motor de Comportamento do Usuário
**Responsabilidade**: Definir padrões realistas de interação do usuário
```php
class UserBehaviorEngine
{
    public function generateUserSession(VirtualUser $user): UserSession;
    public function selectNextAction(UserSession $session): ApiAction;
    public function calculateDelay(ApiAction $action): int;
}
```

#### C. Gerenciador de Cliente HTTP
**Responsabilidade**: Gerenciar conexões e requisições HTTP
```php
class HttpClientManager
{
    private Client $client;
    private TokenManager $tokenManager;
    
    public function executeRequest(ApiRequest $request): ApiResponse;
    public function handleAuthentication(VirtualUser $user): AuthResult;
}
```

#### D. Coletor de Métricas
**Responsabilidade**: Coleta e relatório de métricas em tempo real
```php
class TrafficMetricsCollector
{
    public function recordRequest(ApiRequest $request, ApiResponse $response): void;
    public function recordError(ApiRequest $request, Exception $error): void;
    public function generateReport(): TrafficReport;
}
```

### 3. Arquitetura de Corrotinas

#### Design do Pool de Corrotinas
```php
class CoroutinePool
{
    private int $maxConcurrency;
    private array $activeCoroutines = [];
    
    public function spawn(callable $task): int;
    public function manage(): void;
    public function waitAll(): void;
}
```

#### Gerenciamento de Sessões de Usuário
Cada usuário virtual roda em sua própria corrotina:
```php
go(function() use ($user, $scenarios) {
    $session = new UserSession($user);
    
    while ($session->shouldContinue()) {
        $action = $behaviorEngine->selectNextAction($session);
        $delay = $behaviorEngine->calculateDelay($action);
        
        Coroutine::sleep($delay / 1000);
        
        try {
            $response = $httpClient->executeRequest($action->toRequest());
            $session->recordSuccess($action, $response);
        } catch (Exception $e) {
            $session->recordError($action, $e);
        }
    }
});
```

### 4. Estratégias de Simulação de Usuário

#### A. Arquétipos de Usuário

**Jornada do Usuário Novo**
1. Registro de usuário
2. Criação de perfil
3. Exploração básica (listar usuários, buscar)
4. Atualizações de perfil

**Jornada do Usuário Ativo**
1. Login
2. Operações de gerenciamento de usuários
3. Consultas de roles/permissões
4. Gerenciamento de perfil
5. Logout

**Jornada do Usuário Administrativo**
1. Login
2. Operações CRUD de usuários
3. Gerenciamento de roles/permissões
4. Operações em lote
5. Consultas de monitoramento do sistema

**Jornada do Usuário Avançado**
1. Login
2. Operações complexas de busca
3. Operações em lote de usuários
4. Filtragem avançada
5. Múltiplas sessões

#### B. Padrões de Comportamento Realistas

**Distribuição de Requisições** (baseada no uso típico de API):
- Autenticação: 20%
- Consultas de usuário: 35%
- Mutações de usuário: 15%
- Operações de perfil: 15%
- Operações de role/permissão: 10%
- Operações administrativas: 5%

**Padrões de Temporização**:
- Delays humanizados (100ms - 2000ms)
- Padrões de rajada para operações de busca
- Delays mais longos para envio de formulários
- Clustering de requisições baseado em sessão

### 5. Simulação de Erros

#### Cenários de Erro Realistas
- Falhas de autenticação (credenciais inválidas)
- Falhas de autorização (permissões insuficientes)
- Erros de validação (dados inválidos)
- Erros de não encontrado (recursos inexistentes)
- Cenários de rate limiting
- Timeouts de rede
- Erros de servidor (respostas 5xx)

#### Distribuição de Erros (comportamento realista de API):
- Sucesso: 85%
- Erros de Cliente (4xx): 12%
- Erros de Servidor (5xx): 3%

### 6. Gerenciamento de Dados

#### Geração de Usuário Virtual
```php
class VirtualUserFactory
{
    public function generateUsers(int $count): array
    {
        return array_map(function($i) {
            return new VirtualUser([
                'username' => "sim_user_{$i}",
                'email' => "sim_user_{$i}@simulation.test",
                'password' => 'SimPass123!',
                'name' => "Usuário de Simulação {$i}",
                'profile' => $this->generateProfile(),
                'archetype' => $this->selectArchetype(),
                'behavior_config' => $this->generateBehaviorConfig()
            ]);
        }, range(1, $count));
    }
}
```

#### Estratégia de Limpeza de Dados
```php
class DataCleanupService
{
    public function cleanupSimulationData(): void
    {
        // Remover usuários criados pela simulação
        // Limpar tokens de teste do cache
        // Resetar métricas se necessário
        // Limpar dados temporários
    }
}
```

### 7. Gerenciamento de Configuração

#### Configuração de Simulação
```php
class SimulationConfig
{
    public int $userCount;
    public int $minRequestCount;
    public int $maxRequestCount;
    public int $durationSeconds;
    public string $baseUrl;
    public array $delayRange;
    public string $scenario;
    public bool $cleanupOnExit;
    public array $endpointWeights;
    public array $errorRates;
}
```

#### Configurações Específicas do Ambiente
- Desenvolvimento: Contagens menores de usuários, logging detalhado
- Teste: Carga média, métricas abrangentes
- Staging: Carga similar à produção, monitoramento completo

### 8. Monitoramento e Relatórios

#### Exibição de Progresso em Tempo Real
```bash
Progresso da Simulação de Tráfego:
┌─────────────────────────────────────────┐
│ Usuários: 100/100 Ativos | Reqs: 15.432│
│ Taxa Sucesso: 87.3% | Erros: 1.967     │
│ Resp Média: 245ms | Pico: 1.230ms      │
│ Duração: 05:23 | Restante: ~12:15      │
└─────────────────────────────────────────┘

Atividade Recente:
[12:34:15] User-045: POST /api/v1/users → 201 (123ms)
[12:34:15] User-021: GET /api/v1/users → 200 (89ms)
[12:34:16] User-087: POST /api/v1/auth/login → 401 (45ms)
```

#### Integração de Métricas
O simulador gerará métricas que se integram com coletores Prometheus existentes:
- Volume de requisições por endpoint
- Distribuições de tempo de resposta
- Taxas de erro por endpoint e tipo
- Taxas de sucesso/falha de autenticação
- Durações de sessão de usuário
- Utilização de recursos durante carga

### 9. Fases de Implementação

#### Fase 1: Infraestrutura Principal (Semana 1)
- [ ] Implementação da classe de comando
- [ ] Arquitetura básica de corrotinas
- [ ] Configuração de cliente HTTP
- [ ] Gerenciamento de configuração
- [ ] Simulação simples de usuário

#### Fase 2: Motor de Comportamento (Semana 2)
- [ ] Definições de arquétipos de usuário
- [ ] Padrões de comportamento realistas
- [ ] Algoritmos de temporização de requisições
- [ ] Gerenciamento de estado de sessão
- [ ] Lógica de simulação de erros

#### Fase 3: Recursos Avançados (Semana 3)
- [ ] Jornadas complexas de usuário
- [ ] Mecanismos de limpeza de dados
- [ ] Integração abrangente de métricas
- [ ] Otimização de performance
- [ ] Gerenciamento de memória

#### Fase 4: Prontidão para Produção (Semana 4)
- [ ] Testes extensivos
- [ ] Conclusão da documentação
- [ ] Benchmarking de performance
- [ ] Robustez do tratamento de erros
- [ ] Implantação em produção

## Diretrizes de Implementação

### 1. Gerenciamento de Memória
```php
// Prevenir vazamentos de memória em simulações de longa duração
class MemoryManager
{
    public function periodic_cleanup(): void
    {
        // Limpar caches de resposta
        // Resetar métricas acumuladas
        // Coletar lixo de objetos não utilizados
        // Monitorar uso de memória
    }
}
```

### 2. Desligamento Gracioso
```php
// Lidar com SIGTERM/SIGINT graciosamente
class ShutdownHandler
{
    public function handleShutdown(): void
    {
        $this->logger->info('Desligamento da simulação iniciado');
        $this->stopNewRequests();
        $this->waitForActiveRequests();
        $this->cleanupResources();
        $this->generateFinalReport();
    }
}
```

### 3. Recuperação de Erros
```php
// Implementar padrão circuit breaker
class CircuitBreaker
{
    public function executeWithBreaker(callable $operation): mixed
    {
        if ($this->isOpen()) {
            throw new CircuitBreakerOpenException();
        }
        
        try {
            $result = $operation();
            $this->recordSuccess();
            return $result;
        } catch (Exception $e) {
            $this->recordFailure();
            throw $e;
        }
    }
}
```

### 4. Considerações de Performance

#### Otimização de Corrotinas
- Usar pooling de conexões para clientes HTTP
- Implementar batching de requisições onde possível
- Monitorar contagem de corrotinas para prevenir exaustão de recursos
- Usar padrões async/await para operações de I/O

#### Gerenciamento de Recursos
- Limitar conexões concorrentes por alvo
- Implementar estratégias de backoff para requisições falhadas
- Monitorar recursos do sistema (CPU, memória, conexões)
- Usar inicialização lazy para objetos pesados

## Considerações de Segurança

### 1. Isolamento de Dados de Teste
- Usar banco de dados/ambiente dedicado para teste
- Garantir que dados de simulação sejam claramente marcados
- Implementar mecanismos de limpeza automática
- Prevenir que dados de simulação afetem produção

### 2. Segurança de Autenticação
- Usar credenciais de teste dedicadas
- Implementar gerenciamento de ciclo de vida de tokens
- Garantir limpeza adequada de sessões
- Monitorar abuso de autenticação

### 3. Conformidade com Rate Limiting
- Respeitar limites de taxa existentes
- Implementar estratégias de backoff
- Monitorar degradação de serviço
- Fornecer mecanismos de parada de emergência

## Métricas e KPIs

### Métricas Geradas
1. **Métricas de Requisição**
   - Total de requisições por endpoint
   - Taxas de sucesso/falha
   - Percentis de tempo de resposta
   - Throughput (requisições/segundo)

2. **Métricas de Usuário**
   - Usuários virtuais ativos
   - Durações de sessão
   - Taxas de conclusão de jornada de usuário
   - Distribuições de padrões de comportamento

3. **Métricas de Sistema**
   - Utilização de recursos
   - Taxas de erro por categoria
   - Indicadores de degradação de performance
   - Padrões de uso de memória

### Critérios de Sucesso
- Operação estável por períodos estendidos (24+ horas)
- Padrões de tráfego realistas correspondendo à produção
- Geração abrangente de métricas para Grafana
- Impacto mínimo na performance do sistema
- Gerenciamento confiável de limpeza e recursos

## Exemplos de Configuração

### Ambiente de Desenvolvimento
```bash
php bin/hyperf.php user:populate \
    --users=10 \
    --min-request-count=5 \
    --max-request-count=50 \
    --base-url=http://localhost:9501 \
    --scenario=development
```

### Teste de Carga
```bash
php bin/hyperf.php user:populate \
    --users=500 \
    --min-request-count=100 \
    --max-request-count=5000 \
    --duration=3600 \
    --scenario=load_test
```

### Geração de Métricas
```bash
php bin/hyperf.php user:populate \
    --users=100 \
    --min-request-count=50 \
    --max-request-count=1000 \
    --scenario=metrics_generation
```

## Conclusão

Esta especificação fornece um roadmap abrangente para implementar um simulador sofisticado de tráfego que gerará métricas significativas para visualização no Grafana mantendo estabilidade do sistema e padrões realistas de comportamento do usuário. A implementação aproveita as capacidades de corrotinas do Hyperf para performance otimizada e se integra perfeitamente com a infraestrutura de métricas existente.

O simulador servirá múltiplos propósitos:
- Gerar carga realista para testes de performance
- Criar métricas abrangentes para desenvolvimento de painéis
- Fornecer insights sobre comportamento do sistema sob várias condições de carga
- Suportar fluxos de trabalho de desenvolvimento e teste

A abordagem de implementação em fases garante progresso constante mantendo qualidade de código e estabilidade do sistema durante todo o processo de desenvolvimento.