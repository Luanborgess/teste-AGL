# AgroLogistic

MVP SaaS para operação agrícola, logística, financeiro, estoque e relatórios.

## Como rodar

```bash
npm install
npm run prisma:generate
npm run db:push
npm run dev
```

Abra `http://localhost:3000`.

## Rotas

- `/` landing page premium.
- `/login` login simples do MVP.
- `/onboarding` questionário operacional.
- `/dashboard` painel principal com dados reais.
- `/financeiro` receitas, despesas, saldo e comparação com custos estimados.
- `/estoque` entradas, saídas e saldo, com bloqueio para saída maior que o saldo.
- `/logistica` embarques, volume, status e frete.
- `/relatorios` relatório operacional e exportação PDF.

## Cálculos

As funções puras ficam em `src/lib/calculations.ts`.

O dashboard usa:

- produção estimada = área produtiva x produtividade por hectare
- receita estimada = produção x preço por saca/unidade
- lucro estimado = receita - custos
- margem = lucro / receita
- cargas necessárias = produção / capacidade por carga
- meses para escoar = produção / volume mensal
- custo por hectare = custos / área produtiva
- custo por saca = custos / produção
- estoque atual = estoque inicial + entradas - saídas

Se faltar dado operacional, a interface mostra: `Dados insuficientes. Complete seu perfil operacional.`

## Cenário Mais Soja

1. Entre em `/login`.
2. Acesse `/onboarding`.
3. Clique em `Preencher cenário Mais Soja`.
4. Salve e abra `/dashboard`.

Resultados esperados:

- Produção: 51.000 sacas
- Receita: R$ 6.375.000,00
- Lucro: R$ 1.575.000,00
- Margem: 24,7%
- Cargas necessárias: 102
- Meses para escoar: 5,1
- Custo por hectare: R$ 5.647,06
- Custo por saca: R$ 94,12

## Banco

SQLite local via Prisma. A URL padrão está em `.env.example`:

```env
DATABASE_URL="file:./dev.db"
```

Para migrar futuramente para PostgreSQL, ajuste o `provider` e a `DATABASE_URL` no Prisma.
