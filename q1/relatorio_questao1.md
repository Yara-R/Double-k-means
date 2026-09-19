# Questão 1: Co-clustering da base Spambase com Double K-means

> Este documento resume o notebook principal [`questao1_double_kmeans.ipynb`](questao1_double_kmeans.ipynb),
> onde estão o código, todas as figuras e a análise completa. A análise de robustez está no
> notebook complementar [`questao1_estrategias_alternativas.ipynb`](questao1_estrategias_alternativas.ipynb).

## Resumo

Pelo protocolo do enunciado, o par selecionado é **(K\*, H\*) = (2, 2)**, com silhueta 0,660. A
partição de objetos correspondente é fortemente desbalanceada (4567 × 34 e-mails) e tem
**ARI = −0,005** contra a partição *a priori* spam / não-spam. O resultado não decorre de erro de
implementação. Ele resulta de um artefato da base (34 e-mails com vocabulário exclusivo do doador
dos dados), que a padronização *z-score* transforma em valores extremos. Como o Double K-means
minimiza soma de quadrados, ele isola esse grupo, e a silhueta recompensa a separação obtida.
O mesmo comportamento aparece no k-means clássico, em oito dos nove pares (K,H) e em três blocos
independentes de sementes. Oito estratégias alternativas de pré-processamento não levam o critério
a selecionar uma partição próxima do rótulo, porque a distinção spam / não-spam não é
geometricamente separada em nenhuma das representações avaliadas.

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
| Escalas heterogêneas (`capital_run_*` concentra 99,99% da variância bruta) | Padronizar antes de agrupar |
| 77% de zeros na matriz; 38 das 57 variáveis com >80% de zeros | Valores raros viram escores z extremos após padronização |
| 302 e-mails (6,6%) com algum \|z\| > 10; máximo de **51σ** | Risco real de clusters formados por outliers |
| Bloco fortemente correlacionado (`857`–`415`: **0,996**) | Justifica buscar H > 1 |
| Classes sobrepostas na projeção PCA | Não se deve esperar que um método não supervisionado recupere o rótulo |

## 2. Metodologia

### 2.1 Padronização

Todas as 57 variáveis foram padronizadas por escore z. Sem isso, as três variáveis
`capital_run_length_*`, cuja variância é ordens de grandeza maior, decidiriam sozinhas a
partição. É uma escolha metodológica nossa, fixada antes de observar os resultados e auditada
nas Seções 6 e 7.

### 2.2 Implementação

Double K-means implementado do zero em NumPy, vetorizado, seguindo as equações do material de
apoio (protótipos → partição de objetos → partição de variáveis, até não haver transferências).

**Validação:** em uma matriz sintética com 3 grupos de objetos × 2 grupos de variáveis
plantados, o algoritmo recupera ambas as partições com **ARI = 1,000** e reproduz a matriz de
protótipos verdadeira. A função objetivo é monotonicamente não crescente.

### 2.3 Protocolo

Para cada par (K,H) com K ∈ {2,3,4} e H ∈ {1,…,K} (9 pares), o algoritmo foi executado
**100 vezes** com inicializações aleatórias independentes (**900 execuções**), retendo a de
menor W. Sementes fixas garantem reprodutibilidade. **899 das 900 execuções convergiram** (sem
transferências) antes do teto de 100 iterações. A única exceção, no par (4,2), não é a execução
retida.

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

**Sensibilidade à inicialização.** Com H = 1 toda inicialização encontra o mesmo mínimo. O motivo
é estrutural: com um único grupo de variáveis, a atribuição de cada objeto depende apenas da média
da sua linha, e o problema se reduz a um k-means unidimensional. Já em (3,2) e (4,4), apenas
**1%** das 100 execuções alcança o melhor mínimo. Isso confirma a existência de muitos mínimos
locais e justifica o protocolo de 100 execuções.

**Estabilidade frente às sementes.** Com probabilidade de sucesso p por execução, 100 execuções
alcançam o melhor mínimo com probabilidade 1 − (1 − p)¹⁰⁰, apenas ≈ 63% para p = 1%. Repetimos
o protocolo com outros dois blocos de 100 sementes (notebook complementar, Seção 7). O par
selecionado (2,2) e a sua partição são **idênticos nos três blocos**, e em oito dos nove pares o
melhor de 100 oficial coincide com o melhor de 300. A exceção é (4,4): os blocos adicionais
encontraram W até 0,02% menor, com partições próximas (ARI de 0,88 e 0,98 com a oficial), sem
efeito sobre a escolha de K\*.

