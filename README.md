# devops_aula10ceub — IaC com Docker Compose

**CEUB · Aula 10 · Professor Danilo Silva**

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
