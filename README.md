

# **Projeto 4 - Aprendizado de máquina aplicado na caracterização de lesões de substância branca do Lúpus Eritematoso Sistêmico**

---


Nome: Leonardo Rodrigues da Costa


RA: 146895

Especialização: Física
---
# Apresentacão 
Este projeto foi feito para a disciplina de pós-graduação Ciência e Visualização de Dados em Saúde, oferecida no primeiro semestre de 2022, na Unicamp.

# Introdução
   O Lúpus Eritematoso Sistêmico (LES ou em inglês Systemic Lupus Erythematosus, SLE) é uma doença crônica autoimune que pode afetar diversos órgãos 
   e sistemas do corpo humano. No cérebro, sabe-se que esta condição pode causar lesões desmielinizantes na substância branca, observadas como manchas
   claras no tecido nervoso com o exame de ressonância magnética imagética (RMI). A proposta deste trabalho foi avaliar se as lesões do LES se assemelham 
   mais às lesões da Esclerose Múltipla (EM, outra síndrome desmielinizante) ou do Acidente Vascular Cerebral (AVC) isquêmico (decorrente da obstrução de 
   um vaso sanguíneo), os dois tipos de lesão também se apresentam como manchas claras na RMI. Para esta avaliação, utilizamos dois datasets 
   (um com imagens RMI de pacientes com AVC, outro de pacientes com EM) para treino e validação de um classificador SVM (Support Vector Machine) 
   linear. Subsequentemente, este classificador foi aplicado no dataset de imagens de LES. É importante dizer que as lesões nestas imagens foram 
   previamente demarcadas por médicos, e cada imagem selecionada aqui acompanha uma máscara com sua região de interesse.
 
  O SVM calcula qual é o hiperplano que melhor separa as classes (no caso se é uma imagem de AVC ou EM) usando um vetor 
  de atributos das imagens do conjunto de treino. Este vetor de atributos é construído com propriedades numéricas da imagem, 
  como as estatísticas da distribuição de intensidade de cinza da imagem (ou apenas da máscara) ou de propriedades de textura 
  (que definem relações, ou padrões, que os pontos de uma determinada vizinhança formam).
  
# Ferramentas 
O código foi escrito em Python, em um notebook do Google Colab.
As principais bibliotecas utilizadas foram:

*   Google Drive
*   numpy
*   pandas
*   sklearn
*   matlibplot 

Demais bibliotecas foram declaradas na seção de inicialização do código.

#  Preparo e uso de dados

## Normalização 
A normalização das imagens foi feita usando o método MinMax que mapeia a menor valor da imagem para 0 e o maior valor para 255.


## Montando o vetor de atributos
  O vetor de atributos deste projeto foi feito a com as métricas (média, desvio padrão, obliquidade, curtose, tamanho da máscara/255, argmax)
  de três tipos de histograma:
 
 
1.   Histograma da imagem completa;
2.   Histograma da região da máscara;
3.   Histograma de textura Local Binary Pattern (LBP) da região da máscara.

Além disso levei em consideração o a numeração de cada imagem, considerando que imagens de fatias com a mesma numeração 
poderiam representar regiões similares, ou ao menos próximas, entre os pacientes.

## Seleção de Atributos
Ao montar o vetor de atributos para cada conjunto de dados, o algoritmo também sorteia imagens para imprimir, imprimindo também sua normalização, sua máscara aplicada, a máscara aplicada ao LBP e seus respectivos histogramas (h1, h2, h3) em escala logarítmica, além do LBP.  

A partir das imagens das máscaras dos histogramas podemos fazer algumas análises qualitativas.

A primeira é que as lesões de AVC geralmente cobrem uma região comparativamente mais extensa que as lesões de EM. Além das imagens (onde as vezes fica difícil de observar a região da imagem com máscara de EM), podemos ver isso pela quantidade de pontos nos histogramas (a maior parte dos histogramas com máscara de AVC chega a intensidades com 10^3 pontos, enquanto os de EM mal chegam a 10^2).

A segunda, a partir dos histogramas das máscaras sem LBP, é que as lesões de AVC geralmente apresentam um pico de intensidade em um valor alto com bastante dispersão desta intensidade, ao passo que as de EM apresentam um pico de intensidade média com pouca dispersão.

