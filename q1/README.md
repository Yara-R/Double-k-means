# Questão 1: Double K-means na base Spambase

## Entregáveis

**[`questao1_double_kmeans.ipynb`](questao1_double_kmeans.ipynb)** (notebook principal e
programa-fonte): notebook já executado, da análise exploratória dos dados até a análise crítica
dos resultados. Contém o resultado oficial da Questão 1.

**[`questao1_estrategias_alternativas.ipynb`](questao1_estrategias_alternativas.ipynb)** (material
complementar): testa a robustez do resultado oficial sob oito estratégias de pré-processamento e
sob outros dois blocos de sementes aleatórias. Não substitui o resultado oficial.

### Estrutura do notebook principal

| Seção | Conteúdo |
|---|---|
| Resumo | Resultado, justificativa e evidências, em uma página |
| 0 | Configuração do ambiente e reprodutibilidade |
| 1 | Descrição e **análise exploratória**: escalas, esparsidade, assimetria, outliers, correlação, PCA |
| 2 | Pré-processamento (padronização *z-score*) e sua justificativa |
| 3 | Double K-means: formulação, implementação vetorizada e **validação em dados sintéticos** |
| 4 | Protocolo experimental (900 execuções), convergência e sensibilidade à inicialização |
| 5 | Escolha de K\* pela silhueta |
| 6 | Índice de Rand corrigido e a **silhueta do rótulo verdadeiro** como referência |
| 7 | Resultados para (K\*, H\*): matriz G, **matriz de dados reorganizada**, matriz de confusão, função objetivo × iterações |
| 8 | **Análise crítica**: artefato do doador e seu alcance na grade, caso H = 1, controle com k-means, estudo de sensibilidade, outras estratégias (8.6.1), **ponte para a Questão 2** |
| 9 | Conclusões |

### Estrutura do notebook complementar

| Seção | Conteúdo |
|---|---|
| 1–2 | Configuração e implementação (idêntica à do notebook principal) |
| 3 | Oito estratégias de pré-processamento e diagnóstico de concentração da dispersão |
| 4 | Protocolo completo para cada estratégia e verificação de que E0 reproduz exatamente a tabela oficial |
| 5 | Síntese: par escolhido pela silhueta × melhor par da grade |
| 6 | Silhueta do rótulo verdadeiro em cada espaço |
| 7 | Estabilidade do protocolo oficial em três blocos de 100 sementes |
| 8 | Conclusões |

## Arquivos

| Arquivo | Papel |
|---|---|
| `questao1_double_kmeans.ipynb` | Notebook principal (executado), **programa-fonte** |
| `questao1_estrategias_alternativas.ipynb` | Notebook complementar (executado) |
| `relatorio_questao1.md` | Relatório em texto corrido |
| `requirements.txt` | Dependências Python |
| `results/` | Tabelas CSV, figuras PNG e `resumo.json` do notebook principal |
| `results/estrategias/` | Tabelas e figuras do notebook complementar |

Os dois notebooks são autocontidos: o Double K-means está implementado em cada um deles e não
depende de nenhum módulo `.py` externo. O notebook complementar verifica que a sua cópia da
implementação reproduz exatamente os resultados do principal.

## Como executar

Na raiz do repositório:

```bash
python3 -m venv .venv && source .venv/bin/activate && pip install -r q1/requirements.txt
jupyter lab q1/questao1_double_kmeans.ipynb
```

Ou, para reexecutar tudo de forma não interativa (o principal primeiro, pois o complementar usa as
suas saídas como referência):

```bash
cd q1
jupyter nbconvert --to notebook --execute --inplace questao1_double_kmeans.ipynb
jupyter nbconvert --to notebook --execute --inplace questao1_estrategias_alternativas.ipynb
```

Os notebooks são **determinísticos** (sementes fixas por par (K,H) e por execução). O principal
leva cerca de **2,5 minutos** (900 execuções do protocolo oficial e 900 do estudo de
sensibilidade). O complementar leva cerca de **5,5 minutos** em 8 núcleos (9.000 execuções,
paralelizadas com `joblib`).

## Principais resultados

| Métrica | Valor |
|---|---|
| (K\*, H\*) selecionado pela silhueta | **(2, 2)** |
| Silhueta | 0,6596 |
| W\* | 244.486,70 |
| **ARI vs. partição a priori** | **−0,0049** |
| Tamanho dos clusters de objetos | 4567 / 34 |
| Grupos de variáveis | 46 / 11 |
| Execuções convergidas | 899 de 900 (a única exceção não é a execução retida) |

## Por que os resultados são assim

O ARI nulo **não é um erro de implementação**. O cluster minoritário (34 e-mails, 100%
não-spam) é formado por valores extremos em 11 variáveis ligadas ao doador original da base
(HP Labs: `857`, `415`, `direct`, `telnet`, `technology`, `labs`, `85`, `650`, `lab`, `hp`, `hpl`),
um artefato de coleta. O mecanismo tem três elos:

