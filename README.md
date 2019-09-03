# JStap: A Static Pre-Filter for Malicious JavaScript Detection

This repository contains the code for the ACSAC'19 paper: "JStap: A Static Pre-Filter for Malicious JavaScript Detection".  
Please note that in its current state, the code is a Poc and not a fully-fledged production-ready API.


## Summary
JStap is a modular static malicious JavaScript detection system. Our detector is composed of ten modules, including five different ways of abstracting code (namely tokens, Abstract Syntax Tree (AST), Control Flow Graph (CFG), Program Dependency Graph considering data flow only (PDG-DFG), and PDG), and two ways of extracting features (leveraging n-grams, or Identifier values). Based on the frequency of these specific patterns, we train a random forest classifier for each module. 

## Setup

```
install python3 version 3.6.7
install python3-pip # (tested with 9.0.1)
pip3 install -r requirements.txt # (tested versions indicated in requirements.txt)

install nodejs # (tested with 8.10.0)
install npm # (tested with 3.5.2)
cd pdg_generation
npm install escodegen # (tested with 1.9.1)
cd ../classification
npm install esprima # (tested with 4.0.1)
cd ..
```


## Usage

For the AST, CFG, PDG-DFG, and PDG analyses, you should generate the PDGs of the considered files separately and beforehand. After that, give the folder(s) containing the PDGs as input to the learner or classifier (in the case of an AST-based analysis, for example, we only use the AST information contained in the PDG).
On the contrary for the token-based approach, you should give directly the folder containing the JS files as input to the learner/classifier.


### PDGs Generation

To generate the PDGs of the JS files (.js) from the folder FOLDER\_NAME, launch the following shell command from the ```pdg_generation``` folder location:
```
$ python3 -c "from pdgs_generation import *; store_pdg_folder('FOLDER_NAME')"
```

The corresponding PDGs will be store in FOLDER\_NAME/Analysis/PDG.

Currently, we are using 2 CPUs for the PDGs generation process; this can be changed by modifying the variable NUM\_WORKERS from pdg\_generation/utility\_df.py.


### Learning: Building a Model

>usage: learner.py [-h] [--d DIR [DIR ...]] [--l LABEL [LABEL ...]]  
                  [--vd DIR-VALIDATE [DIR-VALIDATE ...]] [--vl LABEL [LABEL ...]]  
                  [--analysis_path DIR] [--md MODEL-DIR] [--mn MODEL-NAME]  
                  [--level LEVEL] [--features FEATURES\_CHOICE]  

>Given a list of directories, builds a model to classify future JS inputs.

>Arguments:  
  -h, --help            show this help message and exit  
  --d DIR [DIR ...]     directories to be used to build a model from  
  --l LABEL [LABEL ...] labels of the JS directories used to build a model from  
  --vd DIR-VALIDATE [DIR-VALIDATE ...] 2 JS directories (1 benign, 1 malicious) to select the features with chi2  
  --vl LABEL [LABEL ...] labels of the 2 JS directories for the features selection process  
  --analysis_path DIR   folder to store the features' analysis results in  
  --md MODEL-DIR        path to store the model that will be produced  
  --mn MODEL-NAME       name of the model that will be produced  
  --level LEVEL         stands for the level of the analysis (tokens, ast, cfg, pdg-dfg, or pdg)  
  --features FEATURES\_CHOICE features' choice (ngrams, value)  

To build a model from the folders BENIGN and MALICIOUS, containing JS files (for the token-based analysis) or the PDGs (for the other analyses), use the option --d BENIGN/, MALICIOUS/ and add their corresponding ground truth with --l benign malicious  
And to select the features appearing in the training set with chi2 on 2 independent datasets: --vd BENIGN-VALIDATE/, MALICIOUS-VALIDATE/ with their corresponding ground truth --vl benign malicious  
Indicate your analysis level with --level followed by either 'tokens', 'ast', 'cfg', 'pdg-dfg' or 'pdg'.  
Indicate the features that the analysis should use with --features followed by either 'ngrams', 'value'. You can choose where to store the features selected by chi2 with --analysis_path (default JStap/Analysis).  
You can choose the model's name with --mn (default being 'model') and its directory (default JStap/Analysis).