**Seleção:** K\* = arg max Sil → **(K\*, H\*) = (2, 2)**, com Sil = 0,6596.

## 4. Índice de Rand corrigido

**ARI = −0,0049**: concordância equivalente ao acaso. A partição selecionada **não recupera**
a distinção spam/não-spam.

Agravante: entre os 9 pares, o de maior silhueta é exatamente o de **menor** ARI (o único
negativo), enquanto (3,3), com silhueta baixa, alcança ARI = 0,385. Nesta base, os índices
interno e externo se ordenam de forma aproximadamente **inversa** (correlação de Spearman de
−0,72 entre silhueta e ARI nos nove pares).

### Referência: a silhueta do rótulo verdadeiro

Para distinguir "o algoritmo falhou" de "o critério é cego", calculamos a silhueta da própria
partição *a priori* nos mesmos dados padronizados: **Sil = 0,044**, **menor que a dos nove
pares** avaliados (o menor deles, (4,1), tem 0,048). A consequência é forte: mesmo que o
Double K-means tivesse recuperado o rótulo spam/não-spam com perfeição, o critério
K\* = arg max Sil teria descartado essa solução. O ARI nulo não mede uma falha do algoritmo em
encontrar o que existe. Ele mede a cegueira do critério de seleção a essa estrutura neste espaço.

## 5. Resultados para (K\*, H\*) = (2, 2)

### (i) Matriz de protótipos G (em escores z)

| | variáveis G0 (46 vars.) | variáveis G1 (11 vars.) |
|---|---:|---:|
| **objetos G0** (4567 e-mails) | 0,0011 | −0,0511 |
| **objetos G1** (34 e-mails) | −0,1444 | **6,8612** |

O grupo de variáveis G1 reúne `hp`, `hpl`, `650`, `lab`, `labs`, `telnet`, `857`, `415`, `85`,
`technology` e `direct`. A variável `george`, também associada ao doador, ficou em G0: o passo de
atribuição das variáveis considera os 4601 objetos, e no conjunto da base ela se comporta como as
demais de G0.

### (ii) Matriz de confusão

| | não-spam (0) | spam (1) |
|---|---:|---:|
| **cluster 0** (4567) | 2754 | 1813 |
| **cluster 1** (34) | 34 | 0 |

O cluster 1 é 100% não-spam, mas representa 0,7% da base; o cluster 0 contém *todo* o spam.

### (iv) Função objetivo × iterações

A curva é monotonicamente não crescente e converge em 34 iterações, com redução total de 6,78%
de W. É o comportamento esperado de mínimos alternados e confirma a corretude da implementação.

### Matriz de dados reorganizada

Seguindo a apresentação do material de apoio (*matriz inicial → matriz reorganizada → matriz
de protótipos*), a Seção 7.2 do notebook exibe X com as linhas reordenadas pelos clusters de
objetos e as colunas pelos grupos de variáveis (`results/matriz_reorganizada.png`). Na ordem
original nenhuma estrutura é visível. Reorganizada, a matriz expõe os 2 × 2 blocos e a faixa
fina de 34 e-mails, saturada em todas as 11 variáveis de G1 (média de 6,86 desvios). É a
leitura visual do mesmo diagnóstico desenvolvido na Seção 6.

## 6. Por que os resultados são assim

### 6.1 O mecanismo: o artefato do doador

As **onze variáveis** que mais separam o cluster minoritário (médias de 3,5 a 9,5 desvios acima
da base) formam um único conjunto temático: `857`, `415`, `direct`, `telnet`, `technology`,
`labs`, `85`, `650`, `lab`, `hp`, `hpl`. São números de telefone da região de Palo Alto (os
códigos de área `650` e `415` e o prefixo `857`, dos telefones da HP) e o vocabulário da HP Labs:
o **rastro da caixa de e-mails do doador**, e não um sinal generalizável de "não-spam". São
também as mesmas variáveis que formavam o único bloco fortemente correlacionado detectado na
análise exploratória.

O mecanismo tem três elos:

1. **Esparsidade.** Essas variáveis estão entre as mais esparsas da base: são nulas em 76% a
   95,5% dos e-mails.
