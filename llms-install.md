# Installing the emissaoja MCP server (for AI agents such as Cline)

emissaoja is a **remote** MCP server that issues Brazilian service invoices (NFS-e) through Emissor Nacional. There is nothing to clone, build or run: only add the server URL to the client configuration.

- URL: `https://emissaoja.com.br/mcp` (production) or `https://sandbox.emissaoja.com.br/mcp` (simulated tax authority, for tests)
- Transport: Streamable HTTP
- Tools: `status` and `validar_cnpj` work without an account; the others need the user's emissaoja account.

## Cline

Add this entry to `cline_mcp_settings.json` (Cline → MCP Servers → Configure → "Configure MCP Servers"):

```json
{
  "mcpServers": {
    "emissaoja": {
      "type": "streamableHttp",
      "url": "https://emissaoja.com.br/mcp",
      "disabled": false,
      "autoApprove": ["status", "validar_cnpj", "minha_empresa", "listar_tomadores", "listar_servicos", "listar_notas", "acompanhar_pedido", "acompanhar_operacao"]
    }
  }
}
```

Never add `emitir_nota`, `cancelar_nota` or `configurar_credencial` to `autoApprove`: issuing and cancelling invoices have real tax effect and must be confirmed by the user every time.

To use the account tools:

1. If the client supports MCP OAuth, it opens the emissaoja sign-in on the first call: the user signs in with Google and authorizes.
2. Otherwise, ask the user to sign in at https://emissaoja.com.br, create an API key in Conta → API, and add the header to the same entry:

```json
"headers": { "Authorization": "Bearer ej_live_YOUR_KEY" }
```

Never print, log or commit the API key. For tests, use `https://sandbox.emissaoja.com.br/mcp` with an `ej_test_` key created at https://sandbox.emissaoja.com.br.

## Verify

Call `status` (no arguments): the response says whether the service and the tax engine are up. Then call `minha_empresa`: if the credential is `ausente`, the user still has to connect the company with `configurar_credencial` (CNPJ and the Emissor Nacional password, typed by the user).

## Notes

- Before `emitir_nota`, confirm customer, amount and description with the user, and pass a unique `referencia` per invoice: repeating the same `referencia` never issues twice.
- After `emitir_nota`, poll `acompanhar_pedido` until `autorizada`, `recusada` or `incerta`, waiting `proxima_consulta_ms` between calls.
- Docs: https://emissaoja.com.br/docs/ · MCP guide: https://emissaoja.com.br/emitir-nota-fiscal-por-ia
