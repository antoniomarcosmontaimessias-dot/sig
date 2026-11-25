# Documentação Técnica — SIG v1.0
**Município:** Prefeitura Municipal da Estância Turística de Paraguaçu Paulista/SP  
**Versão:** 1.0 (MVP web) · **Data:** novembro/2026  
**Padrões considerados:** Lei 14.133/2021, Lei 13.019/2014 (MROSC), Transferegov, InvestSUS, SIMEC, SP Demandas, AUDESP/TCESP, e-MAG/ABNT NBR 17225, LGPD.

---

## 1. Visão geral

O SIG v1.0 é uma SPA em arquivo único (`index.html`) com:

- **Tailwind CSS via CDN** (`https://cdn.tailwindcss.com`) e CSS crítico embutido como fallback.
- **JavaScript ES6+**, sem framework, com renderização declarativa por strings HTML e delegação de eventos.
- **localStorage** como camada de persistência (`sig_paraguacu_v1`).
- **SheetJS** (XLSX) e **jsPDF** (PDF) via CDN.
- **40 entradas de catálogo**: dashboard, 30+ módulos de cadastro/fluxo, 3 formulários derivados de DOCX, 4 formulários HTML incorporados, BI e Transparência.

### 1.1 Estrutura lógica do catálogo

```
CATALOG[] = {
  id, label, group, icon, type, desc,
  fields[] | sections[].fields[], seed[], src (quando type=embed)
}
```

Tipos de entrada:

| `type` | Comportamento |
|---|---|
| `dashboard` | Painel executivo agregado |
| (padrão) | Lista + CRUD em modal (editar/duplicar/excluir) |
| `form` | Formulário secionado (DOCX convertido) com salvamento próprio |
| `embed` | Formulário oficial incorporado por iframe |
| `reports` | Gerador de relatórios com filtros |
| `transparency` | Espelho público automático |

---

## 2. Modelo de dados (localStorage)

Chave única: `sig_paraguacu_v1`

```jsonc
{
  "records": {
    "convenios-federais": [ { "id": "SIG-...", "...": "..." } ],
    "emendas-diretas": [ ... ],
    "...": [ ... ]
  },
  "movements": [ { "id", "data", "acao", "modulo", "descricao", "usuario" } ],
  "alerts": [ { "id", "titulo", "categoria", "vencimento", "severidade", "modulo", "descricao" } ],
  "documents": [ { "id", "name", "size", "type", "module", "date" } ],
  "settings": { "contrast": false, "largeFont": false, "lgpd": "accepted|refused|null" }
}
```

### 2.1 Dicionário simplificado de campos

#### Entidade: Convênio / Transferência (base financeira)

| Campo | Tipo | Descrição | Obrig. |
|---|---|---|---|
| `numero` | texto | Identificador do instrumento (ex.: `TC-897123/2026`) | Sim |
| `objeto` | texto longo | Descrição do objeto | Sim |
| `origem` | lista | União, Estado, Emendas ou OSCs | Sim |
| `secretaria` | lista | Sigla + denominação (ex.: `PARAG-SMS - Secretaria Municipal de Saúde`) | Sim |
| `valor` | moeda | Valor do instrumento (R$) | Sim |
| `status` | lista | Solicitado, Em Execução, Aprovado, Repassado, Concluído, Suspenso | Sim |
| `dataInicio` / `dataFim` | data | Vigência | Não |
| `concedente` | texto | Órgão repassador | Não |
| `beneficiario` | texto | Entidade beneficiária | Não |
| `plataforma` | lista | Transferegov (3 modalidades), InvestSUS, SIMEC, SP Demandas etc. | Não |
| `contrapartida` | moeda | Contrapartida municipal | Não |
| `dataRepasse` | data | Data efetiva do repasse | Não |
| `prestacaoPrazo` | data | Prazo de prestação de contas | Não |
| `sei` / `planoTrabalho` | texto | Referência de processo SEI | Não |

#### Entidade: Emenda Parlamentar (execução direta)

| Campo | Tipo | Descrição |
|---|---|---|
| `tipoEmenda` | lista | Municipal impositiva, estadual ou federal |
| `numeroEmenda` | texto | Número da emenda |
| `autor` | texto | Autor da emenda/indicação |
| `indicacao` | texto | Indicação parlamentar de origem |
| `orgaoExecutor` | lista | Secretaria executora |
| `tipoDespesa` | lista | Custeio ou Investimento |
| `empenho` / `pagamento` | texto/data | Rastreabilidade financeira |
| `sei` | texto | Processo administrativo |

#### Entidade: Parceria MROSC

| Campo | Tipo | Descrição |
|---|---|---|
| `osc` / `cnpjOsc` | texto | Organização da Sociedade Civil |
| `tipo` (Termos) | lista | Contrato de Gestão, Termo de Parceria, Colaboração, Fomento, Convênio |
| `aditivos` | repetidor | Nº, tipo, data e descrição do aditivo |
| `gestor` / `fiscal` | texto | Gestor da parceria e fiscal designados |
| `nota` / `parecer` | texto | Avaliação e parecer conclusivo |

#### Entidade: Licitação e Contrato

| Campo | Tipo | Descrição |
|---|---|---|
| `modalidade` | lista | Pregão, Concorrência, Concurso, Leilão, Diálogo Competitivo |
| `edital` | texto | Referência do edital/SEI |
| `sessao` | data | Sessão de disputa/julgamento |
| `vencedor` | texto | Fornecedor vencedor |
| `fundamento` | texto | Base legal da contratação direta |
| `quantidade` / `saldo` / `validade` | texto/data | Controle de ARP e adesões |
| `aditivos` | repetidor | Aditivos de valor/prazo e apostilamentos |

