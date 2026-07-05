# Desafio Técnico — Desenvolvedor Full Stack Pleno

## Sistema de Acompanhamento da Execução Orçamentária – SEPLAG

## Contexto

A Secretaria de Estado de Planejamento e Gestão (SEPLAG) acompanha diariamente a execução do orçamento dos órgãos estaduais para garantir que os recursos públicos sejam utilizados conforme o planejamento anual.

Os analistas precisam consultar rapidamente informações como:

* Dotação inicial;
* Dotação atualizada;
* Valor empenhado;
* Valor liquidado;
* Valor pago;
* Saldo disponível;
* Execução por programa e ação;
* Situação dos contratos vinculados.

Hoje essas informações encontram-se distribuídas em diferentes sistemas e planilhas.

Seu desafio será desenvolver um painel web que centralize essas informações, permitindo aos analistas acompanhar a execução orçamentária dos órgãos de forma simples e intuitiva.

---

# Entendendo o domínio

Este desafio usa conceitos comuns da execução orçamentária pública. **Você não precisa ser especialista em orçamento público** — tudo o que é necessário para desenvolver a aplicação está explicado nesta seção. Leia com atenção antes de começar: entender o domínio é parte do desafio e evita erros de modelagem.

## Uma analogia para começar

Pense no orçamento público como o planejamento financeiro de uma família, só que em escala de governo:

* No início do ano, a família define quanto pode gastar com cada coisa: mercado, escola, saúde. Isso é a **dotação inicial**.
* Ao longo do ano, surgem imprevistos: ela remaneja dinheiro de uma categoria para outra. Aumentos são **suplementações**, reduções são **anulações**. O valor final de cada categoria é a **dotação atualizada**.
* Quando a família fecha o pacote da escola do filho e assina o contrato, ela **se comprometeu** com aquele gasto, mesmo que ainda não tenha pagado nada. No governo, isso é o **empenho**.
* Quando o mês de aula acontece e o serviço foi de fato prestado, a dívida se torna real e conferida. No governo, isso é a **liquidação**.
* Quando o boleto é efetivamente pago, o dinheiro sai da conta. No governo, isso é o **pagamento**.

Ou seja: **empenhar** é reservar/comprometer, **liquidar** é confirmar que recebeu o que comprou, **pagar** é transferir o dinheiro. Por isso, em condições normais, o valor pago nunca é maior que o liquidado, que nunca é maior que o empenhado.

## O que é um "orçamento" nesta aplicação

Neste contexto, um registro de orçamento representa **uma autorização de gasto** para um órgão público em determinado ano, vinculada a um programa de governo, uma ação orçamentária, uma natureza de despesa e uma fonte de recurso.

Cada linha da tabela de orçamentos responde, na prática, a esta pergunta:

> "Quanto o órgão X pode gastar, no ano Y, com o tipo de despesa Z, dentro do programa W — e quanto disso já foi gasto?"

Exemplo simplificado:

```
Órgão: Secretaria de Estado de Educação
Programa: Educação pública de qualidade
Ação: Manutenção de unidades escolares
Natureza da despesa: Material de consumo
Fonte de recurso: Tesouro estadual
Ano: 2026
Dotação atualizada: R$ 10.000.000,00
Valor empenhado: R$ 7.000.000,00
Valor liquidado: R$ 5.500.000,00
Valor pago: R$ 4.800.000,00
Saldo: R$ 3.000.000,00
```

## Principais conceitos (glossário)

