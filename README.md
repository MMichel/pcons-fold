PconsFold
=========

![PconsFold pipeline](https://github.com/MMichel/pcons-fold/blob/master/pipeline_horiz.png?raw=true)

A pipeline for protein folding using predicted contacts from PconsC and a  Rosetta folding protocol.

## Remark:
This branch is deprecated! Please use [https://github.com/ElofssonLab/pcons-fold](https://github.com/ElofssonLab/pcons-fold) instead.

## Pipeline overview:

1. __Input:__ fasta file containing one protein sequence
2. Prepare input for PconsC
3. Contact prediction with PconsC
4. Prepare input for Rosetta folding
5. Rosetta folding
6. Extract and relax structures with lowest Rosetta energy
7. __Output:__ the predicted contact map (also as a plot) and the top-ranked structural model(s) relaxed and non-relaxed


## Dependencies:

- [Rosetta v3.5 or weekly built](https://www.rosettacommons.org/software)
- [Jackhmmer from HMMER v3.0 or higher](http://hmmer.janelia.org/)
- [HHblits from HHsuite v2.0.16](http://toolkit.tuebingen.mpg.de/hhblits)
- [PSICOV v1.11](http://bioinfadmin.cs.ucl.ac.uk/downloads/PSICOV/)
- [plmDCA asymmetric](http://plmdca.csc.kth.se/)
- either MATLAB v8.1 or higher
- or [MATLAB Compiler Runtime (MCR) v8.1](http://www.mathworks.se/products/compiler/mcr/)

MATLAB is needed to run plmDCA. However, if MATLAB is not available you can also use a compiled version of plmDCA. For the compiled version to run you need to provide a path to MCR.


## How to run it:

__Make sure__ all dependencies are working correctly and adjust the paths in `localconfig.py`.

To run the full pipeline use:
```
./pcons_fold.py [-c n_cores] [-n n_decoys] [-m n_models]
                [-f factor] [--norelax] [--nohoms] 
                hhblits_database jackhmmer_database sequence_file
```
- Required:
  - `hhblits_database` and `jackhmmer_database` are paths to the databases used by HHblits and Jackhmmer
  - `sequence_file` is the path to the input protein sequence in FASTA format (only single sequences). 
- Optional:
  - `n_cores` specifies the number of cores to use during computation (default: number of available cores). 
  - `n_decoys` specifies  the number of decoy structures generated by Rosetta (default: 2000, see publication). 
  - `n_models` is the number of top-ranked models being extracted and eventually relaxed in the end (default: 10).
  - `factor` determines the number of constraints used to fold the protein, which is: `factor` * length_of_the_input_sequence (default: 1.0). 
  - `norelax` is a flag that supresses relaxation of the final models. This can be used to quickly extract structures in the end. 
  - `nohoms` is a flag that ensures that homologous structures are excluded from fragment picking. This is only useful in test cases if the model quality needs to be evaluated with a known structure.


---


You can also __run PconsC contact prediction__ independently with this command:
```
./pconsc/predict_all.py [-c cores] hhblits_database jackhmmer_database sequence_file
```


And then __fold the protein__ according to given predicted contacts with the following commands:
``` 
./folding/rosetta/prepare_input.py [-f factor] [--nohoms] sequence_file contact_map 

./folding/rosetta/fold.py [-c n_cores] [-n n_decoys] sequence_file rosetta_constraintfile

./folding/rosetta/extract.py [-c n_cores] [-m n_models] [--norelax] number_of_extracted_structures
```

The first script generates the file `(pconsc_output)-(factor).constraints` which is then used by Rosetta in the next step with `rosetta_constraintfile`.

