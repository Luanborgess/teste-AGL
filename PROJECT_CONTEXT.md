# PROJECT_CONTEXT

## Produto

AgroLogistic e um MVP SaaS agricola em portugues brasileiro. O fluxo principal e:

`landing -> login simples -> onboarding operacional -> dashboard -> modulos`.

O questionario operacional e a fonte de verdade dos calculos. O produto nao usa `Math.random` nem mocks como dados reais.

## Inspiracao visual

A referencia do Replit foi usada apenas como inspiracao de UX:

- sidebar fixa com marca AgroLogistic e modulos ERP;
- tema escuro profissional com acentos verdes;
- dashboard com cards de metricas, graficos e listas;
- hierarquia visual de SaaS agricola;
- modulos de financeiro, estoque, logistica e relatorios.

Nenhum codigo externo foi copiado.

## Arquitetura

- Next.js App Router
- TypeScript
- TailwindCSS v4
- shadcn/ui
- Prisma ORM
- SQLite local
- Zod
- React Hook Form
- Recharts
- jsPDF

Principais pastas:

- `src/app`: rotas e API routes.
- `src/components/layout`: shell, sidebar e tema.
- `src/components/dashboard`: cards e blocos de dashboard.
- `src/components/forms`: formularios client-side legados/compartilhados.
- `src/components/financeiro`: modulo financeiro interativo.
- `src/components/producao`: modulo de producao agricola interativo.
- `src/components/estoque`: modulo de estoque agricola interativo.
- `src/components/funcionarios`: modulo de funcionarios, presenca e logistica de equipe.
- `src/components/charts`: graficos.
- `src/lib`: calculos, validacoes, Prisma, auth e formatadores.
- `prisma/schema.prisma`: modelos minimos.

## Modelos

- `User`
- `FarmProfile`
- `FinancialEntry`
- `Transaction`
- `StockMovement`
- `InventoryItem`
- `InventoryMovement`
- `Shipment`
- `Crop`
- `Employee`
- `EmployeeAttendance`
- `TeamAssignment`
- `TeamAssignmentMember`
- `EmployeeSalaryHistory`
- `Machine`
- `MachineMaintenance`
- `MachineFuelLog`
- `MachineHourMeterLog`

`FinancialEntry` foi mantido para compatibilidade com dashboard e fluxo financeiro antigo. O novo modulo `/financeiro` usa `Transaction`.

`StockMovement` foi mantido para compatibilidade com dashboard/calculos legados de estoque de safra. O novo modulo `/estoque` usa `InventoryItem` e `InventoryMovement` para controlar o inventario geral da fazenda.

O modulo `/funcionarios` usa models proprios para equipe operacional, presenca, alocacao em atividades e historico salarial. Todos pertencem a `User` por `userId` obrigatorio.

O modulo `/maquinario` usa models proprios para frota/equipamentos, manutencoes, consumo de combustivel e historico de horimetro. Todos pertencem a `User` por `userId` obrigatorio.

## Modulo de producao agricola

O modulo `/producao` foi implementado como CRUD real de cultivos, usando Prisma e dados persistidos no banco. A pagina usa o shell visual existente do AgroLogistic e componentes dedicados em `src/components/producao`.

A implementacao segue o padrao do modulo financeiro: a pagina server-side carrega os cultivos reais do usuario atual, serializa as datas para o componente client-side e o dashboard client-side faz filtros, ordenacao, paginacao, grafico, modais e atualizacao apos mutacoes via route handlers.

Funcionalidades implementadas:

- cards de resumo para Cultivos ativos, Area plantada total, Producao esperada, Producao colhida e Progresso da safra;
- grafico de Producao por Cultivo com Recharts, comparando producao esperada e colhida;
- tabela com cultivo, variedade, area, plantio, colheita prevista, producao esperada, producao colhida, status e acoes;
- pesquisa por cultivo/variedade;
- filtro por status;
- ordenacao por plantio, colheita prevista, producao e area;
- paginacao;
- estado vazio e estado sem resultados;
- modal para criar, editar e registrar producao colhida;
- modal de detalhes;
- dialog de confirmacao para excluir;
- exportacao CSV;
- loading skeleton e estado de erro ao carregar;
- atualizacao client-side apos criar, editar ou excluir, mantendo filtros quando possivel.

Modelo usado:

- `Crop`
- `CropStatus`: `PLANTADO`, `EM_CRESCIMENTO`, `COLHIDO`, `CANCELADO`

Campos principais do modelo:

- `name`
- `variety`
- `plantedArea`
- `plantingDate`
- `expectedHarvestDate`
- `expectedProduction`
- `harvestedProduction`
- `status`
- `notes`
- `userId`

Os cultivos pertencem a um `User` por `userId` obrigatorio. As APIs sempre usam o usuario atual via `requireProfile()` e filtram por `userId`, evitando mistura de dados entre usuarios.

Endpoints criados:

