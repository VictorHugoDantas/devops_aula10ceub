# devops_aula10ceub — IaC com Docker Compose

**CEUB · Aula 10 · Professor Danilo Silva**
Material base para as próximas aulas: teoria (slides) + atividade prática (código pronto nesta pasta).

## Estrutura do repositório

```
devops_aula10ceub/
├── package.json         # dependências da API (express, pg)
├── server.js            # API Node.js que consulta o Postgres (SELECT NOW())
├── Dockerfile           # receita da imagem da aplicação
├── docker-compose.yml   # orquestração: serviços db + app, rede e volume
└── README.md            # este arquivo (teoria + passo a passo)
```

---

## Parte 1 — Teoria (slides da Aula 10)

### Onde estamos?
No ciclo DevOps (Plan → Code → Build → Test → Release → Deploy → Operate → Monitor), esta aula está na fase **Ops / Deploy**: Infrastructure as Code, Provisioning, Configuration Management, Virtualization e **Containerization**.

### O Modo Antigo × O Modo Compose

| | O Modo Antigo (o problema) | O Modo Compose (a solução) |
|---|---|---|
| **Sintoma / Conceito** | "Na minha máquina funciona!" | Infraestrutura como Código (IaC) aplicada ao ambiente de trabalho diário. |
| **Causa / Resultado** | Instalações manuais e repetitivas; conflitos de versão no SO hospedeiro; horas de configuração para novos devs. | Definição e execução de aplicativos multi-contêineres de forma totalmente padronizada a partir de um único arquivo declarativo. |

### Os Três Pilares do Compose para DevOps
1. **Ambiente Declarativo** — todo o stack (front-end, back-end, banco) é versionado e centralizado em um único arquivo YAML.
2. **Onboarding Imediato** — basta clonar o repositório e executar um comando para ter a infraestrutura rodando com as dependências e versões exatas.
3. **Isolamento Absoluto** — cada projeto tem seu ecossistema fechado. Dá para rodar PostgreSQL 13 em um projeto e PostgreSQL 15 em outro, sem conflitos no host.

### O Papel Estratégico do Orquestrador Local
O **Compose é a ponte** entre o desenvolvimento local e a produção/nuvem: garante que o código nasça num ambiente conteinerizado e padronizado, preparado para orquestradores em nuvem no futuro.

- **Docker Compose (desenvolvimento local):** simplicidade, isolamento, reprodutibilidade imediata.
- **Kubernetes / Pipelines (produção e nuvem):** escalabilidade distribuída, alta disponibilidade, gestão de tráfego massivo.

### A Tríade: a planta baixa da aplicação
Todo `docker-compose.yml` se apoia em três pilares:

| Pilar | Papel |
|---|---|
| **Services** | A Execução |
| **Networks** | A Comunicação |
| **Volumes** | A Persistência |

#### Pilar 1 — Services (os motores da execução)
Um serviço é um contêiner em execução derivado de uma imagem Docker (ex.: API Node.js ou banco PostgreSQL).
- `image:` / `build:` — origem do código que será executado.
- `ports:` — mapeia portas do contêiner isolado para o host (`"HOST:CONTAINER"`).
- `environment:` — configurações, credenciais e variáveis de ambiente injetadas no contêiner.

#### Pilar 2 — Networks (a comunicação invisível)
- **Service Discovery (DNS embutido):** os contêineres não precisam saber o IP uns dos outros; a rede traduz o nome do serviço declarado no YAML para a rota correta (`db:5432` em vez de `192.168.x.x`).
- **Segurança padrão:** serviços na mesma rede se comunicam livremente internamente, mas ficam invisíveis/inacessíveis para o exterior (exceto portas publicadas).

#### Pilar 3 — Volumes (a garantia de persistência)
Contêineres nascem para morrer: se um banco é recriado sem volume, os dados somem.
- **Sem volume:** dados perdidos no reinício.
- **Com volume:** os dados sobrevivem ao `docker-compose down` e ficam intactos no armazenamento do host.

### Lógica de Provisionamento e Ciclo de Vida
- **Inicialização — `depends_on` (gerenciamento de dependências):** um backend falha se tentar conectar a um banco inexistente; `depends_on` força a ordem de subida.
- **Verificação de saúde — Healthchecks (sondas de vida):** apenas ligar o contêiner não basta; com healthcheck, o Compose aguarda o banco reportar que está pronto para receber queries.

### O Desafio Prático — arquitetura
```
┌──────────────── Rede: app-network ────────────────┐
│                                                   │
│  [Serviço 2: API Node.js] ──SELECT NOW()──▶ [Serviço 1: PostgreSQL 15]
│        porta 8080                          porta 5432
│                                                   │
└───────────────────────────────────────────────────┘
                                   Volume: pg_dados (persistência)
```

