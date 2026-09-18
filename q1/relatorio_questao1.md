# Questão 1: Co-clustering da base Spambase com Double K-means

> Este documento resume o notebook [`questao1_double_kmeans.ipynb`](questao1_double_kmeans.ipynb),
> onde estão o código, todas as figuras e a análise completa.

## 1. Descrição sucinta dos dados

A base **Spambase** (UCI ML Repository, dataset 94) contém **N = 4601 e-mails** descritos por
**P = 57 variáveis numéricas contínuas**:

- 48 variáveis `word_freq_*`: percentual de ocorrência de uma palavra específica (0–100);
- 6 variáveis `char_freq_*`: percentual de ocorrência de um caractere (`;`, `(`, `[`, `!`, `$`, `#`);
- 3 variáveis `capital_run_length_*`: comprimento médio, máximo e total das sequências de maiúsculas.

Rótulo **a priori** `spam ∈ {0,1}`, não usado no agrupamento: 2788 não-spam (60,6%) e 1813 spam (39,4%).

Um detalhe da procedência é decisivo: os e-mails legítimos vieram da caixa pessoal de **um
único funcionário da HP Labs**.

### Achados da análise exploratória

| Achado | Consequência metodológica |
|---|---|
| Escalas heterogêneas (`capital_run_*` domina a variância) | Padronizar antes de agrupar |
| 77% de zeros na matriz; 38 das 57 variáveis com >80% de zeros | Valores raros viram escores z extremos após padronização |
| 302 e-mails (6,6%) com algum \|z\| > 10; máximo de **51σ** | Risco real de clusters formados por outliers |
| Bloco fortemente correlacionado (`857`–`415`: **0,996**) | Justifica buscar H > 1 |
| Classes sobrepostas na projeção PCA | Não se deve esperar que um método não supervisionado recupere o rótulo |

## 2. Metodologia

### 2.1 Padronização

Todas as 57 variáveis foram padronizadas por escore z. Sem isso, as três variáveis
`capital_run_length_*`, cuja variância é ordens de grandeza maior, decidiriam sozinhas a
partição. Escolha metodológica nossa, auditada na Seção 6.

### 2.2 Implementação

Double K-means implementado do zero em NumPy, vetorizado, seguindo as equações do material de
apoio (protótipos → partição de objetos → partição de variáveis, até não haver transferências).

**Validação:** em uma matriz sintética com 3 grupos de objetos × 2 grupos de variáveis
plantados, o algoritmo recupera ambas as partições com **ARI = 1,000** e reproduz a matriz de
protótipos verdadeira. A função objetivo é monotonicamente não crescente.

### 2.3 Protocolo

Para cada par (K,H) com K ∈ {2,3,4} e H ∈ {1,…,K} (9 pares), o algoritmo foi executado
**100 vezes** com inicializações aleatórias independentes (**900 execuções**), retendo a de
menor W. Sementes fixas garantem reprodutibilidade.

## 3. Resultados por (K, H)

| K | H | W\* | Silhueta | ARI | Iterações | % execuções que acham W\* |
|---|---|---:|---:|---:|---:|---:|
| 2 | 1 | 256.513,75 | 0,2961 | +0,0686 | 11 | 100% |
| 2 | 2 | 244.486,70 | **0,6596** | −0,0049 | 34 | 7% |
| 3 | 1 | 254.414,01 | 0,2515 | +0,0784 | 40 | 100% |
| 3 | 2 | 239.373,66 | 0,2454 | +0,1984 | 82 | 1% |
| 3 | 3 | 234.585,87 | 0,1409 | **+0,3854** | 33 | 20% |
| 4 | 1 | 253.278,68 | 0,0484 | +0,0916 | 38 | 100% |
| 4 | 2 | 233.358,70 | 0,2428 | +0,1493 | 30 | 25% |
| 4 | 3 | 228.516,44 | 0,1620 | +0,2744 | 47 | 22% |
| 4 | 4 | 227.261,65 | 0,1576 | +0,2451 | 27 | 1% |

**W\* decresce monotonicamente** com K e H, logo não serve como critério de seleção do número
de grupos, o que motiva o uso da silhueta.

**Sensibilidade à inicialização:** com H = 1 toda inicialização encontra o mesmo mínimo, mas
em (3,2) e (4,4) apenas **1%** das 100 execuções alcança o melhor mínimo. Isso confirma a
existência de muitos mínimos locais e justifica quantitativamente o protocolo de 100 execuções.

