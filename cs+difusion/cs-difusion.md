# Reconstrução de RM com Compressed Sensing e modelos de difusão

Esta pasta compara três estratégias de subamostragem do espaço-k para reconstrução acelerada de imagens de Ressonância Magnética (RM), combinando uma reconstrução inicial *Zero-Filled* com refinamento por modelo de difusão. Os experimentos estão nos notebooks [cartesiano](cartesian-cs-difusion.ipynb), [radial](cs-difusao-radial.ipynb) e [espiral](cs-difusion-spiral.ipynb).

O objetivo é avaliar como a trajetória de aquisição e a quantidade de dados preservados afetam a qualidade da imagem reconstruída. Os números abaixo são os resultados experimentais registrados para esta comparação; não são médias de várias imagens, a menos que isso seja indicado explicitamente.

## Contexto e fluxo de reconstrução

Em RM, a imagem é relacionada às amostras adquiridas no domínio de Fourier, chamado espaço-k. A aquisição incompleta reduz o tempo de exame, mas torna a reconstrução um problema inverso: várias imagens podem ser compatíveis com os dados medidos. A subamostragem também causa artefatos na reconstrução direta.

O fluxo comparado nos experimentos pode ser resumido assim:

1. Parte-se de uma imagem de referência.
2. A imagem é transformada para o espaço-k.
3. Aplica-se a máscara ou trajetória de amostragem correspondente ao experimento.
4. Calcula-se uma reconstrução inicial *Zero-Filled* com os dados incompletos.
5. O modelo de difusão refina a reconstrução.
6. Comparam-se a reconstrução inicial e a reconstrução refinada usando PSNR, SSIM e NMSE.

O modelo de difusão fornece um prior aprendido sobre imagens e busca recuperar uma imagem plausível a partir da entrada degradada. O modelo não substitui os dados adquiridos: a quantidade e a distribuição das amostras continuam influenciando a qualidade final.

## Trajetórias e níveis de subamostragem

Os três cenários usam imagens de **192 × 192 pixels** (36.864 pixels). A tabela apresenta a quantidade de medidas e os fatores de subamostragem informados na comparação experimental.

| Trajetória | Como amostra o espaço-k | Medidas | Espaço-k preservado | Dados removidos | Fator de aceleração / compressão | Dificuldade de reconstrução |
| --- | --- | ---: | ---: | ---: | ---: | --- |
| Cartesiana | Amostras organizadas em linhas cartesianas | 12.288 | 33,3% | 66,7% | 3× / 3:1 | Baixa |
| Espiral | Trajetória contínua em espiral | 9.216 | 25,0% | 75,0% | 4× / 4:1 | Média |
| Radial | Projeções dispostas radialmente em torno do centro | 4.608 | 12,5% | 87,5% | 8× / 8:1 | Alta |

A comparação registra a maior degradação no estado final do processo de difusão (`t=T`) para os três cenários. A aquisição cartesiana preserva a maior proporção de amostras e representa o caso menos agressivo. A espiral ocupa uma posição intermediária. A radial usa menos medidas e impõe a maior taxa de subamostragem.

### Diferenças práticas entre as trajetórias

- **Cartesiana:** simples e direta para reconstrução em grade; neste experimento, preserva mais dados. A subamostragem cartesiana pode produzir aliasing estruturado, dependendo da máscara.
- **Radial:** distribui as medidas em raios que atravessam o centro do espaço-k. Neste experimento, foi a trajetória mais subamostrada; a reconstrução é classificada como a mais difícil.
- **Espiral:** percorre o espaço-k por uma trajetória contínua e fica entre os cenários cartesiano e radial em quantidade de medidas preservadas e fator de aceleração.

> A tabela descreve os níveis de amostragem e a dificuldade reportados para estes experimentos; não implica que uma trajetória será sempre superior em qualquer protocolo, conjunto de dados ou implementação.

## Resultados quantitativos

Os resultados comparam a reconstrução inicial *Zero-Filled* com a reconstrução após difusão. **PSNR e SSIM maiores são melhores; NMSE menor é melhor.** Os valores não incluem tempo de execução nem intervalos de incerteza na comparação disponível.