```
$ python3 learner.py --d BENIGN/ MALICIOUS/ --l benign malicious --vd BENIGN-VALIDATE/ MALICIOUS-VALIDATE/ --vl benign malicious --level LEVEL --features FEATURES --mn FEATURES_LEVEL
```


### Classification of Unknown JS Samples
The process is similar for the classification process.

>usage: classifier.py [-h] [--d DIR [DIR ...]] [--l LABEL [LABEL ...]]  
                     [--f FILE [FILE ...]] [--lf LABEL [LABEL ...]]  
                     [--analysis_path DIR] [--m MODEL]  
                     [--level LEVEL] [--features FEATURES\_CHOICE]

>Given a list of directories or file paths, detects the malicious JS inputs.

>Arguments:
  -h, --help            show this help message and exit  
  --d DIR [DIR ...]     directories containing the JS files to be analyzed  
  --l LABEL [LABEL ...] labels of the JS directories to evaluate the model from (to evaluate the accuracy, otherwise do not enter anything)  
  --f FILE [FILE ...]   files to be analyzed  
  --lf LABEL [LABEL ...] labels of the JS files to evaluate the model from (to evaluate the accuracy, otherwise do not enter anything)  
  --analysis_path DIR   folder to store the features' analysis results in  
  --m MODEL             path of the model used to classify the new JS inputs  
  --level LEVEL         stands for the level of the analysis (tokens, ast, cfg, pdg-dfg, pdg)  
  --features FEATURES\_CHOICE features' choice (ngrams, value)

If you known the ground truth and would like to evaluate the accuracy:  
```
$ python3 classifier.py --d BENIGN2/ MALICIOUS2/ --l benign malicious --level LEVEL --features FEATURES --m FEATURES_LEVEL
```

If you would like to classify unknown files:  
```
$ python3 classifier.py --d BENIGN2/ MALICIOUS2/ --level LEVEL --features FEATURES --m FEATURES_LEVEL
```


Currently, we are using 2 CPUs for the learning and classification processes; this can be changed by modifying the variable NUM\_WORKERS from classification/utility.py.


## Cite this work
If you use JStap for academic research, you are highly encouraged to cite the following paper:
```
@inproceedings{fass2019jstap,
    author="Fass, Aurore and Backes, Michael and Stock, Ben",
    title="{\textsc{JStap}: A Static Pre-Filter for Malicious JavaScript Detection}",
    booktitle="Proceedings of the Annual Computer Security Applications Conference~(ACSAC)",
    year="2019"
}
```

### Abstract:

Given the success of the Web platform, attackers have abused its main programming language, namely JavaScript, to mount different types of attacks on their victims. Due to the large volume of such malicious scripts, detection systems rely on static analyses to quickly process the vast majority of samples. These static approaches are not infallible though and lead to misclassifications. Also, they lack semantic information to go beyond purely syntactic approaches.
In this paper, we propose JStap, a modular static JavaScript detection system, which extends the detection capability of existing lexical and AST-based pipelines by also leveraging control and data flow information.
Our detector is composed of ten modules, including five different ways of abstracting code, with differing levels of context and semantic information, and two ways of extracting features. Based on the frequency of these specific patterns, we train a random forest classifier for each module.

In practice, JStap outperforms existing systems, which we reimplemented and tested on our dataset totaling over 270,000 samples. To improve the detection, we also combine the predictions of several modules. A first layer of unanimous voting classifies 93% of our dataset with an accuracy of 99.73%, while a second layer--based on an alternative modules' combination--labels another 6.5% of our initial dataset with an accuracy over 99%. This way, JStap can be used as a precise pre-filter, meaning that it would only need to forward less than 1% of samples to additional analyses. For reproducibility and direct deployability of our modules, we make our system publicly available.


## License

This project is licensed under the terms of the AGPL3 license which you can find in ```LICENSE```.