# Nexo360 – protótipo técnico de apoio ao PIM IV

**Situação do artefato:** código-fonte didático desenvolvido para demonstrar parte dos requisitos do PIM. Não é produto completo, auditado, testado de ponta a ponta ou implantado comercialmente. Não há dados reais ou pagamentos.

## Conteúdo

- `api/Nexo360.Api`: API REST ASP.NET Core .NET 10 com cadastro de cliente, login JWT, catálogo, cadastro de produto para gestor, pedido com múltiplos itens e consulta por titular.
- `banco/banco_nexo360.sql`: esquema SQL Server, restrições, índices, tipo tabular, procedure transacional e trigger de auditoria.
- `web`: portal React/Vite com consulta de produtos, registro/login de cliente, carrinho e pedidos.
- `mobile`: exemplo Flutter de catálogo com comunicação HTTP. Para executar em Android real, configurar HTTPS e endereço acessível pelo dispositivo.
- `.github/workflows/ci.yml`: exemplo de pipeline de compilação (não executada neste ambiente de criação).

## Pré-requisitos

.NET SDK 10, SQL Server 2022 ou Azure SQL, Node.js 22, npm e, se for executar o mobile, Flutter SDK. A autoria deste material não implica que essas ferramentas estejam instaladas no computador do estudante.

## Passo a passo de desenvolvimento local

1. Crie um SQL Server de desenvolvimento (opcionalmente `docker compose up -d` após definir `SA_PASSWORD`).
2. No SSMS ou sqlcmd, execute `banco/banco_nexo360.sql` em uma instância local **descartável**. O script cria banco e insere cinco produtos fictícios; executar novamente na mesma base sem limpar pode gerar conflito.
3. Defina `NEXO_CONNECTION` apontando para a base Nexo360, `NEXO_JWT_KEY` como uma chave aleatória forte de pelo menos 32 caracteres e `NEXO_WEB_ORIGIN=http://localhost:5173`. **Nunca publique esses segredos.**
4. Execute `dotnet run --project api/Nexo360.Api/Nexo360.Api.csproj --urls http://localhost:5082` para testes locais. Para produção, configure HTTPS com certificados válidos.
5. Execute `cd web`, depois `npm install` e `npm run dev`. Abra `http://localhost:5173`.
6. Cadastre uma conta fictícia de cliente na interface, faça login, selecione quantidades e envie um pedido de teste. A API baixa o estoque na mesma transação de gravação do pedido.
7. Para ver permissões administrativas, promova **somente uma conta de teste** a `GESTOR` via SQL de desenvolvimento e realize novo login. Não é fornecida conta administrativa fixa.
8. No aplicativo Flutter, depois de instalar Flutter: dentro de `mobile`, execute `flutter create .` para gerar os arquivos específicos de plataforma caso não existam, em seguida `flutter pub get` e `flutter run --dart-define=API_URL=http://10.0.2.2:5082` para emulador Android. Ajuste AndroidManifest e configuração de rede para desenvolvimento HTTP local; para ambiente real utilize HTTPS.

## Endpoints principais

| Método | URL | Uso |
|---|---|---|
| POST | /api/auth/cadastro | Criar conta de teste do perfil CLIENTE |
| POST | /api/auth/login | Emitir JWT |
| GET | /api/produtos | Consultar catálogo |
| GET | /api/produtos/{id} | Consultar um produto |
| POST | /api/produtos | Cadastrar produto (GESTOR) |
| POST | /api/pedidos | Criar pedido autenticado |
| GET | /api/pedidos/meus | Consultar pedidos do próprio cliente |
| GET | /api/admin/indicadores | Painel simples (GESTOR) |

JSON de `POST /api/pedidos`: `{"itens":[{"produtoId":1,"quantidade":2},{"produtoId":2,"quantidade":1}]}`. Envie `Authorization: Bearer <token>`.

## Limitações a declarar durante a apresentação

- Não foi possível compilar .NET, iniciar SQL Server nem executar o Flutter no ambiente de geração dos arquivos; teste essas partes na sua máquina antes da apresentação.
- O aplicativo mobile demonstra **somente o catálogo**; login, carrinho e histórico são parte da arquitetura proposta, não dessa demonstração mobile.
- Segurança de produção ainda requer proteção contra abuso, políticas de CORS específicas, gestão de certificados, revisão de dependências, testes de penetração e observabilidade centralizada.
- Fluxos administrativos avançados, exclusão, cancelamento de pedido, documentação OpenAPI, testes de integração e infraestrutura Azure são propostas documentadas e permanecem no backlog.
- `TrustServerCertificate=True` no `.env.example` é apenas uma conveniência para SQL Server local; produção deve validar certificados.
- O texto do PIM é um estudo de caso fictício com indicadores ilustrativos, não uma validação econômica real.