- `GET /api/producao/crops`
- `POST /api/producao/crops`
- `GET /api/producao/crops/[id]`
- `PUT /api/producao/crops/[id]`
- `DELETE /api/producao/crops/[id]`

Componentes criados:

- `src/components/producao/producao-dashboard.tsx`
- `src/components/producao/production-summary-cards.tsx`
- `src/components/producao/production-chart.tsx`
- `src/components/producao/crops-table.tsx`
- `src/components/producao/crop-form-modal.tsx`
- `src/components/producao/crop-details-modal.tsx`
- `src/components/producao/crop-delete-dialog.tsx`
- `src/components/producao/crop-filters.tsx`
- `src/components/producao/empty-production-state.tsx`

Validacao:

- Zod em `src/lib/validations.ts`;
- React Hook Form no modal;
- cultivo obrigatorio;
- area plantada maior que zero;
- producao esperada e colhida iguais ou maiores que zero;
- status restrito a `PLANTADO`, `EM_CRESCIMENTO`, `COLHIDO` ou `CANCELADO`;
- data prevista de colheita nao pode ser anterior a data de plantio;
- quando a producao colhida passa da esperada, o modal exibe alerta, mas permite salvar porque pode haver safra acima da previsao.

Calculos:

- funcoes puras em `src/lib/production-calculations.ts`;
- `Cultivos ativos` conta cultivos com status `PLANTADO` ou `EM_CRESCIMENTO`;
- `Area plantada total` soma `plantedArea`;
- `Producao esperada` soma `expectedProduction`;
- `Producao colhida` soma `harvestedProduction`;
- `Progresso da safra` usa `producao colhida / producao esperada`, limitado a 100% nos indicadores;
- grafico usa dados reais dos cultivos, com barras para producao esperada e producao colhida;
- detalhes calculam dias ate colheita, produtividade esperada por hectare e produtividade colhida por hectare.

Integracoes:

- a sidebar ganhou link para `/producao`;
- o dashboard principal ganhou um botao leve para abrir o modulo de Producao;
- `TESTING.md` foi atualizado com o cenario manual de Soja/BRS 1010.

Validacoes executadas apos a implementacao:

- `npm run prisma:generate`
- `npm run db:push`
- `npm run lint`
- `npm run build`

Teste funcional local executado:

- login com sessao real;
- criacao de cultivo Soja/BRS 1010;
- edicao para `20.000` sacas colhidas e status `EM_CRESCIMENTO`, resultando em progresso de `39,2%`;
- edicao para `51.000` sacas colhidas e status `COLHIDO`, resultando em progresso de `100%`;
- exclusao do cultivo;
- verificacao visual no navegador interno de `/producao`, com titulo, botao Novo cultivo, cards renderizados e zero erros de console apos ajuste do `ThemeToggle`.

Observacao de ambiente:

- embora o pedido tenha mencionado Next.js 15 e PostgreSQL, o projeto atual esta em Next.js `16.2.6` e SQLite local via Prisma. A implementacao respeitou o stack real do repositorio.

## Modulo de estoque agricola

O modulo `/estoque` foi substituido por um CRUD real de inventario geral da fazenda, sem mocks no fluxo principal. O antigo `StockMovement` continua no schema para compatibilidade com dashboard/calculos legados, mas o modulo novo usa models proprios de inventario.

Funcionalidades implementadas:

- cards de resumo para Total de Itens, Alertas de Estoque, Valor Total em Estoque, Itens Vencendo e Movimentacoes do Mes;
- tabela moderna com TanStack Table, pesquisa, filtros por categoria/situacao, ordenacao, paginacao e acoes por item;
- cadastro, edicao, exclusao e detalhes de itens;
- movimentacoes reais de Entrada, Saida, Consumo, Perda e Ajuste;
- bloqueio de saida/consumo/perda acima do saldo;
- historico de movimentacoes com filtros por produto, tipo e periodo;
- painel de alertas automaticos;
- geracao de QR Code apontando para `/estoque/[id]`;
- pagina simples de detalhes em `/estoque/[id]`;
- exportacao CSV do inventario filtrado.

Models usados:

- `InventoryItem`
- `InventoryMovement`
- `InventoryCategory`
- `InventoryMovementType`

Os itens e movimentacoes pertencem a um `User` por `userId` obrigatorio. As APIs usam `requireProfile()` e filtram por `userId`, evitando mistura de dados entre usuarios.

Endpoints criados:

- `GET /api/estoque/items`
- `POST /api/estoque/items`
- `GET /api/estoque/items/[id]`
- `PUT /api/estoque/items/[id]`
- `DELETE /api/estoque/items/[id]`
- `GET /api/estoque/movements`
- `POST /api/estoque/movements`
- `GET /api/estoque/items/[id]/movements`

Validacao:

- Zod em `src/lib/validations.ts`;
- React Hook Form nos modais;
- nome, categoria e unidade obrigatorios;
- quantidade, estoque minimo e custo unitario iguais ou maiores que zero;
- data de validade opcional com validacao de data;
- motivo obrigatorio para perda e ajuste.

Calculos:

- funcoes puras em `src/lib/inventory-calculations.ts`;
- `Valor Total em Estoque` soma `quantidade atual x custo unitario`;
- situacao e calculada por validade primeiro, depois saldo/minimo;
- alertas usam somente itens e movimentacoes reais cadastrados;
- movimentacao calcula `balanceAfter` e atualiza o saldo do item em transacao Prisma.

Validacoes executadas apos a implementacao:

- `npm run prisma:generate`
- `npm run db:push`
- `npm run lint`
- `npm run build`

Teste funcional local executado:

- renderizacao visual de `/estoque` no navegador interno sem overlay do Next e sem erros de console;
- criacao do item Fertilizante NPK 20-05-20 via API autenticada;
- consumo de 750 kg, resultando em saldo de 250 kg;
- tentativa de saida de 500 kg bloqueada com HTTP 422;
- entrada de 500 kg com custo unitario R$ 4,80, resultando em saldo de 750 kg;
- verificacao de historico com 2 movimentacoes e persistencia apos reload da rota.

## Modulo financeiro

O modulo `/financeiro` foi implementado como CRUD real de transacoes financeiras, usando Prisma e dados persistidos no banco. A pagina usa o shell visual existente do AgroLogistic e componentes dedicados em `src/components/financeiro`.

Funcionalidades implementadas:

- cards de resumo para Receita Total, Despesas Totais, Investimentos e Lucro Liquido;
- grafico mensal com Recharts alimentado pelas transacoes reais;
- tabela com data, descricao, categoria, tipo, valor e acoes;
- pesquisa por descricao/categoria;
- filtros por tipo e categoria;
- ordenacao por data e valor;
- paginacao;
- estado vazio e estado sem resultados;
- modal para criar, visualizar e editar transacoes;
- dialog de confirmacao para excluir;
- exportacao CSV;
- atualizacao client-side apos criar, editar ou excluir, mantendo filtros quando possivel.

Modelo usado:

- `Transaction`
- `TransactionType`: `RECEITA`, `DESPESA`, `INVESTIMENTO`

As transacoes pertencem a um `User` por `userId`. As APIs sempre usam o usuario atual via `requireProfile()` e filtram por `userId`, evitando mistura de dados entre usuarios.

Endpoints criados:

- `GET /api/financeiro/transactions`
- `POST /api/financeiro/transactions`
- `GET /api/financeiro/transactions/[id]`
- `PUT /api/financeiro/transactions/[id]`
- `DELETE /api/financeiro/transactions/[id]`

Validacao:

- Zod em `src/lib/validations.ts`;
- React Hook Form no modal;
- descricao minima de 3 caracteres;
- valor maior que zero;
- categoria obrigatoria;
- tipo restrito a `RECEITA`, `DESPESA` ou `INVESTIMENTO`;
- data valida.

Calculos:

- funcoes puras em `src/lib/financeiro.ts`;
- `Receita Total` soma `RECEITA`;
- `Despesas Totais` soma `DESPESA`;
- `Investimentos` soma `INVESTIMENTO`;
- `Lucro Liquido` usa a regra operacional `RECEITA - DESPESA`;
- investimentos sao acompanhados separadamente e nao abatem o lucro operacional.

Formatadores:

- `formatCurrencyBRL(value)`;
- `formatDateBR(date)`;
- `parseCurrencyInput(value)`.

Validacoes executadas apos a implementacao:

- `npm run prisma:generate`
- `npm run db:push`
- `npm run lint`
- `npm run build`

Tambem foi feito teste funcional local com sessao real: login, renderizacao de `/financeiro`, criacao, edicao, leitura e exclusao de transacao via API. A verificacao visual automatizada por navegador nao foi concluida porque `agent-browser`/Playwright nao estavam disponiveis no ambiente.

## Modulo de logistica de funcionarios

O modulo `/funcionarios` foi implementado como uma pagina completa para gestao de equipe, escala, presenca, folha salarial e logistica operacional da fazenda. A pagina segue o visual premium dark existente do AgroLogistic, usando o `AppShell`, cards escuros, acentos verdes, badges coloridas, modais responsivos e componentes dedicados em `src/components/funcionarios`.

A implementacao segue o padrao dos modulos reais ja existentes: a pagina server-side carrega dados reais do usuario atual com Prisma, serializa datas para um dashboard client-side, e as mutacoes acontecem via route handlers com atualizacao client-side apos salvar/excluir. Nao ha mocks no fluxo principal, nao ha `Math.random`, e os dados sao filtrados por `userId` para evitar mistura entre usuarios.

Funcionalidades implementadas:

