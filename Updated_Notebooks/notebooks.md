

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

Considerando os três notebooks:

* **Maior nível de degradação:** radial.
* **Maior fator de compressão:** radial (**8:1**).
* **Menor número de medidas:** radial (**≈4.608**).
* **Maior proporção de espaço-k preservado:** cartesiana (**33,3%**).
* **A espiral fica em uma posição intermediária**, preservando aproximadamente **25% do espaço-k** e fornecendo um fator de aceleração próximo de **4×**.
