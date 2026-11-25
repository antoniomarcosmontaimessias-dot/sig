# SIG — Sistema Integrado de Gestão
### Prefeitura Municipal da Estância Turística de Paraguaçu Paulista/SP

MVP funcional (v1.0) em **HTML5 + Tailwind CSS (CDN) + JavaScript ES6+**, com persistência em `localStorage`, 40 módulos/visões, dados mockados de Paraguaçu Paulista e incorporação integral dos formulários oficiais de Terceiro Setor.

---

## 1. Executar

### Opção A — abrir diretamente
Abra `index.html` no navegador (Chrome/Edge recomendado).

### Opção B — servidor local (recomendado)
Os quatro formulários oficiais são carregados por `iframe`; um servidor local evita restrições de origem de arquivos.

```bash
node server.js
# acesse http://localhost:8080
```

> É necessária conexão com a internet para os CDNs de Tailwind, XLSX e jsPDF. Sem conexão, a interface continua utilizável (CSS crítico embutido), mas importação/exportação XLSX/PDF ficam indisponíveis.

---

## 2. Estrutura de arquivos

```
index.html                         SPA v1.0 (arquivo único)
server.js                          Servidor estático opcional
assets/
  brasao_prefeitura.png            Brasão oficial fornecido
  avaliacao_gestor.html            Modelo original (referência)
  declaracao_custeio.docx          Modelo original (referência)
  estudo_tecnico_inicial.html      Modelo original (referência)
  lista_verificacao.html           Modelo original (referência)
  plano_de_trabalho_osc.html       Modelo original (referência)
  plano_trabalho_resumido.docx     Modelo original (referência)
  relatorio_gestao.docx            Modelo original (referência)
  embedded/                        Cópias adaptadas dos 4 formulários HTML
    form-eti.html
    form-plano-trabalho-osc.html
    form-avaliacao-gestor.html
    form-lista-verificacao.html
modelos/docx_texto.txt             Texto extraído dos 3 DOCX
DOCUMENTACAO_TECNICA.md            Dicionário de dados e especificações
ARQUITETURA_V2.md                  Arquitetura de nuvem e escalabilidade
```

---

## 3. O que está implementado

| Requisito | Onde |
|---|---|
| Painel Executivo com 5 cards e detalhamento União/Estado/Emendas/OSCs | Dashboard |
| Linha do tempo de movimentações e alertas vermelho/amarelo/verde | Dashboard |
| 18 secretarias com siglas oficiais + fundos + autarquias | Administração → Estrutura Organizacional |
| 5 perfis de acesso | Administração → Usuários e Perfis |
| Fluxo DFD → PCA → ETP → Pesquisa → Riscos → TR/Projeto Básico | Planejamento |
| Licitações, Dispensa/Inexigibilidade, ARP com saldos, Contratos com aditivos | Licitações e Contratações |
| Convênios Federais (Transferegov/InvestSUS/SIMEC), Estaduais (SP Demandas) | Convênios e Transferências |
| Emendas de execução direta + 3 formulários DOCX convertidos em HTML | 4.3 Emendas Parlamentares |
| ETI, Plano de Trabalho OSC, Avaliação do Gestor e Lista de Verificação **incorporados integralmente** | 4.4 Terceiro Setor |
| Execução direta, de convênios e rastreabilidade de emendas | Módulo 5 |
| MROSC, PC de convênios e auditoria | Módulo 6 |
| BI com 6 filtros e exportação PDF/JSON/CSV/XLSX/AUDESP | Módulo 7 |
| Transparência como espelho automático (6 visões) | Módulo 8 |
| CRUD completo, editar, duplicar com aproveitamento de campos | Ações nas tabelas |
| Importar CSV/JSON/XLSX e inserir documentos em lote | Barra de cada módulo |
| Digitação por voz (Web Speech API) | Botão 🎙 ao lado de campos de texto |
| Assinatura digital (Sisyema / GOV.BR / Certificado) | Botão “Assinar instrumento” |
| Impressão, exportação, acessibilidade e-MAG, LGPD, IA embarcada | Global |

---

## 4. Dados de demonstração

Os dados são fictícios e representativos da realidade local:

- Convênio InvestSUS – UBS: R$ 1.250.000,00
- Convênio FNDE/PDE – EMEFs: R$ 2.300.000,00
- Emenda municipal – pavimentação Jardim Paulista: R$ 480.000,00
- Termo de Colaboração MROSC – OSC de saúde: R$ 210.000,00
- Alertas: InvestSUS, TCESP, vigência de termo e licitação

CNPJs são placeholders (`00.000.000/0001-xx`) e CPFs mascarados. Substitua por dados reais apenas em ambiente autorizado.

---

## 5. Reiniciar a demonstração

No console do navegador:

```js
localStorage.removeItem('sig_paraguacu_v1'); location.reload();
```

---

## 6. Limites conhecidos da v1.0

- Não há backend; todos os dados ficam no navegador de cada usuário.
- Assinatura digital e integrações contábeis são **simuladas** (registro de metadados/hash local).
- A IA é heurística (regras locais), sem chamada a modelos externos.
- A importação XLSX depende do CDN SheetJS; PDF depende do jsPDF.
- Os 4 formulários incorporados mantêm seu próprio `localStorage` e exportações, conforme projeto original.
- Para uso em produção, adote a v2.0 descrita em `ARQUITETURA_V2.md`.