- rota `/funcionarios`;
- header com acoes para `Novo Funcionario`, `Nova escala`, `Registrar presenca` e `Alocar equipe`;
- cards de resumo para Total de Funcionarios, Funcionarios Ativos, Folha de Pagamento, Equipes em Campo, Custo Diario Estimado e Ausencias Hoje;
- busca instantanea por nome, CPF, telefone ou cargo;
- filtros por status, setor, cargo e equipe;
- ordenacao por nome, salario, admissao e status;
- lista em cards premium com iniciais, cargo, setor, salario, telefone, admissao, status, equipe atual e ultima presenca;
- acoes por funcionario para visualizar, editar, excluir, ativar/desativar, registrar presenca e alocar em atividade;
- modal de cadastro/edicao com React Hook Form e Zod;
- mascara de CPF e telefone;
- validacao de nome, CPF, telefone, cargo, salario, admissao, e-mail opcional e status;
- modal de detalhes com dados pessoais, cargo/setor, salario atual, status, admissao, historico de presenca, alocacoes recentes, observacoes, ultima atualizacao e historico salarial;
- sistema simples de presenca com PRESENTE, AUSENTE, ATRASADO e JUSTIFICADO;
- aba `Logistica de Equipe` com criacao, edicao e exclusao de alocacoes/escala;
- alocacao com atividade, local/talhao, data, horarios, funcionarios selecionados, transporte, veiculo, motorista, observacoes e status;
- status de alocacao PLANEJADA, EM_CAMPO, CONCLUIDA, CANCELADA e ATRASADA;
- historico salarial simples: ao alterar salario, registra salario anterior, novo salario, data e motivo opcional;
- estados de loading, erro, vazio, sem resultado, salvando, excluindo e confirmacao de delete;
- atualizacao automatica dos cards, lista, presenca, alocacoes e folha apos mutacoes.

Models usados:

- `Employee`
- `EmployeeAttendance`
- `TeamAssignment`
- `TeamAssignmentMember`
- `EmployeeSalaryHistory`
- `EmployeeStatus`: `ATIVO`, `INATIVO`, `AFASTADO`, `DEMITIDO`
- `AttendanceStatus`: `PRESENTE`, `AUSENTE`, `ATRASADO`, `JUSTIFICADO`
- `TeamAssignmentStatus`: `PLANEJADA`, `EM_CAMPO`, `CONCLUIDA`, `CANCELADA`, `ATRASADA`

Endpoints criados:

- `GET /api/funcionarios/employees`
- `POST /api/funcionarios/employees`
- `GET /api/funcionarios/employees/[id]`
- `PUT /api/funcionarios/employees/[id]`
- `DELETE /api/funcionarios/employees/[id]`
- `GET /api/funcionarios/attendance`
- `POST /api/funcionarios/attendance`
- `GET /api/funcionarios/assignments`
- `POST /api/funcionarios/assignments`
- `PUT /api/funcionarios/assignments/[id]`
- `DELETE /api/funcionarios/assignments/[id]`

Validacao:

- Zod em `src/lib/validations.ts`;
- React Hook Form nos modais;
- CPF obrigatorio e unico no banco;
- presenca usa upsert por funcionario/data para evitar duplicidade do ponto do mesmo dia;
- alocacoes validam que todos os funcionarios selecionados pertencem ao usuario atual;
- APIs usam `requireProfile()` e sempre filtram por `userId`.

Calculos:

- funcoes puras em `src/lib/employee-calculations.ts`;
- `Total de Funcionarios` conta todos os funcionarios cadastrados;
- `Funcionarios Ativos` conta status `ATIVO`;
- `Folha de Pagamento` soma salarios de funcionarios ativos;
- `Equipes em Campo` conta alocacoes de hoje com status `EM_CAMPO`;
- `Custo Diario Estimado` soma `salario / 30` dos funcionarios alocados hoje, sem duplicar funcionario repetido;
- `Ausencias Hoje` conta presencas de hoje com status `AUSENTE`;
- tambem ha helpers para ultima presenca, equipe atual, agrupamentos por setor/status e labels de status.

Formatadores:

- `formatCPF(value)`;
- `formatPhoneBR(value)`;
- `getInitials(value)`;
- reutiliza `formatCurrencyBRL(value)` e `formatDateBR(value)`.

Componentes criados:

- `src/components/funcionarios/funcionarios-dashboard.tsx`
- `src/components/funcionarios/employee-summary-cards.tsx`
- `src/components/funcionarios/employee-filters.tsx`
- `src/components/funcionarios/employee-card.tsx`
- `src/components/funcionarios/employee-form-modal.tsx`
- `src/components/funcionarios/employee-details-modal.tsx`
- `src/components/funcionarios/employee-delete-dialog.tsx`
- `src/components/funcionarios/attendance-modal.tsx`
- `src/components/funcionarios/attendance-table.tsx`
- `src/components/funcionarios/team-assignment-modal.tsx`
- `src/components/funcionarios/team-assignments-list.tsx`
- `src/components/funcionarios/empty-employees-state.tsx`

