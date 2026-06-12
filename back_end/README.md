# Back-End - Nação Nutrida

Esta pasta contém o código do back-end da aplicação "Nação Nutrida", desenvolvido em Node.js com TypeScript e Express. O back-end é responsável por fornecer a API RESTful que suporta as funcionalidades da plataforma de doação de alimentos.

## Estrutura da Pasta

- **server/**: Pasta principal contendo todo o código do back-end.
  - `src/`: Código fonte organizado em camadas.
    - `controllers/`: Controladores para cada entidade (UsuarioController, CampanhaController, etc.).
    - `services/`: Lógica de negócio e regras de aplicação.
    - `routes/`: Definição das rotas da API.
    - `schemas/`: Validações usando Zod.
    - `middlewares/`: Middlewares de autenticação (JWT) e validação.
    - `types/`: Definições de tipos TypeScript.
  - `prisma/`: Configuração do banco de dados com Prisma ORM.
    - `schema.prisma`: Schema do banco de dados MongoDB.
    - `seed.ts`: Script para popular o banco com dados iniciais.
  - `config/`: Configurações adicionais, como integração com API do IBGE.
  - `server.ts`: Ponto de entrada da aplicação.
  - `package.json`: Dependências e scripts do projeto.
  - `tsconfig.json`: Configuração do TypeScript.

## Tecnologias Utilizadas

- **Node.js**: Ambiente de execução JavaScript no servidor.
- **Express**: Framework web para criação da API.
- **TypeScript**: Superset do JavaScript com tipagem estática.
- **Prisma**: ORM para interação com o banco de dados MongoDB.
- **MongoDB**: Banco de dados NoSQL.
- **Azure Service Bus**: Mensageria (fila) para processamento assíncrono de eventos de doação.
- **JWT**: Autenticação baseada em tokens.
- **Bcrypt**: Hashing de senhas.
- **Zod**: Validação de dados.
- **CORS**: Suporte a requisições cross-origin.

## Funcionalidades Principais

- **Autenticação e Autorização**: Login, registro e validação de usuários (JWT, expiração de 12h; senhas com Bcrypt).
- **Gerenciamento de Usuários**: CRUD de doadores e organizações.
- **Campanhas**: Criação, listagem e gerenciamento de campanhas de doação.
- **Doações**: Registro e histórico de doações de alimentos.
- **Mensageria (Azure Service Bus)**: Cada doação publica um evento `nova_doacao` na fila `fila1`. Um consumidor no próprio backend lê o evento de forma assíncrona e recalcula o progresso da campanha.
- **Encerramento Automático de Campanhas**: Quando todas as metas de alimento são atingidas, a campanha é desativada automaticamente (`fg_campanha_ativa = false`) pelo consumidor da fila.
- **Recomendações (Mineração de Dados)**: Endpoints que expõem as regras de associação (Apriori) geradas em Python, sugerindo alimentos/campanhas com base no histórico do doador.
- **Chat**: Sistema de mensagens entre usuários.
- **Integração Externa**: API do IBGE para busca de localidades brasileiras.

## Como Executar

1. Instale as dependências:
   ```bash
   npm install
   ```

2. Configure as variáveis de ambiente. Crie um arquivo `.env` em `server/` com:
   ```env
   DATABASE_URL="<string de conexão do MongoDB>"
   JWT_SECRET="<segredo para assinar os tokens JWT>"
   JWT_EXPIRES_IN="12h"
   SERVICEBUS_CONNECTION_STRING="<connection string do Azure Service Bus>"
   PORT=5000
   ```
   > Se a `SERVICEBUS_CONNECTION_STRING` estiver ausente, a API sobe normalmente, mas o consumidor da fila não é iniciado.

3. Sincronize o schema com o banco (MongoDB usa `db push`, não `migrate`):
   ```bash
   npx prisma db push
   ```

4. Inicie o servidor:
   ```bash
   npm start      # produção (node server.ts)
   npm run dev    # desenvolvimento (nodemon)
   ```

O servidor estará rodando em `http://localhost:5000` (ou na porta definida em `PORT`). A documentação interativa (Swagger) fica em `http://localhost:5000/docs`.

## Endpoints Principais

> Todas as rotas usam o prefixo **`/api`**. As rotas marcadas com 🔒 exigem token JWT no header `Authorization: Bearer <token>`.

**Usuário**
- `POST /api/usuarioCadastro`: Registro de usuário
- `POST /api/usuarioLogin`: Login (retorna o token JWT)
- `GET /api/perfil` 🔒: Dados do usuário logado
- `PATCH /api/usuario/:id` 🔒: Atualizar usuário

**Campanha**
- `GET /api/campanhas`: Listar campanhas
- `GET /api/campanhas/buscar`: Buscar campanhas por localização
- `GET /api/campanhas/:id`: Detalhes de uma campanha
- `POST /api/campanhas` 🔒: Criar campanha
- `PATCH /api/campanhas/desativar/:id` 🔒: Desativar campanha

**Doação**
- `POST /api/doacoes` 🔒: Registrar doação (publica evento no Service Bus)
- `GET /api/doacoes/minhas` 🔒: Histórico de doações do usuário

**Alimento**
- `GET /api/alimentos` 🔒: Listar alimentos por tipo

**Mineração / Recomendações** 🔒
- `GET /api/mineracao/recomendacoes?alimento=Arroz`: Recomendações para um alimento
- `POST /api/mineracao/recomendacoes`: Recomendações para múltiplos alimentos
- `GET /api/mineracao/regras` · `GET /api/mineracao/estatisticas` · `GET /api/mineracao/historico`

**Chat** 🔒
- `GET|POST /api/chat/conversations`: Listar/criar conversas
- `GET|POST /api/chat/messages`: Listar/enviar mensagens

**Localidade**
- `GET /api/...`: Integração com a API do IBGE (estados/cidades)

Para a documentação completa e interativa, acesse **`/docs`** (Swagger) ou consulte os arquivos de rotas em `server/src/routes/`.