* **Órgão**: entidade pública responsável por executar parte do orçamento, como SEPLAG, SEEDUC (Educação), SES (Saúde), DETRAN ou Polícia Militar. Pense como "departamentos" do governo estadual. É o nível mais alto de agrupamento dos dados.
* **Unidade gestora**: unidade administrativa que executa o orçamento **dentro** de um órgão. Exemplo: dentro da SES (Secretaria de Saúde), o "Hospital Estadual X" e o "Fundo Estadual de Saúde" podem ser unidades gestoras diferentes. Relação: um órgão tem várias unidades gestoras.
* **Programa**: conjunto de iniciativas do governo com um objetivo comum, por exemplo "Atenção à saúde" ou "Infraestrutura rodoviária". É o "para quê" do gasto, em nível estratégico. Um programa pode ser executado por mais de um órgão.
* **Ação**: atividade, projeto ou operação específica **dentro de um programa**, por exemplo "Construção de unidades escolares" dentro do programa "Educação pública de qualidade". Relação: um programa tem várias ações.
* **Função e subfunção**: classificação da **área de atuação** do gasto, como saúde, educação, segurança pública ou transporte. Serve para responder "quanto o estado gasta com saúde?", somando gastos de todos os órgãos. A subfunção detalha a função (ex.: função "Saúde", subfunção "Atenção hospitalar").
* **Natureza da despesa**: classificação do **tipo** de gasto — pessoal, material de consumo, serviços de terceiros, obras, equipamentos. Responde "gastou com o quê?" (enquanto o programa responde "gastou para quê?").
* **Fonte de recurso**: **origem do dinheiro** utilizado — tesouro estadual (impostos arrecadados pelo próprio estado), convênios, transferências federais ou recursos próprios do órgão. Importa porque certos recursos só podem ser usados para certas finalidades.

Em resumo, cada registro orçamentário é uma combinação dessas classificações: **quem gasta** (órgão + unidade gestora), **para quê** (programa + ação + função/subfunção), **com o quê** (natureza da despesa) e **com dinheiro de onde** (fonte de recurso).

## Valores orçamentários

* **Dotação inicial**: valor aprovado no início do ano para aquele orçamento. É o "teto de gasto" original.
* **Suplementações**: acréscimos realizados ao longo do ano (o órgão recebeu autorização para gastar mais).
* **Anulações**: reduções realizadas ao longo do ano (parte do valor foi cortada ou remanejada para outro lugar).
* **Dotação atualizada**: o teto de gasto **vigente**, após somar suplementações e subtrair anulações. É sempre a dotação atualizada (e não a inicial) que serve de base para calcular saldo e percentual de execução.

```
dotacao_atualizada = dotacao_inicial + suplementacoes - anulacoes
```

## Etapas da execução da despesa

Toda despesa pública passa por três estágios, **nesta ordem**:

1. **Empenhado**: o órgão assumiu o compromisso e reservou o valor (assinou o contrato, emitiu a ordem de compra). O dinheiro ainda não saiu, mas já está comprometido e não pode ser usado para outra coisa.
2. **Liquidado**: o bem foi entregue ou o serviço foi prestado, e o governo conferiu e atestou. A dívida agora é real e exigível.
3. **Pago**: o dinheiro foi efetivamente transferido ao fornecedor.

E, derivado deles:

* **Saldo**: quanto do teto de gasto ainda está livre, ou seja, não comprometido.

```
saldo = dotacao_atualizada - valor_empenhado
percentual_execucao = (valor_empenhado / dotacao_atualizada) * 100
```

### Exemplo passo a passo

A SEEDUC tem uma dotação atualizada de **R$ 10 milhões** para "Manutenção de unidades escolares" em 2026:

1. Ela assina contratos de reforma no total de **R$ 7 milhões** → `valor_empenhado = 7.000.000` e `saldo = 3.000.000`. Ela ainda pode assumir novos compromissos de até R$ 3 milhões.
2. As empreiteiras concluem e entregam **R$ 5,5 milhões** em obras, conferidas pelos fiscais → `valor_liquidado = 5.500.000`. Há R$ 1,5 milhão empenhado mas ainda não entregue.
3. O estado paga **R$ 4,8 milhões** dos valores liquidados → `valor_pago = 4.800.000`. Há R$ 700 mil de obras já entregues aguardando pagamento.

Repare que cada etapa "afunila" a anterior:

```
valor_pago (4,8M) <= valor_liquidado (5,5M) <= valor_empenhado (7M) <= dotacao_atualizada (10M)
```

