# Serviço 08 — Cadastro / Identidade - Guilherme Caetano dos Santos e Eduarda Gadanha Guerra

Serviço responsável por validar tokens de autenticação e devolver o perfil do usuário correspondente, dentro do sistema distribuído da pizzaria (Arquitetura de Serviços em Nuvem).

**Tecnologias:** n8n (orquestração dos workflows) + Redis (armazenamento)

## Sumário

- [URL Base](#url-base)
- [Headers obrigatórios](#headers-obrigatórios)
- [Endpoints](#endpoints)
- [GET /v1/cadastro/health](#get-v1cadastrohealth)
- [POST /v1/cadastro/validar](#post-v1cadastrovalidar)
- [Códigos de erro](#códigos-de-erro)
- [Exemplo real de chamada](#exemplo-real-de-chamada)
- [Observabilidade](#observabilidade)

## URL Base

```
https://pzaas.online/webhook
```

Todos os endpoints abaixo devem ser combinados com essa URL base.

## Headers obrigatórios

| Header | Valor | Obrigatório em |
|---|---|---|
| `Content-Type` | `application/json` | Todas as requisições |
| `x-api-key` | `turma2026` | Todas as requisições, exceto `/health` |
| `x-pedido-id` | `<id gerado pelo Gateway>` | Todas as requisições |

Se o `x-api-key` estiver ausente ou incorreto, a API retorna `403`.

## Endpoints

### `GET /v1/cadastro/health`

**URL completa:** `https://pzaas.online/webhook/v1/cadastro/health`

Verifica se o serviço está no ar. Não requer autenticação.

**Resposta (200 OK)**
```json
{ "status": "ok" }
```

### `POST /v1/cadastro/validar`

**URL completa:** `https://pzaas.online/webhook/v1/cadastro/validar`

Valida um token e devolve o perfil do usuário correspondente, consultando a base de usuários no Redis.

**Body da requisição**
```json
{ "token": "abc123" }
```

**Resposta de sucesso (200 OK)**
```json
{
  "usuario_id": "u001",
  "nome": "João Silva",
  "email": "joao@teste.com",
  "telefone": "11999990001",
  "valido": true
}
```

## Códigos de erro

| Código | Motivo | Exemplo de resposta |
|---|---|---|
| `400` | campo `token` ausente no body | `{ "erro": "campo token ausente" }` |
| `403` | `x-api-key` ausente ou inválida | `{ "erro": "x-api-key inválida" }` |
| `404` | token não encontrado na base | `{ "erro": "token não encontrado" }` |

## Exemplo real de chamada

**Requisição:**
```
POST https://pzaas.online/webhook/v1/cadastro/validar
Content-Type: application/json
x-api-key: turma2026
x-pedido-id: teste001

{
  "token": "abc123"
}
```

**Resposta (200 OK):**
```json
{
  "usuario_id": "u001",
  "nome": "João Silva",
  "email": "joao@teste.com",
  "telefone": "11999990001",
  "valido": true
}
```

## Status da integração

Este serviço é consultado de forma independente (não faz parte da orquestração do checkout do API Gateway — serviço 01). Qualquer serviço ou cliente que precise validar um token de usuário pode chamar `POST /v1/cadastro/validar` diretamente.

## Observabilidade

Este serviço envia logs estruturados para o Logger (serviço 09), de forma assíncrona a cada chamada de `/v1/cadastro/validar`, no seguinte formato:

```json
{
  "pedido_id": "<vindo do header x-pedido-id>",
  "servico": "cadastro",
  "nivel": "info",
  "mensagem": "Validação de token realizada com sucesso",
  "timestamp": "<data/hora ISO no momento da chamada>"
}
