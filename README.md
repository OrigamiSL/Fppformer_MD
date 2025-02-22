# Inconsistent Multivariate Time Series Forecasting
![Python 3.11](https://img.shields.io/badge/python-3.11-green.svg?style=plastic)
![PyTorch 2.1.0](https://img.shields.io/badge/PyTorch%20-%23EE4C2C.svg?style=plastic)
![CUDA 11.8](https://img.shields.io/badge/cuda-11.8-green.svg?style=plastic)
![License CC BY-NC-SA](https://img.shields.io/badge/license-CC_BY--NC--SA--green.svg?style=plastic)

This is the origin Pytorch implementation of FPPformer-MD in the following paper: 
[Inconsistent Multivariate Time Series Forecasting] (Manuscript submitted to IEEE Transactions on Knowledge and Data Engineering).

## Model Architecture
This work intends to solve two general problems involved in deep MTSF. (1) The existing approaches for addressing variable correlations, including CD and CI approaches, consistently ignore or extract all possible variable correlations, making them either insufficient or inappropriate. (2) The existing data augmentation methods hardly exploit variable correlations to generate more training instances. Moreover, since we mainly combine the proposed inconsistent MTSF approach with the FPPformer, this work also attempts to compensate for the inadequacy of the input features in the embedding layer of the FPPformer. To address these problems, we propose an MVCI method that dynamically identifies local variable correlations and an ICVA module that adaptively extracts the cross-variable features of the correlated variables, thus solving general problem (1). Moreover, the MODWT smooths produced by MVCI can additionally provide interpretable input features to enrich the input of the FPPformer. Then, the unique problem of the FPPformer is solved. We also propose a CVDA method to generate more training instances by conducting DMD on the multivariate input sequences, hence addressing general problem (2). As shown in Figure 1, five major steps are required to upgrade the FPPformer to our proposed FPPformer-MD model:

Frequency-Scale Decomposition: Given an arbitrary multivariate input sequence $\mathbfcal{X}_{t_{1}:t_{2}}$, it is decomposed into $j+1$ frequency scales, which are represented by $\{\boldsymbol{v}_{j}, \boldsymbol{w}_{1}, \boldsymbol{w}_{2}, \dots, \boldsymbol{w}_{j}\mid\boldsymbol{v}, \boldsymbol{w} \in \mathbb{R}^{L_{in} \times V} \}$, via the MODWT.}
\item {\textbf{Adjacency Matrix Generation}: The variances of different frequency scales are computed to obtain the most significant frequency scale for each univariate sequence in $\mathbfcal{X}_{t_{1}:t_{2}}$. Then, an adjacency matrix $\boldsymbol{A} \in {Bool}^{V \times V}$, where the value 0 indicates the existence of a correlation and the value 1 indicates the opposite scenario, is generated to express the existence of a correlation between any variable pair.

\item {\textbf{Enriching the Input Features with MODWT Smooths}: The MODWT-based MRA process is performed based on the decomposition results obtained in step (1) to acquire the MODWT smooths $\{ \boldsymbol{S}_{1}, \boldsymbol{S}_{2}, \dots, \boldsymbol{S}_{j} \mid \boldsymbol{S} \in \mathbb{R}^{L_{in} \times V} \}$ of $\mathbfcal{X}_{t_{1}:t_{2}}$. They are concatenated with $\mathbfcal{X}_{t_{1}:t_{2}}$ in the temporal dimension to enrich the input features.
}
\item {
	\textbf{DMD-Based Data Augmentation}: $\mathbfcal{X}_{t_{1}:t_{2}}$ is split into at most $j+1$ groups according to step (2). This process has a chance to augment each variable group individually with DMD to provide more instances during training.
}
\item {\textbf{Masking the Uncorrelated Variables via ICVA}: An ICVA module is placed after each temporal patchwise attention module in the encoder of the FPPformer. ICVA performs attention along the variable dimension, and its attention score is masked with $\boldsymbol{A}$ from step (2) to reduce the connections among uncorrelated variables.
}
<p align="center">
<img src="./img/FPPformer-MD.jpg" height = "300" alt="" align=center />
<br><br>
<b>Figure 1.</b> The architecture of FPPformer-MD with a $N$-stage encoder and a $M$-stage decoder  ($N=M=6$ in experiment). The colored modules are the novel methods proposed in this work.
</p>


## Requirements
- python == 3.11.4
- numpy == 1.24.3
- pandas == 1.5.3
- scipy == 1.11.3
- torch == 2.1.0+cu118
- scikit-learn == 1.3.0
- PyWavelets == 1.4.1
- astropy == 6.1
- h5py == 3.7.0
- geomstat == 2.5.0

Dependencies can be installed using the following command:
```bash
pip install -r requirements.txt
```

## Data

ETT, ECL, Traffic and Weather dataset were acquired at: [here](https://drive.google.com/drive/folders/1ZOYpTUa82_jCcxIdTmyr0LXQfvaM9vIy?usp=sharing). Solar dataset was acquired at: [Solar](https://drive.google.com/drive/folders/1Gv1MXjLo5bLGep4bsqDyaNMI2oQC9GH2?usp=sharing). PeMSD3, PeMSD4, PeMSD7 and PeMSD8 were acquired at: [PeMS](https://github.com/guoshnBJTU/ASTGNN/tree/main/data). PeMS-Bay was acquired at: [PeMS-Bay](https://drive.google.com/drive/folders/10FOTa6HXPqX8Pf5WRoRwcFnW9BrNZEIX).

### Data Preparation
After you acquire raw data of all datasets, please separately place them in corresponding folders at `./data`. 

We place ETT in the folder `./ETT-data`, ECL in the folder `./electricity`  and weather in the folder `./weather` of [here](https://drive.google.com/drive/folders/1ZOYpTUa82_jCcxIdTmyr0LXQfvaM9vIy?usp=sharing) (the folder tree in the link is shown as below) into folder `./data` and rename them from `./ETT-data`,`./electricity`, `./traffic` and `./weather` to `./ETT`, `./ECL`, `./Traffic` and`./weather` respectively. We rename the file of ECL/Traffic from `electricity.csv`/`traffic.csv` to `ECL.csv`/`Traffic.csv` and rename its last variable from `OT`/`OT` to original `MT_321`/`Sensor_861` separately.
```
The folder tree in https://drive.google.com/drive/folders/1ZOYpTUa82_jCcxIdTmyr0LXQfvaM9vIy?usp=sharing:
|-autoformer
| |-ETT-data
| | |-ETTh1.csv
| | |-ETTh2.csv
| | |-ETTm1.csv
| | |-ETTm2.csv
| |
| |-electricity
| | |-electricity.csv
| |
| |-traffic
| | |-traffic.csv
| |
| |-weather
| | |-weather.csv
```

We place Solar in the folder `./financial` of [Solar](https://drive.google.com/drive/folders/1Gv1MXjLo5bLGep4bsqDyaNMI2oQC9GH2?usp=sharing) (the folder tree in the link is shown as below) into the folder `./data` and rename them as `./Solar` respectively. 

```
The folder tree in https://drive.google.com/drive/folders/1Gv1MXjLo5bLGep4bsqDyaNMI2oQC9GH2?usp=sharing:
|-dataset
| |-financial
| | |-solar_AL.txt
```

We place the NPZ files ('PEMS03.npz', 'PEMS04.npz', 'PEMS07.npz', 'PEMS08.npz') of PeMSD3, PeMSD4, PeMSD7 and PeMSD8 in the folder `./PEMS03`, `./PEMS03`, `./PEMS03` and `./PEMS03` of [PeMS](https://github.com/guoshnBJTU/ASTGNN/tree/main/data) 
 into the folder `./data/PEMS`. We place the H5 file ('pems-bay.h5') of PeMS-Bay in the [PeMS-Bay](https://drive.google.com/drive/folders/10FOTa6HXPqX8Pf5WRoRwcFnW9BrNZEIX) into the folder `./data/PEMS`. The folder trees in the mentioned two links are shown as below:

```
The folder tree in https://github.com/guoshnBJTU/ASTGNN/tree/main/data:
|-PEMS03
| |-PEMS03.csv
| |-PEMS03.npz
| |-PEMS03.txt
|-PEMS04
| |-PEMS04.csv
| |-PEMS04.npz
| |-PEMS04.txt
|-PEMS07
| |-PEMS07.csv
| |-PEMS07.npz
| |-PEMS07.txt
|-PEMS08
| |-PEMS08.csv
| |-PEMS08.npz
| |-PEMS08.txt

The folder tree in https://drive.google.com/drive/folders/10FOTa6HXPqX8Pf5WRoRwcFnW9BrNZEIX:
|-metr-la.h5
|-pems-bay.h5
```

After you process all the datasets, you will obtain folder tree:
```
|-data
| |-ECL
| | |-ECL.csv
| |
| |-ETT
| | |-ETTh1.csv
| | |-ETTh2.csv
| | |-ETTm1.csv
| | |-ETTm2.csv
| |
| |-PEMS
| | |-PEMS03.npz
| | |-PEMS04.npz
| | |-PEMS07.npz
| | |-PEMS08.npz
| | |-pems-bay.h5
| |
| |-Solar
| | |-solar_AL.txt
| |
| |-Traffic
| | |-Traffic.csv
| |
| |-weather
| | |-weather.csv

```

## Usage
Commands for training and testing FPPformer-MD of all datasets are in `./scripts/Main.sh`. 

More parameter information please refer to `main.py`.

We provide a complete command for training and testing FPPformer-MD:

```
python -u main.py --data <data> --input_len <input_len> --pred_len <pred_len> --encoder_layer <encoder_layer> --layer_stack <layer_stack> --MODWT_level<MODWT_level> --patch_size<patch_size> --d_model <d_model> --augmentation_len<augmentation_len> --augmentation_ratio<augmentation_ratio>  --learning_rate <learning_rate> --dropout <dropout> --batch_size <batch_size> --train_epochs <train_epochs> --itr <itr> --train --decoder_IN --patience <patience> --decay<decay>
```

Here we provide a more detailed and complete command description for training and testing the model:

| Parameter name |                                          Description of parameter                                          |
|:--------------:|:----------------------------------------------------------------------------------------------------------:|
|      data      |                                              The dataset name                                              |
|   root_path    |                                       The root path of the data file                                       |
|   data_path    |                                             The data file name                                             |
|  checkpoints   |                                       Location of model checkpoints                                        |
|   input_len    |                                           Input sequence length                                            |
|    pred_len    |                                         Prediction sequence length                                         |
|     enc_in     |                                                 Input size                                                 |
|    dec_out     |                                                Output size                                                 |
|    d_model     |                                             Dimension of model                                             |
|  encoder_layer |                                            The number of stages                                            |
|   layer_stack  |                                       The number of layers per stage                                       |
|   patch_size   |                                The initial patch size in patch-wise attention                              |
|  MODWT_level   |                                           The level of MODWT/MRA                                           |
|augmentation_method   |                                           Augmentation method                                           |
|  augmentation_ratio   |                                           Augmentation ratio                                           |
|  augmentation_len   |                                           Augmentation length                                           |
|  decoder_IN   |                                          whether to perform IN for decoder inputh                                           |
|    dropout     |                                                  Dropout                                                   |
|    num_workers     |                                                  Data loader num workers                                                   |
|      itr       |                                             Experiments times                                              |
|  train_epochs  |                                      Train epochs of the second stage                                      |
|   batch_size   |                         The batch size of training input data in the second stage                          |
|   decay   |                         Decay rate of learning rate per epoch                         |
|    patience    |                                          Early stopping patience                                           |
| learning_rate  |                                          Optimizer learning rate                                           |


## Results
The experiment parameters of each data set are formated in the `Main.sh` files in the directory `./scripts/`. You can refer to these parameters for experiments, and you can also adjust the parameters to obtain better mse and mae results or draw better prediction figures. We also provide the commands for obtain the results of FPPformer-MD with longer input sequence length (336) in the file `./scripts/Main.sh`. We present the full results of multivariate forecasting results in 

<p align="center">
<img src="./img/Multivariate.png" height = "500" alt="" align=center />
<br><br>
<b>Figure 2.</b> Multivariate forecasting results
</p>

<p align="center">
<img src="./img/Univariate.png" height = "400" alt="" align=center />
<br><br>
<b>Figure 3.</b> Univariate forecasting results
</p>

### Full results
Moreover, we present the full results of multivariate forecasting results with long input sequence lengths in Figure 4, that of ablation study in Figure 5 and that of parameter sensitivity in Figure 6.
<p align="center">
<img src="./img/Long.png" height = "500" alt="" align=center />
<br><br>
<b>Figure 4.</b> Multivariate forecasting results with long input lengths
</p>
<p align="center">
<img src="./img/Ablation.png" height = "400" alt="" align=center />
<br><br>
<b>Figure 5.</b> Ablation results with the prediction length of 720
</p>
<p align="center">
<img src="./img/Parameter.png" height = "500" alt="" align=center />
<br><br>
<b>Figure 6.</b> Results of parameter sensitivity on stage numbers
</p>

## Contact
If you have any questions, feel free to contact Li Shen through Email (shenli@buaa.edu.cn) or Github issues. Pull requests are highly welcomed!