Essa é a regra em uma situação **ideal**. Porém, sistemas reais têm dados problemáticos — e a base fictícia deste desafio **deve conter inconsistências propositais** (ex.: pago maior que liquidado, saldo negativo, campos nulos) justamente para avaliar como a aplicação identifica, sinaliza e não quebra diante desses casos.

## Interpretando os números (visão do analista)

Para entender o que o usuário do painel procura ao olhar esses dados:

* **Percentual de execução baixo perto do fim do ano** → o órgão pode estar com dificuldade de executar; recursos podem ser perdidos ou remanejados.
* **Saldo negativo** → foi empenhado mais do que a dotação permite; erro grave que precisa ficar visível no painel.
* **Empenhado alto e pago baixo** → obras/serviços contratados mas entregas ou pagamentos atrasados.
* **Pago maior que liquidado** → inconsistência (pagou-se algo que não foi conferido); deve ser sinalizada, nunca escondida.
* **Orçamento sem nenhum empenho** → dinheiro parado, sem execução.

## Contratos vinculados

Contratos representam acordos firmados com fornecedores para execução de serviços, compra de materiais, obras ou fornecimento de bens. É pelo contrato que o compromisso assumido no empenho se materializa: o governo contrata a empresa Y para entregar o objeto Z por R$ N, dentro de um período de vigência.

Um contrato consome recursos de uma dotação específica, portanto deve estar **vinculado a um orçamento**. A modelagem esperada é:

* Um orçamento pode ter **vários** contratos (relação 1:N);
* Todo contrato pertence a **exatamente um** orçamento;
* Devem existir orçamentos **sem nenhum contrato** (situação comum na prática, ex.: despesas de pessoal).

Sobre a situação do contrato: **vigente** significa dentro do período de vigência; **vencido** significa que a data de fim passou sem encerramento formal; **encerrado** significa concluído/finalizado formalmente; **suspenso** significa temporariamente paralisado.

Exemplo:

```
Contrato: 045/2026
Fornecedor: Alpha Serviços de Engenharia Ltda.
Objeto: Reforma de escolas estaduais
Valor: R$ 2.400.000,00
Status: Vigente
Orçamento vinculado: Manutenção de unidades escolares
```

## O que os dados devem permitir analisar

A base fictícia deve permitir que o usuário responda perguntas como:

* Quais órgãos têm maior orçamento?
* Quais órgãos já executaram a maior parte da dotação?
* Quais programas concentram mais gastos?
* Qual valor foi empenhado, liquidado e pago?
* Quais orçamentos ainda têm saldo disponível?
* Quais orçamentos estão sem execução?
* Quais contratos estão vencidos, vigentes ou encerrados?
* Quais registros possuem inconsistências ou informações ausentes?
* Quais orçamentos já foram revisados por um analista?

---

# O que construir

## Backend

Desenvolver uma API REST utilizando **Laravel 12**.

Você poderá utilizar **MySQL ou PostgreSQL** como banco de dados.

O projeto deverá possuir migrations, seeders e factories para popular automaticamente a base com dados fictícios.

O repositório deverá conter um arquivo:

```
database/seeders/OrcamentoSeeder.php
```

que gere todos os dados automaticamente.

A escolha do banco deverá ser documentada no README.

---

## Autenticação

Criar autenticação utilizando JWT (Laravel Sanctum ou JWT Auth).

Usuário de teste

```
E-mail:
analista@seplag.rj.gov.br

Senha:
orcamento@2026
```

O token deverá conter o campo:

```
preferred_username
```

com o e-mail do usuário autenticado.

O token deverá possuir tempo de expiração. A estratégia de renovação (refresh token, renovação silenciosa ou redirecionamento para novo login ao expirar) fica a seu critério — documente a escolha no README do projeto.

---

# Endpoints obrigatórios

## POST /auth/login

Autenticação do usuário.

Retorna:

```
JWT
```

---

## GET /dashboard

Retorna os indicadores principais do painel.