Integracoes:

- a sidebar ganhou link para `/funcionarios`;
- `TESTING.md` ganhou um roteiro manual especifico com Joao Silva, presenca, alocacao em Colheita, reajuste salarial e desativacao.

Validacoes executadas apos a implementacao:

- `npm run prisma:generate`
- `npm run db:push`
- `npm run lint`
- `npm run build`

Teste funcional local executado:

- dev server em `http://localhost:3000`;
- login com sessao real;
- abertura de `/funcionarios` com HTTP 200;
- criacao de funcionario Joao Silva via API autenticada;
- verificacao de Total 1, Ativos 1 e Folha R$ 3.500;
- registro de presenca PRESENTE para a data atual;
- criacao de alocacao Colheita / Talhao 1 com transporte Caminhonete e status `EM_CAMPO`;
- verificacao de Equipes em Campo 1 e Ausencias Hoje 0;
- edicao de salario para R$ 4.000 com historico salarial registrado;
- desativacao do funcionario, deixando Ativos 0 e Folha R$ 0;
- reload de `/funcionarios` com HTTP 200, confirmando persistencia;
- o registro de teste foi removido ao final para deixar o CPF do roteiro livre para testes manuais.

Observacao de ambiente:

- o pedido mencionou Next.js 15 e PostgreSQL, mas o projeto atual esta em Next.js `16.2.6` e SQLite local via Prisma. A implementacao respeitou o stack real do repositorio.
- `agent-browser` nao estava disponivel no PATH, entao a validacao manual foi feita via HTTP/API autenticada e logs do dev server, sem erros em `dev-funcionarios.err.log`.

## Modulo de maquinario agricola

O modulo `/maquinario` foi implementado como uma pagina completa para gestao de frota e equipamentos agricolas, usando Prisma e dados persistidos no banco. A pagina segue o visual premium dark existente do AgroLogistic, com `AppShell`, cards escuros, acentos verdes, badges coloridas, alertas em amarelo/vermelho, modais responsivos e componentes dedicados em `src/components/maquinario`.

A implementacao segue o padrao dos modulos reais ja existentes: a pagina server-side carrega maquinas, manutencoes e consumos reais do usuario atual, serializa datas para um dashboard client-side, e as mutacoes acontecem via route handlers com atualizacao client-side apos salvar/excluir. Nao ha mocks no fluxo principal, nao ha `Math.random`, e os dados sao filtrados por `userId` para evitar mistura entre usuarios.

Funcionalidades implementadas:

- rota `/maquinario`;
- header com acoes para `Nova Maquina`, `Registrar manutencao`, `Registrar consumo` e `Atualizar horimetro`;
- cards de resumo para Total de Maquinas, Operacionais, Manutencao/Reparo, Alertas de Revisao, Custo de Manutencao e Consumo Medio;
- banner de alerta de manutencao com revisoes vencidas e proximas nos proximos 15 dias;
- busca por nome, tipo, marca e modelo;
- filtros por status, tipo, marca, ano e manutencao pendente;
- ordenacao por nome, ano, horimetro, custo de manutencao e proxima manutencao;
- lista em cards premium com nome, tipo, marca, modelo, ano, identificacao, horimetro, consumo medio, custo operacional, proxima manutencao, custo de manutencao e status;
- acoes por maquina para visualizar detalhes, editar, excluir, alterar status, registrar manutencao, registrar consumo e atualizar horimetro;
- modal de cadastro/edicao com React Hook Form e Zod;
- modal de detalhes com dados operacionais, custos, observacoes, manutencoes recentes e consumos recentes;
- dialog de confirmacao para excluir maquina;
- modal de alteracao de status;
- registro real de manutencao preventiva, corretiva, revisao, troca de oleo, troca de peca, reparo emergencial e outro;
- historico de manutencao em tabela com busca, filtros por maquina/tipo/periodo, ordenacao por data/custo e paginacao;
- registro de consumo de combustivel com calculo automatico de `litros x valor por litro`;
- atualizacao de horimetro com bloqueio para valor menor que o horimetro atual;
- estados de loading, erro, vazio, sem resultado, salvando, excluindo e sucesso via toast;
- atualizacao automatica dos cards, lista, alertas, historicos e custos apos mutacoes.

Models usados:

- `Machine`
- `MachineMaintenance`
- `MachineFuelLog`
- `MachineHourMeterLog`
- `MachineStatus`: `OPERACIONAL`, `MANUTENCAO`, `EM_REPARO`, `PARADA`, `VENDIDA`
- `MachineType`: `TRATOR`, `COLHEITADEIRA`, `PULVERIZADOR`, `PLANTADEIRA`, `CAMINHAO`, `CARRETA`, `PA_CARREGADEIRA`, `IMPLEMENTO`, `IRRIGACAO`, `VEICULO_APOIO`, `OUTRO`
- `MaintenanceType`: `PREVENTIVA`, `CORRETIVA`, `REVISAO`, `TROCA_OLEO`, `TROCA_PECA`, `REPARO_EMERGENCIAL`, `OUTRO`

