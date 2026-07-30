# TESTING

## Validação obrigatória executada

Com o cenário:

- Fazenda: Mais Soja
- Responsável: Ícaro Almeida dos Santos Machado
- Tipo: Produtor rural
- Área total: 1.000 ha
- Área produtiva: 850 ha
- Cultura: Soja
- Produtividade: 60 sacas/ha
- Preço: R$ 125
- Custos: R$ 4.800.000
- Estoque inicial: 5.000 sacas
- Máquinas: 8
- Capacidade por carga: 500 sacas
- Volume mensal: 10.000 sacas

Os cálculos em `src/lib/calculations.ts` retornam:

- Produção: 51.000 sacas
- Receita: R$ 6.375.000,00
- Lucro: R$ 1.575.000,00
- Margem: 24,7%
- Cargas necessárias: 102
- Meses para escoar: 5,1
- Custo por hectare: R$ 5.647,06
- Custo por saca: R$ 94,12

## Comandos de verificação

```bash
npm run prisma:generate
npm run db:push
npm run build
```

## Teste manual recomendado

1. Rodar `npm run dev`.
2. Abrir `/login`.
3. Entrar com nome e e-mail.
4. Em `/onboarding`, clicar em `Preencher cenário Mais Soja`.
5. Salvar.
6. Conferir `/dashboard`.
7. Criar e editar um cultivo em `/producao`.
8. Criar um lançamento em `/financeiro`.
9. Criar entrada e saída em `/estoque`; tentar uma saída maior que o saldo.
10. Criar um embarque em `/logistica`.
11. Exportar PDF em `/relatorios`.

## Teste manual da producao agricola

1. Abrir `/producao`.
2. Criar cultivo:
   - cultivo: Soja
   - variedade: BRS 1010
   - area plantada: 850 ha
   - data de plantio: 2026-01-10
   - colheita prevista: 2026-05-20
   - producao esperada: 51.000 sacas
   - producao colhida: 0
   - status: PLANTADO
3. Verificar:
   - Cultivos ativos: 1
   - Area plantada total: 850 ha
   - Producao esperada: 51.000 sacas
   - Producao colhida: 0 sacas
   - Progresso: 0%
4. Editar o cultivo:
   - producao colhida: 20.000 sacas
   - status: EM_CRESCIMENTO
5. Verificar:
   - Producao colhida: 20.000 sacas
   - Progresso aproximado: 39,2%
   - Grafico atualizado
   - Tabela atualizada
6. Editar novamente:
   - producao colhida: 51.000 sacas
   - status: COLHIDO
7. Verificar:
   - Progresso: 100%
   - Status: COLHIDO
   - Apos refresh, os dados continuam porque sao salvos no banco via Prisma.

## Teste manual do financeiro

Regra adotada: `INVESTIMENTO` e acompanhado em card/grafico proprio e nao abate o lucro operacional. O card `Lucro Liquido` usa `RECEITA - DESPESA`.

1. Abrir `/financeiro`.
2. Criar receita:
   - descricao: Venda de soja
   - valor: R$ 100.000
   - categoria: Vendas
   - tipo: Receita
   - data: hoje
3. Criar despesa:
   - descricao: Fertilizantes
   - valor: R$ 35.000
   - categoria: Insumos
   - tipo: Despesa
   - data: hoje
4. Criar investimento:
   - descricao: Manutencao de maquina
   - valor: R$ 12.000
   - categoria: Maquinas
   - tipo: Investimento
   - data: hoje

Resultado esperado:

- Receita Total: R$ 100.000,00
- Despesas Totais: R$ 35.000,00
- Investimentos: R$ 12.000,00
- Lucro Liquido: R$ 65.000,00
- A tabela deve mostrar as 3 transacoes.
- O grafico mensal deve refletir os valores no mes usado.
- Pesquisa, filtro de tipo/categoria, ordenacao por data/valor e paginacao devem continuar funcionando apos criar, editar ou excluir.
- Apos refresh da pagina, os dados devem continuar porque sao salvos no banco via Prisma.

