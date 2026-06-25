# Reconstrução de Imagens de Ressonância Magnética com Modelos de Difusão e Diferentes Estratégias de Amostragem do Espaço-k

## Visão Geral

Este repositório implementa modelos probabilísticos de difusão para reconstrução de imagens de Ressonância Magnética (RM) a partir de dados subamostrados no espaço de Fourier (*k-space*).

O objetivo principal é investigar o impacto de diferentes trajetórias de aquisição no desempenho da reconstrução utilizando modelos generativos baseados em difusão.

As seguintes estratégias de amostragem foram implementadas:

- Amostragem Cartesiana
- Amostragem Radial
- Amostragem Espiral

---

# Motivação

A aquisição completa do espaço-k em exames de RM possui elevado custo temporal, aumentando:

- tempo total de aquisição;
- suscetibilidade a movimento do paciente;
- desconforto durante o exame;
- custos operacionais.

Uma abordagem clássica para reduzir o tempo de aquisição consiste em subamostrar o espaço-k, adquirindo apenas uma fração das medidas originalmente necessárias.

Entretanto, a subamostragem transforma o problema de reconstrução em um problema inverso mal condicionado:

$$
y = PFx + n
$$

onde:

- $x$ representa a imagem desconhecida;
- $F$ é a Transformada Discreta de Fourier bidimensional;
- $P$ representa o operador de subamostragem;
- $y$ corresponde às medidas adquiridas;
- $n$ representa ruído de aquisição.

Como o número de medidas torna-se inferior ao número de incógnitas:

$$
M < N
$$

existem infinitas soluções compatíveis com as observações disponíveis.

Modelos de difusão fornecem um mecanismo probabilístico para restringir esse espaço de soluções e recuperar imagens fisicamente plausíveis.

---

# Modelos de Difusão

Os modelos de difusão pertencem à classe dos modelos probabilísticos generativos baseados em processos de Markov.

O processo direto consiste em adicionar ruído gaussiano progressivamente à imagem:

$$
q(x_t|x_{t-1}) = \mathcal{N}(x_t;\sqrt{1-\beta_t}x_{t-1},\beta_t I)
$$

Após um número suficientemente grande de etapas:

$$
x_T \sim \mathcal{N}(0,I)
$$

Durante o treinamento, a rede aprende o processo reverso:

$$
p_\theta(x_{t-1}|x_t)
$$

permitindo reconstruir iterativamente a imagem original a partir de amostras ruidosas.

---

# Formulação Matemática

O processo de difusão direto pode ser escrito como:

$$
x_t = \sqrt{\bar{\alpha}_t}x_0 + \sqrt{1-\bar{\alpha}_t}\epsilon
$$

onde:

- $x_0$ é a imagem original;
- $\epsilon \sim \mathcal{N}(0,I)$;
- $\bar{\alpha}_t$ representa o produto acumulado dos coeficientes de preservação do sinal.

A rede é treinada para estimar o ruído adicionado:

$$
\epsilon_\theta(x_t,t)
$$

utilizando a função perda:

$$
L = \mathbb{E}\left[\left\|\epsilon-\epsilon_\theta(x_t,t)\right\|_2^2\right]
$$

---

# Pipeline Experimental

## 1. Aquisição da imagem de referência

Uma imagem de RM totalmente amostrada é utilizada como referência (*ground truth*).

---

## 2. Transformada de Fourier

A imagem é convertida para o domínio da frequência:

$$
K = \mathcal{F}(x)
$$

---

## 3. Aplicação da máscara de amostragem

Uma máscara específica é aplicada ao espaço-k:

$$
K_u = M \odot K
$$

onde:

- $M$ representa a máscara de amostragem;
- $\odot$ representa multiplicação elemento a elemento.

---

## 4. Reconstrução inicial

Uma reconstrução inicial é obtida utilizando a transformada inversa:

$$
x_u = \mathcal{F}^{-1}(K_u)
$$