**Seleção:** K\* = arg max Sil → **(K\*, H\*) = (2, 2)**, com Sil = 0,6596.

## 4. Índice de Rand corrigido

**ARI = −0,0049**: concordância equivalente ao acaso. A partição selecionada **não recupera**
a distinção spam/não-spam.

Agravante: entre os 9 pares, o de maior silhueta é exatamente o de **menor** ARI (o único
negativo), enquanto (3,3), com silhueta baixa, alcança ARI = 0,385. Nesta base, os índices
interno e externo se ordenam de forma aproximadamente **inversa**.

### Referência: a silhueta do rótulo verdadeiro

Para distinguir "o algoritmo falhou" de "o critério é cego", calculamos a silhueta da própria
partição *a priori* nos mesmos dados padronizados: **Sil = 0,044**, **menor que a dos nove
pares** avaliados (o menor deles, (4,1), tem 0,048). A consequência é forte: mesmo que o
Double K-means tivesse recuperado o rótulo spam/não-spam com perfeição, o critério
K\* = arg max Sil teria descartado essa solução. O ARI nulo não mede uma falha do algoritmo em
encontrar o que existe — mede a cegueira do critério de seleção a essa estrutura neste espaço.

## 5. Resultados para (K\*, H\*) = (2, 2)

### (i) Matriz de protótipos G (em escores z)

| | variáveis G0 (46 vars.) | variáveis G1 (11 vars.) |
|---|---:|---:|
| **objetos G0** (4567 e-mails) | 0,0011 | −0,0511 |
| **objetos G1** (34 e-mails) | −0,1444 | **6,8612** |

O grupo de variáveis G1 reúne `hp`, `hpl`, `george`, `650`, `lab`, `labs`, `telnet`, `857`,
`415`, `85`, `technology`, `direct`.

### (ii) Matriz de confusão

| | não-spam (0) | spam (1) |
|---|---:|---:|
| **cluster 0** (4567) | 2754 | 1813 |
| **cluster 1** (34) | 34 | 0 |

O cluster 1 é 100% não-spam, mas representa 0,7% da base; o cluster 0 contém *todo* o spam.

### (iv) Função objetivo × iterações

Curva monotonicamente não crescente, convergindo em 34 iterações com redução total de 6,78%
de W, comportamento esperado de mínimos alternados, confirmando a corretude da implementação.

### Matriz de dados reorganizada

Seguindo a apresentação do material de apoio (*matriz inicial → matriz reorganizada → matriz
de protótipos*), a Seção 7.2 do notebook exibe X com as linhas reordenadas pelos clusters de
objetos e as colunas pelos grupos de variáveis (`results/matriz_reorganizada.png`). Na ordem
original nenhuma estrutura é visível; reorganizada, a matriz expõe os 2 × 2 blocos e a faixa
fina de 34 e-mails, saturada em todas as 11 variáveis de G1 (média de 6,86 desvios). É a
leitura visual do mesmo diagnóstico desenvolvido na Seção 6.

## 6. Análise dos resultados

### 6.1 O artefato do doador

As **onze variáveis** que mais separam o cluster minoritário (médias de 3,5 a 9,5 desvios acima
da base) formam um único conjunto temático: `857`, `415`, `direct`, `telnet`, `technology`,
`labs`, `85`, `650`, `lab`, `hp`, `hpl`, seguidas de `george`. São, respectivamente, códigos
de área da Califórnia, o vocabulário da HP Labs e um nome próprio: o **rastro da caixa de
e-mails do doador**, não um sinal generalizável de "não-spam". São também as mesmas variáveis
que formavam o único bloco fortemente correlacionado detectado na análise exploratória.

O mecanismo é claro: essas variáveis estão entre as mais esparsas da base (`george` é zero em
83% dos e-mails); ao padronizar, seus raros valores não nulos viram escores z de 6 a 9. O
Double K-means minimiza soma de quadrados e **é sensível a valores discrepantes** (limitação
listada no material de apoio). Isolar esses pontos gera um cluster artificialmente compacto,
exatamente o que a silhueta premia.

### 6.2 Quatro evidências que confirmam o diagnóstico

1. **Controle com k-means clássico.** Um k-means com K=2 sobre os mesmos dados padronizados
   converge para a **partição idêntica**: ARI = 1,000 entre as duas partições, **zero** objetos
   classificados de forma diferente, os mesmos 34 e-mails isolados. Logo, a limitação **não é
   do Double K-means**: é da geometria da base sob esse pré-processamento.