Exemplo:

```
{
    "total_orgaos": 20,
    "total_contratos": 300,
    "orcamento_total": 428000000,
    "empenhado": 321000000,
    "liquidado": 287000000,
    "pago": 241000000,
    "saldo": 107000000,
    "percentual_execucao": 75.0
}
```

---

## GET /orgaos

Lista todos os órgãos.

Filtros:

* nome ou sigla (busca parcial)
* status (`ativo` ou `inativo`)
* paginação

---

## GET /orcamentos

Lista todos os registros orçamentários.

Filtros:

* órgão
* programa
* ação
* ano
* situação da execução
* percentual mínimo executado
* percentual máximo executado

Paginação obrigatória.

---

## GET /orcamentos/:id

Detalhamento completo do orçamento.

Exemplo:

* órgão
* unidade gestora
* programa
* ação
* natureza da despesa
* fonte de recurso
* dotação inicial
* suplementações
* anulações
* dotação atualizada
* empenhado
* liquidado
* pago
* saldo
* contratos vinculados

---

## GET /contratos

Lista contratos vinculados aos orçamentos.

Filtros:

* órgão
* situação
* fornecedor

---

## PATCH /orcamentos/:id/revisao

Marca um orçamento como revisado pelo analista autenticado.

Necessita JWT.

Registrar:

* usuário
* data
* observação

---

## GET /graficos

Retorna dados agregados para gráficos.

Exemplos:

* execução por órgão
* execução por programa
* empenhado x pago
* top 10 maiores contratos
* evolução mensal da execução

---

# Frontend

Desenvolver utilizando:

* React 19
* Vite
* TypeScript

---

## Esperamos encontrar

### Login

* autenticação
* armazenamento do JWT
* renovação/logout ao expirar
* proteção de rotas

---

### Dashboard

Consumindo:

```
GET /dashboard
```

Apresentando:

* Cards
* Indicadores
* Percentuais
* Barras de progresso
* Última atualização

---

### Tela de Orçamentos

Tabela com:

* filtros
* pesquisa
* paginação
* ordenação

---

### Detalhamento

Exibir todas as informações do orçamento.

Caso existam dados ausentes, a interface deverá informar claramente:

```
Informação não disponível.
```

ao invés de simplesmente deixar o campo vazio.

---

### Revisão

Botão:

```
Marcar como Revisado
```

Consumindo

```
PATCH /orcamentos/:id/revisao
```

Apresentar feedback visual.

---

### Dashboard Analítico

Criar gráficos utilizando sua biblioteca preferida.

Sugestões:

* Execução por órgão
* Empenhado x Pago
* Evolução Mensal
* Execução por Programa
* Ranking dos maiores contratos

---

## Responsividade

A aplicação deverá funcionar entre:

```
375px
```

e

```
1440px
```

pensando em analistas que utilizam notebooks corporativos e tablets durante reuniões.

---

# Stack

| Camada         | Tecnologia                                         |
| -------------- | -------------------------------------------------- |
| Backend        | Laravel 12                                         |
| Frontend       | React 19 + TypeScript                              |
| Banco          | MySQL ou PostgreSQL                                |
| Estilização    | Bootstrap 5 ou Tailwind CSS (justifique a escolha) |
| Infraestrutura | Docker                                             |

---

## Docker

O comando

```
docker compose up
```

deverá subir automaticamente:

* Backend
* Frontend
* Banco
* Executar migrations
* Executar seeders

Sem necessidade de configuração adicional.

---

# Base de Dados Fictícia

Para que você não perca tempo inventando nomes de órgãos, programas, ações e fornecedores, o repositório inclui o arquivo:

```
dados-referencia.json
```

com listas prontas de órgãos (com sigla e nome), programas, ações (já vinculadas aos seus programas), funções/subfunções, naturezas de despesa, fontes de recurso, exemplos de unidades gestoras e fornecedores fictícios.

