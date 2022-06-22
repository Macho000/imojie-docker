# About This Repo
This repo is reproduced on [IMOJIE](https://github.com/dair-iitd/imojie) for Docker Environment

# How to Start
```
git clone https://github.com/dair-iitd/imojie
docker-compose build
docker-compose up -d
```

The other steps follows IMOJIE instruction BELOW!

# IMoJIE

Iterative Memory based Joint OpenIE

A BERT-based OpenIE system that generates extraction using an iterative Seq2Seq model, as described in the following publication, in ACL 2020, [link](https://arxiv.org/abs/2005.08178)

## Installation Instructions
Use a python-3.6 environment and install the dependencies using,
```
pip install -r requirements.txt
```
This will install custom versions of allennlp and pytorch_transformers based on the code in the folder.

All reported results are based on pytorch-1.2 run on a TeslaV100 GPU (CUDA 10.0). Results may vary slightly with change in environment.

## Execution Instructions
### Data Download
bash download_data.sh 

This downloads the (train, dev, test) data

### Running the code
IMoJIE (on OpenIE-4, ClausIE, RnnOIE bootstrapping data with QPBO filtering)
```
python allennlp_script.py --param_path imojie/configs/imojie.json --s models/imojie --mode train_test 
```

Arguments:
- param_path: file containing all the parameters for the model
- s:  path of the directory where the model will be saved
- mode: train, test, train_test

Important baselines:

IMoJIE (on OpenIE-4 bootstrapping)
```
python allennlp_script.py --param_path imojie/configs/ba.json --s models/ba --mode train_test 
```

CopyAttn+BERT (on OpenIE-4 bootstrapping)
```
python allennlp_script.py --param_path imojie/configs/be.json --s models/be --mode train_test --type single --beam_size 3
```

### Generating aggregated data

Score using bert_encoder trained on oie4: 
```
python imojie/aggregate/score.py --model_dir models/score/be --inp_fp data/train/4cr_comb_extractions.tsv --out_fp data/train/4cr_comb_extractions.tsv.be 
```

Score using bert_append trained on comb_4cr (random): 
```            
python imojie/aggregate/score.py --model_dir models/score/4cr_rand --inp_fp data/train/4cr_comb_extractions.tsv.be --out_fp data/train/4cr_comb_extractions.tsv.ba
```

Filter using QPBO:
```
python imojie/aggregate/filter.py --inp_fp data/train/4cr_comb_extractions.tsv.ba --out_fp data/train/4cr_qpbo_extractions.tsv
```

### Note on the notation
We have been internally calling our model as "bert-append" (ba) until the day of submission of the paper and CopyAttention + BERT as "bert-encoder" (be). So you will find similar references throughout the code-base. In this context, IMoJIE is bert-append trained on qpbo filtered data.

### Expected Results
Format: (Prec/Rec/F1-Optimal, AUC, Prec/Rec/F1-Last)

models/imojie/test/carb_1/best_results.txt \
(64.70/45.60/53.50, 33.30, 63.80/45.80/53.30)

models/ba/test/carb_1/best_results.txt \
(Prec/Rec/F1-Optimal, AUC, Prec/Rec/F1-Last) \
(63.50/45.80/53.20, 33.10, 60.40/46.30/52.40)

models/be/test/carb_3/best_results.txt \
(Prec/Rec/F1-Optimal, AUC, Prec/Rec/F1-Last) \
(59.50/45.50/51.60, 32.80, 52.90/46.70/49.60)

## Resources

Downloading the pre-trained models:
```
zenodo_get 3779954
```

Downloading the data:
```
zenodo_get 3775983
```

Downloading the results:
```
zenodo_get 3780045
```

### Generating extractions
```
python standalone.py --inp input.txt --out output.txt
```
input.txt contains one sentence in each line 
output.txt contains the corresponding OpenIE extractions

This requires downloading the pre-trained models

# Data Example
## Train Datals
A more obvious influence on the early Hercule Poirot stories is that of Arthur Conan Doyle .	<arg1> A more obvious influence on the early Hercule Poirot stories </arg1> <rel>  is </rel> <arg2>  that of Arthur Conan Doyle </arg2>	0.94
Hercule Poirot also bears a striking resemblance to A. E. W. Mason 's fictional detective -- Inspector Hanaud of the French Sûreté -- who , first appearing in the 1910 novel At the Villa Rose , predates the writing of the first Hercule Poirot novel by six years .	<arg1> Hercule Poirot </arg1> <rel>  bears </rel> <arg2>  a striking resemblance to A. E. W. Mason 's fictional detective </arg2>	0.95




## Dev Data (Carb)
Blagoja ` Billy ' Celeski is an Australian footballer who plays as a midfielder for the Newcastle Jets .
1 (Blagoja ` Billy ' Celeski ; is ; an Australian footballer)
1 (Blagoja ` Billy ' Celeski ; plays ; as a midfielder)
1 (Blagoja ` Billy ' Celeski ; plays ; for the Newcastle Jets)
1 (the Newcastle Jets ; have ; midfielder)

# Test Data (Carb)
A Democrat , he became the youngest mayor in Pittsburgh 's history in September 2006 at the age of 26 .
1 (he ; became ; the youngest mayor in  Pittsburgh 's history ; in September 2006)
1 (he ; was ; A Democrat)
1 (the youngest mayor in Pittsburgh 's history ; was ; A Democrat)
1 (he ; became ; the youngest mayor in the history of ; Pittsburgh ; in September 2006)
1 (he ; became the youngest mayor in Pittsburgh 's history at ; the age of 26 ; in September 2006)
1 (the youngest mayor in Pittsburgh 's history ; was ; 26)