2. **Padronização.** Ao padronizar, seus raros valores não nulos viram escores z extremos. Os
   34 e-mails, 0,74% da base, passam a concentrar **8,7% da soma de quadrados total**, cerca de
   12 vezes a sua participação proporcional.
3. **Função objetivo e critério.** O Double K-means minimiza soma de quadrados e **é sensível a
   valores discrepantes** (limitação listada no material de apoio). Isolar esses pontos reduz W
   e gera um cluster artificialmente compacto e muito afastado, exatamente o que a silhueta
   premia.

### 6.2 Evidências que confirmam o diagnóstico

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
   base). A única exceção é (2,1), com 1018 objetos.

   A exceção **não** se explica por H = 1 em si, pois (3,1) e (4,1) também têm H = 1 e isolam
   grupos de 32 e 22 objetos. Com H = 1, o custo de atribuir o objeto *i* ao grupo *k* é
   ‖xᵢ‖² − 2P·g_k·x̄ᵢ + P·g_k², que depende de xᵢ apenas pela média da linha x̄ᵢ. O algoritmo se
   reduz, portanto, a um k-means unidimensional, cujos grupos são intervalos disjuntos de x̄ᵢ
   (verificado no notebook). Os 34 e-mails do doador estão todos entre os 70 de maior média. Com
   K = 2, o único corte da reta separa e-mails de atividade geral alta e baixa (1018 × 3583), e
   os 34 ficam diluídos no grupo de 1018. Com K = 3 e 4, sobra um intervalo para a cauda
   extrema, formado majoritariamente por eles (25 dos 32 e 20 dos 22 objetos).

   Isso também explica por que (3,3) chega a ARI 0,385 carregando o mesmo cluster degenerado:
   com K ≥ 3 sobram clusters para dividir o corpo da base depois de gastar um com a franja de
   outliers; com K = 2, não sobra.

4. **Estudo de sensibilidade com log(1+x).** Repetindo o protocolo completo com as mesmas
   sementes após atenuar a assimetria (o maior \|z\| cai de 51 para 27), os clusters degenerados
   desaparecem em K = 2. O menor grupo passa de 34 para 1203–1384 objetos, e as silhuetas caem
   para a faixa normal (0,00–0,19), o que revela que o 0,66 original **era o artefato**. O ARI do
   par (2,2) sobe de −0,005 para **+0,536**, mas a silhueta passa a escolher (2,1), com ARI 0,12.

### 6.3 Outras estratégias de pré-processamento (notebook complementar)

Para verificar se **alguma** escolha de pré-processamento faria o protocolo recuperar a partição
spam / não-spam, o protocolo completo (9 pares × 100 execuções, mesmas sementes) foi executado
sob oito estratégias. O rótulo é usado apenas para avaliar. A estratégia E0 reproduz exatamente
a tabela oficial.

| Estratégia | (K\*, H\*) | Sil | ARI em K\* | Tamanhos em K\* | Maior ARI da grade (par; posição pela silhueta) | Sil do rótulo |
|---|---|---:|---:|---|---|---:|
| E0 · *z-score* (oficial) | (2,2) | 0,660 | −0,005 | 34 / 4567 | 0,385 ((3,3); 8º de 9) | 0,044 |
| E1 · dados brutos | (2,1) | 0,844 | 0,042 | 238 / 4363 | 0,100 ((4,2); 9º de 9) | 0,197 |
| E2 · min-max | (2,1) | 0,223 | 0,095 | 1139 / 3462 | 0,204 ((3,3); 5º de 9) | 0,044 |
| E3 · *z-score* recortado em ±3 | (2,1) | 0,190 | 0,142 | 1140 / 3461 | 0,461 ((2,2); 4º de 9) | 0,076 |
| E4 · log(1+x) + *z-score* | (2,1) | 0,189 | 0,120 | 1203 / 3398 | 0,536 ((2,2); 6º de 9) | 0,068 |
| E5 · log(1+x) + min-max | (3,3) | 0,129 | 0,391 | 265 / 1717 / 2619 | 0,391 ((3,3); 1º de 9) | 0,085 |
| E6 · quantis → normal | (3,3) | 0,183 | 0,276 | 557 / 1252 / 2792 | 0,276 ((3,3); 1º de 9) | 0,104 |
| E7 · binarização | (2,1) | 0,192 | 0,115 | 1646 / 2955 | 0,229 ((4,2); 3º de 9) | 0,109 |

Conclusões:

- **O artefato pode ser removido, mas o rótulo não é recuperado.** As estratégias que atenuam a
  cauda (E3, E5, E6, E7) reduzem a concentração da dispersão em poucos e-mails e eliminam o
  cluster degenerado em todos os pares (menor cluster ≥ 174 objetos). Ainda assim, o par
  escolhido pela silhueta tem ARI de no máximo 0,39, e em seis das oito estratégias não passa de
  0,14. Em cinco delas o escolhido é (2,1), que separa os e-mails pelo nível geral de atividade,
  e não pelo conteúdo.
- **Os pares de ARI alto existem, mas o critério não os alcança.** O maior ARI da grade (0,536)
  ocupa o 6º lugar de 9 no ranking de silhueta. A correlação de Spearman entre silhueta e ARI
  varia de −0,93 a +0,80 conforme a estratégia: não há relação estável entre as duas medidas.
- **O rótulo tem silhueta baixa em todos os espaços** (0,044 a 0,197). A estrutura spam /
  não-spam não é compacta e separada em nenhuma das representações. Um critério que mede
  compacidade e separação não pode selecioná-la, qualquer que seja o pré-processamento.
- **Escolher a estratégia pelo ARI seria inválido.** E5 e E6 só se destacam quando se olha o
  rótulo; *a priori*, as oito estratégias são igualmente defensáveis, e silhuetas calculadas em
  espaços diferentes não são comparáveis entre si. Promover uma delas a resultado oficial
  transformaria a Questão 1 em um problema supervisionado. E, mesmo nesses casos, K\* seria 3,
  e não 2.

Por isso, o resultado oficial permanece o do pipeline *z-score*, fixado antes de observar os
resultados. As análises desta seção são diagnósticas e reportadas como tal.

### 6.4 Lição metodológica

Maximizar a silhueta seleciona a partição *geometricamente* melhor separada, que não é
necessariamente a mais *informativa*. Um valor alto de silhueta deve sempre ser auditado:
pode estar medindo a compacidade de um grupo de outliers.

### 6.5 O que o co-clustering entregou de valor

Apesar do resultado no eixo dos objetos, o agrupamento de **variáveis** é coerente e
interpretável: o grupo G1 isolou exatamente o bloco de termos mutuamente correlacionados
identificado na análise exploratória. O algoritmo capturou corretamente a estrutura de
co-ocorrência, e esse é o ganho do co-clustering sobre agrupar apenas os objetos.

## 7. Conclusão e ponte para a Questão 2

O número de clusters de objetos selecionado é **K\* = 2**, valor a ser usado como variável
resposta alternativa na Questão 2. A partição está em `results/particao_objetos_Kstar.csv`.

Registramos, porém, que essa partição é **degenerada** (4567 × 34), o que condiciona o desenho
experimental da Questão 2 (detalhado na Seção 8.7 do notebook):

1. **Bayesiano gaussiano:** a covariância estimada da classe minoritária tem **posto 16 de 57**
   e é singular, menos até que o limite n − 1 = 33, porque dezenas de variáveis esparsas são
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

Caso a orientação do curso seja usar uma variável resposta não degenerada, a alternativa coerente
com o protocolo é aplicar **o mesmo critério** sob um pré-processamento fixado *a priori*. Por
exemplo, sob log(1+x) o critério seleciona (2,1), com classes de 1203 e 3398 e-mails. Não se deve
adotar a partição de maior ARI, pois isso seria escolher a variável resposta com base no próprio
rótulo (Seção 6.3).

## 8. Arquivos

- **Notebook principal** (programa-fonte, autocontido): `questao1_double_kmeans.ipynb`.
- **Notebook complementar** (robustez): `questao1_estrategias_alternativas.ipynb`.
- **Resultados do notebook principal** em `results/`: `tabela_KH.csv`,
  `tabela_KH_sensibilidade_log.csv`, `matriz_G_Kstar.csv`, `grupos_variaveis_Kstar.csv`,
  `matriz_confusao_Kstar.csv`, `particao_objetos_Kstar.csv`, `alcance_do_artefato.csv`,
  `diagnostico_classes_questao2.csv`, `resumo.json` e as figuras PNG.
- **Resultados do notebook complementar** em `results/estrategias/`: `grade_completa.csv`,
  `sintese_por_estrategia.csv`, `silhueta_do_rotulo_por_estrategia.csv`,
  `concentracao_dispersao.csv`, `estabilidade_sementes.csv` e as figuras PNG.
