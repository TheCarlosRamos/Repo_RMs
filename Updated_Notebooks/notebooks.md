

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

O modelo **espiral** opera em um regime intermediário, preservando aproximadamente um quarto das medições e exigindo maior capacidade de inferência do modelo de difusão.

O modelo **radial** representa o experimento mais agressivo, utilizando apenas cerca de 12,5% das medições originais e exigindo que o modelo reconstrua aproximadamente 87,5% das informações ausentes.

