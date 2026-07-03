# Comparação

| Métrica             | Cartesiana | Espiral | Radial  |
| ------------------- | ---------- | ------- | ------- |
| Dimensão            | 192×192    | 192×192 | 192×192 |
| Pixels              | 36.864     | 36.864  | 36.864  |
| Maior degradação    | `t=T`      | `t=T`   | `t=T`   |
| Espaço-k amostrado  | 33,3%      | 25,0%   | 12,5%   |
| Medidas             | 12.288     | 9.216   | 4.608   |
| Fator de aceleração | 3×         | 4×      | 8×      |
| Compressão          | 3:1        | 4:1     | 8:1     |
| Dados removidos     | 66,7%      | 75,0%   | 87,5%   |

## Considerando os três modelos

* **Maior nível de degradação:** radial.
* **Maior fator de compressão:** radial (**8:1**).
* **Menor número de medidas:** radial (**4.608**).
* **Maior proporção de espaço-k preservado:** cartesiana (**33,3%**).
* **A espiral ocupa uma posição intermediária**, preservando aproximadamente **25% do espaço-k** e utilizando fator de aceleração de **4×**.

## Desempenho Comparativo dos Modelos

| Modelo     | Dimensão da Imagem | Medidas Adquiridas | Espaço-k Preservado | Dados Removidos | Fator de Compressão | Fator de Aceleração | Dificuldade da Reconstrução |
| ---------- | ------------------ | ------------------ | ------------------- | --------------- | ------------------- | ------------------- | --------------------------- |
| Cartesiano | 192 × 192          | 12.288             | 33,3%               | 66,7%           | 3:1                 | 3×                  | Baixa                       |
| Espiral    | 192 × 192          | 9.216              | 25,0%               | 75,0%           | 4:1                 | 4×                  | Média                       |
| Radial     | 192 × 192          | 4.608              | 12,5%               | 87,5%           | 8:1                 | 8×                  | Alta                        |

## Interpretação

### Modelo Cartesiano

* Mantém a maior quantidade de informações do espaço-k.
* Apresenta a menor dificuldade de reconstrução.
* Corresponde ao cenário menos severo de subamostragem.
* Obteve os melhores resultados quantitativos após a reconstrução por difusão.

### Modelo Espiral

* Representa um cenário intermediário entre os modelos cartesiano e radial.
* Remove aproximadamente três quartos das medições originais.
* Exige maior capacidade de inferência do modelo para recuperar as frequências ausentes.

### Modelo Radial

* Corresponde ao cenário mais severo de subamostragem.
* Utiliza apenas aproximadamente 12,5% das medições originais.
* Apresenta o maior fator de compressão e a maior dificuldade de reconstrução.
* Mesmo nesse cenário extremo, o modelo de difusão foi capaz de produzir melhorias quantitativas significativas.

## Resumo

O modelo **cartesiano** apresenta o cenário mais favorável para reconstrução, preservando aproximadamente um terço do espaço-k original.

O modelo **espiral** opera em um regime intermediário, preservando aproximadamente um quarto das medições originais.

O modelo **radial** representa o experimento mais agressivo, exigindo que o modelo reconstrua aproximadamente **87,5% das informações ausentes**.

# Resultados Experimentais

Foram comparados os desempenhos das reconstruções iniciais (*Zero-Filled*) e das reconstruções obtidas pelo modelo de difusão para os três padrões de amostragem investigados.

## PSNR (Peak Signal-to-Noise Ratio)

Valores maiores indicam melhor qualidade de reconstrução.

| Padrão     | Zero-Filled | Difusão   | Ganho        |
| ---------- | ----------- | --------- | ------------ |
| Cartesiano | 31.05       | **38.40** | **+7.35 dB** |
| Espiral    | 30.33       | **32.72** | **+2.39 dB** |
| Radial     | 31.84       | **34.10** | **+2.26 dB** |

### Observações

* O modelo de difusão aumentou o PSNR em todos os cenários avaliados.
* O maior ganho absoluto foi obtido pela trajetória **cartesiana**.
* Mesmo com apenas 12,5% das medições disponíveis, a trajetória **radial** apresentou melhora superior a 2 dB.

---

## SSIM (Structural Similarity Index Measure)

Valores maiores indicam melhor preservação estrutural.

| Padrão     | Zero-Filled | Difusão    | Ganho       |
| ---------- | ----------- | ---------- | ----------- |
| Cartesiano | 0.8828      | **0.9632** | **+0.0803** |
| Espiral    | 0.8520      | **0.9030** | **+0.0510** |
| Radial     | 0.8434      | **0.9027** | **+0.0593** |

### Observações

* O modelo de difusão aumentou o SSIM em todos os experimentos.
* A trajetória **cartesiana** apresentou a melhor preservação estrutural final.
* A trajetória **radial** apresentou ganho estrutural superior ao observado na trajetória espiral.

---

## NMSE (Normalized Mean Squared Error)

Valores menores indicam melhor reconstrução.

| Padrão     | Zero-Filled | Difusão     | Redução      |
| ---------- | ----------- | ----------- | ------------ |
| Cartesiano | 0.02402     | **0.00451** | **-0.01951** |
| Espiral    | 0.02849     | **0.01663** | **-0.01186** |
| Radial     | 0.02013     | **0.01199** | **-0.00814** |

### Observações

* O modelo de difusão reduziu o NMSE em todos os cenários avaliados.
* A trajetória **cartesiana** apresentou a maior redução absoluta do erro.
* A trajetória **radial**, mesmo utilizando apenas 12,5% do espaço-k, apresentou redução superior a 40% do NMSE.

---

## Comparação Geral

| Critério                                | Melhor Desempenho |
| --------------------------------------- | ----------------- |
| Maior PSNR final                        | Cartesiano        |
| Maior SSIM final                        | Cartesiano        |
| Menor NMSE final                        | Cartesiano        |
| Maior ganho em PSNR                     | Cartesiano        |
| Maior ganho em SSIM                     | Cartesiano        |
| Maior redução de NMSE                   | Cartesiano        |
| Maior robustez sob subamostragem severa | Radial            |

---

## Conclusões

Os resultados demonstram que o modelo de difusão produziu melhorias quantitativas consistentes em relação às reconstruções *Zero-Filled* em todos os cenários avaliados.

A trajetória **cartesiana** apresentou o melhor desempenho global, alcançando:

* **PSNR:** 38,40 dB
* **SSIM:** 0,9632
* **NMSE:** 0,00451

Além disso, foi o cenário que apresentou os maiores ganhos proporcionados pelo modelo de difusão.

A trajetória **radial**, apesar de preservar apenas **12,5%** do espaço-k original e utilizar um fator de aceleração de **8×**, apresentou desempenho superior ao da trajetória espiral em PSNR e NMSE, indicando que a geometria da amostragem exerce papel importante no processo de reconstrução.

A trajetória **espiral** apresentou desempenho intermediário, confirmando que a quantidade de amostras preservadas não é o único fator determinante para a qualidade da reconstrução.

De forma geral, os resultados indicam que modelos de difusão constituem uma abordagem promissora para reconstrução acelerada de imagens de Ressonância Magnética, sendo capazes de recuperar informações ausentes e melhorar significativamente a qualidade das reconstruções obtidas a partir de dados subamostrados.