| Trajetória | PSNR Zero-Filled (dB) | PSNR difusão (dB) | Ganho PSNR | SSIM Zero-Filled | SSIM difusão | Ganho SSIM | NMSE Zero-Filled | NMSE difusão | Redução NMSE |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Cartesiana | 31,05 | **38,40** | **+7,35 dB** | 0,8828 | **0,9632** | **+0,0803** | 0,02402 | **0,00451** | **−0,01951** |
| Espiral | 30,33 | **32,72** | **+2,39 dB** | 0,8520 | **0,9030** | **+0,0510** | 0,02849 | **0,01663** | **−0,01186** |
| Radial | 31,84 | **34,10** | **+2,26 dB** | 0,8434 | **0,9027** | **+0,0593** | 0,02013 | **0,01199** | **−0,00814** |

### Leitura dos resultados

- A difusão melhorou as três métricas nos três cenários: elevou PSNR e SSIM e reduziu NMSE.
- **Cartesiana** obteve os melhores valores finais e os maiores ganhos reportados: PSNR de 38,40 dB, SSIM de 0,9632 e NMSE de 0,00451.
- **Radial** preservou apenas 12,5% do espaço-k (aceleração de 8×), mas ainda apresentou ganho de PSNR de 2,26 dB e de SSIM de 0,0593 após a difusão.
- **Espiral** apresentou resultados intermediários: PSNR final de 32,72 dB e SSIM final de 0,9030.
- A ordenação das métricas não é idêntica antes e depois do refinamento. Por exemplo, a radial começa com o menor NMSE entre as reconstruções *Zero-Filled*, enquanto a cartesiana apresenta o menor NMSE após a difusão. Isso reforça a importância de considerar conjuntamente subamostragem, reconstrução inicial e refinamento.

## Comparação resumida de trajetória e resultado

| Trajetória | Regime de aquisição neste experimento | PSNR final | SSIM final | NMSE final | Resultado resumido |
| --- | --- | ---: | ---: | ---: | --- |
| Cartesiana | Menos agressivo: 33,3% preservado; 3× | **38,40 dB** | **0,9632** | **0,00451** | Melhor resultado final nas três métricas e maior ganho de PSNR. |
| Espiral | Intermediário: 25,0% preservado; 4× | 32,72 dB | 0,9030 | 0,01663 | Desempenho intermediário nesta comparação. |
| Radial | Mais agressivo: 12,5% preservado; 8× | 34,10 dB | 0,9027 | 0,01199 | Aceleração mais alta; apesar da subamostragem severa, melhora com difusão e supera a espiral em PSNR e NMSE finais. |

## Métricas

- **PSNR (Peak Signal-to-Noise Ratio):** expressa a relação entre o sinal e o erro de reconstrução em escala logarítmica; valores maiores indicam menor erro relativo.
- **SSIM (Structural Similarity Index Measure):** avalia semelhança estrutural, considerando luminância, contraste e estrutura; valores maiores indicam maior semelhança.
- **NMSE (Normalized Mean Squared Error):** erro quadrático médio normalizado pela referência; valores menores indicam menor erro.

Os ganhos e as diferenças na tabela reproduzem os valores reportados na documentação comparativa do repositório, com as precisões ali apresentadas. Pequenas discrepâncias aritméticas podem decorrer do arredondamento dos valores exibidos.

## Conclusão

Nos resultados apresentados, o modelo de difusão melhora a reconstrução *Zero-Filled* para todas as trajetórias e métricas avaliadas. A aquisição **cartesiana** oferece a melhor qualidade final, mas também preserva a maior quantidade de dados e usa a menor aceleração. A aquisição **radial** é a mais agressiva e permite a maior aceleração (8×), com qualidade final inferior à cartesiana, mas superior à espiral em PSNR e NMSE. A **espiral** fica entre as outras em subamostragem, embora seus valores finais não superem os da radial nesta comparação.

Portanto, os resultados evidenciam um compromisso entre aceleração e qualidade: a trajetória com menos medidas não necessariamente produz o pior resultado em todas as métricas, e a difusão melhora consistentemente a reconstrução nos três casos. Estas conclusões se limitam às configurações e aos resultados registrados para os notebooks desta pasta.
