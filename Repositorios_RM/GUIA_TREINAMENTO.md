# Guia de Treinamento - Modelos de Reconstrução de Ressonância Magnética

Este documento identifica os arquivos de treinamento disponíveis neste repositório e como executá-los.

## ⚠️ AVISO IMPORTANTE - Compatibilidade

**Os 3 projetos têm requisitos INCOMPATÍVEIS entre si!**

Cada projeto precisa de um **ambiente virtual ou conda separado** devido a:
- **DuDoRNet**: PyTorch 0.4.1 (muito antigo, 2018) + Python 3.7
- **MoDL**: TensorFlow 1.x (antigo) + Python 2.7 ou 3.6  
- **score-MRI**: PyTorch moderno + Python 3.8

**⚠️ NÃO instale todas as dependências no mesmo ambiente Python!**

**✅ SOLUÇÃO:** Use ambientes virtuais/conda separados para cada projeto (veja seção "Configuração de Ambientes" abaixo).

---

## 📋 Checklist Antes de Começar

Antes de executar qualquer projeto, verifique:

- [ ] **Dados disponíveis:**
  - DuDoRNet: Dados em `../Data/PROC/` (TRAIN/VALI/TEST)
  - MoDL: Arquivo `dataset.hdf5` no diretório `modl/`
  - score-MRI: Dados configurados no arquivo de config

- [ ] **GPU disponível** (recomendado, mas não obrigatório)
- [ ] **Espaço em disco** suficiente para modelos treinados
- [ ] **Ambientes virtuais** configurados (um por projeto)

---

## 🔧 Configuração de Ambientes

### Opção 1: Usando Conda (Recomendado)

```bash
# Ambiente para DuDoRNet
conda create -n dudornet python=3.7
conda activate dudornet
conda install pytorch=0.4.1 torchvision cudatoolkit=10.0 -c pytorch
pip install scipy scikit-image opencv-python tqdm

# Ambiente para MoDL
conda create -n modl python=3.6
conda activate modl
conda install tensorflow-gpu=1.15 h5py tqdm matplotlib
# OU para CPU: conda install tensorflow=1.15 h5py tqdm matplotlib

# Ambiente para score-MRI
conda create -n score-mri python=3.8
conda activate score-mri
cd score-MRI
bash install.sh
```

### Opção 2: Usando venv

```bash
# DuDoRNet
python3.7 -m venv venv_dudornet
source venv_dudornet/bin/activate  # Linux/Mac
# ou: venv_dudornet\Scripts\activate  # Windows
pip install torch==0.4.1 torchvision scipy scikit-image opencv-python tqdm

# MoDL
python3.6 -m venv venv_modl
source venv_modl/bin/activate
pip install tensorflow==1.15 h5py tqdm matplotlib

# score-MRI
python3.8 -m venv venv_score_mri
source venv_score_mri/bin/activate
cd score-MRI
pip install -r requirements.txt
bash install.sh
```

---

## 📁 Estrutura dos Projetos

### 1. **DuDoRNet** (PyTorch)
**Localização:** `DuDoRNet/`

**Arquivo de Treinamento:**
- `train.py` - Script principal de treinamento

**Scripts de Exemplo:**
- `scripts/train_DuDoRN_R4_1Do_0pT1_Cartesian.sh`
- `scripts/train_DuDoRN_R4_1Do_1pT1_Cartesian.sh`

**Como Executar:**
```bash
cd DuDoRNet
python train.py \
  --experiment_name 'train_DuDoRN_R4_1Do_0pT1_Cartesian' \
  --data_root '../Data/PROC/' \
  --dataset 'Cartesian' \
  --net_G 'DRDN' \
  --n_recurrent 4 \
  --batch_size 6 \
  --eval_epochs 10 \
  --save_epochs 10 \
  --lr 5e-4 \
  --gpu_ids 0
```

**Ou usando o script:**
```bash
cd DuDoRNet
bash scripts/train_DuDoRN_R4_1Do_0pT1_Cartesian.sh
```

**Requisitos:**
- Python 3.7 ⚠️ (versão específica)
- PyTorch 0.4.1 ⚠️ (versão muito antiga)
- CUDA 10.0 (recomendado)
- Dados organizados em `../Data/PROC/` (TRAIN/VALI/TEST)
- scipy, scikit-image, opencv-python, tqdm

**⚠️ Problemas Conhecidos:**
- PyTorch 0.4.1 é muito antigo e pode não funcionar em sistemas modernos
- Pode ser necessário usar Docker ou ambiente isolado
- Verifique se os dados estão no formato correto (.mat com variável 'kspace_py')

**Parâmetros Importantes:**
- `--experiment_name`: Nome do experimento (define pasta de saída)
- `--data_root`: Diretório dos dados
- `--n_recurrent`: Número de blocos recorrentes (padrão: 4)
- `--batch_size`: Tamanho do batch
- `--gpu_ids`: IDs das GPUs a usar
- `--use_prior`: Flag para usar prior (T1 como referência)
- `--protocol_ref`: Modalidade de referência (T1/T2/FLAIR)
- `--protocol_tag`: Modalidade alvo para reconstrução