Todos os registros pertencem a um `User` por `userId` obrigatorio. As APIs usam `requireProfile()` e sempre filtram por `userId`, evitando mistura de dados entre usuarios.

Endpoints criados:

- `GET /api/maquinario/machines`
- `POST /api/maquinario/machines`
- `GET /api/maquinario/machines/[id]`
- `PUT /api/maquinario/machines/[id]`
- `DELETE /api/maquinario/machines/[id]`
- `GET /api/maquinario/maintenances`
- `POST /api/maquinario/maintenances`
- `GET /api/maquinario/fuel-logs`
- `POST /api/maquinario/fuel-logs`
- `POST /api/maquinario/hour-meter`

Validacao:

- Zod em `src/lib/validations.ts`;
- React Hook Form nos modais;
- nome, tipo, marca, ano e status obrigatorios para maquinas;
- ano entre 1900 e ano atual + 1;
- horimetro, consumo medio e custo operacional iguais ou maiores que zero;
- proxima manutencao opcional com validacao de data;
- manutencao valida maquina do usuario antes de salvar;
- consumo valida maquina do usuario antes de salvar;
- horimetro nao permite novo valor menor que o horimetro atual.

Calculos:

- funcoes puras em `src/lib/machine-calculations.ts`;
- `Total de Maquinas` conta todas as maquinas cadastradas;
- `Operacionais` conta status `OPERACIONAL`;
- `Manutencao/Reparo` conta status `MANUTENCAO` ou `EM_REPARO`;
- `Alertas de Revisao` conta maquinas com `nextMaintenance` vencida ou nos proximos 15 dias;
- revisao vencida usa `nextMaintenance < hoje`;
- revisao proxima usa `nextMaintenance` entre hoje e 15 dias;
- `Custo de Manutencao` soma `MachineMaintenance.cost`;
- `Custo de Combustivel` soma `MachineFuelLog.totalCost`;
- `Consumo Medio` calcula a media de `Machine.fuelConsumption`;
- custo real por hora usa `(custos de manutencao + custos de combustivel) / horimetro`, quando ha dados suficientes.

Formatadores:

- reutiliza `formatCurrencyBRL(value)` e `formatDateBR(value)`;
- adiciona `formatHours(value)`;
- adiciona `formatLiters(value)`;
- labels de status, tipo de maquina e tipo de manutencao ficam em `src/lib/machine-calculations.ts`.

Componentes criados:

- `src/components/maquinario/maquinario-dashboard.tsx`
- `src/components/maquinario/machine-summary-cards.tsx`
- `src/components/maquinario/machine-maintenance-alert.tsx`
- `src/components/maquinario/machine-filters.tsx`
- `src/components/maquinario/machine-card.tsx`
- `src/components/maquinario/machine-form-modal.tsx`
- `src/components/maquinario/machine-details-modal.tsx`
- `src/components/maquinario/machine-delete-dialog.tsx`
- `src/components/maquinario/machine-status-modal.tsx`
- `src/components/maquinario/maintenance-form-modal.tsx`
- `src/components/maquinario/maintenance-history-table.tsx`
- `src/components/maquinario/fuel-log-modal.tsx`
- `src/components/maquinario/hour-meter-modal.tsx`
- `src/components/maquinario/empty-machines-state.tsx`

Integracoes:

- a sidebar ganhou link para `/maquinario`;
- `TESTING.md` ganhou um roteiro manual especifico com Trator John Deere 6110J, manutencao preventiva, consumo, horimetro e alteracao de status.

Validacoes executadas apos a implementacao:

- `npm run prisma:generate`
- `npm run db:push`
- `npm run lint`
- `npm run build`

Teste funcional local executado:

- dev server em `http://localhost:3000`;
- login com sessao real;
- abertura de `/maquinario` com HTTP 200;
- criacao da maquina Trator John Deere 6110J via API autenticada;
- verificacao de maquina cadastrada com status `OPERACIONAL`;
- registro de manutencao `PREVENTIVA` com custo R$ 2.500 e proxima revisao;
- registro de consumo de 100 litros a R$ 6,00/L, com custo total R$ 600;
- atualizacao de horimetro para 1220;
- tentativa de reduzir horimetro para 1000 bloqueada com HTTP 422;
- alteracao de status para `MANUTENCAO`;
- reload de `/maquinario` com HTTP 200, confirmando persistencia;
- o registro de teste foi removido ao final para deixar o roteiro livre para testes manuais;
- logs do dev server ficaram sem erros em `dev-maquinario.err.log`.

Observacao de ambiente:

