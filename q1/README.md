# Questão 1: Double K-means na base Spambase

## Entregável principal

**[`questao1_double_kmeans.ipynb`](questao1_double_kmeans.ipynb)**: notebook Jupyter completo,
já executado (todas as saídas e as 20 figuras embutidas), cobrindo da análise exploratória
dos dados até a análise crítica dos resultados.

### Estrutura do notebook

| Seção | Conteúdo |
|---|---|
| 0 | Configuração do ambiente e reprodutibilidade |
| 1 | Descrição e **análise exploratória**: escalas, esparsidade, assimetria, outliers, correlação, PCA |
| 2 | Pré-processamento (padronização z-score) e sua justificativa |
| 3 | Double K-means: formulação, implementação vetorizada e **validação em dados sintéticos** |
| 4 | Protocolo experimental (900 execuções) e sensibilidade à inicialização |
| 5 | Escolha de K\* pela silhueta |
| 6 | Índice de Rand corrigido e a **silhueta do rótulo verdadeiro** como referência |
| 7 | Resultados para (K\*, H\*): matriz G, **matriz de dados reorganizada**, matriz de confusão, função objetivo × iterações |
| 8 | **Análise crítica**: artefato do doador e seu alcance no grid, controle com k-means, estudo de sensibilidade, **ponte para a Questão 2** |
| 9 | Conclusões |

## Arquivos

| Arquivo | Papel |
|---|---|
| `questao1_double_kmeans.ipynb` | Notebook principal (executado), **é o programa-fonte** |
| `relatorio_questao1.md` | Relatório em texto corrido |
| `requirements.txt` | Dependências Python |
| `results/` | Tabelas CSV, figuras PNG e `resumo.json` |

O algoritmo Double K-means está implementado dentro do próprio notebook (Seção 3), que é
autocontido: não depende de nenhum módulo `.py` externo.

## Como executar

```bash
python3 -m venv .venv && source .venv/bin/activate && pip install -r q1/requirements.txt
```

Depois, abra o notebook:

```bash
jupyter lab q1/questao1_double_kmeans.ipynb
```

Ou reexecute tudo de forma não interativa:

```bash
cd q1 && jupyter nbconvert --to notebook --execute --inplace questao1_double_kmeans.ipynb
```

O notebook é **determinístico** (sementes fixas por par (K,H) e por execução) e leva cerca de
**2,5 minutos**: 900 execuções do protocolo oficial + 900 do estudo de sensibilidade.

## Principais resultados

| Métrica | Valor |
|---|---|
| (K\*, H\*) selecionado pela silhueta | **(2, 2)** |
| Silhueta | 0,6596 |
| W\* | 244.486,70 |
| **ARI vs. partição a priori** | **−0,0049** |
| Tamanho dos clusters de objetos | 4567 / 34 |
| Grupos de variáveis | 46 / 11 |

**Diagnóstico.** O ARI nulo não é um erro de implementação. O cluster minoritário (34 e-mails)
é formado por valores extremos em variáveis ligadas ao doador original da base (HP Labs:
`857`, `415`, `hp`, `hpl`, `george`, …), um artefato de coleta. Três evidências confirmam:

1. um **k-means clássico** com K=2 converge para a **partição idêntica** (ARI = 1,000 entre as
   duas, zero objetos diferentes), logo o problema não é do co-clustering;
2. o par (3,3), com silhueta baixa, alcança **ARI = 0,385**: a silhueta escolhe justamente o
   pior par segundo o ARI;
3. o mesmo cluster degenerado (22 a 34 objetos, 5,8 a 7,4 desvios nas variáveis do doador)
   aparece em **oito dos nove pares** — a exceção é (2,1), o único com H = 1;
4. atenuando a assimetria com `log(1+x)`, os clusters degenerados desaparecem e o ARI do par
   (2,2) sobe de −0,005 para **+0,536**.

**O critério, e não o algoritmo.** A silhueta da própria partição *a priori* é **0,044**, menor
que a dos nove pares avaliados. Mesmo uma recuperação perfeita do rótulo teria sido descartada
por K\* = arg max Sil.

**Atenção para a Questão 2.** A partição de K\* = 2 é degenerada (4567 × 34). A Seção 8.7 do
notebook quantifica o que isso implica: a covariância estimada da classe minoritária tem posto
16 de 57 (singular, logo o bayesiano gaussiano exige regularização), e cada fold de teste da
validação cruzada 10-folds recebe ~3 exemplos dessa classe.
