# Aprendizado de Máquina Aplicado à Detecção de Propostas Anômalas em Processos de Licitações no Setor Público Pernambucano

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

## Diagrama do Projeto 

<img width="450" height="735" alt="image" src="https://github.com/user-attachments/assets/381c2ad0-478d-4579-a535-293eb0983253" />


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