## Teste manual do estoque agricola

1. Abrir `/estoque`.

2. Criar item:
   - nome: Fertilizante NPK 20-05-20
   - categoria: Fertilizantes
   - unidade: kg
   - quantidade atual: 1000
   - estoque minimo: 300
   - custo unitario: R$ 4,50
   - fornecedor: AgroFornecedora
   - validade: daqui 90 dias
   - localizacao: Galpao 1

3. Verificar:
   - Total de Itens: 1
   - Valor Total: R$ 4.500,00
   - Situacao: OK

4. Registrar consumo:
   - quantidade: 750 kg
   - responsavel: Icaro
   - observacao: Aplicacao no talhao 1

5. Verificar:
   - quantidade atual: 250 kg
   - situacao: Critico
   - alertas de estoque: 1
   - historico contem a movimentacao
   - valor total atualizado: R$ 1.125,00

6. Tentar registrar saida de 500 kg.
   - Deve bloquear ou mostrar erro de saldo insuficiente.

7. Registrar entrada:
   - quantidade: 500 kg
   - custo unitario: R$ 4,80

8. Verificar:
   - quantidade atual: 750 kg
   - situacao: OK
   - historico atualizado
   - cards atualizados

9. Dar refresh.
   - Os dados devem continuar salvos porque sao persistidos via Prisma.

## Teste manual de logistica de funcionarios

1. Abrir `/funcionarios`.

2. Criar funcionario:
   - nome: Joao Silva
   - CPF: 123.456.789-09
   - telefone: (31) 99999-9999
   - email: joao@fazenda.com
   - cargo: Operador de Maquina
   - setor: Campo
   - salario: R$ 3.500
   - data de admissao: 2026-01-10
   - status: ATIVO

3. Verificar:
   - Total de Funcionarios: 1
   - Funcionarios Ativos: 1
   - Folha de Pagamento: R$ 3.500,00
   - funcionario aparece na lista

4. Registrar presenca:
   - funcionario: Joao Silva
   - status: PRESENTE
   - data: hoje

5. Criar alocacao:
   - atividade: Colheita
   - local: Talhao 1
   - data: hoje
   - funcionario: Joao Silva
   - transporte: Caminhonete
   - status: EM_CAMPO

6. Verificar:
   - Equipes em Campo: 1
   - Joao aparece alocado
   - ausencia hoje continua 0

7. Editar salario:
   - novo salario: R$ 4.000

8. Verificar:
   - Folha de Pagamento: R$ 4.000,00
   - historico salarial registrado nos detalhes do funcionario

9. Desativar funcionario.
   - Funcionarios Ativos deve virar 0
   - Folha de pagamento dos ativos deve virar R$ 0,00

10. Dar refresh.
   - Os dados devem continuar salvos porque sao persistidos via Prisma.

## Teste manual de maquinario agricola

1. Abrir `/maquinario`.

2. Criar maquina:
   - nome: Trator John Deere 6110J
   - tipo: TRATOR
   - marca: John Deere
   - modelo: 6110J
   - ano: 2022
   - identificacao: TR-001
   - horimetro: 1200
   - consumo medio: 12 litros/h
   - custo operacional: R$ 180/h
   - status: OPERACIONAL
   - proxima manutencao: daqui 10 dias

3. Verificar:
   - Total de Maquinas: 1
   - Operacionais: 1
   - Alertas de Revisao: 1
   - card da maquina aparece

4. Registrar manutencao:
   - tipo: PREVENTIVA
   - descricao: Revisao geral
   - custo: R$ 2.500
   - data realizada: hoje
   - proxima revisao: daqui 90 dias
   - pecas trocadas: filtros e oleo
   - responsavel: Joao

5. Verificar:
   - historico de manutencao atualizado
   - custo total de manutencao: R$ 2.500
   - alerta de revisao recalculado