#### Entidade: Execução

| Campo | Tipo | Descrição |
|---|---|---|
| `ordem` | texto | Ordem de fornecimento/serviço |
| `valorMedido` | moeda | Valor medido |
| `recebimento` | lista | Provisório, Definitivo, Pendente |
| `metaFisica` | número | Percentual de execução física |
| `dataPrevista` / `dataRealizada` | data | Cronograma de desembolso |

#### Entidade: Controle e Transparência

| Campo | Tipo | Descrição |
|---|---|---|
| `severidade` | lista | Baixa, Média, Alta, Crítica |
| `prazo` | data | Prazo de resposta/apresentação |
| `parecer` | texto longo | Parecer do Controle Interno |
| `origem` | texto | Módulo gerador do dado (transparência/auditoria) |

---

## 3. Regras de negócio implementadas

1. **Agregação financeira:** `Total Solicitado = Σ valor` dos módulos financeiros; `Repassado = Σ status ∈ {Repassado, Pago, Concluído}`; `Saldo = Solicitado − Repassado` (mínimo zero). Detalhamento por `origem`.
2. **Alertas:** severidade calculada por faixa de vencimento e categoria (prestação de contas, vigência, TCESP/CGU, licitações).
3. **Duplicação consciente:** ao duplicar, o usuário escolhe quais campos são reaproveitados; identificadores e datas são limpos por padrão e o registro é aberto para ajuste.
4. **Rastreabilidade:** toda criação, edição, exclusão, importação, assinatura e consentimento LGPD gera evento em `movements`.
5. **Transparência:** nenhuma edição é permitida; os dados são lidos diretamente dos módulos (`TRANSPARENCY_VIEWS`).
6. **Relatórios:** filtros combináveis por ano, fonte, secretaria, status, beneficiário e inadimplência; exportação no formato AUDESP inclui envelope com órgão/exercício/sistema.

---

## 4. Fluxos legalmente ordenados

```
DFD → PCA → ETP → Pesquisa de Preços → Matriz de Riscos → TR / Projeto Básico
   → Licitação ou Contratação Direta → ARP/Contrato → Execução
   → Prestação de Contas → Controle Interno → Transparência
```

Parcerias MROSC:

```
Cadastro OSC → Credenciamento → Chamamento (ou Justificativa) → ETI
→ Plano de Trabalho → Termo (+ Aditivos) → Execução
→ Monitoramento/Avaliação → Parecer Conclusivo → Transparência
```

Emendas:

```
Indicação → Plano de Trabalho Resumido → Declaração de Custeio
→ Empenho → Execução/Pagamento → Relatório de Gestão → TCESP → Transparência
```

---

## 5. Formulários oficiais incorporados

| Módulo | Origem | Tratamento |
|---|---|---|
| Plano de Trabalho Resumido | `plano_trabalho_resumido.docx` | 11 seções convertidas em formulário HTML com validação e assinatura |
| Declaração de Custeio | `declaracao_custeio.docx` | Texto oficial mantido, campos do beneficiário e base de cálculo |
| Relatório de Gestão | `relatorio_gestao.docx` | Quadro técnico, execução da meta (repetidor) e conclusão |
| ETI | `estudo_tecnico_inicial.html` | Cópia adaptada em `assets/embedded/form-eti.html` |
| Plano de Trabalho OSC | `plano_de_trabalho_osc.html` | Cópia adaptada em `assets/embedded/form-plano-trabalho-osc.html` |
| Avaliação do Gestor | `avaliacao_gestor.html` | Cópia adaptada em `assets/embedded/form-avaliacao-gestor.html` |
| Lista de Verificação | `lista_verificacao.html` | Cópia adaptada em `assets/embedded/form-lista-verificacao.html` |

**Adaptações aplicadas:** troca das variáveis de cor para o design system institucional (`#1e3a8a` / `#15803d`), inclusão automática do brasão e faixa de identidade. A lógica original (autosave, versionamento, documentos e exportações) foi preservada integralmente.

---

## 6. Importação e exportação

### Importação
- **CSV** com delimitador `;` ou `,` e aspas duplas; cabeçalhos normalizados (sem acentos/maiúsculas).
- **JSON** array de objetos.
- **XLSX/XLS** primeira planilha via SheetJS.
- Mapeamento automático por `campo.k` ou `campo.l` (removendo parênteses, ex.: “Valor (R$)” → `valor`).

### Exportação
| Formato | Observação |
|---|---|
| PDF |jsPDF, até 200 registros por arquivo |
| JSON | Estrutura original |
| CSV | UTF-8 com BOM para Excel |
| XLSX | Planilha “SIG” |
| AUDESP/TCESP | JSON com envelope `{orgao, exercicio, sistema, geradoEm, registros}` |

---

## 7. Acessibilidade, LGPD e IA

- **e-MAG:** `lang="pt-BR"`, skip link, landmarks, `aria-label`, `aria-live`, foco visível, tabelas com `scope="col"`, contraste alto e ampliação de fonte.
- **LGPD:** banner de consentimento, política com bases legais (art. 7º, II e III), finalidade, retenção e direitos do titular; trilha de consentimento.
- **IA embarcada:** painel com três modos (análise de risco, minutas ETP/DFD, resumo executivo) e auditoria de campos obrigatórios/números duplicados. É heurística e **não substitui validação técnica/jurídica**.

---

## 8. Limites e evolução

A v1.0 é deliberadamente local e monofásica. Para produção, multiusuário, integrações reais e trilha imutável, migrar para a arquitetura descrita em `ARQUITETURA_V2.md`.