1. **Esparsidade.** Essas variáveis são nulas em 76% a 95,5% dos e-mails.
2. **Padronização.** O *z-score*, necessário para que `capital_run_length_*` não domine a distância
   (99,99% da variância bruta), transforma os seus raros valores não nulos em escores de 3,5 a 9,5
   desvios-padrão. Os 34 e-mails, 0,74% da base, passam a concentrar 8,7% da soma de quadrados
   total.
3. **Função objetivo e critério.** O Double K-means minimiza soma de quadrados e é sensível a
   valores discrepantes. Isolar esses e-mails reduz W e cria dois grupos muito afastados, o que a
   silhueta recompensa.

Evidências (seções do notebook principal):

1. **Controle com k-means clássico (8.5).** Um k-means com K = 2 converge para a **partição
   idêntica** (ARI = 1,000 entre as duas, zero objetos diferentes). Logo, o fenômeno não é
   específico do co-clustering.
2. **Ordenação inversa (6).** O par (3,3), com silhueta baixa, alcança **ARI = 0,385**. A silhueta
   escolhe justamente o pior par segundo o ARI.
3. **Alcance do artefato (8.3).** O mesmo cluster degenerado (22 a 34 objetos, 5,8 a 7,4 desvios
   nas variáveis do doador) aparece em **oito dos nove pares**. A exceção é (2,1), e não por
   H = 1 em si, pois (3,1) e (4,1) também têm H = 1 e isolam grupos de 32 e 22 objetos. Com
   H = 1, o Double K-means se reduz a um k-means unidimensional sobre a **média de cada linha**.
   Os 34 e-mails estão entre os 70 de maior média, mas com K = 2 há um único corte na reta, que
   separa atividade geral alta e baixa (1018 × 3583). Com K = 3 e 4, sobra um intervalo para a
   cauda extrema (25 dos 32 e 20 dos 22 objetos são desses e-mails).
4. **Sensibilidade a `log(1+x)` (8.6).** Atenuando a assimetria, os clusters degenerados
   desaparecem em K = 2 e o ARI do par (2,2) sobe de −0,005 para **+0,536**. Mas a silhueta passa
   a escolher (2,1), com ARI 0,12.

**O critério, e não o algoritmo.** A silhueta da própria partição *a priori* é **0,044**, menor
que a dos nove pares avaliados. Mesmo uma recuperação perfeita do rótulo teria sido descartada
por K\* = arg max Sil.

## Outras estratégias não "consertam" o resultado

O notebook complementar executa o protocolo completo sob oito pré-processamentos, com as mesmas
sementes. O rótulo é usado apenas para avaliar.

| Estratégia | (K\*, H\*) | Sil | ARI em K\* | Maior ARI da grade (posição pela silhueta) | Sil do rótulo |
|---|---|---:|---:|---|---:|
| E0 · *z-score* (oficial) | (2,2) | 0,660 | −0,005 | 0,385 (8º de 9) | 0,044 |
| E1 · dados brutos | (2,1) | 0,844 | 0,042 | 0,100 (9º de 9) | 0,197 |
| E2 · min-max | (2,1) | 0,223 | 0,095 | 0,204 (5º de 9) | 0,044 |
| E3 · *z-score* recortado em ±3 | (2,1) | 0,190 | 0,142 | 0,461 (4º de 9) | 0,076 |
| E4 · log(1+x) + *z-score* | (2,1) | 0,189 | 0,120 | 0,536 (6º de 9) | 0,068 |
| E5 · log(1+x) + min-max | (3,3) | 0,129 | 0,391 | 0,391 (1º de 9) | 0,085 |
| E6 · quantis → normal | (3,3) | 0,183 | 0,276 | 0,276 (1º de 9) | 0,104 |
| E7 · binarização | (2,1) | 0,192 | 0,115 | 0,229 (3º de 9) | 0,109 |

- As estratégias que atenuam a cauda (E3, E5, E6, E7) eliminam o cluster degenerado, mas o par
  escolhido tem ARI de no máximo 0,39, e em seis das oito estratégias não passa de 0,14.
- O rótulo verdadeiro tem silhueta baixa em **todos** os espaços. A estrutura spam / não-spam não
  é compacta e separada em nenhuma das representações avaliadas.
- E5 e E6 só se destacam quando se olha o rótulo. Promovê-las a resultado oficial seria escolher o
  pré-processamento pela variável resposta (vazamento de informação). O resultado oficial
  permanece o do *z-score*.

**Robustez às sementes.** Repetido com dois outros blocos de 100 sementes, o protocolo seleciona o
mesmo par (2,2) com a mesma partição. Só o par (4,4) apresenta um mínimo ligeiramente melhor em
outro bloco (W até 0,02% menor), sem efeito sobre K\*.

## Atenção para a Questão 2

A partição de K\* = 2 é degenerada (4567 × 34). A Seção 8.7 do notebook principal quantifica o que
isso implica. A covariância estimada da classe minoritária tem posto 16 de 57: é singular, logo o
bayesiano gaussiano exige regularização. Além disso, cada fold de teste da validação cruzada
10-folds recebe ~3 exemplos dessa classe.