- o pedido mencionou Next.js 15 e PostgreSQL, mas o projeto atual esta em Next.js `16.2.6` e SQLite local via Prisma. A implementacao respeitou o stack real do repositorio.
- o primeiro `prisma generate` falhou por lock do Windows/OneDrive no `query_engine-windows.dll.node` enquanto o dev server antigo estava ativo; apos parar o processo Node do workspace, `prisma generate` e `db:push` passaram normalmente.

## Landing page AgroLogistic

A rota inicial `/` foi substituida por uma landing page completa, premium e responsiva para o AgroLogistic, mantendo o fluxo real do produto:

`landing -> login simples -> onboarding operacional -> dashboard -> modulos`.

A implementacao foi inspirada apenas em estrutura de UX, hierarquia visual e organizacao de copy de referencias SaaS premium, sem copiar textos, imagens, codigo, identidade visual ou marca de terceiros.

Mensagem central da landing:

- `Veja o futuro da sua fazenda, nao apenas o passado.`

Submensagem:

- `Controle producao, estoque, logistica, financeiro, equipe e maquinario com dados reais da sua operacao agricola.`

SEO configurado na home:

- `title`: `AgroLogistic | ERP agricola inteligente`
- `description`: `Controle producao, estoque, logistica, financeiro, maquinario e equipe da fazenda com dados reais em um painel simples.`

Componentes criados:

- `src/components/landing/landing-navbar.tsx`
- `src/components/landing/landing-hero.tsx`
- `src/components/landing/landing-trust-signals.tsx`
- `src/components/landing/landing-feature-grid.tsx`
- `src/components/landing/landing-how-it-works.tsx`
- `src/components/landing/landing-operational-intelligence.tsx`
- `src/components/landing/landing-example-calculation.tsx`
- `src/components/landing/landing-security.tsx`
- `src/components/landing/landing-pricing.tsx`
- `src/components/landing/landing-faq.tsx`
- `src/components/landing/landing-final-cta.tsx`
- `src/components/landing/landing-footer.tsx`
- `src/components/landing/landing-section-heading.tsx`

Arquivo alterado:

- `src/app/page.tsx`

Secoes implementadas:

- navbar sticky premium com links para `Funcionalidades`, `Como funciona`, `Seguranca`, `Planos` e `FAQ`;
- menu mobile com sheet/hamburguer funcional;
- hero com badge `ERP agricola inteligente`, titulo forte, CTAs e microcopy;
- mockup visual do dashboard com exemplo fixo de operacao, claramente apresentado como exemplo, contendo producao prevista, receita estimada, estoque, cargas e grafico simples;
- faixa de sinais de confianca com segmentos agricolas, sem logos ou clientes falsos;
- grid de funcionalidades para fazenda/talhoes, producao, estoque, financeiro, logistica, maquinario, funcionarios e relatorios;
- secao `Configure em minutos. Decida com clareza todos os dias.` com 3 passos;
- secao de inteligencia operacional conectando talhao, estoque, maquina, equipe, carga, financeiro e dashboard;
- exemplo de calculo usando o cenario `850 ha`, `60 sacas/ha`, `R$ 125`, `R$ 4.800.000`, resultando em `51.000 sacas`, `R$ 6.375.000`, `R$ 1.575.000`, `102 cargas` e `5,1 meses`;
- secao de seguranca com alegacoes honestas: dados por usuario, banco real, controle operacional e estrutura pronta para evoluir;
- planos Starter, Pro e Enterprise sem preco fixo, usando `Sob consulta`, `Em breve`, `MVP` e `Fale conosco`;
- FAQ com accordion usando `details/summary`;
- CTA final;
- footer com links e texto `MVP em desenvolvimento`.

Conexoes com o app:

- `Comecar agora` aponta para `/login` quando nao ha sessao;
- `Comecar agora` aponta para `/onboarding` quando ha usuario logado sem perfil operacional;
- `Comecar agora` aponta para `/dashboard` quando ha usuario logado com perfil operacional;
- `Ver demonstracao` aponta para `/login` quando nao ha demo segura para usuario sem sessao;
- `Ver demonstracao` aponta para `/dashboard` quando ha usuario logado com perfil operacional;
- `Entrar` aponta para `/login`;
- links internos usam anchors: `#funcionalidades`, `#como-funciona`, `#seguranca`, `#planos`, `#faq`.

Cuidados adotados:

- nao foram alterados `/login`, `/onboarding`, `/dashboard` ou modulos internos;
- nao foram criados mocks no fluxo principal do app;
- os numeros exibidos na landing foram rotulados como exemplo de operacao/calculo, nao como dados reais do usuario;
- nao foram inventados clientes, logos, certificacoes, integracoes, IA avancada ou promessas de infraestrutura nao implementadas;
- visual segue o padrao dark premium do AgroLogistic, com acentos verdes, cards, glassmorphism leve, bordas sutis, responsividade e lucide-react;
- a landing usa Server Component em `src/app/page.tsx` para detectar o usuario atual via `getCurrentUser()` e passar CTAs corretos para componentes client-side quando necessario.

