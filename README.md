# Double k-means

Este repositório foi organizado para responder ao projeto acadêmico descrito em [Projeto-AM-2026-2.pdf](Projeto-AM-2026-2.pdf), com foco em análise de agrupamento e co-clustering da base Spambase.

## Objetivo do repositório

O objetivo principal deste projeto é desenvolver a solução para o problema proposto no enunciado do trabalho, aplicando o algoritmo Double K-means à base de e-mails Spambase e avaliando a estrutura dos grupos de objetos e variáveis.

O repositório já contém a implementação e a análise da questão 1, que corresponde ao estudo do Double K-means sobre os dados da base.

## O que já temos

- [q1/questao1_double_kmeans.ipynb](q1/questao1_double_kmeans.ipynb): notebook principal com a implementação, análise exploratória, protocolo experimental e resultados executados;
- [q1/relatorio_questao1.md](q1/relatorio_questao1.md): relatório textual detalhando a metodologia e os resultados;
- [q1/README.md](q1/README.md): documentação específica da questão 1;
- [q1/results](q1/results): tabelas, matrizes e figuras produzidas pelo estudo.

## Contexto do problema

A base de estudo é a Spambase, composta por 4.601 e-mails e 57 variáveis numéricas contínuas, com a informação de rótulo spam/não-spam disponível apenas como referência de comparação, não como entrada do algoritmo de agrupamento.

O objetivo da questão 1 é investigar a seguinte questão:

- o algoritmo Double K-means consegue revelar grupos de objetos e grupos de variáveis de forma coerente e interpretável;
- como a escolha do número de clusters influencia a solução;
- como interpretar a capacidade do critério de silhueta em um cenário com artefato de dados;
- qual é o efeito do vocabulário de origem do doador na formação de clusters extremos.

## Estrutura do repositório

- `q1/`: conteúdo da questão 1;
- `q1/results/`: saídas do experimento;
- `spambase.data`: base de dados original;
- `spambase.names`: descrição da base e das variáveis;
- `LICENSE`: licença do projeto.

## Requisitos

- Python 3.10+
- pip
- Jupyter Notebook / Jupyter Lab

## Como executar

```bash
cd ../Double-k-means
python3 -m venv .venv
source .venv/bin/activate
pip install -r q1/requirements.txt
```

Em seguida:

```bash
jupyter lab q1/questao1_double_kmeans.ipynb
```

## Resultado principal da questão 1

A análise conduzida no notebook mostra que o melhor par selecionado pela silhueta foi:

- (K*, H*) = (2, 2)
- silhueta = 0,6596
- ARI em relação à partição a priori = -0,0049

Esse resultado aponta para um diagnóstico importante: a solução mais “bem separada” segundo a silhueta pode não ser a mais informativa ou a mais alinhada com a estrutura semântica da classe alvo. O estudo demonstra que o cluster minoritário foi influenciado por um artefato ligado ao vocabulário do doador original da base, e não por uma diferenciação genuína de spam vs. não-spam.

## Próximo passo

Este repositório já resolve a questão 1 do projeto. O próximo passo, se houver, será continuar a partir do diagnóstico e da partição obtida para desenvolver a questão seguinte do trabalho acadêmico descrito em [Projeto-AM-2026-2.pdf](Projeto-AM-2026-2.pdf).

## Licença

Este projeto está sob a licença disponível em [LICENSE](LICENSE).
