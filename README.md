<p align="center"><img src="assets/logo-400.png" width="96" alt="emissaoja"></p>

# emissaoja MCP server: issue Brazilian NFS-e from your AI assistant

Remote [Model Context Protocol](https://modelcontextprotocol.io) server to **issue Brazilian service invoices (NFS-e, Nota Fiscal de Serviço eletrônica)** through the national **Emissor Nacional** (nfse.gov.br), **without a digital certificate**. Built for MEI and Simples Nacional companies. Works in Claude, ChatGPT, Claude Code, Cursor, VS Code, Cline and any MCP client that supports Streamable HTTP.

```
https://emissaoja.com.br/mcp
```

This repository holds the public documentation and the [MCP Registry](https://registry.modelcontextprotocol.io) manifest ([`server.json`](server.json), name `br.com.emissaoja/nfse`). The server is hosted by [emissaoja](https://emissaoja.com.br/emitir-nota-fiscal-por-ia); there is nothing to install or run.

[![Install in Cursor](https://cursor.com/deeplink/mcp-install-dark.svg)](https://cursor.com/install-mcp?name=emissaoja&config=eyJ1cmwiOiJodHRwczovL2VtaXNzYW9qYS5jb20uYnIvbWNwIn0=) [![Install in VS Code](https://img.shields.io/badge/VS_Code-Install_emissaoja_MCP-0098FF?logo=visualstudiocode&logoColor=white)](https://vscode.dev/redirect/mcp/install?name=emissaoja&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Femissaoja.com.br%2Fmcp%22%7D)

## Tools

| Tool | What it does | Kind |
|---|---|---|
| `status` | Whether emissaoja and the tax engine are up | read-only, no account |
| `validar_cnpj` | Checks format and check digits of a CNPJ | read-only, no account |
| `consultar_cnpj` | Company record from Receita Federal (legal name, status, CNAE, address, Simples/MEI, partners with masked CPF) | read-only |
| `minha_empresa` | The account's company: CNPJ, credential state, onboarding, import, warnings. Start here | read-only |
| `configurar_credencial` | Connects the company to Emissor Nacional with CNPJ and the portal password (checked on the portal, never stored in clear or returned) | write |
| `situacao_credencial` | State of the credential check and the next step | read-only |
| `perguntas_fiscais` | Tax questions the registry does not answer by itself (Simples rate, accounting regime) | read-only |
| `responder_perguntas_fiscais` | Saves the answers the user gave (never invented) | write |
| `listar_tomadores` | Customers (service takers), with search | read-only |
| `criar_tomador` | Registers a customer by CNPJ (name and address from Receita Federal) or by CPF with the full name | write |
| `listar_servicos` | Registered services (national service code, ISS, main service) | read-only |
| `emitir_nota` | **Issues an NFS-e** for a registered customer. Preview first; returns what is missing instead of issuing when blocked. Idempotent by `referencia` | write, real tax effect |
| `acompanhar_pedido` | State of an issuing request: queued → issuing → authorized (number, access key), refused (reason) or uncertain | read-only |
| `listar_notas` | Issued invoices (local and imported from the portal), by period, status and search | read-only |
| `cancelar_nota` | **Cancels an invoice** within the Emissor Nacional deadline | write, destructive |
| `acompanhar_operacao` | State of a cancellation | read-only |

`emitir_nota` and `cancelar_nota` have real tax effect: their descriptions tell the assistant to confirm customer, amount and description (or the invoice and reason) with you before calling.

## Authentication

- **OAuth (recommended):** Claude, ChatGPT, Claude Code, Cursor and VS Code open the emissaoja sign-in the first time a tool needs an account. Sign in with Google and authorize. The connection lasts 60 days without use and can be revoked in Conta → API.
- **API key:** clients without OAuth send `Authorization: Bearer ej_live_...`. Create the key in [emissaoja.com.br](https://emissaoja.com.br/?origem=mcp-github) → Conta → API.
- **Sandbox:** `https://sandbox.emissaoja.com.br/mcp` with `ej_test_` keys runs against a simulated Emissor Nacional (nothing reaches the tax authority). Example company: CNPJ `11111111000191`, any password.

## Connect

**Claude (claude.ai, Desktop, mobile):** Settings → Connectors → *Add custom connector* → URL `https://emissaoja.com.br/mcp`.

**ChatGPT:** Settings → Apps & Connectors → enable Developer mode → *Add custom MCP server* → URL `https://emissaoja.com.br/mcp`, authentication OAuth.

**Claude Code:**

```bash
claude mcp add --transport http emissaoja https://emissaoja.com.br/mcp
# or with an API key:
claude mcp add --transport http emissaoja https://emissaoja.com.br/mcp --header "Authorization: Bearer ej_live_YOUR_KEY"
```

**Cursor:** use the install button above, the [emissaoja Cursor plugin](https://github.com/Easymovel/emissaoja-cursor-plugin), or add to `.cursor/mcp.json`:

```json
{ "mcpServers": { "emissaoja": { "url": "https://emissaoja.com.br/mcp" } } }
```

**VS Code:** use the install button above, or add to `.vscode/mcp.json`:

```json
{ "servers": { "emissaoja": { "type": "http", "url": "https://emissaoja.com.br/mcp" } } }
```

**Cline:** see [llms-install.md](llms-install.md).

**Other clients (API key):**

```json
{
  "mcpServers": {
    "emissaoja": {
      "type": "http",
      "url": "https://emissaoja.com.br/mcp",
      "headers": { "Authorization": "Bearer ej_live_YOUR_KEY" }
    }
  }
}
```

## Example prompts

- "Emite uma nota de R$ 1.500 para a Alfa Ltda, CNPJ 12.345.678/0001-95, referente à consultoria de novembro."
- "Quais notas eu emiti este mês e qual o total?"
- "A nota 42 saiu com o valor errado: cancela por erro na emissão e emite outra de R$ 1.200."
- "Minha empresa já está pronta para emitir? O que falta configurar?"
- "Consulta o CNPJ 12.345.678/0001-95 antes de cadastrar como cliente."

## Em português

Servidor MCP remoto do [emissaoja](https://emissaoja.com.br/emitir-nota-fiscal-por-ia) para **emitir NFS-e pelo seu assistente de IA** (Claude, ChatGPT, Cursor, VS Code): o assistente consulta sua empresa, cadastra clientes pelo CNPJ, emite a nota no Emissor Nacional **sem certificado digital**, acompanha a autorização e cancela, sempre confirmando com você antes de emitir ou cancelar. Para MEI e empresas do Simples Nacional. Plano Grátis com 1 nota por mês; planos em [emissaoja.com.br/#precos](https://emissaoja.com.br/#precos). Documentação da API: [emissaoja.com.br/docs](https://emissaoja.com.br/docs/).

Bibliotecas oficiais da mesma API: [Node.js/TypeScript](https://github.com/Easymovel/emissaoja-js) · [Python](https://github.com/Easymovel/emissaoja-py) · [PHP/Laravel](https://github.com/Easymovel/emissaoja-php) · [.NET](https://github.com/Easymovel/emissaoja-dotnet) · [Go](https://github.com/Easymovel/emissaoja-go) · [Ruby](https://github.com/Easymovel/emissaoja-ruby) · [n8n](https://github.com/Easymovel/n8n-nodes-emissaoja) · [WooCommerce](https://github.com/Easymovel/emissaoja-woocommerce).

## Limits and scope

- Companies: MEI, ME and EPP under Simples Nacional, in a municipality that issues through Emissor Nacional. One company per account.
- Rate limits: 60 calls/min per key, 10 issuances/min. An invoice counts toward the plan when Emissor Nacional accepts it; refused invoices do not count.
- Support: [suporte@emissaoja.com.br](mailto:suporte@emissaoja.com.br) · Terms: https://emissaoja.com.br/termos/ · Privacy: https://emissaoja.com.br/privacidade/

emissaoja is a private service, operated by BINOBO LTDA, with no affiliation with Receita Federal or Emissor Nacional.
