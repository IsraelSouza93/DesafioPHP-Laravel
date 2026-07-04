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

Este desafio usa conceitos comuns da execução orçamentária pública. A pessoa candidata não precisa ser especialista em orçamento público, mas precisa representar os dados de forma coerente na aplicação.

## O que é um orçamento

Neste contexto, um orçamento representa uma autorização de gasto para um órgão público em determinado ano, vinculada a um programa de governo, uma ação orçamentária, uma natureza de despesa e uma fonte de recurso.

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

## Principais conceitos

* **Órgão**: entidade pública responsável por executar parte do orçamento, como SEPLAG, SEEDUC, SES, DETRAN ou Polícia Militar.
* **Unidade gestora**: unidade administrativa que executa o orçamento dentro de um órgão.
* **Programa**: conjunto de iniciativas do governo com um objetivo comum, por exemplo "Atenção à saúde" ou "Infraestrutura rodoviária".
* **Ação**: atividade, projeto ou operação específica dentro de um programa, por exemplo "Construção de unidades escolares".
* **Função e subfunção**: classificação da área de atuação do gasto, como saúde, educação, segurança pública ou transporte.
* **Natureza da despesa**: classificação do tipo de gasto, como pessoal, material de consumo, serviços de terceiros, obras ou equipamentos.
* **Fonte de recurso**: origem do dinheiro utilizado, como tesouro estadual, convênios, transferências federais ou recursos próprios.

## Valores orçamentários

* **Dotação inicial**: valor aprovado inicialmente para aquele orçamento no começo do exercício.
* **Suplementações**: acréscimos realizados ao orçamento ao longo do ano.
* **Anulações**: reduções realizadas no orçamento ao longo do ano.
* **Dotação atualizada**: valor final disponível após suplementações e anulações.

Uma forma simples de calcular:

```
dotacao_atualizada = dotacao_inicial + suplementacoes - anulacoes
```

## Etapas da execução da despesa

* **Empenhado**: valor reservado para uma despesa assumida pelo órgão. Indica que o governo se comprometeu com aquele gasto.
* **Liquidado**: valor correspondente a bens entregues ou serviços prestados e conferidos.
* **Pago**: valor efetivamente pago ao fornecedor.
* **Saldo**: valor ainda disponível em relação à dotação atualizada.

Uma forma simples de calcular:

```
saldo = dotacao_atualizada - valor_empenhado
percentual_execucao = (valor_empenhado / dotacao_atualizada) * 100
```

Em uma situação ideal, os valores costumam seguir esta ordem:

```
valor_pago <= valor_liquidado <= valor_empenhado <= dotacao_atualizada
```

Porém, a base fictícia deve conter também inconsistências propositais para avaliar como a aplicação lida com dados reais problemáticos.

## Contratos vinculados

Contratos representam acordos firmados com fornecedores para execução de serviços, compra de materiais, obras ou fornecimento de bens.

Um contrato deve estar vinculado a um orçamento quando ele consome recursos daquela dotação. Um orçamento pode ter vários contratos, um contrato deve estar associado a um orçamento, e também devem existir orçamentos sem contrato para simular situações reais.

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
    "total_orgaos": 12,
    "total_contratos": 186,
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

* nome
* secretaria
* status
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

A aplicação deverá conter aproximadamente:

## Órgãos

20 órgãos estaduais

Exemplos

* Casa Civil
* SEPLAG
* DER-RJ
* SEEDUC
* SES
* Polícia Civil
* Polícia Militar
* FAETEC
* DETRAN
* EMOP

---

## Orçamentos

500 registros.

Cada registro orçamentário deve representar uma combinação de ano, órgão, unidade gestora, programa, ação, natureza da despesa e fonte de recurso.

Campos sugeridos:

```
id

ano

orgao

unidade_gestora

programa

acao

funcao

subfuncao

natureza_despesa

fonte_recurso

dotacao_inicial

suplementacoes

anulacoes

dotacao_atualizada

valor_empenhado

valor_liquidado

valor_pago

saldo

percentual_execucao

status

revisado

revisado_por

data_revisao
```

Regras esperadas para geração dos valores:

* `dotacao_atualizada` deve ser calculada a partir de `dotacao_inicial`, `suplementacoes` e `anulacoes`.
* `saldo` deve ser calculado preferencialmente a partir de `dotacao_atualizada - valor_empenhado`.
* `percentual_execucao` deve representar o percentual empenhado sobre a dotação atualizada.
* `status` pode indicar situações como `sem_execucao`, `em_execucao`, `executado`, `saldo_negativo` ou `inconsistente`.
* Alguns registros devem conter valores nulos para testar o tratamento de dados ausentes na interface.

---

## Contratos

300 contratos.

Cada contrato deve representar uma contratação vinculada a um orçamento, com fornecedor, objeto, valor, período de vigência e situação.

Campos:

```
numero_contrato

objeto

fornecedor

valor

inicio_vigencia

fim_vigencia

status

orcamento_id
```

Status sugeridos:

* `vigente`
* `vencido`
* `encerrado`
* `suspenso`

---

## Programas

Gerar aproximadamente:

* 30 programas governamentais

Exemplos:

* Atenção à saúde
* Educação pública de qualidade
* Segurança cidadã
* Infraestrutura rodoviária
* Modernização da gestão pública

---

## Ações

Gerar aproximadamente:

* 80 ações orçamentárias

Exemplos:

* Manutenção de unidades escolares
* Aquisição de medicamentos
* Reforma de hospitais
* Conservação de rodovias
* Capacitação de servidores
* Modernização de sistemas corporativos

---

## Casos especiais

Os dados deverão conter propositalmente situações reais como:

* orçamento totalmente executado;
* orçamento sem nenhum empenho;
* orçamento com saldo negativo;
* contrato vencido;
* contrato encerrado;
* orçamento sem contratos;
* órgão sem orçamento;
* pagamento maior que o liquidado (inconsistência proposital);
* orçamento revisado;
* orçamento ainda não revisado;
* registros com campos nulos.

O comportamento da aplicação diante desses cenários faz parte da avaliação.

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

Esse desafio simula um cenário próximo ao encontrado em equipes de desenvolvimento que atuam com sistemas de acompanhamento orçamentário no setor público, avaliando tanto a capacidade técnica quanto as decisões de arquitetura, organização do código e a experiência oferecida ao usuário final.
