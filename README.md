<div align="center">

# 👀 Olhaí

**Viu um problema na rua? Olhaí.**

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

Buraco na rua, poste apagado, lixo acumulado, bueiro aberto. As reclamações se perdem em canais diferentes, ninguém enxerga o padrão e o cidadão nunca sabe se o problema foi resolvido.

## A solução

O cidadão reporta em segundos (foto + GPS). O Olhaí:

- **Junta reportes iguais** em vez de criar duplicatas
- **Encaminha ao órgão certo** pela área de cobertura
- **Acompanha cada mudança de status** com histórico imutável
- **Funciona offline**, porque o problema costuma estar onde o sinal é ruim
- **Expõe publicamente** o tempo médio de resolução por bairro e órgão

---

## Funcionalidades

| Área | O que tem |
|---|---|
| 📱 **App mobile** | Reporte com foto e GPS, mapa, confirmação de problemas de terceiros, push de status, modo offline |
| 🗺️ **Mapa** | Filtros, clusterização por zoom, mapa de calor, busca por endereço |
| 🧠 **Inteligência** | Deduplicação em raio de 30 m (PostGIS), score de prioridade, roteamento por polígono |
| 🖥️ **Painel do gestor** | Fila priorizada, troca de status, atribuição, exportação CSV/PDF, webhook e e-mail |
| 🛡️ **Moderação** | Denúncias, reputação de usuários, detecção de foto reaproveitada, rate limit |
| 🔎 **Transparência** | Página pública com estatísticas, rankings e API de dados abertos |
| 🔐 **Privacidade** | Remoção de EXIF, desfoque de rostos e placas, exclusão de conta (LGPD) |

A lista completa, com prioridades e regras de negócio, está em [`docs/SPEC.md`](docs/SPEC.md).

---

## Arquitetura

Monólito modular: um deploy, módulos com fronteiras claras.

```
┌──────────────┐   ┌──────────────┐
│ App RN (Expo)│   │ Painel React │
└──────┬───────┘   └──────┬───────┘
       │ HTTPS / SignalR  │
       └────────┬─────────┘
         ┌──────▼───────┐
         │ API ASP.NET  │── Hangfire (jobs)
         │   (.NET 9)   │── SignalR (tempo real)
         └──┬─────┬───┬─┘
            │     │   └──────► MinIO / S3 (fotos)
            │     └──────────► Redis (cache, rate limit)
            ▼
   PostgreSQL + PostGIS
```

Decisões de arquitetura estão registradas em [`docs/adr`](docs/adr).

## Stack

| Camada | Tecnologias |
|---|---|
| **Mobile** | React Native (Expo), TypeScript, TanStack Query, MapLibre, SQLite |
| **Web** | React, Vite, TypeScript, MapLibre GL, Recharts, Tailwind |
| **API** | .NET 9, ASP.NET Core, EF Core + NetTopologySuite, MediatR, FluentValidation |
| **Dados** | PostgreSQL 16 + PostGIS, Redis |
| **Jobs / tempo real** | Hangfire, SignalR |
| **Storage** | MinIO (dev) / S3 compatível (prod) |
| **Observabilidade** | OpenTelemetry, Serilog |
| **Testes** | xUnit, Testcontainers, Playwright, Jest |
| **CI/CD** | GitHub Actions, Docker |

---

## Como rodar

### Pré-requisitos

- [Docker](https://www.docker.com/) e Docker Compose
- [.NET SDK 9](https://dotnet.microsoft.com/download)
- [Node.js 20+](https://nodejs.org/) e npm
- (Mobile) app **Expo Go** no celular ou um emulador Android/iOS

### Subir a infraestrutura e a API

```bash
git clone https://github.com/SEU_USUARIO/olhai.git
cd olhai

# Postgres+PostGIS, Redis, MinIO
docker compose up -d

# Migrações e dados de exemplo (cidade fictícia)
dotnet run --project tools/seed

# API
dotnet run --project src/Olhai.Api
```

A API sobe em `http://localhost:5000` e a documentação OpenAPI em `http://localhost:5000/swagger`.

### Painel web

```bash
cd apps/web
npm install
npm run dev
```

### App mobile

```bash
cd apps/mobile
npm install
npx expo start
```

### Variáveis de ambiente

Copie os exemplos e ajuste:

```bash
cp .env.example .env
cp apps/web/.env.example apps/web/.env
cp apps/mobile/.env.example apps/mobile/.env
```

> Nunca versione arquivos `.env` com segredos reais.

---

## Estrutura do repositório

```
olhai/
├── apps/
│   ├── mobile/            # Expo (React Native)
│   └── web/               # Painel do gestor + página pública
├── src/
│   ├── Olhai.Api/
│   ├── Olhai.Modules.Identity/
│   ├── Olhai.Modules.Reports/
│   ├── Olhai.Modules.Organizations/
│   ├── Olhai.Modules.Moderation/
│   ├── Olhai.Modules.Notifications/
│   ├── Olhai.Modules.Analytics/
│   └── Olhai.SharedKernel/
├── tests/
│   ├── Olhai.UnitTests/
│   └── Olhai.IntegrationTests/
├── tools/seed/            # gerador de dados de exemplo
├── docs/                  # SPEC, ADRs, arquitetura
└── docker-compose.yml
```

## Testes

```bash
# Back-end (unitários + integração com Testcontainers)
dotnet test

# Web (E2E)
cd apps/web && npx playwright test

# Mobile
cd apps/mobile && npm test
```

---

## Roadmap

- [ ] **Fase 0:** fundação (repo, Docker Compose, CI)
- [ ] **Fase 1:** MVP (auth, criar reporte, mapa, painel básico)
- [ ] **Fase 2:** núcleo inteligente (deduplicação, score, roteamento, push, página pública)
- [ ] **Fase 3:** robustez (offline-first, moderação, LGPD, observabilidade)
- [ ] **Fase 4:** diferenciais (classificação por imagem, desfoque de rostos, multi-cidade, API aberta)

Detalhes em [`docs/SPEC.md`](docs/SPEC.md).

---

## Como contribuir

Contribuições são bem-vindas, de código a documentação e testes.

1. Leia o [`CONTRIBUTING.md`](CONTRIBUTING.md)
2. Escolha uma issue com o rótulo `good first issue`
3. Abra um PR pequeno e focado, seguindo [Conventional Commits](https://www.conventionalcommits.org/pt-br/)

Ao participar, você concorda em seguir o [Código de Conduta](CODE_OF_CONDUCT.md).

## Privacidade e LGPD

O Olhaí coleta apenas os dados necessários, remove metadados sensíveis das fotos e permite exclusão de conta e exportação de dados. Se uma foto sua aparece na plataforma e você quer removê-la, abra uma issue ou escreva para o contato do projeto.

## Licença

Distribuído sob a licença **AGPL-3.0**. Veja [`LICENSE`](LICENSE).

---

<div align="center">

Feito para tornar a cidade um pouco mais visível. 👀

</div>
