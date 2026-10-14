<div align="center">

# 🏙️ Cidade Viva

**Viu um problema na rua? Cidade Viva.**

Plataforma open source para reportar, acompanhar e cobrar a resolução de problemas urbanos, com deduplicação geoespacial, modo offline e painel público de transparência.

![Status](https://img.shields.io/badge/status-em%20desenvolvimento-yellow)
![.NET](https://img.shields.io/badge/.NET-9-512BD4)
![React](https://img.shields.io/badge/React-18-61DAFB)
![React Native](https://img.shields.io/badge/React%20Native-Expo-000020)
![PostGIS](https://img.shields.io/badge/PostgreSQL-PostGIS-336791)
![License](https://img.shields.io/badge/license-AGPL--3.0-blue)

</div>

---

## O problema

Buracos nas ruas, postes apagados, lixo acumulado e bueiros abertos. As reclamações se perdem em canais diferentes, ninguém enxerga os padrões e o cidadão muitas vezes não sabe se o problema foi resolvido.

## A solução

O cidadão registra uma ocorrência em segundos, utilizando uma foto e a localização GPS.

O **Cidade Viva**:

- **Agrupa reportes semelhantes**, evitando ocorrências duplicadas.
- **Encaminha ao órgão responsável**, considerando sua área de cobertura.
- **Registra cada mudança de status**, mantendo um histórico de acompanhamento.
- **Funciona offline**, permitindo registrar problemas mesmo sem conexão com a internet.
- **Promove a transparência pública**, apresentando indicadores sobre o tempo de resolução por bairro e órgão responsável.

---

## Funcionalidades

| Área | Recursos |
|---|---|
| 📱 **Aplicativo mobile** | Registro com foto e GPS, mapa, confirmação de problemas de terceiros, notificações push e modo offline. |
| 🗺️ **Mapa interativo** | Filtros, agrupamento de marcadores por zoom, mapa de calor e busca por endereço. |
| 🧠 **Inteligência geoespacial** | Deduplicação em um raio de 30 metros com PostGIS, pontuação de prioridade e encaminhamento por polígonos geográficos. |
| 🖥️ **Painel de gestão** | Fila priorizada, atualização de status, atribuição de responsáveis, exportação CSV/PDF, webhooks e e-mail. |
| 🛡️ **Moderação** | Denúncias, reputação de usuários, detecção de imagens reutilizadas e limitação de requisições. |
| 🔎 **Transparência pública** | Estatísticas, rankings e API de dados abertos. |
| 🔐 **Privacidade** | Remoção de metadados EXIF, desfoque de rostos e placas e exclusão de contas, considerando a LGPD. |

A especificação completa, incluindo prioridades e regras de negócio, está em [`docs/SPEC.md`](docs/SPEC.md).

---

## Arquitetura

O Cidade Viva utiliza uma arquitetura de **monólito modular**: um único deploy, com módulos organizados por responsabilidades bem definidas.

```text
┌─────────────────┐   ┌─────────────────┐
│  App RN (Expo)  │   │  Painel React   │
└────────┬────────┘   └────────┬────────┘
         │ HTTPS / SignalR     │
         └──────────┬──────────┘
            ┌───────▼────────┐
            │ API ASP.NET    │── Hangfire (jobs)
            │    (.NET 9)    │── SignalR (tempo real)
            └───┬──────┬─────┘
                │      │
                │      └──────────► MinIO / S3 (fotos)
                ├─────────────────► Redis (cache, rate limit)
                ▼
       PostgreSQL + PostGIS
```

As decisões de arquitetura estão registradas em [`docs/adr`](docs/adr).

---

## Stack tecnológica

| Camada | Tecnologias |
|---|---|
| **Mobile** | React Native (Expo), TypeScript, TanStack Query, MapLibre, SQLite |
| **Web** | React, Vite, TypeScript, MapLibre GL, Recharts, Tailwind CSS |
| **API** | .NET 9, ASP.NET Core, EF Core + NetTopologySuite, MediatR, FluentValidation |
| **Dados** | PostgreSQL 16 + PostGIS, Redis |
| **Jobs e tempo real** | Hangfire, SignalR |
| **Armazenamento** | MinIO (desenvolvimento) / S3 compatível (produção) |
| **Observabilidade** | OpenTelemetry, Serilog |
| **Testes** | xUnit, Testcontainers, Playwright, Jest |
| **CI/CD** | GitHub Actions, Docker |

---

## Como executar

### Pré-requisitos

- [Docker](https://www.docker.com/) e Docker Compose.
- [.NET SDK 9](https://dotnet.microsoft.com/download).
- [Node.js 20+](https://nodejs.org/) e npm.
- Para o aplicativo mobile, Expo Go no celular ou um emulador Android/iOS.

### 1. Clonar o repositório

Substitua `SEU_USUARIO` pelo nome de usuário ou organização responsável pelo repositório no GitHub.

```bash
git clone https://github.com/SEU_USUARIO/cidade-viva.git
cd cidade-viva
```

### 2. Subir a infraestrutura

```bash
docker compose up -d
```

O ambiente deverá disponibilizar os serviços configurados no Docker Compose, incluindo PostgreSQL com PostGIS, Redis e MinIO.

### 3. Configurar as variáveis de ambiente

```bash
cp .env.example .env
cp apps/web/.env.example apps/web/.env
cp apps/mobile/.env.example apps/mobile/.env
```

Ajuste as configurações conforme o ambiente local.

> Nunca versione arquivos `.env` com senhas, tokens ou outros segredos reais.

### 4. Gerar dados de exemplo

```bash
dotnet run --project tools/seed
```

### 5. Executar a API

```bash
dotnet run --project src/CidadeViva.Api
```

Endereços previstos, dependendo da configuração da aplicação:

- **API:** `http://localhost:5000`
- **Swagger:** `http://localhost:5000/swagger`

### 6. Executar o painel web

```bash
cd apps/web
npm install
npm run dev
```

### 7. Executar o aplicativo mobile

```bash
cd apps/mobile
npm install
npx expo start
```

---

## Estrutura do repositório

```text
cidade-viva/
├── apps/
│   ├── mobile/                       # Aplicativo React Native com Expo
│   └── web/                          # Painel de gestão e página pública
│
├── src/
│   ├── CidadeViva.Api/               # API ASP.NET Core
│   ├── CidadeViva.Modules.Identity/  # Identidade e autenticação
│   ├── CidadeViva.Modules.Reports/   # Registro de ocorrências
│   ├── CidadeViva.Modules.Organizations/ # Órgãos e áreas de cobertura
│   ├── CidadeViva.Modules.Moderation/ # Moderação e denúncias
│   ├── CidadeViva.Modules.Notifications/ # Notificações
│   ├── CidadeViva.Modules.Analytics/  # Indicadores e transparência
│   └── CidadeViva.SharedKernel/      # Componentes compartilhados
│
├── tests/
│   ├── CidadeViva.UnitTests/         # Testes unitários
│   └── CidadeViva.IntegrationTests/  # Testes de integração
│
├── tools/
│   └── seed/                         # Gerador de dados de exemplo
│
├── docs/
│   ├── SPEC.md                       # Especificação funcional
│   └── adr/                          # Decisões de arquitetura
│
├── docker-compose.yml
├── .env.example
├── CONTRIBUTING.md
├── CODE_OF_CONDUCT.md
├── LICENSE
└── README.md
```

---

## Testes

### Back-end

```bash
dotnet test
```

### Painel web

```bash
cd apps/web
npx playwright test
```

### Aplicativo mobile

```bash
cd apps/mobile
npm test
```

---

## Roadmap

### Fase 0: Fundação
- [ ] Estrutura inicial do repositório.
- [ ] Configuração do Docker Compose.
- [ ] Pipeline de CI com GitHub Actions.

### Fase 1: MVP
- [ ] Autenticação.
- [ ] Registro de ocorrências.
- [ ] Mapa interativo.
- [ ] Painel básico de gestão.

### Fase 2: Núcleo inteligente
- [ ] Deduplicação geoespacial.
- [ ] Pontuação de prioridade.
- [ ] Encaminhamento por área de cobertura.
- [ ] Notificações push.
- [ ] Página pública de transparência.

### Fase 3: Robustez
- [ ] Funcionamento offline.
- [ ] Moderação de conteúdo.
- [ ] Recursos de privacidade e conformidade com a LGPD.
- [ ] Observabilidade.

### Fase 4: Diferenciais
- [ ] Classificação de ocorrências por imagem.
- [ ] Desfoque de rostos e placas.
- [ ] Suporte a múltiplas cidades.
- [ ] API de dados abertos.

Consulte [`docs/SPEC.md`](docs/SPEC.md) para conhecer os requisitos e as regras de negócio detalhadas.

---

## Como contribuir

Contribuições são bem-vindas, seja por meio de código, documentação, testes ou sugestões de melhorias.

1. Leia o [`CONTRIBUTING.md`](CONTRIBUTING.md).
2. Escolha uma issue com o rótulo `good first issue`.
3. Desenvolva uma solução pequena e focada.
4. Abra um Pull Request seguindo [Conventional Commits](https://www.conventionalcommits.org/pt-br/).

Ao participar, você concorda em seguir o [Código de Conduta](CODE_OF_CONDUCT.md).

---

## Privacidade e LGPD

O Cidade Viva foi concebido para coletar apenas os dados necessários ao funcionamento da plataforma e reduzir a exposição de informações sensíveis.

Os recursos previstos incluem:

- Remoção de metadados EXIF das fotografias.
- Desfoque de rostos e placas.
- Controles para acesso e tratamento de dados pessoais.
- Exclusão de contas e exportação de dados.

A implementação deverá considerar os requisitos aplicáveis da Lei Geral de Proteção de Dados (LGPD).

---

## Licença

Distribuído sob a licença **GNU Affero General Public License v3.0 (AGPL-3.0)**.

Consulte o arquivo [`LICENSE`](LICENSE) para conhecer os termos completos.

---

<div align="center">

**Feito para tornar a cidade um pouco mais visível. 🏙️**

*Cidade Viva. Uma cidade melhor começa quando alguém decide não ignorar o problema.*

</div>