---

### 2. **MoDL** (TensorFlow)
**Localização:** `modl/`

**Arquivo de Treinamento:**
- `trn.py` - Script principal de treinamento

**Como Executar:**
```bash
cd modl
python trn.py
```

**Configurações no Arquivo (linhas 66-72):**
```python
nLayers=5          # Número de camadas da CNN
epochs=50          # Número de épocas
batchSize=1        # Tamanho do batch
gradientMethod='AG' # 'AG' ou 'MG'
K=1                # Número de iterações (máximo 20 para GPU 16GB)
sigma=0.01         # Desvio padrão do ruído Gaussiano
restoreWeights=False # Carregar pesos pré-treinados
```

**Requisitos:**
- TensorFlow 1.7+ ⚠️ (TensorFlow 1.x está descontinuado)
- Python 2.7 ou 3.6 ⚠️ (Python 2.7 está descontinuado, use 3.6)
- h5py (para dados HDF5)
- tqdm
- matplotlib
- Dataset: `dataset.hdf5` (download necessário: https://zenodo.org/records/6481291)

**⚠️ Problemas Conhecidos:**
- TensorFlow 1.x não é mais suportado oficialmente
- Pode precisar de ajustes para funcionar em sistemas modernos
- O arquivo `dataset.hdf5` tem ~3GB, certifique-se de ter espaço

**📥 Download do Dataset:**
```bash
cd modl
# Baixe o dataset.hdf5 de: https://zenodo.org/records/6481291
# Coloque o arquivo no diretório modl/
```

**Nota:** O modelo treinado é salvo em `savedModels/` com nome gerado automaticamente baseado em data/hora e parâmetros.

---

### 3. **score-MRI** (PyTorch - Score-based Diffusion)
**Localização:** `score-MRI/`

**Arquivo de Treinamento:**
- `main_fastmri.py` - Script principal (modo: train)

**Script de Exemplo:**
- `train_fastmri_knee.sh`

**Como Executar:**
```bash
cd score-MRI
python main_fastmri.py \
  --config=configs/ve/fastmri_knee_320_ncsnpp_continuous.py \
  --mode='train' \
  --workdir=workdir/fastmri_multicoil_knee_320
```

**Ou usando o script:**
```bash
cd score-MRI
bash train_fastmri_knee.sh
```

**Requisitos:**
- Python 3.8
- PyTorch (versão moderna, instalada via install.sh)
- CUDA 10.2+ (recomendado)
- Dependências instaladas via `install.sh`
- Dados de treinamento configurados no arquivo de config

**Instalação:**
```bash
cd score-MRI
bash install.sh
# Isso vai:
# 1. Baixar pesos do modelo pré-treinado
# 2. Criar ambiente conda (score-POCS)
# 3. Instalar PyTorch e dependências
```

**Modos Disponíveis:**
- `train`: Treinamento normal
- `train_regression`: Treinamento de regressão
- `eval`: Avaliação

**⚠️ Nota:** O script `install.sh` cria automaticamente um ambiente conda. Se preferir usar seu próprio ambiente, instale manualmente as dependências do `requirements.txt`.

---

## 🚀 Início Rápido

### Para DuDoRNet:
1. **Ative o ambiente virtual:**
   ```bash
   conda activate dudornet
   # ou: source venv_dudornet/bin/activate
   ```

2. **Verifique se os dados existem:**
   ```bash
   ls -la ../Data/PROC/TRAIN/
   # Deve ter subpastas: T1/, T2/, FLAIR/ com arquivos .mat
   ```

3. **Execute o treinamento:**
   ```bash
   cd DuDoRNet
   python train.py --help  # Ver todas as opções
   # Ou use o script:
   bash scripts/train_DuDoRN_R4_1Do_0pT1_Cartesian.sh
   ```

### Para MoDL:
1. **Ative o ambiente virtual:**
   ```bash
   conda activate modl
   # ou: source venv_modl/bin/activate
   ```

2. **Verifique se o dataset existe:**
   ```bash
   cd modl
   ls -lh dataset.hdf5
   # Se não existir, baixe de: https://zenodo.org/records/6481291
   ```

3. **Ajuste os parâmetros no arquivo `trn.py` (linhas 66-72)**

4. **Execute o treinamento:**
   ```bash
   python trn.py
   ```

### Para score-MRI:
1. **Instale as dependências (primeira vez apenas):**
   ```bash
   cd score-MRI
   bash install.sh
   ```

2. **Ative o ambiente (se criado pelo install.sh):**
   ```bash
   conda activate score-POCS
   ```

3. **Configure os dados de treinamento** (edite o arquivo de config se necessário)

4. **Execute o treinamento:**
   ```bash
   bash train_fastmri_knee.sh
   # ou manualmente:
   python main_fastmri.py --config=configs/ve/fastmri_knee_320_ncsnpp_continuous.py --mode='train' --workdir=workdir/fastmri_multicoil_knee_320
   ```

---

## 🔍 Verificação Pré-Execução

### Verificar Dados:

**DuDoRNet:**
```bash
# Verifique se a estrutura de dados existe
ls -la ../Data/PROC/TRAIN/T1/kspace/
ls -la ../Data/PROC/VALI/T1/kspace/
ls -la ../Data/PROC/TEST/T1/kspace/
```

**MoDL:**
```bash
cd modl
python -c "import h5py; f=h5py.File('dataset.hdf5','r'); print(list(f.keys()))"
# Deve mostrar: ['trnOrg', 'trnCsm', 'trnMask', 'tstOrg', 'tstCsm', 'tstMask']
```

**score-MRI:**
```bash
cd score-MRI
# Verifique o arquivo de config
cat configs/ve/fastmri_knee_320_ncsnpp_continuous.py
# Verifique se os caminhos dos dados estão corretos
```

### Verificar GPU:
```bash
# Verifique se CUDA está disponível
nvidia-smi

# No Python:
python -c "import torch; print(torch.cuda.is_available())"  # PyTorch
python -c "import tensorflow as tf; print(tf.test.is_gpu_available())"  # TensorFlow
```

---

## 📝 Notas Importantes

1. **DuDoRNet**: 
   - Requer estrutura específica de dados (TRAIN/VALI/TEST com subpastas T1/T2/FLAIR)
   - PyTorch 0.4.1 é muito antigo - pode ter problemas de compatibilidade
   - Arquivos .mat devem ter variável 'kspace_py'

2. **MoDL**: 
   - Usa TensorFlow 1.x (descontinuado)
   - Funciona melhor com Python 3.6 (evite Python 2.7)
   - Dataset de 3GB necessário para treinamento

3. **score-MRI**: 
   - Baseado em modelos de difusão (score-based), mais moderno
   - Requer mais memória GPU
   - Tem script de instalação automatizado

---

## 🐛 Troubleshooting Comum

### Problema: "ModuleNotFoundError" ou "ImportError"
**Solução:** Verifique se está no ambiente virtual correto e se todas as dependências estão instaladas.

### Problema: "CUDA out of memory"
**Solução:** 
- Reduza o `batch_size` no código
- Para MoDL, reduza `K` (número de iterações)
- Use CPU (mais lento): `CUDA_VISIBLE_DEVICES="" python train.py ...`

### Problema: "FileNotFoundError" para dados
**Solução:** 
- Verifique os caminhos dos dados
- Para DuDoRNet, certifique-se que `--data_root` aponta para o diretório correto
- Para MoDL, verifique se `dataset.hdf5` está no diretório `modl/`

### Problema: "TensorFlow/PyTorch version mismatch"
**Solução:** Use ambientes virtuais separados para cada projeto (veja seção "Configuração de Ambientes")

### Problema: PyTorch 0.4.1 não instala
**Solução:** 
- Tente instalar de uma fonte alternativa
- Considere usar Docker com ambiente pré-configurado
- Ou tente atualizar o código para versão mais recente do PyTorch (pode requerer modificações)

---

## 📊 Resumo de Executabilidade

| Projeto | Facilidade | Status | Observações |
|---------|-----------|--------|-------------|
| **DuDoRNet** | ⚠️ Média | Requer dados + ambiente isolado | PyTorch 0.4.1 muito antigo |
| **MoDL** | ⚠️ Média | Requer dataset.hdf5 + ambiente isolado | TensorFlow 1.x descontinuado |
| **score-MRI** | ✅ Mais fácil | Tem script de instalação automático | Mais moderno, melhor suportado |

**Resposta à pergunta: "Consigo executar os 3 projetos?"**

✅ **SIM, mas com ressalvas:**
- ✅ Você **pode** executar os 3 projetos
- ⚠️ Mas precisa de **ambientes virtuais separados** para cada um
- ⚠️ Precisa ter os **dados de treinamento** para cada projeto
- ⚠️ Alguns projetos têm versões antigas que podem dar problemas em sistemas modernos
- ✅ O projeto **score-MRI** é o mais fácil de executar (tem instalação automatizada)

**Recomendação:** Comece pelo **score-MRI** (mais fácil), depois **MoDL**, e por último **DuDoRNet** (mais problemático devido ao PyTorch antigo).

---

## 📞 Suporte

Para dúvidas específicas de cada projeto, consulte os README.md em cada diretório:
- `DuDoRNet/README.md`
- `modl/README.md`
- `score-MRI/README.md`

**Links Úteis:**
- DuDoRNet Paper: http://openaccess.thecvf.com/content_CVPR_2020/papers/Zhou_DuDoRNet_Learning_a_Dual-Domain_Recurrent_Network_for_Fast_MRI_Reconstruction_CVPR_2020_paper.pdf
- MoDL Dataset: https://zenodo.org/records/6481291
- MoDL Paper: https://arxiv.org/abs/1712.02862
- score-MRI Paper: https://arxiv.org/abs/2110.05243