Validacoes executadas apos a implementacao:

- `npm run lint`
- `npm run build`

Teste local executado:

- `/` retornou HTTP 200 e conteudo esperado da landing;
- `/login` retornou HTTP 200;
- `/onboarding` sem sessao redirecionou com HTTP 307 para `/login`;
- `/dashboard` sem sessao redirecionou com HTTP 307 para `/login`;
- checagem de conteudo confirmou hero, funcionalidades, como funciona, seguranca, planos, FAQ e CTA final renderizados;
- nao foi detectado overlay de erro do Next.js via HTML retornado.

Observacao de ambiente:

- `agent-browser` e Playwright nao estavam disponiveis no ambiente, entao nao foi possivel gerar screenshot automatizado da landing. A validacao foi feita por lint, build e HTTP local.
- `git` nao estava disponivel no PATH do ambiente, entao nao foi possivel consultar `git status`/`git diff` ao final.

## Producao real

Ainda faltam autenticacao robusta, controle multiusuario/multiempresa, permissoes, auditoria, testes automatizados com runner dedicado, migrations versionadas para producao, observabilidade, backups e storage externo para PDFs.

## Modulo de clima agricola inteligente

O modulo `/clima` foi implementado como uma pagina completa para previsao climatica agricola, risco operacional, alertas e recomendacoes inteligentes. A pagina usa dados reais da Open-Meteo com base na fazenda cadastrada em `/fazenda`, sem mocks, sem `Math.random` e sem expor chave no frontend.

Fonte climatica:

- Open-Meteo Forecast API para clima atual, previsao diaria, chuva, vento, temperatura, umidade calculada por hora e UV;
- Open-Meteo Geocoding API para resolver cidade/estado da fazenda;
- a API usada nao exige chave para o fluxo atual.

Prioridade de localizacao:

1. coordenadas da fazenda, quando informadas;
2. cidade/estado da fazenda via geocoding;
3. coordenadas de talhao como fallback tecnico;
4. estado vazio pedindo completar localizacao quando nao for possivel resolver.

Funcionalidades implementadas:

- rota `/clima`;
- header com acoes para atualizar clima, editar localizacao e ver fazenda;
- card de clima atual com fazenda, localizacao, temperatura, sensacao termica, condicao, umidade, vento, rajadas, precipitacao, UV quando disponivel e atualizacao;
- cards de resumo para temperatura, chuva prevista, umidade, vento, risco agricola e melhor janela operacional;
- previsao dos proximos dias em cards responsivos;
- graficos Recharts de temperatura, chuva, umidade e vento;
- alertas climaticos deterministas baseados somente nos dados reais recebidos;
- recomendacoes para producao, estoque, logistica, maquinario e equipe;
- fallback local quando IA nao esta configurada ou falha;
- disclaimer discreto de apoio a decisao tecnica;
- estados para fazenda ausente, localizacao ausente, loading, erro de API, cache e refresh.

Models usados:

- `WeatherSnapshot`
- `WeatherRiskLevel`: `BAIXO`, `MEDIO`, `ALTO`

Endpoints criados:

- `GET /api/clima/current`
- `GET /api/clima/forecast`
- `POST /api/clima/refresh`
- `POST /api/clima/analyze`

Calculos:

- funcoes puras em `src/lib/weather-calculations.ts`;
- `Indice AgroClimatico` de 0 a 100 considera chuva, vento, temperatura, umidade e alertas operacionais;
- `0-39`: Ruim;
- `40-69`: Atencao;
- `70-100`: Favoravel;
- risco agricola deriva dos mesmos dados climaticos e dos alertas calculados.

Cache:

- snapshots climaticos sao salvos em `WeatherSnapshot`;
- dados com menos de 60 minutos sao reutilizados;
- `Atualizar clima` forca nova chamada real e salva novo snapshot.

IA:

- quando `AI_PROVIDER=openai` e `AI_API_KEY` estao configurados, `/api/clima/analyze` tenta gerar JSON estruturado via OpenAI;
- sem IA, usa regras locais deterministicas e avisa a interface;
- a IA recebe somente dados reais da fazenda, talhoes e previsao normalizada.

Validacoes executadas:

- `npm run prisma:generate`
- `npm run db:push`
- `npm run lint`
- `npm run build`

Teste funcional local executado:

- dev server em `http://localhost:3000`;
- `/api/clima/current` autenticado retornou `READY` para Mais Soja em Luis Eduardo Magalhaes/BA;
- `/clima` renderizou clima atual, previsao, graficos, alertas, score e recomendacoes;
- `/api/clima/refresh` fez nova busca real na Open-Meteo;
- `/api/clima/analyze` retornou fallback local quando IA nao estava configurada;
- snapshots foram salvos no banco com provider `open-meteo`, score e risco.
