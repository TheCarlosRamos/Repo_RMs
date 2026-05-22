

#  MRI Reconstruction with Conditioned Diffusion (Cartesian, Radial & Spiral)

Este projeto implementa reconstrução de imagens de ressonância magnética (MRI) subamostradas utilizando **modelos de difusão condicionada**, explorando diferentes estratégias de amostragem no espaço-k:

*  Cartesiana
*  Radial
*  Espiral

O objetivo é comparar o impacto do padrão de aquisição na qualidade da reconstrução.

***

##  Dataset

* Dataset: **fastMRI (multi-coil knee)**
* Caminho utilizado:
  ```bash
  /kaggle/input/datasets/arafatshovon/fastmri-kneemulticoil
  ```
* Total de arquivos: **194 volumes** 
Cada amostra:

* Seleciona um slice aleatório
* Trabalha no espaço-k complexo (real + imaginário)

***

##  Pipeline

### 1. Subamostragem do espaço-k

Cada abordagem utiliza uma máscara distinta:

####  Cartesiana

* Linhas horizontais com:
  * Região central densa (low-frequency)
  * Amostragem aleatória nas demais
* Fator de aceleração \~4 
####  Radial

* Linhas passando pelo centro em múltiplos ângulos
* Número fixo de raios (ex: 30) 
####  Espiral

* Máscara baseada em função senoidal radial + angular
* Padrão contínuo não-linear 
***

### 2. Conversão para imagem

Transformação:

```python
ifft2 → magnitude → combinação de coils → normalização
```

***

### 3. Modelo de Difusão

* Passos de difusão: **T = 50** * Ruído progressivo:
  * β ∈ \[1e-4, 0.02]
* Predição de ruído (ε-model)

#### Arquitetura (UNet simplificada):

* Entrada: 2 canais (imagem ruidosa + condição)
* Camadas:
  * Conv → ReLU → Conv → ReLU (stack)
* Saída: 1 canal (ruído previsto) 
***

### 4. Condicionamento

O modelo recebe:

```text
[x_t (imagem ruidosa), imagem subamostrada]
```

***

### 5. Loss

Mean Squared Error (MSE):

```math
L = || ε_pred - ε_real ||²
```

***

### 6. Reconstrução (Sampling)

Processo reverso:

* Inicia com ruído puro
* Aplica passos de denoising
* **Impõe consistência no espaço-k** a cada etapa:

```python
k = mask * k_us + (1-mask) * k_pred
```

Isso garante fidelidade aos dados medidos.

***

##  Resultados

###  Cartesiana

* **PSNR médio:** 26.36 dB
* **SSIM médio:** 0.6105 
 Melhor desempenho geral  
 Estrutura preservada  
 Pode gerar artefatos de aliasing direcional

***

###  Radial

* **PSNR:** 22.28 dB
* **SSIM:** 0.5393 
 Robustez a aliasing  
 Menos artefatos estruturados  
 Menor fidelidade quantitativa

***

###  Espiral

* **PSNR:** 23.01 dB
* **SSIM:** 0.6754 
 Melhor SSIM (percepção estrutural)  
 Reconstruções mais suaves  
 Maior instabilidade durante treino (picos de loss observados)

***

##  Comparação Geral

| Trajetória | PSNR ↑      | SSIM ↑      | Característica principal    |
| ---------- | ----------- | ----------- | --------------------------- |
| Cartesiana |  Mais alto | Médio       | Melhor fidelidade numérica  |
| Radial     | Baixo       | Baixo       | Robustez a ruído            |
| Espiral    | Médio       |  Mais alto | Melhor qualidade perceptual |

***

##  Observações

* O **condicionamento com imagem subamostrada** foi essencial para estabilidade.
* A **consistência no espaço-k** melhorou bastante os resultados.
* Modelos de difusão conseguem:
  * Reconstruir detalhes finos
  * Reduzir artefatos clássicos de undersampling

***

##  Possíveis melhorias

* Usar UNet mais profunda (skip-connections reais)
* Aumentar número de steps (T)
* Treinar com batch > 1
* Adicionar atenção espacial
* Testar outras strategies de máscara (Poisson-disc)
* Aplicar multi-scale supervision

***

##  Conclusão

* A escolha da trajetória impacta diretamente:
  * Qualidade perceptual
  * Fidelidade quantitativa

 **Cartesiana**: melhor PSNR (mais precisa)  
 **Espiral**: melhor SSIM (mais agradável visualmente)  
 **Radial**: mais robusta, porém menos precisa

O modelo de difusão condicionado demonstrou ser uma abordagem eficaz e flexível para reconstrução de MRI sob diferentes padrões de aquisição.


