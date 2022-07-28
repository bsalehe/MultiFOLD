# MultiFOLD_docker
Docker implementation of MultiFOLD for predicting 3D protein models for quaternary structures. MoltiFOLD is still under development and is part of the component methods of the IntFOLD 7 (https://www.reading.ac.uk/bioinf/IntFOLD/), which is a server for predicting 3D structures of preoteins and their functions.

The docker container for the MultiFOLD run the following programs:
- colabfold_batch (https://github.com/sokrypton/ColabFold) and local colabfold (https://github.com/YoshitakaMo/localcolabfold)
- DockQ (https://github.com/bjornwallner/DockQ)
- OpenStructure (https://openstructure.org/docs/2.3.1/install/)
- Voronota-js-voromqa (https://github.com/kliment-olechnovic/voronota/tree/master/expansion_js)

## Running colabfold_batch
#### Instructions on running DockQ in the multifold container from the host
- To run colabfold_batch from the host the following step should be done
  Input files directory:
  - `export COLABFOLD_INPUT=/home/intfold/bajuna_test_docker_colabfold_batch/input_files/`
  Output files directory:
  - `export COLABFOLD_OUTPUT=/home/intfold/bajuna_test_docker_colabfold_batch/output_files/`
- Run this command:
`sudo docker run -it --gpus all --name colabfold_batch -v $COLABFOLD_INPUT:/colabfold_batch/input_files -v $COLABFOLD_OUTPUT:/colabfold_batch/output_files multifold /colabfold_batch/bin/colabfold_batch /colabfold_batch/input_files /colabfold_batch/output_files`

## Running DockQ
### Inside container
 ./DockQ.py examples/1A2K_r_l_b.model.pdb examples/1A2K_r_l_b.pdb -native_chain1 A B`

### Outside container
`sudo docker run --rm --gpus all -v /home/intfold/bajuna_test_docker_DockQ/model:/DockQ/input_model -v /home/intfold/bajuna_test_docker_DockQ/native:/DockQ/native_model multifold $DOCKQ_PYTHONDIR/python3 /DockQ/DockQ.py /DockQ/input_model/1EXB_r_l_b.model.pdb /DockQ/native_model/1EXB_r_l_b.pdb`

#### Output:
`Multi-chain model need sets of chains to group`
`use -native_chain1 and/or -model_chain1 if you want a different mapping than 1-1`
`Model chains  : ['A', 'B', 'D', 'C', 'E', 'G', 'F', 'H']`
`Native chains : ['A', 'B', 'D', 'C', 'E', 'G', 'F', 'H']`

#### Instructions on running DockQ in the multifold container from the host
- To run DockQ from the host the following step should be done
  - export DOCKQ_PYTHONDIR=/localcolabfold-1.0.0/colabfold/colabfold-conda/bin
- Create directory where actual model pdb file reside
  - Example: /home/intfold/bajuna_test_docker_DockQ/model
- Create directory where actual native pdb file reside
  - /home/intfold/bajuna_test_docker_DockQ/native
- Run the above command.

## Running OpenStructure
- Running OpenStructure user after pulling the image can just type:
  `sudo docker run --rm -it --gpus all multifold ost`

## Running voronota-js-voromqa
To run voronoa-js-voromqa from the host the following step should be done
 Input file directory:
 - Example: /home/intfold/bajuna_test_docker_voronota_js_voromqa/
- Run this command:
`sudo docker run --rm -it --gpus all -v /home/intfold/bajuna_test_docker_voronota_js_voromqa/:/voronota-js_release/user_data/ multifold voronota-js-voromqa -i /voronota-js_release/user_data/1EXB_r_l_b.pdb`
