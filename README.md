# Aprendizado de Máquina Aplicado à Detecção de Propostas Anômalas em Processos de Licitações no Setor Público Brasileiro

**Autora:** Letícia Gomes da Silva  
**Orientadora:** Laura Pacífico  
**Instituição:** CESAR School — Graduação em Ciência da Computação  
**Período:** 8º Período — 2026.2

---

## Sobre o Projeto

Este repositório contém o pipeline de coleta, tratamento e modelagem de dados do Trabalho de Conclusão de Curso (TCC), cujo objetivo é comparar modelos de Aprendizado de Máquina para a geração de uma **tabela ordenada de risco** de propostas anômalas em processos licitatórios dos municípios de Pernambuco.

> *Como a comparação entre modelos de Machine Learning, considerando variáveis financeiras, administrativas e eleitorais, pode auxiliar na detecção de anomalias ainda na fase de licitação dos municípios de Pernambuco, apoiando a construção de planos de ação de auditoria preventiva?*

O produto esperado é uma tabela ordenada por *score* de anomalia que permita a auditores do TCE-PE **priorizar** quais processos licitatórios merecem atenção preventiva — sem substituir o julgamento humano.

---

## Objetivos Específicos

| # | Objetivo |
|---|---|
| OE1 | Definir os conceitos de fraude e anomalia no contexto de licitações municipais |
| OE2 | Coletar e organizar dados públicos de licitações e contratos dos municípios de PE |
| OE3 | Construir variáveis financeiras, administrativas e eleitorais (incl. distância temporal para eleições) |
| OE4 | Aplicar modelos não supervisionados: Isolation Forest, LOF e One-Class SVM |
| OE5 | Comparar os modelos quanto à identificação de propostas com maior risco de anomalia |
| OE6 | Elaborar uma tabela ordenada de risco para apoiar auditorias preventivas |

---

## Fontes de Dados

| Fonte | O que fornece | API |
|---|---|---|
| **Compras.gov.br** | Licitações (Lei 8.666/93) e contratações (Lei 14.133/2021) | `dadosabertos.compras.gov.br` |
| **PNCP** | Resultados por item — propostas vencedoras | `pncp.gov.br/api/pncp/v1` |
| **BrasilAPI** | Dados cadastrais de fornecedores (CNPJ) | `brasilapi.com.br/api/cnpj/v1` |
| **IBGE** | Coordenadas geográficas dos 185 municípios de PE | `servicodados.ibge.gov.br` |

---

## Pipeline de Dados

01_extracao_uasg.ipynb
→ data/raw/uasg_pe.csv                    (1.222 UASGs ativas em PE)
↓
02_extracao_licitacoes.ipynb
→ data/raw/licitacoes_legado_pe.csv        (1.328 licitações Lei 8.666, 2020–2024)
→ data/raw/contratacoes14133_pe.csv        (11.353 contratações Lei 14.133, 2023–2024)
↓
02b_extracao_resultados.ipynb
→ data/raw/propostas_pe.csv               (75.502 propostas vencedoras, 9.458 CNPJs)
↓
03_enriquecimento_cnpj.ipynb
→ data/raw/fornecedores_cnpj.csv          (dados cadastrais dos fornecedores)
↓
04_coordenadas_ibge.ipynb
→ data/raw/municipios_pe.csv              (185 municípios PE com lat/lon)
↓
05_engenharia_variaveis.ipynb
→ data/processed/dataset_ml.csv          (76.218 linhas × 27 features)



---

## Dataset Final — `dataset_ml.csv`

**76.218 propostas** × **27 features** | Período: 2023–2024 | 13,4 MB

