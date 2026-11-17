# Projeto: CRUD de Usuários (login)

## Contexto
Projeto de aprendizado. O dono (Caio) é engenheiro de dados sênior (SQL, Python, Azure/Databricks),
com background web antigo (PHP 5 + Apache + AJAX). Forte em dados; precisa de explicação em:
Vue, Docker, Git moderno, DevOps, arquitetura de APIs.

## Modo de trabalho (importante)
- Antes de cada fase, proponha um plano (plan mode) e espere aprovação.
- Passos pequenos, um tema por vez. Um commit lógico por passo (só quando eu pedir o commit).
- Ao fim de cada passo: explique em até ~10 linhas o que foi feito e por quê. Quando houver paralelo
  com PHP 5 / AJAX / Apache, cite o paralelo.
- Se discordar de uma decisão ou enxergar risco, diga ANTES de implementar.
- Peça confirmação antes de: git push, apagar arquivos/pastas, dropar banco, instalar algo global.
- Rode testes e lint antes de dizer que um passo terminou e mostre o resultado.

## Stack
- Backend: Python 3.13, FastAPI, Pydantic v2, SQLAlchemy 2.0 (síncrono), Alembic, SQLite, Pytest, Ruff, uv
- Frontend: Vue 3 (Composition API, `<script setup>`), Vite, Vue Router, `fetch`. JavaScript puro (sem TypeScript por ora), sem Pinia por ora
- Portas: API 8000, front 5173. CORS libera apenas http://localhost:5173

## Estrutura (monorepo)
```
backend/
  app/
    main.py          # cria o FastAPI, inclui routers, CORS
    core/config.py   # pydantic-settings, lê .env
    db/session.py    # engine, SessionLocal, get_db
    models/          # SQLAlchemy
    schemas/         # Pydantic (Create / Update / Read separados)
    routers/         # endpoints finos
    services/        # regras de negócio (hash de senha, unicidade de login)
    scripts/         # seed do primeiro usuário
  alembic/
  tests/
  pyproject.toml
frontend/            # criado na Fase 2 com create-vue
CLAUDE.md
.env.example
```

## Modelo de dados — tabela `usuario`
| Coluna | Tipo | Regra |
|---|---|---|
| id | VARCHAR(36) | PK, UUID4 gerado no backend |
| nome | VARCHAR(120) | obrigatório |
| login | VARCHAR(60) | obrigatório, ÚNICO |
| senha_hash | VARCHAR(255) | hash Argon2; NUNCA guardar senha em texto puro |
| data_atualizacao | DATETIME | UTC, atualizada a cada alteração |

## API
| Método | Rota | Resposta |
|---|---|---|
| POST | /auth/login | 200 se login/senha válidos, 401 se não |
| GET | /usuarios | 200 lista |
| GET | /usuarios/{id} | 200 / 404 |
| POST | /usuarios | 201 / 409 (login duplicado) / 422 |
| PUT | /usuarios/{id} | 200 / 404 / 409; senha opcional (vazia = mantém) |
| DELETE | /usuarios/{id} | 204 / 404 |

Telas: /login, /usuarios (listagem + botão "Novo"), /usuarios/novo, /usuarios/:id/editar,
confirmação de exclusão (tela ou modal, decidir na Fase 2).

## Convenções
- Nomes de domínio em português (`usuario`, `login`, `senha`); infraestrutura e termos técnicos em inglês.
- Schemas de resposta NUNCA incluem senha nem hash.
- Models via `Mapped[...]` / `mapped_column`. Sessão por request com `Depends(get_db)`.
- Schema do banco só muda por migration Alembic (nada de `create_all`). No `env.py`, usar `render_as_batch=True` (SQLite).
- Configuração por `.env` (commitar só `.env.example`). Arquivo `.db` fora do Git.
- Testes: pytest + TestClient com banco isolado via override de `get_db`. Cobrir sucesso e erros de cada endpoint.
- Git: branch por fase (`fase-1-backend`...), Conventional Commits, merge via PR na `main`.

## Comandos (atualizar conforme criarmos)
```
cd backend
uv sync
uv run fastapi dev app/main.py     # API em :8000, docs em /docs
uv run pytest
uv run ruff check . && uv run ruff format .
uv run alembic upgrade head
```

## Limitações conhecidas (intencionais, Fase 1)
- /auth/login não emite token; as rotas de CRUD ficam abertas. Temporário: autenticação real (JWT) entra depois.
- Banco vazio = ninguém loga. O primeiro usuário vem de um script de seed (`app/scripts/seed.py`).