Essa imagem contém artefatos decorrentes da subamostragem.

---

## 5. Processo de difusão

A imagem reconstruída é progressivamente refinada pelo modelo de difusão até a obtenção da reconstrução final:

$$
\hat{x}
$$

---

# Estratégias de Amostragem

## 1. Amostragem Cartesiana

Arquivo correspondente:

```text
difusao12-cartesiana.ipynb
```

### Características

- aquisição em linhas horizontais do espaço-k;
- compatibilidade direta com FFT convencional;
- implementação computacional simples;
- padrão dominante na maioria dos scanners clínicos.

### Vantagens

- menor custo computacional;
- reconstrução rápida;
- implementação simples.

### Desvantagens

- elevada sensibilidade a movimento;
- artefatos periódicos de aliasing quando subamostrada.

---

## 2. Amostragem Radial

Arquivo correspondente:

```text
difusao12-radial.ipynb
```

### Características

- aquisição ao longo de projeções angulares;
- cobertura densa da região central do espaço-k;
- elevada robustez a movimento.

### Vantagens

- melhor preservação das baixas frequências;
- redução de artefatos de movimento;
- degradação visual mais suave.

### Desvantagens

- necessidade de reconstrução não cartesiana;
- maior custo computacional.

---

## 3. Amostragem Espiral

Arquivo correspondente:

```text
difusao12-spiral12.ipynb
```

### Características

- trajetória contínua em espiral;
- elevada eficiência amostral;
- cobertura uniforme do espaço-k.

### Vantagens

- maior eficiência de aquisição;
- menor tempo de escaneamento;
- excelente cobertura das baixas frequências.

### Desvantagens

- elevada complexidade computacional;
- maior sensibilidade a imperfeições do gradiente;
- necessidade de reconstrução via NUFFT ou gridding.

---

# Comparação das Estratégias

| Método | Cobertura do Espaço-k | Robustez ao Movimento | Artefato Predominante | Complexidade |
|--------|----------------------|-----------------------|----------------------|-------------|
| Cartesiana | Linear | Baixa | Aliasing periódico | Baixa |
| Radial | Angular | Alta | Streaking | Média |
| Espiral | Contínua | Alta | Blur residual | Alta |

---

# Métricas de Avaliação

## Mean Squared Error (MSE)

$$
MSE = \frac{1}{N}\sum_{i=1}^{N}(x_i-\hat{x}_i)^2
$$

---

## Peak Signal-to-Noise Ratio (PSNR)

$$
PSNR = 20\log_{10}\left(\frac{MAX_I}{\sqrt{MSE}}\right)
$$

---

## Structural Similarity Index Measure (SSIM)

$$
SSIM(x,\hat{x}) =
\frac{
(2\mu_x\mu_y+C_1)(2\sigma_{xy}+C_2)
}{
(\mu_x^2+\mu_y^2+C_1)(\sigma_x^2+\sigma_y^2+C_2)
}
$$

---

# Dependências

Instale as dependências utilizando:

```bash
pip install -r requirements.txt
```

Pacotes principais:

```text
numpy
scipy
matplotlib
torch
torchvision
scikit-image
opencv-python
tqdm
```

Dependências opcionais para trajetórias não cartesianas:

```text
torchkbnufft
sigpy
bart
```


---

# Objetivos Experimentais

Este projeto busca avaliar:

- desempenho de modelos de difusão em reconstrução de RM;
- impacto do padrão de amostragem sobre a qualidade da reconstrução;
- robustez a elevadas taxas de subamostragem;
- preservação estrutural das imagens reconstruídas;
- viabilidade da utilização clínica de modelos generativos em aquisição acelerada.

---

# Possíveis Extensões

- DDPM condicionais;
- DDIM para aceleração da inferência;
- reconstrução multi-coil;
- integração com Parallel Imaging;
- integração com Compressed Sensing;
- utilização de dados clínicos reais;
- treinamento em domínio complexo;
- modelos latentes de difusão.

