

# Comparação 

| Métrica             | Cartesiana | Espiral | Radial  |
| ------------------- | ---------- | ------- | ------- |
| Dimensão            | 192×192    | 192×192 | 192×192 |
| Pixels              | 36.864     | 36.864  | 36.864  |
| Maior degradação    | `t=T`      | `t=T`   | `t=T`   |
| Espaço-k amostrado  | 33,3%      | ≈25%    | ≈12,5%  |
| Medidas             | 12.288     | ≈9.216  | ≈4.608  |
| Fator de aceleração | 3×         | ≈4×     | 8×      |
| Compressão          | 3:1        | ≈4:1    | 8:1     |
| Dados removidos     | 66,7%      | ≈75%    | 87,5%   |

## Considerando os três modelos:

* **Maior nível de degradação:** radial.
* **Maior fator de compressão:** radial (**8:1**).
* **Menor número de medidas:** radial (**≈4.608**).
* **Maior proporção de espaço-k preservado:** cartesiana (**33,3%**).
* **A espiral fica em uma posição intermediária**, preservando aproximadamente **25% do espaço-k** e fornecendo um fator de aceleração próximo de **4×**.

## Desempenho Comparativo dos Modelos

| Modelo     | Dimensão da Imagem | Medidas Adquiridas | Espaço-k Preservado | Dados Removidos | Fator de Compressão | Fator de Aceleração | Dificuldade da Reconstrução |
| ---------- | ------------------ | ------------------ | ------------------- | --------------- | ------------------- | ------------------- | --------------------------- |
| Cartesiano | 192 × 192          | 12.288             | 33,3%               | 66,7%           | 3:1                 | 3×                  | Baixa                       |
| Espiral    | 192 × 192          | ≈ 9.216            | ≈ 25%               | ≈ 75%           | ≈ 4:1               | ≈ 4×                | Média                       |
| Radial     | 192 × 192          | ≈ 4.608            | ≈ 12,5%             | ≈ 87,5%         | 8:1                 | 8×                  | Alta                        |

### Interpretação

* **Modelo Cartesiano**

  * Mantém a maior quantidade de informações do espaço-k.
  * Apresenta a menor dificuldade de reconstrução.
  * Serve como cenário de menor degradação entre os experimentos realizados.

* **Modelo Espiral**

  * Representa um cenário intermediário entre os modelos cartesiano e radial.
  * Remove aproximadamente três quartos das medições originais.
  * Exige maior capacidade de generalização do modelo de difusão para recuperar as frequências ausentes.

* **Modelo Radial**

  * Corresponde ao cenário mais severo de subamostragem.
  * Utiliza apenas aproximadamente 12,5% das medições originais.
  * Apresenta o maior fator de compressão e a maior dificuldade de reconstrução.


### Resumo

O modelo **cartesiano** apresenta o cenário mais favorável para reconstrução, preservando aproximadamente um terço do espaço-k original.

# Resultados Experimentais

Foram comparados os desempenhos das reconstruções iniciais (*Zero-Filled*) e das reconstruções obtidas pelo modelo de difusão para os três padrões de amostragem investigados: cartesiano, radial e espiral.

## PSNR (Peak Signal-to-Noise Ratio)

Valores maiores indicam melhor qualidade de reconstrução.

| Padrão     | Zero-Filled | Difusão    | Variação |
| ---------- | ----------- | ---------- | -------- |
| Espiral    | 0.1448      | 0.1238     | -0.0210  |
| Radial     | **0.1602**  | **0.1356** | -0.0246  |
| Cartesiano | 0.1245      | 0.1093     | -0.0152  |

### Observações

* O padrão **radial** apresentou o maior PSNR tanto na reconstrução inicial quanto após a difusão.
* O modelo de difusão reduziu o PSNR em todos os cenários avaliados.
* A menor degradação relativa ocorreu no padrão **cartesiano**.

---

## SSIM (Structural Similarity Index Measure)

Valores maiores indicam melhor preservação estrutural.

| Padrão     | Zero-Filled | Difusão    | Variação |
| ---------- | ----------- | ---------- | -------- |
| Espiral    | 0.4022      | 0.3740     | -0.0282  |
| Radial     | 0.4518      | 0.4479     | -0.0039  |
| Cartesiano | **0.4975**  | **0.4777** | -0.0198  |

### Observações

* O padrão **cartesiano** apresentou a melhor preservação estrutural.
* O padrão **radial** foi o mais estável após a aplicação da difusão, apresentando a menor redução de SSIM.
* O padrão **espiral** apresentou a maior degradação estrutural após o processo difusivo.

---

## NMSE (Normalized Mean Squared Error)

Valores menores indicam melhor reconstrução.

| Padrão     | Zero-Filled | Difusão | Variação |
| ---------- | ----------- | ------- | -------- |
| Espiral    | **1.0413**  | 1.0458  | +0.0045  |
| Radial     | **1.0378**  | 1.0433  | +0.0055  |
| Cartesiano | **1.0436**  | 1.0461  | +0.0025  |

### Observações

* O menor NMSE inicial foi obtido pelo padrão **radial**.
* O modelo de difusão aumentou o erro em todos os cenários avaliados.
* O padrão **cartesiano** apresentou a menor deterioração relativa do NMSE.

---

## Comparação Geral

| Critério                                  | Melhor Desempenho |
| ----------------------------------------- | ----------------- |
| Maior PSNR                                | Radial            |
| Maior SSIM                                | Cartesiano        |
| Menor NMSE                                | Radial            |
| Maior estabilidade após difusão           | Radial            |
| Menor impacto negativo da difusão no PSNR | Cartesiano        |
| Menor impacto negativo da difusão no SSIM | Radial            |

---

## Conclusões

Os resultados indicam que o processo de difusão, na configuração atualmente utilizada, não produziu ganhos quantitativos em relação às reconstruções *Zero-Filled* para nenhum dos padrões de amostragem avaliados.

O padrão **radial** apresentou o melhor desempenho global em termos de PSNR e NMSE, além de demonstrar elevada robustez estrutural após a aplicação da difusão.

O padrão **cartesiano** obteve o maior SSIM absoluto, indicando melhor preservação das estruturas anatômicas presentes na imagem reconstruída.

O padrão **espiral** apresentou o comportamento menos favorável, especialmente em relação à preservação estrutural medida pelo SSIM.


O modelo **espiral** opera em um regime intermediário, preservando aproximadamente um quarto das medições e exigindo maior capacidade de inferência do modelo de difusão.

O modelo **radial** representa o experimento mais agressivo, utilizando apenas cerca de 12,5% das medições originais e exigindo que o modelo reconstrua aproximadamente 87,5% das informações ausentes.