| Grupo | Variável | Descrição |
|---|---|---|
| **Identificadores** | `orgaoEntidadeCnpj`, `anoCompraPncp`, `sequencialCompraPncp`, `numeroItem`, `cnpj_forn_norm` | Chaves de rastreabilidade |
| **Preço** | `desvio_preco_mediana` | (valor_proposto − mediana_item) / mediana_item — sobrepreço relativo |
| | `ratio_preco_estimado` | valor_proposto / valor_estimado da contratação |
| **Concorrência** | `n_propostas_item` | Nº de vencedores registrados por item |
| | `fornecedor_unico` | 1 se único fornecedor registrado no item (89,7% das linhas) |
| **Temporal** | `ano`, `mes`, `trimestre`, `dia_semana` | Sazonalidade |
| **Eleitoral** | `dias_para_eleicao` | Distância (dias) até a eleição municipal mais próxima |
| | `ano_eleitoral` | 1 se ano eleitoral (2020 ou 2024) |
| | `pre_eleitoral_180d` | 1 se ≤ 180 dias antes de uma eleição (59,1% das linhas) |
| **Geográfica** | `dist_km_forn_orgao` | Distância Haversine (km) entre fornecedor e órgão (32,3% preenchido) |
| | `fornecedor_fora_pe` | 1 se fornecedor é de outro estado (63,1%) |
| **Empresa** | `tipo_fornecedor` | `pessoa_juridica` (91%) / `pessoa_fisica` (9%) |
| | `idade_empresa_anos` | Idade da empresa na data da proposta |
| | `empresa_nova` | 1 se empresa com < 1 ano de existência (6,8%) |
| | `freq_forn_orgao` | Nº de contratações distintas do fornecedor com este órgão |
| | `opcao_pelo_mei` | 1 se MEI |
| | `opcao_pelo_simples` | 1 se Simples Nacional |
| **Contexto** | `modalidadeNome` | Modalidade licitatória |
| | `porte` | Porte da empresa (micro, pequena etc.) |
| | `cnae_fiscal` | Código CNAE principal do fornecedor |

---

## Estrutura do Repositório

notebooks/
  ├── 01_extracao_uasg.ipynb          # Coleta de UASGs ativas em PE
  ├── 02_extracao_licitacoes.ipynb    # Coleta de licitações (Lei 8.666 e 14.133)
  ├── 02b_extracao_resultados.ipynb   # Coleta de resultados por item (PNCP)
  ├── 03_enriquecimento_cnpj.ipynb    # Dados cadastrais dos fornecedores (BrasilAPI)
  ├── 04_coordenadas_ibge.ipynb       # Coordenadas geográficas dos municípios
  ├── 05_engenharia_variaveis.ipynb   # Joins e engenharia de features
  └── data/
  ├── raw/
  │   ├── uasg_pe.csv
  │   ├── licitacoes_legado_pe.csv
  │   ├── contratacoes14133_pe.csv
  │   ├── propostas_pe.csv
  │   ├── fornecedores_cnpj.csv
  │   └── municipios_pe.csv
  └── processed/
  └── dataset_ml.csv          # Dataset final para modelagem

---

## Como Executar

### Pré-requisitos

```bash
pip install pandas numpy requests tqdm jupyter
```

Execução sequencial
Execute os notebooks na ordem numérica. Cada notebook depende dos arquivos gerados pelo anterior:

```bash
jupyter nbconvert --to notebook --execute 01_extracao_uasg.ipynb
jupyter nbconvert --to notebook --execute 02_extracao_licitacoes.ipynb
jupyter nbconvert --to notebook --execute 02b_extracao_resultados.ipynb
jupyter nbconvert --to notebook --execute 03_enriquecimento_cnpj.ipynb
jupyter nbconvert --to notebook --execute 04_coordenadas_ibge.ipynb
jupyter nbconvert --to notebook --execute 05_engenharia_variaveis.ipynb
```
Ou abra e execute manualmente cada notebook no Jupyter Lab/Notebook.

Atenção: Os notebooks 02_extracao_licitacoes.ipynb e 02b_extracao_resultados.ipynb realizam muitas chamadas a APIs públicas com rate limiting. A execução completa pode levar várias horas dependendo da conexão e da disponibilidade das APIs.

---

Observações Técnicas
fornecedor_unico (89,7%): A API do PNCP retorna apenas o vencedor por item, não todos os licitantes. Esse campo reflete a ausência de múltiplos vencedores registrados — não necessariamente ausência de concorrência.
dist_km_forn_orgao (32,3% preenchido): Apenas fornecedores de PE têm coordenadas no municipios_pe.csv. Fornecedores de outros estados resultam em nulo. A variável binária fornecedor_fora_pe (100% preenchida) pode ser usada como proxy.
NB03 — 55% de erros no enriquecimento CNPJ: Combinação de CPFs de pessoa física, CNPJs inválidos e empresas canceladas/inaptas. As variáveis de empresa têm ~75% de cobertura.

---

Licença
Dados utilizados são públicos, disponibilizados pelas APIs do Governo Federal do Brasil sob a Política de Dados Abertos (Decreto nº 8.777/2016).

---