Por fim, observando os histogramas de máscara com o LBP, podemos inferir que as lesões de EM apresentam padrões mais uniformes que as de AVC.

A partir dessas observações podemos definir uma ordem de prioridade dos atributos, de modo que as estatísticas de h2 >h3 >h1.

### AVC
![avc_print](https://user-images.githubusercontent.com/51934983/177860852-b43ae532-1106-4987-bc4d-e334e4d05685.png)

### EM
![em_print](https://user-images.githubusercontent.com/51934983/177860928-1da6dab0-0f68-46b4-a375-7f3e7cc90da2.png)

### TEST 
![test_print](https://user-images.githubusercontent.com/51934983/177861384-872ef520-ed28-415c-96d8-13740792f593.png)

### SLE
![sle_print](https://user-images.githubusercontent.com/51934983/177861534-9319f2a2-5b61-42fa-85d9-027087713cd9.png)

# Metodologia
## Separando os pacientes em grupo de treino e validação
Como sugerido na proposta da A11 não dividi os dados de um mesmo paciente entre teste e validação. Porém, por conta disso, não consegui implementar os métodos de validação cruzada do sklearn. Testei o classificador para múltiplas iterações desta separação aleatória (de aproximadamente 75% treino/ 25% validação), uma a cada duas vezes a classificação foi satisfatória. Minha hipótese é que esta separação é mais propícia a agrupar os outliers.
## Classificador SVM
Para classificação optei por um SVM linear com 50 iterações, imaginei que esta arquitetura seria menos propícia a overfit, mesmo que perdesse alguma acurácia.

## Resultados de validação
Para avaliar os resultados de validação, utilizei o 'Classification Report' e a matriz de confusão.

![confusion matrix](https://user-images.githubusercontent.com/51934983/177864476-c1ec1540-d07e-49a2-b0da-8cbeae4c6b5d.png)


|        | precision  |  recall |f1-score |support|
| ------ | ---------- | ------- | ------- | ----- |
|AVC     |0.94      |0.99      | 0.97     |  133  |
| EM     |0.99      |0.93      | 0.96     |  113  |
|accuracy|          |          | 0.96     |  246  |
|macro avg|   0.97  |  0.96    |  0.96    |  246  |
|weighted avg|  0.96| 0.96     |  0.96    |  246  |


### Testando a classifiação (A11)
Este classificador obteve 95% de acurácia na Atividade 11 (antes de algumas modificações que fiz aqui no P4).

[Resultados A11 - 146895.txt](https://github.com/LCosta845/Ci-ncia-de-Dados-Aplicada-a-Sa-de/files/9067097/Resultados.A11.-.146895.txt)

# Resultado e Discussão 
## Resultado final

|EM  |  504|
| -- | --- |
|AVC |  190|

Aplicado no dataset de Lúpus, o preditor apontou similaridade com EM para 504 de 694 lesões de LES (72.6%), contra 190 para AVC (27.3%).
## Discussão

O resultado final da classificação foi satisfatório, uma vez que o tipo de lesão com a manifestação mais similar ao Lúpus Eritematoso Sistêmico
é realmente a Esclerose Múltipla (ambas causam lesões desmielinizantes). Porém, imagino que a arquitetura do classificador não foi suficientemente 
robusta, por alguns motivos. Utilizei uma normalização rude, não implementei validação cruzada, não selecionei as features automaticamente, o kernel
do SVM linear não traça hiperplanos muito complexos no espaço de características, o classificador não convergiu em 50 iterações, e também houve muita 
alteração do resultado de validação para diferentes divisões do grupo de treino/validação. No entanto, uma escolha dos atributos de classificação qualitativa,
baseada na visualização das imagens e distribuições, pode ter compensando esses defeitos do classificador.  

### Futuro
Com mais tempo, eu teria tentado normalizações diferentes e outras propriedades de textura. Por exemplo, na A11 utilizei o GLCM mas não consegui
rodar nos dados de SLE. Além disso, queria corrigir a separação treino/validação para validação cruzada, e testar outras arquiteturas do classificador. 
Finalmente, com outro projeto do mesmo tipo, gostaria de tentar uma rede de deep learning convolucional, com mais tempo.
