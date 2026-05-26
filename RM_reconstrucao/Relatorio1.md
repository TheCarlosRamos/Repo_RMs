

**Relatório Técnico: Avaliação de Trajetórias de Amostragem no k-space para Reconstrução de Imagens por Difusão**

**1. Introdução e Objetivo**

O presente experimento tem como objetivo avaliar o impacto de diferentes trajetórias de amostragem no k-space (Cartesiana, Radial e Espiral) na reconstrução de imagens de Ressonância Magnética (MRI). O processo de reconstrução utiliza um modelo de difusão condicionado (UNet com predição de ruído) com consistência de dados. As métricas de avaliação incluem a relação sinal-ruído de pico (PSNR), o índice de similaridade estrutural (SSIM) e a evolução da função de perda (loss) durante o treinamento.

**2. Metodologia**

Foram avaliadas as seguintes configurações de trajetória:

1.  **Cartesiana (Cartesian):** Amostragem linear padrão.
2.  **Radial (Radial):** Amostragem com 30 linhas (configuração antiga) e 70 linhas (configuração nova).
3.  **Espiral (Spiral):** Amostragem baseada em curvas espirais.

O treinamento do modelo foi conduzido por 3000 passos (steps) para cada configuração, utilizando o conjunto de dados fastMRI (multicanal). As métricas finais de PSNR e SSIM foram consolidadas para comparação.

**3. Resultados**

Os resultados obtidos para cada configuração são apresentados a seguir e resumidos na Tabela 1.

**3.1. Trajetória Radial (30 linhas)**
- Perda final (Loss): ~0,02
- PSNR: 22,27
- SSIM: 0,54
- Diagnóstico: Subamostragem excessiva, resultando em baixa qualidade.

**3.2. Trajetória Radial (70 linhas)**
- Perda final (Loss): ~0,018 – 0,02
- PSNR: 23,23
- SSIM: 0,573
- Diagnóstico: Melhoria significativa em relação à configuração com 30 linhas, indicando que o aumento da densidade angular aprimora a reconstrução.

**3.3. Trajetória Cartesiana**
- Perda final (Loss): ~0,015
- PSNR: 26,36
- SSIM: 0,61
- Diagnóstico: Melhor desempenho em termos de precisão por pixel (PSNR).

**3.4. Trajetória Espiral**
- Perda final (Loss): ~0,02
- PSNR: 23,01
- SSIM: 0,67
- Diagnóstico: Melhor desempenho em termos de preservação da estrutura anatômica (SSIM).

**Tabela 1 – Comparação de Métricas por Trajetória de Amostragem**

| Trajetória           | PSNR (dB) | SSIM   | Status de Qualidade                     |
| -------------------- | --------- | ------ | --------------------------------------- |
| Cartesiana           | 26,36     | 0,61   | Melhor precisão de pixel.               |
| Espiral              | 23,01     | **0,67** | Melhor preservação estrutural.        |
| Radial (70 linhas)   | **23,23** | 0,57   | Qualidade intermediária; melhoria clara.|
| Radial (30 linhas)   | 22,27     | 0,54   | Qualidade inferior; subamostragem severa.|

**4. Análise e Discussão**

**4.1. Relação entre Loss e Qualidade da Imagem**
Observa-se que todos os experimentos atingiram um valor de perda (loss) similar (~0,02). Este indicador confirma que o modelo de difusão aprendeu corretamente o processo de denoising em todos os casos. No entanto, é crucial notar que um **loss baixo não é sinônimo de imagem de alta qualidade**, pois o loss reflete o erro na predição do ruído, enquanto o PSNR e o SSIM avaliam a fidelidade e a estrutura da imagem final reconstruída.

**4.2. Análise do PSNR (Precisão Numérica)**
A métrica PSNR, que avalia a precisão pixel a pixel, apresentou a seguinte ordem de desempenho: **Cartesiana > Radial > Espiral**. A trajetória Cartesiana, por sua estrutura regular em grade, facilita a interpolação e apresenta menor ambiguidade para o modelo, resultando em maior precisão numérica. Conclui-se que o PSNR está diretamente relacionado à simplicidade matemática da trajetória.

**4.3. Análise do SSIM (Qualidade Estrutural)**
A métrica SSIM, que avalia a preservação de bordas e estruturas anatômicas, apresentou ordem de desempenho distinta: **Espiral > Cartesiana > Radial**. A trajetória Espiral cobre o k-space de forma contínua, capturando melhor as transições suaves e as estruturas globais da imagem, o que justifica seu SSIM superior. Conclui-se que o SSIM está mais associado à qualidade da informação amostrada do que à sua regularidade.

**4.4. Análise da Trajetória Radial**
O experimento demonstra que a trajetória radial é altamente dependente da densidade de amostragem. A configuração com 30 linhas mostrou-se excessivamente agressiva, resultando em perda de informação e baixos PSNR e SSIM. O aumento para 70 linhas proporcionou maior cobertura angular, elevando ambas as métricas. A conclusão experimental é direta: **mais linhas na amostragem radial levam a uma melhor qualidade de reconstrução**.

**5. Conclusões e Trabalhos Futuros**

**5.1. Conclusões Principais**
Os experimentos realizados demonstram, de forma conclusiva, que a trajetória de amostragem no k-space tem um impacto direto e significativo na qualidade da reconstrução obtida por modelos de difusão. Os principais achados são:

- A trajetória **Cartesiana maximiza a precisão por pixel (PSNR)**.
- A trajetória **Espiral maximiza a preservação estrutural (SSIM)**.
- A trajetória **Radial apresenta qualidade intermediária e melhora com o aumento da densidade angular**.

A descoberta fundamental é expressa na seguinte observação: *A qualidade da amostragem radial melhora significativamente com o aumento da densidade angular, mas ainda fica aquém da amostragem espiral em termos de preservação estrutural.*

**5.2. Recomendações para Trabalhos Futuros**
Para dar continuidade a esta pesquisa e potencialmente atingir um melhor equilíbrio entre PSNR e SSIM, sugerem-se as seguintes abordagens:

1.  **Aumento da Densidade Radial:** Testar a trajetória radial com 90 linhas.
2.  **Radial Não Uniforme:** Implementar uma distribuição angular não linear (ex: usando `np.linspace(0, np.pi, num_lines)**1.5`) para aumentar a densidade de linhas no centro do k-space.
3.  **Máscara Gaussiana:** Combinar a amostragem radial com uma densidade gaussiana, concentrando a amostragem no centro para melhorar o SSIM.
4.  **Aumento do Treinamento:** Estender o treinamento de 3000 para 6000 passos.
5.  **Validação Robusta:** Avaliar as métricas em uma amostra maior de imagens (de 10 para 50).

**Resumo:** O experimento confirmou o aprendizado do modelo (loss), evidenciou o trade-off entre PSNR (Cartesiana) e SSIM (Espiral), e demonstrou a melhoria da trajetória radial com o aumento do número de linhas. 
