# Tech Challenge — Fase 1: NPS Preditivo

**Análise exploratória da experiência do cliente no e-commerce**

Projeto desenvolvido por **Gabriel Salvador**, na pós-graduação **AI Scientist — FIAP**.

O trabalho investiga fatores associados às notas de recomendação dos clientes e identifica indicadores relevantes para uma futura modelagem preditiva. Nesta fase, a entrega é uma **análise exploratória de dados (EDA)**, sem treinamento ou validação de modelos.

[Abrir no Google Colab](https://colab.research.google.com/github/gabriel-salvad/tech-challenge-nps/blob/main/notebooks/tech_challenge_nps.ipynb) · [Consultar o notebook](notebooks/tech_challenge_nps.ipynb)

## Problema de negócio

A nota de NPS é coletada após a experiência de compra. Para orientar melhorias e, futuramente, antecipar situações de risco, a empresa precisa entender quais aspectos dos pedidos, das entregas e do atendimento aparecem associados a avaliações negativas.

**Pergunta central:** quais indicadores da jornada de compra estão associados à variação do NPS e podem apoiar decisões para melhorar a experiência do cliente?

## Base de dados e qualidade

O notebook utiliza o arquivo `nps.csv`, com **2.500 registros e 19 variáveis originais**.

| Grupo | Variáveis da base |
| --- | --- |
| Identificação | `customer_id`, `order_id` |
| Perfil do cliente | `customer_age`, `customer_region`, `customer_tenure_months` |
| Pedido e condições comerciais | `order_value`, `items_quantity`, `discount_value`, `payment_installments` |
| Logística | `delivery_time_days`, `delivery_delay_days`, `freight_value`, `delivery_attempts` |
| Atendimento | `customer_service_contacts`, `resolution_time_days`, `complaints_count` |
| Avaliações e recompra | `nps_score`, `csat_internal_score`, `repeat_purchase_30d` |

As verificações registradas no notebook encontraram:

- nenhum valor ausente;
- nenhuma linha duplicada;
- nenhum identificador de pedido duplicado;
- nenhum valor fora das regras básicas testadas, como notas fora de 0 a 10, valores monetários negativos ou quantidades de itens menores que 1;
- registros das cinco regiões brasileiras.

Esses testes verificam a integridade básica dos dados, mas não substituem a validação da origem, da definição e do momento de coleta das variáveis.

## Método e análises realizadas

1. Inspeção da estrutura, dos tipos de dados e das regras de qualidade.
2. Classificação das notas em detratores, neutros e promotores.
3. Estatísticas descritivas: médias, medianas, desvios padrão, quartis e extremos para os grupos de variáveis selecionados.
4. Comparações por categoria de NPS, utilizando tabelas, boxplots e barras empilhadas com percentuais por faixa.
5. Teste de Shapiro–Wilk para a distribuição de `nps_score`.
6. Correlações de Spearman entre NPS e variáveis financeiras, de entrega e atendimento.
7. Comparações por faixa etária, tempo de relacionamento, região e recompra em 30 dias.

O teste de Shapiro–Wilk apresentou **W = 0,9817** e **p = 2,143 × 10⁻¹⁷**, indicando desvio da normalidade. A correlação de Spearman foi utilizada para investigar relações monotônicas, considerando também a presença de contagens e variáveis ordinais.

Nem todas as 19 variáveis receberam análise individual de associação com o NPS. Por exemplo, `csat_internal_score` aparece na inspeção e nas regras de qualidade, mas não possui análise específica de associação nesta versão.

## Distribuição do NPS

Como a base contém notas decimais, o código aplica os seguintes intervalos:

| Categoria | Intervalo aplicado | Quantidade | Participação |
| --- | --- | ---: | ---: |
| Detrator | De 0 até menos de 7 | 2.109 | 84,36% |
| Neutro | De 7 até menos de 9 | 281 | 11,24% |
| Promotor | De 9 a 10 | 110 | 4,40% |
| **Total** | | **2.500** | **100,00%** |

A partir dessas contagens, o indicador agregado é:

**NPS = % de promotores − % de detratores = 4,40 − 84,36 = −79,96 pontos.**

A nota individual `nps_score` e o NPS agregado são medidas diferentes. A predominância de detratores também exige atenção ao comparar grupos e avaliar futuros modelos.

## Principais resultados

### Entrega e atendimento

| Variável em relação a `nps_score` | Spearman (ρ) | Valor-p |
| --- | ---: | ---: |
| Dias de atraso na entrega | −0,59 | 1,088 × 10⁻²³⁰ |
| Quantidade de reclamações | −0,49 | 2,477 × 10⁻¹⁵¹ |
| Contatos com o atendimento | −0,34 | 3,441 × 10⁻⁶⁷ |
| Tempo de resolução | −0,19 | 8,026 × 10⁻²² |
| Tempo total de entrega | ≈ 0,00 | 0,9881 |

Coeficientes arredondados conforme os resultados exibidos no notebook.

**O atraso apresentou a associação de maior magnitude com o NPS entre as variáveis testadas.** Atrasos maiores aparecem acompanhados de notas menores. O tempo total de entrega, por sua vez, não apresentou associação monotônica evidente.

| Faixa de atraso | Detratores na faixa | Registros |
| --- | ---: | ---: |
| Sem atraso | 51,6% | 277 |
| 1–2 dias | 81,1% | 1.261 |
| 3–4 dias | 97,6% | 795 |
| 5 dias ou mais | 100,0% | 167 |

No atendimento, a participação de detratores passou de **17,4% entre os 23 registros sem reclamações** para **86,1% entre os 1.107 registros com 3–4 reclamações**. O grupo sem reclamações é pequeno e exige cautela na interpretação.

A mediana do tempo de resolução foi de **6 dias entre detratores**, **4 entre neutros** e **3 entre promotores**. Mais contatos com o atendimento também apareceram associados a notas menores, mas a base não identifica o motivo de cada contato.

### Variáveis financeiras

Valor do pedido, parcelamento, descontos e frete apresentaram associações monotônicas muito fracas com o NPS:

| Variável | Spearman (ρ), arredondado |
| --- | ---: |
| Valor do pedido | 0,036 |
| Número de parcelas | 0,027 |
| Valor do desconto | 0,012 |
| Valor do frete | −0,040 |

Embora existam diferenças entre as médias dos grupos, as correlações ficaram próximas de zero. Essas comparações não demonstram que aumentar descontos ou alterar preços elevará o NPS.

### Perfil dos clientes

As comparações descritivas por idade, tempo de relacionamento e região não mostraram diferenças expressivas entre as categorias de NPS. Nas regiões, a proporção de detratores variou de **83,3% a 85,6%**. Esse resultado descritivo não equivale a demonstrar ausência de qualquer efeito do perfil.

### Recompra em 30 dias

| Categoria | Registros com recompra | Recompra dentro da categoria |
| --- | ---: | ---: |
| Detratores | 0 de 2.109 | 0,0% |
| Neutros | 108 de 281 | 38,4% |
| Promotores | 110 de 110 | 100,0% |

A separação entre os grupos é muito acentuada. Antes de usar essa variável, é necessário verificar sua construção e a ordem temporal entre a pesquisa de NPS e a recompra. O notebook não a considera adequada como variável explicativa para uma futura previsão sem essa validação, pelo risco de vazamento de informação.

## Implicações para o negócio

Os resultados orientam a investigação de três frentes:

- **Logística:** acompanhar o cumprimento dos prazos e investigar situações de atraso.
- **Atendimento:** analisar motivos de reclamações, contatos repetidos e demora na resolução.
- **Gestão da experiência:** acompanhar os indicadores por faixa e avaliar os resultados de intervenções.

A diferença entre atraso e tempo total de entrega sugere a hipótese de que cumprir o prazo combinado seja relevante para a avaliação. Essa hipótese precisa ser validada; a análise não demonstra causa e efeito.

## Limitações e próximos passos

- Os resultados descrevem associações na base analisada e não garantem que uma intervenção produzirá melhora no NPS.
- Grupos com poucos registros e o desequilíbrio entre categorias limitam algumas comparações.
- Correlação próxima de zero não descarta relações não monotônicas ou efeitos combinados entre variáveis.
- A construção e o momento de coleta dos indicadores precisam ser verificados antes da modelagem.

Como evolução proposta, o projeto poderá desenvolver e validar um modelo para estimar o risco de avaliação negativa. Será necessário definir o momento da previsão, selecionar somente informações disponíveis nesse instante, separar treinamento e avaliação e comparar o modelo com uma referência simples.

## Tecnologias utilizadas

**Python**, **pandas**, **NumPy**, **Matplotlib**, **Seaborn** e **SciPy**, em notebook compatível com **Google Colab / Jupyter**.

## Organização do repositório

| Caminho | Finalidade |
| --- | --- |
| `notebooks/tech_challenge_nps.ipynb` | Notebook principal |
| `data/` | Pasta destinada à base de dados |
| `reports/` | Pasta destinada à apresentação e aos relatórios |
| `README.md` | Apresentação e instruções do projeto |

## Como executar no Google Colab

1. Abra o notebook pelo link no início deste README.
2. Conecte-se a um ambiente Python.
3. Obtenha a base `nps.csv`. Se estiver disponível em `data/`, baixe-a dessa pasta; caso contrário, solicite-a ao autor.
4. No painel **Arquivos** do Colab, envie `nps.csv` para a pasta de trabalho da sessão.
5. Execute as células em ordem, do início ao fim.

O notebook lê o arquivo com:

```python
df = pd.read_csv("nps.csv")
```

Se o CSV estiver em outro local, ajuste esse caminho. Abrir o notebook no Colab não carrega automaticamente o conteúdo da pasta `data/` do GitHub.

Caso alguma dependência esteja ausente, execute antes das importações:

```python
%pip install pandas numpy matplotlib seaborn scipy
```

## Autor

**Gabriel Salvador** — Pós-graduação AI Scientist, FIAP.

[Perfil no GitHub](https://github.com/gabriel-salvad)
# tech-challenge-nps
Análise exploratória de dados para identificação dos fatores associados à satisfação de clientes em um e-commerce.
