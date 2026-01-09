# Projeto do Bot de Mensagens

## Visão Geral

Este projeto consiste em uma plataforma web para a automação de campanhas de mensagens em massa via WhatsApp. A arquitetura é separada em um back-end robusto e uma interface de front-end, projetada para ser implantada na nuvem (AWS) como um serviço (SaaS).

---

## Backend

Construído como uma API RESTful para gerenciar toda a lógica de negócio, desde a importação de contatos até a execução e controle das campanhas de envio.

### Responsabilidades

* Gerenciar a autenticação de usuários (Registro e Login) via JWT.
* Gerenciar o ciclo de vida de Listas de Contatos (importação de .csv/.xlsx, análise, validação de telefones e armazenamento).
* Gerenciar o ciclo de vida de Campanhas (criação com templates condicionais, edição, exclusão e estratégias de deduplicação).
* Preparar campanhas, filtrando contatos que já receberam mensagens ou que possuem dados inválidos.
* Orquestrar a fila de envio de mensagens, incluindo controles de Pausar, Retomar e Cancelar.
* Manter um histórico detalhado de todos os envios (success, failed, skipped) para fins de relatório e auditoria.
* Gerenciar a conexão com o WhatsApp via `whatsapp-web.js`, incluindo o ciclo de autenticação por QR Code via WebSocket.

### Documentação da API (Visão Geral)

A API é a interface de comunicação para o front-end. 

> ⚠️ Todas as rotas (exceto `/api/auth`) exigem um **Token JWT** (`Authorization: Bearer <token>`)

### 1. Autenticação `/api/auth`
| Método | Rota | Descrição |
|---------|------|------------|
| POST | `/register` | Registra um novo usuário |
| POST | `/login` | Autentica e retorna token JWT |

### 2. Listas de Contatos `/api/lists`
| Método | Rota | Descrição |
|---------|------|------------|
| POST | `/analyze` | Analisa arquivo CSV/XLSX (prévia dos dados) |
| POST | `/confirm` | Valida telefones e salva lista |
| GET | `/` | Retorna todas as listas |
| GET | `/:id` | Retorna detalhes da lista |
| GET | `/:id/columns/:columnName/unique-values` | Extrai valores únicos de uma coluna |
| DELETE | `/:id` | Exclui uma lista e seus dados associados |

### 3. Campanhas `/api/campaigns`
| Método | Rota | Descrição |
|---------|------|------------|
| GET/POST | `/` | Lista ou cria campanhas |
| GET/PUT/DELETE | `/:id` | Gerencia campanha específica |
| POST | `/:id/prepare` | Prepara contatos e gera resumo |
| POST | `/:id/start` | Inicia envio da campanha |
| GET | `/:id/report` | Gera relatório detalhado |

### 4. Controle de Fila `/api/campaigns/control`
| Método | Rota | Descrição |
|---------|------|------------|
| GET | `/status` | Retorna status atual da fila |
| POST | `/pause` | Pausa o envio |
| POST | `/resume` | Retoma envio |
| POST | `/cancel` | Cancela a campanha |

### 5. Histórico `/api/history`
| Método | Rota | Descrição |
|---------|------|------------|
| GET | `/` | Consulta histórico com filtros |
| GET | `/export` | Exporta histórico em `.csv` |
| DELETE | `/:recordId` | Remove registro único |
| DELETE | `/` | Limpa todo histórico de uma campanha |

### 6. WhatsApp `/api/whatsapp`
| Método | Rota | Descrição |
|---------|------|------------|
| GET | `/status` | Verifica conexão |
| POST | `/connect` | Inicia autenticação (QR via WebSocket) |
| POST | `/logout` | Desconecta cliente |

---

## Frontend

*(Esta seção será preenchida com os detalhes da interface do usuário, tecnologias e como executá-la.)*

---

## 🚀 Como Rodar o Backend (Ambiente de Desenvolvimento)

O projeto utiliza Docker e a extensão Dev Containers do VS Code para criar um ambiente de desenvolvimento isolado e consistente.

### Pré-requisitos
- [Docker Desktop](https://www.docker.com/)
- [Visual Studio Code](https://code.visualstudio.com/)
- Extensão **Dev Containers** (Microsoft)

### Passos para Instalação

1.  **Clone o repositório.**
2.  **Abra a pasta `backend` no VS Code.**
3.  **Crie o arquivo de ambiente:**
    * Crie um arquivo chamado `.env` dentro da pasta `backend`.
    * Adicione a chave secreta para a autenticação JWT. O `DATABASE_URL` já é fornecido pelo Docker Compose.

    ```dotenv
    # backend/.env
    JWT_SECRET=sua-chave-secreta-aqui-bem-longa-e-segura
    ```

4.  **Inicie o Dev Container:**
    * Abra a paleta de comandos (`Ctrl+Shift+P` ou `F1`).
    * Execute **`Dev Containers: Reopen in Container`**.
    * Aguarde o ambiente ser construído (o Docker irá baixar a imagem do Postgres e construir o container do Node.js).

5.  **Execute a Migração do Banco de Dados:**
    * No terminal do VS Code (que agora está *dentro* do container), execute a migração do Prisma para criar as tabelas no banco de dados:
    ```bash
    npx prisma migrate dev
    ```

### Executando o Servidor

No terminal do Dev Container, inicie o servidor com:

```bash
npm start
```
O servidor estará disponível em `http://localhost:4000`.
