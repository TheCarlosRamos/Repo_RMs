
# Relatório de Otimização – Reconstrução CS-MRI

## 1. Visão Geral do Experimento

Este relatório documenta o processo de otimização de um pipeline de reconstrução por *Compressed Sensing* (CS) aplicado a imagens de ressonância magnética (MRI) do cérebro. O objetivo foi maximizar a qualidade das imagens reconstruídas (PSNR e SSIM) e minimizar o tempo de processamento, ajustando três fatores principais:

- **Número de iterações** do algoritmo FISTA.
- **Taxa de decaimento da regularização Wavelet** (parâmetro de *continuation*).
- **Utilização de hardware** (migração parcial das operações para GPU).

---

## 2. Metodologia e Alterações Implementadas

### 2.1 Hardware
- **Versão Original**: Todas as operações (FFT, Wavelet, TV) executadas na CPU via NumPy e PyWavelets.
- **Versão Otimizada**: Implementação híbrida. As operações de FFT, Gradiente da TV e *Data Consistency* foram migradas para GPU (PyTorch). A transformada Wavelet permaneceu na CPU (PyWavelets) por questões de compatibilidade, resultando em um excelente equilíbrio entre velocidade e simplicidade.

### 2.2 Regularização com *Continuation*
O parâmetro de regularização wavelet ($\lambda_{wavelet}$) não é fixo. Ele começa em um valor alto (ex: 0.005) e decai exponencialmente ao longo das iterações para permitir que detalhes finos emergiam gradualmente.

A fórmula original era:
```python
lam_w_seq = lam_wavelet * (10.0 ** torch.linspace(0, -2, n_iter))
```
Isso fazia com que $\lambda_{wavelet}$ chegasse ao valor final de `0.005 * 10⁻² = 0.00005`.

**Alteração Crítica**: Modificamos o expoente final de `-2` para `-1.5`:
```python
lam_w_seq = lam_wavelet * (10.0 ** torch.linspace(0, -1.5, n_iter))
```
Agora o valor mínimo de $\lambda_{wavelet}$ é `0.005 * 10⁻¹·⁵ ≈ 0.000158`, garantindo que a regularização wavelet nunca desapareça completamente no final do processo.

---

## 3. Resultados Comparativos

Foram testadas três configurações distintas. A tabela abaixo resume as médias obtidas com 30 imagens do dataset:

| Configuração | Iterações | Decaimento (10^exp) | Hardware | PSNR Médio (dB) | SSIM Médio | NMSE Médio | Tempo Médio (s) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Baseline (Original)** | 200 | -2.0 | CPU (NumPy) | 27,93 | 0,780 | 0,00894 | 1,41 |
| **Teste (Mais iterações)** | 500 | -2.0 | Híbrido (GPU/CPU) | 27,91 | 0,780 | 0,00890 | 1,83 |
| **Otimizado (Melhor)** | 200 | **-1.5** | Híbrido (GPU/CPU) | **30,42** | **0,840** | **0,00796** | **0,71** |

---

## 4. Análise Detalhada dos Experimentos

### Experimento 1: Baseline (200 iterações, decaimento -2.0)
- **Desempenho**: Sólido, com PSNR médio de ~28 dB.
- **Limitação**: Nos estágios finais, o $\lambda_{wavelet}$ torna-se extremamente baixo (0.00005). Com uma regularização praticamente nula, o algoritmo começa a "ajustar" o ruído e pequenos artefatos presentes nos dados subamostrados, degradando a qualidade final.

### Experimento 2: Aumento de iterações (500 iterações, decaimento -2.0)
- **Resultado**: PSNR e SSIM praticamente idênticos ao baseline.
- **Conclusão**: Apenas aumentar o número de iterações **não resolve** o problema de overfitting causado pela regularização zero no final. Além disso, o tempo de processamento aumentou em 30%, tornando esta abordagem ineficiente.

### Experimento 3: Ajuste do Decaimento (200 iterações, decaimento -1.5) 
- **Resultado**: **Salto de qualidade de ~2,5 dB** no PSNR e melhora significativa no SSIM.
- **Explicação**: Ao limitar o decaimento em `-1.5`, mantemos uma regularização residual de aproximadamente `0.00016`. Essa pequena penalidade wavelet atua como um **filtro suavizante suave** nas últimas iterações, suprimindo ruídos de alta frequência sem borrar as bordas anatômicas. O resultado são imagens mais nítidas e com maior fidelidade estrutural.
- **Eficiência**: O tempo médio caiu para **0,71 segundos por imagem** (quase a metade do tempo da CPU original) devido ao uso da GPU nas operações mais pesadas.

---

## 5. Estatísticas Detalhadas da Configuração Otimizada (Melhor Caso)

Para validar a robustez do modelo otimizado, analisamos a distribuição das métricas:

| Métrica | Média | Desvio Padrão | Mínimo | Máximo |
| :--- | :--- | :--- | :--- | :--- |
| **PSNR (CS)** | 30,42 dB | 3,33 | 24,28 dB | 36,22 dB |
| **SSIM (CS)** | 0,840 | 0,066 | 0,701 | 0,931 |
| **NMSE (CS)** | 0,00796 | 0,00179 | 0,00502 | 0,01161 |
| **Tempo (CS)** | 0,71 s | 0,025 | 0,68 s | 0,77 s |

**Observação**: O desvio padrão de 3,33 dB indica que algumas imagens (com menor conteúdo estrutural ou maior ruído intrínseco) ainda são desafiadoras. A imagem com PSNR de 24,28 dB sugere que ajustes finos nos hiperparâmetros iniciais ($\lambda_{wavelet}$ e $\lambda_{tv}$) podem ser necessários para casos específicos.

---

## 6. Conclusões e Recomendações Finais

A otimização do pipeline de CS-MRI foi extremamente bem-sucedida. As principais lições extraídas são:

1. **Regularização é mais importante que iterações**: A qualidade da reconstrução depende criticamente de manter uma força de regularização adequada durante todo o processo. Deixar a regularização zerar no final causa *overfitting* aos dados ruidosos.
2. **GPU híbrida é a escolha certa**: A migração seletiva (FFT e TV na GPU, Wavelet na CPU) reduziu o tempo pela metade, mesmo com uma implementação não nativa, demonstrando excelente custo-benefício.
3. **Parâmetros ideais encontrados**:
   - `n_iter = 200`
   - `lam_wavelet = 0.005`
   - `lam_tv = 0.001`
   - `Decaimento = -1.5`

### Próximos Passos Sugeridos:
- **Busca em Grade Fina**: Testar `lam_wavelet` entre 0.003 e 0.008 e `lam_tv` entre 0.0005 e 0.002 para tentar elevar o PSNR médio para > 31 dB.
- **Early Stopping**: Implementar um critério de parada baseado na variação da solução (`torch.norm(x - x_prev)`) pode reduzir ainda mais o tempo em imagens mais fáceis.
- **Visualização**: Gerar gráficos de convergência (curva de PSNR vs iterações) para ilustrar o comportamento do decaimento -1.5 vs -2.0.

---

## 7. Impacto Visual (Qualitativo)
Embora não haja imagens inseridas neste relatório, o ganho de 2,5 dB no PSNR geralmente se traduz em:
- **Bordas mais nítidas** (menos efeito *blur*).
- **Supressão eficaz de artefatos* em regiões de baixo contraste.
- **Preservação de pequenas estruturas** (como sulcos corticais) que tendem a ser suavizadas quando a regularização é muito fraca ou muito forte.