### Passo 1 — A Receita da Aplicação (Dockerfile)
| Instrução | O que faz |
|---|---|
| `FROM node:18-alpine` | Imagem base oficial, leve e otimizada. |
| `WORKDIR` / `COPY package*.json ./` / `RUN npm install` / `COPY . .` | Prepara o terreno: copia arquivos e instala dependências (Express e PG). |
| `EXPOSE 8080` / `CMD ["node", "server.js"]` | Define a porta interna e o comando final que dá vida à API. |

### Passo 2 — O Coração da Orquestração (docker-compose.yml)
- `services` → provisiona as imagens e usa o DNS embutido (`DB_HOST: db`).
- `volumes: pg_dados` → volume físico para retenção dos dados do Postgres.
- `networks: app-network (bridge)` → rede isolada para comunicação segura entre app e db.

### Passo 3 — O Ciclo de Execução (CLI)
| Etapa | Comando | O que faz |
|---|---|---|
| A Criação | `docker-compose up -d` | Constrói a imagem Node, baixa o Postgres, cria redes/volumes e sobe tudo em background (detached). |
| A Verificação | `docker-compose ps` | Mostra os contêineres com status `Up` e as portas mapeadas. |
| O Desmonte | `docker-compose down` | Remove contêineres e redes, **preservando os volumes de dados**. |

### Validação — o que o resultado prova
Ao abrir a aplicação no navegador e ver **"Comunicação com sucesso! / Data do Banco de Dados: …"**:
- ✅ O serviço Node.js foi construído e está roteado no host.
- ✅ O PostgreSQL provisionou a base corretamente.
- ✅ O Service Discovery resolveu o host `db` pela rede.
- ✅ A query `SELECT NOW()` confirmou o acesso de ponta a ponta.

### Check-list do Orquestrador
| | |
|---|---|
| **O Paradigma** | Docker Compose é IaC focada em simplicidade, reprodutibilidade e onboarding imediato para desenvolvimento local. |
| **A Estrutura (Tríade)** | Services (processamento), Networks (comunicação isolada), Volumes (persistência). |
| **O Fluxo de Vida** | Da declaração no YAML à orquestração via CLI (`up -d`, `ps`, `down`). |

> **O fim do "Na minha máquina funciona". O código é a sua infraestrutura.**

---

## Parte 2 — Atividade Prática: Orquestração Multi-contêiner com Docker Compose

**Objetivo:** provisionar e integrar uma aplicação web com um banco de dados relacional usando IaC.

**Antes de iniciar:** criar um novo repositório no GitHub para a atividade e subir os 4 arquivos desta pasta (`package.json`, `server.js`, `Dockerfile`, `docker-compose.yml`).

### 1. Arquivos da aplicação (Node.js)
`package.json` e `server.js` — API simples que conecta no banco e retorna a data atual do servidor, provando que a comunicação funcionou.
> No `server.js`: *"verificar a porta a ser utilizada com o professor"* (padrão: `8080`).

### 2. Receita da aplicação
`Dockerfile` (sem extensão) na mesma pasta.

### 3. Orquestrar a infraestrutura
`docker-compose.yml` — define os serviços `db` e `app`, a rede `app-network` e o volume `pg_dados`.
> Troque `meunome` em `container_name` pelo seu nome (ex.: `meu_postgres_dantas`, `minha_api_node_dantas`) para não colidir com colegas no servidor compartilhado.
> Ajuste a porta do host em `ports: "8050:8080"` para a porta designada pelo professor.

### 4. Executar e validar
Acesse o ambiente via SSH (o professor coloca no quadro) e crie uma pasta com seu nome:
```bash
mkdir meunome
cd meunome
```

Vincule o repositório recém-criado no GitHub (dentro da sua pasta no SSH):
```bash
git init
git branch -M main
git remote add origin https://github.com/meunome/seurepositorio.git
git pull origin main
```

Suba a infraestrutura:
```bash
sudo docker compose up -d
```

Confirme se os contêineres estão `Up`:
```bash
sudo docker compose ps
```
> Na folha da atividade está escrito `docker-compose os` — é erro de digitação; o correto é `ps`.

Abra no navegador: `http://<ip-no-quadro>:<sua-porta>` (a folha cita `:8000`, mas o compose publica `8050`; use **a porta que o professor designar e que estiver no seu `ports:`**).

Se aparecer **"Comunicação com sucesso!"** com a data do banco, a orquestração funcionou.

### 5. Desmontagem (Clean Up)
Após os testes e os *prints* para a entrega:
```bash
sudo docker compose down
```

> `docker-compose` (v1, com hífen) e `docker compose` (v2, plugin) fazem a mesma coisa; use o que estiver instalado no servidor.