2. **Ordenação inversa silhueta × ARI.** O par escolhido pela silhueta é o pior por ARI; o par
   (3,3) tem o melhor ARI (0,385) e uma das piores silhuetas.

3. **O alcance do artefato.** Se a causa é o vocabulário do doador sob padronização, e não
   algo particular ao par (2,2), o mesmo cluster deve reaparecer nos demais pares. Reaparece:
   em **oito dos nove pares** o menor cluster tem 22 a 34 objetos, com escores z médios de 5,8
   a 7,4 desvios nas variáveis do doador e taxa de spam entre 0% e 12,5% (contra 39,4% na
   base). A única exceção é (2,1), com 1018 objetos — justamente o par em que H = 1 impede o
   algoritmo de isolar o bloco de 11 variáveis. Isso também explica por que (3,3) chega a ARI
   0,385 carregando o mesmo cluster degenerado: com K ≥ 3 sobram clusters para dividir o corpo
   da base depois de gastar um com a franja de outliers; com K = 2, não sobra.

4. **Estudo de sensibilidade com log(1+x).** Atenuando a assimetria antes de padronizar (maior
   \|z\| cai de 51 para 27), os clusters degenerados desaparecem: o menor grupo com K=2 passa
   de 34 para ~1300 objetos, as silhuetas caem para a faixa normal (0,00–0,19), revelando que
   o 0,66 original **era o artefato**, e o ARI do par (2,2) sobe de −0,005 para **+0,536**.

   *Observação:* os resultados oficiais reportados são os do pipeline padrão (z-score),
   conforme protocolo fixado antes de observar os resultados. Este estudo é uma análise
   diagnóstica *a posteriori*, reportada como tal.

### 6.3 Lição metodológica

Maximizar a silhueta seleciona a partição *geometricamente* melhor separada, que não é
necessariamente a mais *informativa*. Um valor alto de silhueta deve sempre ser auditado:
pode estar medindo a compacidade de um grupo de outliers.

### 6.4 O que o co-clustering entregou de valor

Apesar do resultado no eixo dos objetos, o agrupamento de **variáveis** é coerente e
interpretável: o grupo G1 isolou exatamente o bloco de termos mutuamente correlacionados
identificado na análise exploratória. O algoritmo capturou corretamente a estrutura de
co-ocorrência; este é o ganho do co-clustering sobre agrupar apenas os objetos.

## 7. Conclusão e ponte para a Questão 2

O número de clusters de objetos selecionado é **K\* = 2**, valor a ser usado como variável
resposta alternativa na Questão 2; a partição está em `results/particao_objetos_Kstar.csv`.

Registramos, porém, que essa partição é **degenerada** (4567 × 34), o que condiciona o desenho
experimental da Questão 2 (detalhado na Seção 8.7 do notebook):

1. **Bayesiano gaussiano:** a covariância estimada da classe minoritária tem **posto 16 de 57**
   e é singular — menos até que o limite n − 1 = 33, porque dezenas de variáveis esparsas são
   constantes dentro desses 34 e-mails. Serão necessárias regularização (*shrinkage*,
   covariância comum ou diagonal) ou redução de dimensionalidade, documentadas como parte do
   ajuste de hiper-parâmetros.
2. **Validação cruzada 30 × 10-folds:** cada fold de teste recebe ~3 exemplos da classe
   minoritária, o que torna precisão, cobertura e F-measure dessa classe muito instáveis e
   alarga os intervalos de confiança. Convém reportar métricas por classe e macro-médias, não
   só a taxa de erro global (uma regra trivial já a deixaria em 0,7%).
3. **k-vizinhos, Parzen e voto majoritário:** com P(ω₁) ≈ 0,0074, a classe minoritária
   raramente vence a comparação das probabilidades a posteriori, exceto sobre os próprios
   outliers que a definem.

Existe uma alternativa defensável, caso a orientação do curso seja usar uma variável resposta
não degenerada: a partição (2,2) sob log(1+x), equilibrada (1370 × 3231) e com ARI = 0,536.

## 8. Arquivos

Notebook (programa-fonte, autocontido): `questao1_double_kmeans.ipynb`.
Resultados em `results/`: `tabela_KH.csv`,
`tabela_KH_sensibilidade_log.csv`, `matriz_G_Kstar.csv`, `grupos_variaveis_Kstar.csv`,
`matriz_confusao_Kstar.csv`, `particao_objetos_Kstar.csv`, `resumo.json` e as figuras PNG.
