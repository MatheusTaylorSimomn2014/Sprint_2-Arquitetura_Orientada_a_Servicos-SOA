# Sprint 2 - Ford VIN Share: Impulsionando a Retenção no Pós-Venda

---

Uma solução integrada para aumentar o **VIN Share** na rede de concessionárias Ford da América do Sul, combinando **análise de dados, modelagem preditiva e otimização da jornada do cliente** em uma arquitetura orientada a serviços (SOA) robusta.

---

# Nomes e RM´s dos Integrantes do  Grupo
- Matheus Taylor, RM556211
- Henrique Maldonado, RM557270
- Yuri Silveira, RM557475
- Igor Soos, RM556010

---

## Índice

- [Visão Geral](#-visão-geral)
- [Arquitetura do Sistema](#-arquitetura-do-projeto)
- [Tecnologias Utilizadas](#-tecnologias-utilizadas)
- [Pré-requisitos](#-pré-requisitos)
- [Configuração e Execução](#-configuração-e-execução)
- [Principais Endpoints](#-principais-endpoints)
- [Estrutura do Projeto](#-estrutura-do-projeto)
- [Scripts de migração disponíveis](#-scripts-de-migração-disponíveis)
- [Padrões e Boas Práticas](#-padrões-e-boas-práticas)
- [Contribuição](#-contribuição)
- [Licença](#-licença)

---

## Visão Geral

O desafio central é reter clientes no serviço de pós-venda da Ford, monitorando e impulsionando o indicador-chave **VIN Share** — a porcentagem de veículos Ford que utilizam a rede oficial para manutenções.

### Funcionalidades Entregues

- **Análise e Visualização de Dados**  
  Dashboards interativos com métricas de retenção por idade do veículo, tipo de serviço, detecção de anomalias e análise de coorte.

- **Geração de Leads e Modelagem Preditiva**  
  Algoritmos híbridos (LSTM + XGBoost) que identificam veículos com alta probabilidade de precisar de serviço ou em risco de abandonar a rede, gerando leads proativos para as concessionárias.

- **Otimização da Jornada do Cliente**  
  Visão 360° do cliente e do veículo, integrando dados de CRM, telemetria (FordPass Connect/Smartcar) e histórico de manutenção para comunicações personalizadas.

---

## Arquitetura do Sistema

<img width="1408" height="768" alt="Arquitetura_Do_Projeto_Imagem" src="https://github.com/user-attachments/assets/c33939b1-aa4c-4fe9-bafd-99e893e0549c" />

O sistema segue os princípios de **Arquitetura Orientada a Serviços (SOA)**, com serviços independentes, reutilizáveis e comunicação via **APIs RESTful**.

---

## Tecnologias Utilizadas

| Categoria | Tecnologia | Versão |
|-----------|-----------|--------|
| Backend Framework | Spring Boot | 3.2+ |
| Linguagem | Java | 17 |
| Build | Maven | 3.9+ |
| Banco Relacional | PostgreSQL | 15+ |
| Cache Distribuído | Redis | 7.x |
| Banco NoSQL | MongoDB | 7.x |
| Migrations | Flyway | 9.x |
| Documentação API | SpringDoc OpenAPI | 3.0 |
| Resiliência | Resilience4j | 2.x |
| Service Registry | Netflix Eureka | 4.x |

---

## Pré-requisitos

- **JDK 17** ou superior
- **Maven 3.9+**
- **Docker** e **Docker Compose** (recomendado para bancos de dados)
- **PostgreSQL 15**, **Redis 7** e **MongoDB 7** (se executar localmente sem Docker)

---

## Configuração e Execução

### 1. Clonar o Repositório

```bash
git clone https://github.com/ford/vin-share-service.git
cd vin-share-service
```

### 2. Subir os Bancos de Dados com Docker

```bash
docker-compose up -d
```
O arquivo `docker-compose.yml` incluído provisiona PostgreSQL, Redis e MongoDB.

### 3. Configurar Variáveis de Ambiente
Crie um arquivo `.env `na raiz ou configure as seguintes variáveis:
```properties
DB_USERNAME=ford_app
DB_PASSWORD=ford_pass
REDIS_HOST=localhost
REDIS_PORT=6379
MONGO_HOST=localhost
MONGO_USER=ford_app
MONGO_PASSWORD=ford_pass
```

### 4. Executar as Migrações (Flyway)
As migrações são executadas automaticamente na inicialização da aplicação. Para executar manualmente:

```bash
mvn flyway:migrate
```

### 5. Compilar e Executar a Aplicação
```bash
mvn clean install
mvn spring-boot:run
```
A aplicação estará disponível em: `http://localhost:8080`

Documentação da API
A documentação interativa da API (Swagger UI) está disponível em:
```text
http://localhost:8080/swagger-ui/index.html
```

Recursos disponíveis no Swagger:
- Testar todos os endpoints diretamente do navegador
- Visualizar modelos de requisição/resposta
- Autenticação via JWT (se configurada)

---

## Principais Endpoints

### Analytics Service
| Método | Endpoint | Descrição |
|-----------|-----------|--------|
| `GET` | `/api/v1/analytics/vin-share/{dealerId}`| Retorna o VIN Share da concessionária |
| `GET` | `/api/v1/analytics/dealers/{dealerId}/retention-cohort` | Análise de coorte de retenção por idade |
| `POST` | `/api/v1/analytics/anomalies/detect` | Detecta anomalias no comportamento de serviço |

### Predictive Service
| Método | Endpoint | Descrição |
|-----------|-----------|--------|
| `GET` | `/api/v1/predictive/vehicles/{vin}/service-probability`| Probabilidade de necessidade de serviço |
| `GET` | `/api/v1/predictive/vehicles/{vin}/churn-risk` | Score de risco de churn (0-100) |


### Lead Service
| Método | Endpoint | Descrição |
|-----------|-----------|--------|
| `POST` | `/api/v1/leads/generate`| Gera leads proativos para uma concessionária |
| `GET` | `/api/v1/leads/dealers/{dealerId}/prioritized` | Lista leads priorizados |

### Customer 360 Service
| Método | Endpoint | Descrição |
|-----------|-----------|--------|
| `GET` | `/api/v1/customers/{customerId}/360`| Visão 360° do cliente e seus veículos |


### Vehicle Service
| Método | Endpoint | Descrição |
|-----------|-----------|--------|
| `GET` | `/api/v1/vehicles/{vin}`| Detalhes do veículo |
| `PATCH` | `/api/v1/vehicles/{vin}/maintenance-record` | Atualiza registro de manutenção |

---

## Estrutura do Projeto
```text
src/
├── main/
│   ├── java/
│   │   └── com/
│   │       └── ford/
│   │           └── vin/
│   │               └── share/
│   │                   ├── controller/                 # Endpoints REST (sem lógica de negócio)
│   │                   │   ├── AnalyticsController.java
│   │                   │   ├── PredictiveController.java
│   │                   │   ├── LeadController.java
│   │                   │   └── Customer360Controller.java
│   │                   ├── service/                    # Interfaces de serviço (contratos)
│   │                   │   ├── AnalyticsService.java
│   │                   │   ├── PredictiveService.java
│   │                   │   ├── LeadService.java
│   │                   │   └── impl/                   # Implementações concretas
│   │                   │       ├── AnalyticsServiceImpl.java
│   │                   │       ├── PredictiveServiceImpl.java   # Lógica LSTM + XGBoost
│   │                   │       └── LeadServiceImpl.java
│   │                   ├── repository/                 # Spring Data JPA
│   │                   │   ├── VehicleRepository.java
│   │                   │   ├── ServiceRecordRepository.java
│   │                   │   ├── CustomerRepository.java
│   │                   │   └── DealerRepository.java
│   │                   ├── entity/                     # Entidades JPA (ORM)
│   │                   │   ├── Vehicle.java
│   │                   │   ├── ServiceRecord.java
│   │                   │   ├── Customer.java
│   │                   │   └── Dealer.java
│   │                   ├── dto/                        # Objetos de transferência de dados
│   │                   │   ├── request/                # DTOs de entrada com validação
│   │                   │   │   ├── LeadGenerationRequest.java
│   │                   │   │   └── AnomalyDetectionRequest.java
│   │                   │   └── response/               # DTOs de saída + wrapper
│   │                   │       ├── VinShareResponse.java
│   │                   │       ├── ServiceProbabilityResponse.java
│   │                   │       └── ApiResponse.java
│   │                   ├── config/                     # Configurações (Swagger, Security, Cache)
│   │                   │   ├── OpenApiConfig.java
│   │                   │   ├── SecurityConfig.java
│   │                   │   └── CacheConfig.java
│   │                   ├── exception/                  # Tratamento global de exceções
│   │                   │   ├── GlobalExceptionHandler.java
│   │                   │   ├── BusinessException.java
│   │                   │   └── ResourceNotFoundException.java
│   │                   ├── client/                     # Clientes para APIs externas
│   │                   │   └── FordTelemetryClient.java
│   │                   └── util/                       # Utilitários
│   │                       ├── VinValidator.java
│   │                       └── DateUtils.java
│   └── resources/
│       ├── application.yml                             # Configurações principais
│       ├── db/
│       │   └── migration/                              # Scripts Flyway
│       └── static/                                     # Recursos estáticos (Swagger UI custom)
└── test/                                               # Testes unitários e de integração
```

---

## Modelagem de Dados e Migrações
O banco de dados transacional (PostgreSQL) utiliza um esquema estrela para facilitar análises:
- Tabela Fato: `fato_servico` – registros de serviços realizados
- Dimensões: `dim_veiculo`, `dim_concessionaria`, `dim_tempo`, `dim_tipo_servico`, `dim_cliente`
As migrações são gerenciadas pelo Flyway, garantindo versionamento e reprodutibilidade do schema.

--- 

## Scripts de migração disponíveis
```text
src/main/resources/db/migration/
├── V1__create_vehicle_table.sql
├── V2__create_customer_table.sql
├── V3__create_dealer_table.sql
├── V4__create_service_record_table.sql
├── V5__create_fato_servico_table.sql
└── ...
```

---

## Padrões e Boas Práticas
- RESTful: Nomeação de recursos no plural, versionamento via URI, códigos HTTP adequados
- DTO Pattern: Separação clara entre entidades JPA e objetos de transferência
- Tratamento Global de Exceções: `@RestControllerAdvice` com respostas padronizadas (`ApiResponse<T>`)
- Validação: Bean Validation (`@Valid`, `@NotNull`, `@Min`, etc.)
- Cache: Redis com anotações `@Cacheable` para métricas de VIN Share e dashboards
- Circuit Breaker: Resilience4j para chamadas às APIs externas da Ford (telemetria)
- Documentação Viva: OpenAPI 3.0 (Swagger) gerada automaticamente

---

## Contribuição
Contribuições são bem-vindas! Siga o fluxo padrão:

1. Fork o repositório
2. Crie uma branch: `feature/sua-funcionalidade`
3. Faça commit das alterações seguindo Conventional Commits ( https://www.conventionalcommits.org/ ) Commits
4. Envie um Pull Request detalhando as mudanças

### Padrão de Código
- Siga as convenções do Google Java Style Guide ( https://google.github.io/styleguide/javaguide.html )
- Utilize Lombok para reduzir boilerplate (`@Data`, `@Builder`, `@RequiredArgsConstructor`)
- Mantenha cobertura de testes acima de 80%

---

## Licença
Este projeto está licenciado sob a Apache License 2.0. Consulte o arquivo LICENSE para mais detalhes.

---

Desenvolvido por Alunos da FIAP em colaboração com a Ford Motor Company - Equipe de Inovação em Pós-Venda América do Sul

📧 apisupport@ford.com | 🌐 https://developer.ford.com