6. Registrar consumo:
   - litros: 100
   - preco por litro: R$ 6,00
   - custo total esperado: R$ 600,00
   - horimetro: 1210

7. Verificar:
   - consumo salvo
   - custos atualizados, se exibidos

8. Atualizar horimetro:
   - novo horimetro: 1220

9. Verificar:
   - horimetro da maquina atualizado
   - nao permite valor menor que o atual

10. Alterar status para MANUTENCAO.
    - Operacionais deve virar 0
    - Manutencao/Reparo deve virar 1

11. Dar refresh.
    - Os dados devem continuar salvos porque sao persistidos via Prisma.

## Teste manual do modulo Fazenda

1. Abrir `/fazenda` sem fazenda cadastrada.

Resultado esperado:

- modal ou estado obrigatorio para cadastrar fazenda aparece.
- o dashboard da fazenda fica bloqueado ate o cadastro inicial.

2. Criar fazenda:
   - nome: Mais Soja
   - proprietario: Icaro Almeida dos Santos Machado
   - cidade: Luis Eduardo Magalhaes
   - estado: BA
   - area total: 1000 ha
   - tipo principal de producao: Soja
   - observacoes: Fazenda teste do AgroLogistic

3. Verificar:
   - botao `Nova Fazenda` nao existe
   - card principal mostra Mais Soja
   - area total: 1000 ha
   - talhoes: 0
   - area utilizada: 0 ha
   - area disponivel: 1000 ha

4. Criar talhao:
   - nome: Talhao 1
   - area: 350 ha
   - tipo de solo: Argiloso
   - cultura atual: Soja
   - produtividade esperada: 60 sacas/ha
   - status: ATIVO

5. Verificar:
   - talhoes: 1
   - area utilizada: 350 ha
   - area disponivel: 650 ha
   - cultura ativa: Soja
   - produtividade media: 60 sacas/ha

6. Criar segundo talhao:
   - nome: Talhao 2
   - area: 500 ha
   - tipo de solo: Misto
   - cultura atual: Milho
   - produtividade esperada: 120 sacas/ha
   - status: EM_PREPARO

7. Verificar:
   - talhoes: 2
   - area utilizada: 850 ha
   - area disponivel: 150 ha
   - culturas ativas: Soja e Milho
   - produtividade media: 90 sacas/ha

8. Tentar criar talhao de 300 ha.

Resultado esperado:

- sistema bloqueia com aviso de que a soma ultrapassa a area total da fazenda.

9. Editar fazenda.

Resultado esperado:

- os dados atualizam e nao cria nova fazenda.

10. Dar refresh.

Resultado esperado:

- fazenda e talhoes continuam salvos porque sao persistidos via Prisma.

## Teste manual do clima agricola inteligente

1. Criar ou confirmar fazenda em `/fazenda` com:
   - nome: Mais Soja
   - cidade: Luis Eduardo Magalhaes
   - estado: BA
   - cultura principal: Soja

2. Abrir `/clima`.

3. Verificar:
   - clima atual aparece
   - localizacao da fazenda aparece
   - previsao dos proximos dias aparece
   - graficos carregam
   - alertas sao calculados com base na previsao real
   - recomendacoes inteligentes aparecem

4. Clicar em `Atualizar clima`.

5. Verificar:
   - nova busca ocorre na Open-Meteo
   - cache no banco e atualizado
   - toast de sucesso aparece

6. Remover localizacao da fazenda ou testar fazenda sem localizacao valida.

7. Verificar:
   - sistema pede para completar localizacao da fazenda

8. Simular falha da IA removendo `AI_API_KEY` ou deixando `AI_PROVIDER` vazio.

9. Verificar:
   - recomendacoes locais continuam funcionando
   - pagina avisa que a avaliacao avancada por IA nao esta ativa

10. Dar refresh.

11. Verificar:
   - dados climaticos carregam do cache ou da API corretamente