**Use o JSON como base nos seus seeders/factories.** Não é obrigatório usar exatamente todos os itens do arquivo — ele existe para que você gere dados **coerentes** (ex.: ações combinando com seus programas, contratos com fornecedores plausíveis) sem perder tempo inventando nomes. O importante é respeitar as quantidades aproximadas pedidas abaixo e manter a consistência entre as entidades.

A aplicação deverá gerar automaticamente uma base fictícia com aproximadamente:

* 20 órgãos estaduais;
* 500 registros orçamentários;
* 300 contratos;
* 30 programas governamentais;
* 80 ações orçamentárias.

Os seeders/factories devem usar o `dados-referencia.json` como apoio para popular nomes, siglas, programas, ações, fornecedores, fontes de recurso e demais referências.

Os dados também devem incluir situações comuns em bases reais, como orçamento sem execução, saldo negativo, contrato vencido, orçamento sem contrato, registros revisados e registros com informações ausentes. A aplicação deve tratar esses cenários de forma clara, sem quebrar a interface nem esconder inconsistências.

---

# README

O projeto deverá conter documentação explicando:

* Como executar localmente
* Como utilizar Docker
* Como executar migrations
* Como executar seeders
* Estrutura do projeto
* Decisões arquiteturais
* Justificativa das bibliotecas utilizadas
* Credenciais de acesso
* Principais trade-offs
* Melhorias que seriam implementadas com mais tempo

---

# O que avaliamos

* Organização do projeto
* Arquitetura da aplicação
* Clean Code
* SOLID
* Componentização
* Padrões Laravel
* Consumo correto da API
* Tratamento de erros
* Qualidade da experiência do usuário
* Responsividade
* Estrutura do banco de dados
* Qualidade das consultas
* Performance
* Segurança da autenticação
* Documentação

---

# Diferenciais

Não são obrigatórios, mas agregam valor:

* Utilização de **React Query (TanStack Query)** para gerenciamento de estado servidor.
* Componentes com **shadcn/ui**.
* Testes unitários (PHPUnit/Pest).
* Testes de integração da API.
* Testes de componentes no React.
* Testes E2E com Playwright.
* Gráficos interativos (ApexCharts, Recharts ou Chart.js).
* Modo escuro (Dark Mode).
* Exportação dos dados em PDF e Excel.
* Filtros avançados com múltiplas combinações.
* Dashboard em tempo real utilizando WebSockets (Laravel Reverb).
* Deploy publicado (Vercel para o frontend e Railway/Render para o backend).

---

# Entrega

## Prazo

**7 dias corridos** a partir do recebimento deste desafio.

## Formato

* Repositório **único** (mono-repo) no GitHub, contendo `/backend`, `/frontend` e o `docker-compose.yml` na raiz.
* O repositório deverá ser **público**. Repositórios privados não serão avaliados.
* Registre a entrega, informando o link do repositório, em **https://subpep.rj.gov.br/suborc/entrega-desafio/** até o fim do prazo.

## Dúvidas

Dúvidas sobre o enunciado durante o desafio podem ser registradas no mesmo portal da entrega: **https://subpep.rj.gov.br/suborc/entrega-desafio/**. Interpretar requisitos faz parte do trabalho, mas preferimos esclarecer a deixar você travado.

## Histórico de commits

O histórico de commits **faz parte da avaliação**. Commits incrementais, com mensagens claras, que contam a evolução do projeto, são muito melhores do que um único commit com o projeto pronto.

## Uso de inteligência artificial

Ferramentas de IA (Copilot, ChatGPT, Claude etc.) **podem ser usadas livremente**, desde que você declare no README do projeto como as utilizou. Você deverá ser capaz de explicar e defender tecnicamente qualquer parte do código em conversa posterior — entender profundamente o que entregou é essencial.

---

Esse desafio simula um cenário próximo ao encontrado em equipes de desenvolvimento que atuam com sistemas de acompanhamento orçamentário no setor público, avaliando tanto a capacidade técnica quanto as decisões de arquitetura, organização do código e a experiência oferecida ao usuário final.
