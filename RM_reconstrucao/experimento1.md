
#   DOCUMENTAÇÃO DO NOTEBOOK — MRI RECONSTRUCTION

***

#  1. Objetivo

Este notebook implementa um pipeline de **reconstrução de imagens de ressonância magnética (MRI)** a partir de dados **subamostrados no domínio k-space**, utilizando Deep Learning.

### Problema

Reconstruir uma imagem MRI de alta qualidade a partir de aquisição incompleta (undersampled), reduzindo tempo de aquisição no scanner.

***

#  2. Pipeline Geral

O sistema segue a seguinte estrutura:

    k-space completo (GT)
            ↓
    aplicação de máscara (undersampling)
            ↓
    k-space subamostrado
            ↓
    UNet (rede neural)
            ↓
    k-space reconstruído
            ↓
    IFFT → imagem
            ↓
    avaliação (PSNR, SSIM)

***

#  3. Tipo de Aquisição (Trajetória)

##  Utilizado no notebook:

###  Cartesian Undersampling (1D mask)

*   Linhas horizontais no k-space
*   Região central totalmente amostrada
*   Restante amostrado aleatoriamente

```python
mask[row] = 1 com probabilidade 1/accel
```

***

##  Limitações

O método não simula trajetórias reais como:

*    radial
*    spiral
*    gaussian / variable density realista

***

##  Justificativa

*   Mais simples de implementar
*   Padrão em benchmarks (fastMRI)
*   Facilita treinamento supervisionado

***

#  4. Dataset

*    **fastMRI knee multicoil**
*   \~194 volumes
*   uso de slices aleatórios por batch

***

## Pré-processamento

*   Conversão para PyTorch complex tensor
*   Separação real/imag (2 canais)
*    **Normalização com mean/std (CRÍTICA)**

```python
kspace = (kspace - mean) / std
```

 Necessária para evitar colapso do modelo

***

#  5. Modelo

##  Arquitetura

*   Rede baseada em **UNet simplificada**
*   Entrada: `[B, Nc, H, W, 2]`
*   Processamento coil-wise
*   saída: mesma dimensão

***

##  Residual Learning (CRÍTICO)

```python
output = input + network(input)
```

***

##  Por que usar residual?

*   evita saída zero
*   acelera convergência
*   facilita aprendizado da correção

***

#  6. Funções principais

***

##  FFT Inversa

```python
ifft2c → magnitude da imagem
```

***

##  Data Consistency

```python
k_final = mask * k_us + (1-mask) * k_pred
```

***

###  Uso correto:

| Fase        | Uso        |
| ----------- | ---------- |
| Treinamento |  NÃO usar |
| Inferência  |  usar     |

***

#  7. Loss Function

##  Loss final utilizada:

```python
loss = || (k_pred - k_gt) * (1 - mask) ||^2
```

***

##  Ideia

*   penaliza apenas regiões faltantes
*   força o modelo a reconstruir o que não foi medido

***

##  Motivo

Evita solução trivial:

    copiar entrada → loss ~ 0 

***

#  8. Estratégias críticas aplicadas

***

##  1. Normalização (mean/std)

✔ resolve escala  
✔ estabiliza gradiente

***

##  2. Remoção de data consistency no treino

✔ libera aprendizado  
✔ evita bloqueio do modelo

***

##  3. Residual learning

✔ evita colapso para zero

***

##  4. Loss nas regiões faltantes

✔ força reconstrução real

***

#  9. Treinamento

*   Otimizador: Adam
*   LR: `1e-4`
*   Steps: 400
*   Batch size: 1

***

## Comportamento da loss

*   não converge para zero
*   oscila (normal)
*   aprende padrões estruturais

***

#  10. Avaliação

***

##  Métricas

### PSNR

    28.15 

→ alta qualidade

***

### SSIM

    0.95 

→ excelente preservação estrutural

***

##  Interpretação

| Métrica    | Significado          |
| ---------- | -------------------- |
| PSNR > 28  | reconstrução boa     |
| SSIM > 0.9 | estrutura preservada |

***

#  11. Resultados Visuais

### Observado:

| Tipo           | Resultado  |
| -------------- | ---------- |
| Undersampled   | borrado    |
| Reconstruction | nítido    |
| Ground Truth   | referência |

***

##  Conclusão visual

*   estrutura anatômica recuperada
*   ainda leve suavização (esperado)

***

#  12. Resultado Final

✔ pipeline funcional  
✔ converge corretamente  
✔ reconstrução real  
✔ métricas elevadas

***

#  13. Limitações

***

## 1. Máscara simples

*   não representa trajetórias reais MRI

***

## 2. Modelo pequeno

*   UNet simplificada
*   limita qualidade máxima

***

## 3. Diretamente supervisionado

*   não modela incerteza

***

#  14. Próximas melhorias

***

##  Curto prazo

*   gaussian mask
*   poisson sampling
*   UNet mais profundo

***

##  Médio prazo

*   variational networks
*   iterative reconstruction

***

##  Avançado (pesquisa)

*   diffusion models
*   física MRI + aprendizado
*   radial/spiral sampling

***

#  15. Tipo de Abordagem

##  Classificação do modelo:

| Categoria           | Status |
| ------------------- | ------ |
| Reconstrução direta | ✅      |
| Determinística      | ✅      |
| Probabilística      | ❌      |
| Difusão             | ❌      |

***

##  Conclusão técnica

Este notebook implementa:

 **Deep Learning supervisionado para reconstrução MRI com UNet residual e undersampling cartesian**

***

#  16. Conclusão Geral

O sistema demonstra que:

*   é possível reconstruir MRI a partir de dados incompletos
*   técnicas simples (UNet + mask) já produzem resultados fortes
*   escolhas corretas de loss, normalização e arquitetura são determinantes

***




