# Double K-means

Este repositório reúne a solução do projeto acadêmico descrito em
[Projeto-AM-2026-2.pdf](Projeto-AM-2026-2.pdf) (Aprendizagem de Máquina, CIn/UFPE), com foco em
agrupamento simultâneo de objetos e variáveis (*co-clustering*) na base Spambase.

## Objetivo

Aplicar o algoritmo Double K-means à base de e-mails Spambase e avaliar a estrutura dos grupos de
objetos e de variáveis, conforme o protocolo do enunciado. O repositório contém, até o momento, a
solução completa da Questão 1.

## Conteúdo

| Arquivo | Papel |
|---|---|
| [q1/questao1_double_kmeans.ipynb](q1/questao1_double_kmeans.ipynb) | **Notebook principal** (programa-fonte): análise exploratória, implementação, protocolo do enunciado, resultados e análise crítica |
| [q1/questao1_estrategias_alternativas.ipynb](q1/questao1_estrategias_alternativas.ipynb) | **Material complementar**: oito estratégias de pré-processamento e estabilidade do resultado frente às sementes |
| [q1/relatorio_questao1.md](q1/relatorio_questao1.md) | Relatório textual da Questão 1 |
| [q1/README.md](q1/README.md) | Documentação específica da Questão 1 |
| [q1/results/](q1/results) | Tabelas, matrizes e figuras produzidas pelos notebooks |
| `spambase.data`, `spambase.names` | Base de dados original (UCI, dataset 94) e sua descrição |
| `Double k-means.pdf` | Material de apoio do algoritmo |

## Contexto

A Spambase contém 4.601 e-mails descritos por 57 variáveis numéricas contínuas. O rótulo
spam / não-spam não entra no agrupamento: é usado apenas como partição *a priori* para validação
externa (índice de Rand corrigido e matriz de confusão).

A Questão 1 pede que o Double K-means seja executado 100 vezes para cada par $(K, H)$, com
$K \in \{2,3,4\}$ e $H \in \{1,\dots,K\}$, retendo a melhor solução segundo a função objetivo, e que
o número de clusters seja escolhido por $K^* = \arg\max_{(K,H)} Sil(K,H)$.

## Resultado principal da Questão 1

| Métrica | Valor |
|---|---|
| $(K^*, H^*)$ selecionado pela silhueta | **(2, 2)** |
| Silhueta | 0,6596 |
| Índice de Rand corrigido (ARI) contra a partição *a priori* | **−0,0049** |
| Tamanho dos clusters de objetos | 4567 / 34 |
| Tamanho dos grupos de variáveis | 46 / 11 |

A partição selecionada é fortemente desbalanceada e não recupera a distinção spam / não-spam
(concordância equivalente ao acaso).

### Por que o resultado é este

O resultado **não decorre de erro de implementação**. Ele resulta da combinação de três fatores:

1. **Os dados.** Os e-mails legítimos da Spambase vieram da caixa de correio de um funcionário da
   HP Labs. Um grupo de 34 deles usa com alta frequência um vocabulário que quase não aparece no
   restante da base (`857`, `415`, `hp`, `hpl`, `telnet`, `technology`, …). Essas variáveis são
   nulas em 76% a 95,5% dos e-mails.
2. **O pré-processamento.** A padronização *z-score* é necessária, pois sem ela as três variáveis
   `capital_run_length_*` concentram 99,99% da variância e decidem sozinhas a partição. Porém, ela
   converte os valores raros daquelas variáveis esparsas em escores de 3,5 a 9,5 desvios-padrão.
   Nessa escala, os 34 e-mails (0,74% da base) concentram 8,7% da soma de quadrados total, cerca de
   12 vezes a sua participação proporcional.
3. **O algoritmo e o critério.** O Double K-means minimiza uma soma de quadrados e é sensível a
   valores discrepantes, limitação apontada no próprio material de apoio. Isolar esse pequeno grupo
   reduz a função objetivo e produz dois grupos muito afastados, configuração que a silhueta
   recompensa com um valor alto.

Em síntese: **a silhueta mede compacidade e separação geométrica, e a distinção spam / não-spam
não é geometricamente separada nesta base.** O critério seleciona, corretamente segundo a sua
definição, a partição mais separada, que é a de um pequeno grupo de valores discrepantes.

### Evidências

- **Implementação validada.** Em dados sintéticos com estrutura plantada, o algoritmo recupera as
  partições de objetos e de variáveis com ARI = 1.
- **Controle com k-means clássico.** Um k-means com $K = 2$ sobre os mesmos dados converge para a
  partição idêntica. O fenômeno não é específico do co-clustering.
- **Alcance do artefato.** O mesmo grupo de 22 a 34 e-mails reaparece em oito dos nove pares
  $(K,H)$. A exceção, $(2,1)$, ocorre porque com $H = 1$ o algoritmo se reduz a um k-means
  unidimensional sobre a média de cada linha, e com $K = 2$ há um único corte disponível.
- **Silhueta do rótulo verdadeiro.** A partição *a priori* tem silhueta 0,044, inferior à dos nove
  pares avaliados. Nem uma recuperação perfeita do rótulo seria escolhida pelo critério.
- **Outros pré-processamentos não "consertam" o resultado.** Sob oito estratégias (o *z-score*
  oficial e sete alternativas: dados brutos, min-max, recorte em ±3, $\log(1+x)$ com *z-score* e
  com min-max, transformação por quantis e binarização), o cluster degenerado desaparece nas que
  atenuam a cauda. Ainda assim, o par
  escolhido pela silhueta tem ARI de no máximo 0,39, e em seis das oito estratégias não passa de
  0,14. O rótulo verdadeiro tem silhueta baixa (0,044 a 0,197) em todos os espaços.
- **Robustez às sementes.** O par selecionado e a sua partição são idênticos em três blocos
  independentes de 100 sementes.

O resultado oficial permanece o do pipeline *z-score*, fixado antes da observação dos resultados.
Escolher o pré-processamento pelo ARI significaria usar o rótulo para ajustar um método não
supervisionado, o que é vazamento de informação. Por isso as estratégias alternativas são
reportadas apenas como análise de robustez.

### Consequência para a Questão 2

A segunda versão da base na Questão 2 usa a partição de $K^*$ como variável resposta. Com classes
de 4567 e 34 exemplos, a covariância estimada da classe minoritária é singular (posto 16 de 57), e
cada fold de teste da validação cruzada recebe cerca de 3 exemplos dessa classe. O notebook
principal (Seção 8.7) detalha essas implicações para o desenho experimental.

## Requisitos

- Python 3.12 (os resultados versionados foram gerados com Python 3.12.13)
- Dependências em [q1/requirements.txt](q1/requirements.txt), com as versões usadas na execução

## Como executar

Na raiz do repositório:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r q1/requirements.txt
```

Para abrir os notebooks de forma interativa:

```bash
jupyter lab q1/questao1_double_kmeans.ipynb
```

Para reexecutar tudo de forma não interativa, execute o notebook principal **antes** do complementar,
pois este usa as saídas daquele como referência:

```bash
cd q1
jupyter nbconvert --to notebook --execute --inplace questao1_double_kmeans.ipynb
jupyter nbconvert --to notebook --execute --inplace questao1_estrategias_alternativas.ipynb
```

Os dois notebooks são determinísticos (sementes fixas). O principal leva cerca de 2,5 minutos e o
complementar cerca de 5,5 minutos em uma máquina de 8 núcleos (9.000 execuções do Double K-means,
paralelizadas).

## Licença

Este projeto está sob a licença disponível em [LICENSE](LICENSE).
