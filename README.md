# NOTE: The actual docker package for MultiFOLD, MultiFOLD_refine and ModFOLDdock is available here: https://hub.docker.com/r/mcguffin/multifold

Below is just a log of the procedure that was used to implement the initial container...

# MultiFOLD docker container
Docker implementation of MultiFOLD for predicting 3D protein models for quaternary structures. MultiFOLD is still under development and is part of the component methods of the IntFOLD 7 (https://www.reading.ac.uk/bioinf/IntFOLD/), which is a server for predicting 3D structures of proteins and their functions. This page shows how to run MultiFOLD container in your machine installed with docker client.

The docker container for the MultiFOLD runs the following programs:
- colabfold_batch (https://github.com/sokrypton/ColabFold) and local colabfold (https://github.com/YoshitakaMo/localcolabfold)
- DockQ (https://github.com/bjornwallner/DockQ)
- OpenStructure (https://openstructure.org/docs/2.3.1/install/)
- Voronota-js-voromqa (https://github.com/kliment-olechnovic/voronota/tree/master/expansion_js)
- maxit (binary - https://sw-tools.rcsb.org/apps/MAXIT/source.html, installation - https://sw-tools.rcsb.org/apps/MAXIT/README-source)
- parallel

## Running colabfold_batch
#### Instructions on running DockQ in the multifold container from the host
- To run colabfold_batch from the host the following step should be done
  Input files directory:
  - `export COLABFOLD_INPUT=/home/intfold/bajuna_test_docker_colabfold_batch/input_files/`
  Output files directory:
  - `export COLABFOLD_OUTPUT=/home/intfold/bajuna_test_docker_colabfold_batch/output_files/`
- And then run the following command:

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

### Running voronota-js-voromqa in multiple pdb files
To run multifold with voromqa in multiple pdb files, please use for loop commands as follow:-

Example
```
for pdbfile in *.pdb; do sudo docker run --rm -it --gpus all -v /home/intfold/bajuna_test_docker_voronota_js_voromqa/:/voronota-js_release/user_data/ multifold voronota-js-voromqa -i /voronota-js_release/user_data/$pdbfile; done

```

## Generating a full dockerfile from the multifold:latest image
The code below was used for this purpose
`alias dfimage="sudo docker run -v /var/run/docker.sock:/var/run/docker.sock --rm alpine/dfimage'
Then type: `dfimage -sV=1.36 multifold:latest`

For the full explanation regarding generating dockerfile from the image please read from the following links:
- https://stackoverflow.com/questions/19104847/how-to-generate-a-dockerfile-from-an-image
- https://hub.docker.com/r/alpine/dfimage

This command `dfimage -sV=1.36 multifold:latest` produced the following output:
```
Unable to find image 'alpine/dfimage:latest' locally
latest: Pulling from alpine/dfimage
df20fa9351a1: Pull complete 
820dbffe2156: Pull complete 
Digest: sha256:4a271e763d51b7f3cca72eac9bf508502c032665dde0e4c8d5fcf6376600f64a
Status: Downloaded newer image for alpine/dfimage:latest
Analyzing multifold:latest
Docker Version: 20.10.16
GraphDriver: overlay2
Environment Variables
|PATH=/usr/local/nvidia/bin:/usr/local/cuda/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
|NVARCH=x86_64
|NVIDIA_REQUIRE_CUDA=cuda>=11.2 brand=tesla,driver>=418,driver<419
|NV_CUDA_CUDART_VERSION=11.2.72-1
|NV_CUDA_COMPAT_PACKAGE=cuda-compat-11-2
|CUDA_VERSION=11.2.0
|LD_LIBRARY_PATH=/usr/local/nvidia/lib:/usr/local/nvidia/lib64
|NVIDIA_VISIBLE_DEVICES=all
|NVIDIA_DRIVER_CAPABILITIES=compute,utility

Image user
|User is root

Potential secrets:
|Found match openstructure/modules/io/tests/testfiles/sdf/compound-view.sdf Database file \.sdf$ 5327ce49bfe68842a46d97002022496c9d11f822b6330dfa67bb2fe7d7b8ae3b/layer.tar
|Found match openstructure/modules/io/tests/testfiles/sdf/compound.sdf Database file \.sdf$ 5327ce49bfe68842a46d97002022496c9d11f822b6330dfa67bb2fe7d7b8ae3b/layer.tar
|Found match openstructure/modules/io/tests/testfiles/sdf/empty_dataheader.sdf Database file \.sdf$ 5327ce49bfe68842a46d97002022496c9d11f822b6330dfa67bb2fe7d7b8ae3b/layer.tar
|Found match openstructure/modules/io/tests/testfiles/sdf/multiple.sdf Database file \.sdf$ 5327ce49bfe68842a46d97002022496c9d11f822b6330dfa67bb2fe7d7b8ae3b/layer.tar
|Found match openstructure/modules/io/tests/testfiles/sdf/properties.sdf Database file \.sdf$ 5327ce49bfe68842a46d97002022496c9d11f822b6330dfa67bb2fe7d7b8ae3b/layer.tar
|Found match openstructure/modules/io/tests/testfiles/sdf/simple.sdf Database file \.sdf$ 5327ce49bfe68842a46d97002022496c9d11f822b6330dfa67bb2fe7d7b8ae3b/layer.tar
|Found match openstructure/modules/io/tests/testfiles/sdf/wrong_atomcount.sdf Database file \.sdf$ 5327ce49bfe68842a46d97002022496c9d11f822b6330dfa67bb2fe7d7b8ae3b/layer.tar
|Found match openstructure/modules/io/tests/testfiles/sdf/wrong_atomlinelength.sdf Database file \.sdf$ 5327ce49bfe68842a46d97002022496c9d11f822b6330dfa67bb2fe7d7b8ae3b/layer.tar
|Found match openstructure/modules/io/tests/testfiles/sdf/wrong_atompos.sdf Database file \.sdf$ 5327ce49bfe68842a46d97002022496c9d11f822b6330dfa67bb2fe7d7b8ae3b/layer.tar
|Found match openstructure/modules/io/tests/testfiles/sdf/wrong_bondatomnumber.sdf Database file \.sdf$ 5327ce49bfe68842a46d97002022496c9d11f822b6330dfa67bb2fe7d7b8ae3b/layer.tar
|Found match openstructure/modules/io/tests/testfiles/sdf/wrong_bondcount.sdf Database file \.sdf$ 5327ce49bfe68842a46d97002022496c9d11f822b6330dfa67bb2fe7d7b8ae3b/layer.tar
|Found match openstructure/modules/io/tests/testfiles/sdf/wrong_bondlinelength.sdf Database file \.sdf$ 5327ce49bfe68842a46d97002022496c9d11f822b6330dfa67bb2fe7d7b8ae3b/layer.tar
|Found match openstructure/modules/io/tests/testfiles/sdf/wrong_bondtype.sdf Database file \.sdf$ 5327ce49bfe68842a46d97002022496c9d11f822b6330dfa67bb2fe7d7b8ae3b/layer.tar
|Found match openstructure/modules/io/tests/testfiles/sdf/wrong_charge.sdf Database file \.sdf$ 5327ce49bfe68842a46d97002022496c9d11f822b6330dfa67bb2fe7d7b8ae3b/layer.tar
|Found match openstructure/modules/io/tests/testfiles/sdf/wrong_dataheader.sdf Database file \.sdf$ 5327ce49bfe68842a46d97002022496c9d11f822b6330dfa67bb2fe7d7b8ae3b/layer.tar
|Found match boost_1_79_0/libs/geometry/example/data/cities.sql Database file \.sql$ a165191e30fcc05e8fedcc2a92e454a300e9695fb7d4572d5397cbb793b98f66/layer.tar
|Found match boost_1_79_0/libs/thread/test/sync/conditions/condition_variable/default_pass.cpp dbman default.pass a165191e30fcc05e8fedcc2a92e454a300e9695fb7d4572d5397cbb793b98f66/layer.tar
|Found match boost_1_79_0/libs/thread/test/sync/conditions/condition_variable_any/default_pass.cpp dbman default.pass a165191e30fcc05e8fedcc2a92e454a300e9695fb7d4572d5397cbb793b98f66/layer.tar
|Found match boost_1_79_0/libs/thread/test/sync/futures/future/default_pass.cpp dbman default.pass a165191e30fcc05e8fedcc2a92e454a300e9695fb7d4572d5397cbb793b98f66/layer.tar
|Found match boost_1_79_0/libs/thread/test/sync/futures/promise/default_pass.cpp dbman default.pass a165191e30fcc05e8fedcc2a92e454a300e9695fb7d4572d5397cbb793b98f66/layer.tar
|Found match boost_1_79_0/libs/thread/test/sync/futures/shared_future/default_pass.cpp dbman default.pass a165191e30fcc05e8fedcc2a92e454a300e9695fb7d4572d5397cbb793b98f66/layer.tar
|Found match boost_1_79_0/libs/thread/test/sync/mutual_exclusion/locks/lock_guard/default_pass.cpp dbman default.pass a165191e30fcc05e8fedcc2a92e454a300e9695fb7d4572d5397cbb793b98f66/layer.tar
|Found match boost_1_79_0/libs/thread/test/sync/mutual_exclusion/locks/nested_strict_lock/default_pass.cpp dbman default.pass a165191e30fcc05e8fedcc2a92e454a300e9695fb7d4572d5397cbb793b98f66/layer.tar
|Found match boost_1_79_0/libs/thread/test/sync/mutual_exclusion/locks/shared_lock/cons/default_pass.cpp dbman default.pass a165191e30fcc05e8fedcc2a92e454a300e9695fb7d4572d5397cbb793b98f66/layer.tar
|Found match boost_1_79_0/libs/thread/test/sync/mutual_exclusion/locks/shared_lock_guard/default_pass.cpp dbman default.pass a165191e30fcc05e8fedcc2a92e454a300e9695fb7d4572d5397cbb793b98f66/layer.tar
|Found match boost_1_79_0/libs/thread/test/sync/mutual_exclusion/locks/strict_lock/default_pass.cpp dbman default.pass a165191e30fcc05e8fedcc2a92e454a300e9695fb7d4572d5397cbb793b98f66/layer.tar
|Found match boost_1_79_0/libs/thread/test/sync/mutual_exclusion/locks/unique_lock/cons/default_pass.cpp dbman default.pass a165191e30fcc05e8fedcc2a92e454a300e9695fb7d4572d5397cbb793b98f66/layer.tar
|Found match boost_1_79_0/libs/thread/test/sync/mutual_exclusion/locks/upgrade_lock/cons/default_pass.cpp dbman default.pass a165191e30fcc05e8fedcc2a92e454a300e9695fb7d4572d5397cbb793b98f66/layer.tar
|Found match boost_1_79_0/libs/thread/test/sync/mutual_exclusion/mutex/default_pass.cpp dbman default.pass a165191e30fcc05e8fedcc2a92e454a300e9695fb7d4572d5397cbb793b98f66/layer.tar
|Found match boost_1_79_0/libs/thread/test/sync/mutual_exclusion/null_mutex/default_pass.cpp dbman default.pass a165191e30fcc05e8fedcc2a92e454a300e9695fb7d4572d5397cbb793b98f66/layer.tar
|Found match boost_1_79_0/libs/thread/test/sync/mutual_exclusion/recursive_mutex/default_pass.cpp dbman default.pass a165191e30fcc05e8fedcc2a92e454a300e9695fb7d4572d5397cbb793b98f66/layer.tar
|Found match boost_1_79_0/libs/thread/test/sync/mutual_exclusion/recursive_timed_mutex/default_pass.cpp dbman default.pass a165191e30fcc05e8fedcc2a92e454a300e9695fb7d4572d5397cbb793b98f66/layer.tar
|Found match boost_1_79_0/libs/thread/test/sync/mutual_exclusion/shared_mutex/default_pass.cpp dbman default.pass a165191e30fcc05e8fedcc2a92e454a300e9695fb7d4572d5397cbb793b98f66/layer.tar
|Found match boost_1_79_0/libs/thread/test/sync/mutual_exclusion/timed_mutex/default_pass.cpp dbman default.pass a165191e30fcc05e8fedcc2a92e454a300e9695fb7d4572d5397cbb793b98f66/layer.tar
|Found match boost_1_79_0/libs/thread/test/threads/thread/constr/default_pass.cpp dbman default.pass a165191e30fcc05e8fedcc2a92e454a300e9695fb7d4572d5397cbb793b98f66/layer.tar
|Found match cmake-3.23.2/Templates/Windows/Windows_TemporaryKey.pfx Private client certificate \.pfx$ a165191e30fcc05e8fedcc2a92e454a300e9695fb7d4572d5397cbb793b98f66/layer.tar
|Found match cmake-3.23.2/Tests/VSWinStorePhone/Direct3DApp1/Direct3DApp1_TemporaryKey.pfx Private client certificate \.pfx$ a165191e30fcc05e8fedcc2a92e454a300e9695fb7d4572d5397cbb793b98f66/layer.tar
|Found match cmake-3.23.2/Tests/VSXaml/VSXaml_TemporaryKey.pfx Private client certificate \.pfx$ a165191e30fcc05e8fedcc2a92e454a300e9695fb7d4572d5397cbb793b98f66/layer.tar
|Found match usr/local/share/cmake-3.23/Templates/Windows/Windows_TemporaryKey.pfx Private client certificate \.pfx$ a165191e30fcc05e8fedcc2a92e454a300e9695fb7d4572d5397cbb793b98f66/layer.tar
|Found match colabfold_batch/colabfold-conda/share/terminfo/p/p12 Potential cryptographic key bundle .p12$ a6d5988734614a037169b73295b7338670014e738695280ede31db1268fa2b24/layer.tar
|Found match colabfold_batch/conda/pkgs/conda-content-trust-0.1.1-pyhd3eb1b0_0/info/test/tests/testdata/test_key_1_268B62D0.pri.asc Potential cryptographic key bundle \.asc$ a6d5988734614a037169b73295b7338670014e738695280ede31db1268fa2b24/layer.tar
|Found match colabfold_batch/conda/pkgs/conda-content-trust-0.1.1-pyhd3eb1b0_0/info/test/tests/testdata/test_key_1_268B62D0.pub.asc Potential cryptographic key bundle \.asc$ a6d5988734614a037169b73295b7338670014e738695280ede31db1268fa2b24/layer.tar
|Found match colabfold_batch/conda/pkgs/conda-content-trust-0.1.1-pyhd3eb1b0_0/info/test/tests/testdata/test_key_2_7DB43643.pri.asc Potential cryptographic key bundle \.asc$ a6d5988734614a037169b73295b7338670014e738695280ede31db1268fa2b24/layer.tar
|Found match colabfold_batch/conda/pkgs/conda-content-trust-0.1.1-pyhd3eb1b0_0/info/test/tests/testdata/test_key_2_7DB43643.pub.asc Potential cryptographic key bundle \.asc$ a6d5988734614a037169b73295b7338670014e738695280ede31db1268fa2b24/layer.tar
|Found match colabfold_batch/conda/pkgs/ncurses-6.3-h7f8727e_2/share/terminfo/p/p12 Potential cryptographic key bundle .p12$ a6d5988734614a037169b73295b7338670014e738695280ede31db1268fa2b24/layer.tar
|Found match colabfold_batch/conda/share/terminfo/p/p12 Potential cryptographic key bundle .p12$ a6d5988734614a037169b73295b7338670014e738695280ede31db1268fa2b24/layer.tar
|Found match etc/ssh/ssh_config Client SSH Config .?ssh_config[\s\S]* cdf5066252b0ccd7b133a8416989232fe279c045c34881f02d49635d94d17954/layer.tar
|Found match etc/ssh/ssh_config.d/ Client SSH Config .?ssh_config[\s\S]* cdf5066252b0ccd7b133a8416989232fe279c045c34881f02d49635d94d17954/layer.tar
|Found match usr/local/cuda-11.2/nsight-systems-2020.4.3/documentation/nsys-exporter/_static/composite.report32.sql Database file \.sql$ cdf5066252b0ccd7b133a8416989232fe279c045c34881f02d49635d94d17954/layer.tar
|Found match usr/local/cuda-11.2/nsight-systems-2020.4.3/documentation/nsys-exporter/_static/cuda_bt.report705.sql Database file \.sql$ cdf5066252b0ccd7b133a8416989232fe279c045c34881f02d49635d94d17954/layer.tar
|Found match usr/local/cuda-11.2/nsight-systems-2020.4.3/documentation/nsys-exporter/_static/cuda_kernel_api.report32.sql Database file \.sql$ cdf5066252b0ccd7b133a8416989232fe279c045c34881f02d49635d94d17954/layer.tar
|Found match usr/local/cuda-11.2/nsight-systems-2020.4.3/documentation/nsys-exporter/_static/extract_jsonl.ftrace.sql Database file \.sql$ cdf5066252b0ccd7b133a8416989232fe279c045c34881f02d49635d94d17954/layer.tar
|Found match usr/local/cuda-11.2/nsight-systems-2020.4.3/documentation/nsys-exporter/_static/fecs.gpuctxsw.sql Database file \.sql$ cdf5066252b0ccd7b133a8416989232fe279c045c34881f02d49635d94d17954/layer.tar
|Found match usr/local/cuda-11.2/nsight-systems-2020.4.3/documentation/nsys-exporter/_static/fps_histogram.dx12.sql Database file \.sql$ cdf5066252b0ccd7b133a8416989232fe279c045c34881f02d49635d94d17954/layer.tar
|Found match usr/local/cuda-11.2/nsight-systems-2020.4.3/documentation/nsys-exporter/_static/func_table.report32.sql Database file \.sql$ cdf5066252b0ccd7b133a8416989232fe279c045c34881f02d49635d94d17954/layer.tar
|Found match usr/local/cuda-11.2/nsight-systems-2020.4.3/documentation/nsys-exporter/_static/map.cunvtx.sql Database file \.sql$ cdf5066252b0ccd7b133a8416989232fe279c045c34881f02d49635d94d17954/layer.tar
|Found match usr/local/cuda-11.2/nsight-systems-2020.4.3/documentation/nsys-exporter/_static/origin.simpleCudaGraphs.sql Database file \.sql$ cdf5066252b0ccd7b133a8416989232fe279c045c34881f02d49635d94d17954/layer.tar
|Found match usr/local/cuda-11.2/nsight-systems-2020.4.3/documentation/nsys-exporter/_static/osrt.report32.sql Database file \.sql$ cdf5066252b0ccd7b133a8416989232fe279c045c34881f02d49635d94d17954/layer.tar
|Found match usr/local/cuda-11.2/nsight-systems-2020.4.3/documentation/nsys-exporter/_static/overhead.report32.sql Database file \.sql$ cdf5066252b0ccd7b133a8416989232fe279c045c34881f02d49635d94d17954/layer.tar
|Found match usr/local/cuda-11.2/nsight-systems-2020.4.3/documentation/nsys-exporter/_static/sched.report32.sql Database file \.sql$ cdf5066252b0ccd7b133a8416989232fe279c045c34881f02d49635d94d17954/layer.tar
|Found match usr/local/cuda-11.2/nsight-systems-2020.4.3/documentation/nsys-exporter/_static/sli.report109.sql Database file \.sql$ cdf5066252b0ccd7b133a8416989232fe279c045c34881f02d49635d94d17954/layer.tar
|Found match usr/local/cuda-11.2/nsight-systems-2020.4.3/documentation/nsys-exporter/_static/streams.report32.sql Database file \.sql$ cdf5066252b0ccd7b133a8416989232fe279c045c34881f02d49635d94d17954/layer.tar
|Found match usr/local/cuda-11.2/nsight-systems-2020.4.3/documentation/nsys-exporter/_static/syscall_histogram.ftrace.sql Database file \.sql$ cdf5066252b0ccd7b133a8416989232fe279c045c34881f02d49635d94d17954/layer.tar
|Found match localcolabfold-1.0.0/colabfold/colabfold-conda/etc/jupyter/nbconfig/notebook.d/widgetsnbextension.json Jupyter Configuration file jupyter[^ ]*config[^ ]*.json edd08ff686dabd5f346105c2c43f7ea1df88de8c143adefe212fb7f5ebc3fd40/layer.tar
|Found match localcolabfold-1.0.0/colabfold/colabfold-conda/lib/python3.7/site-packages/tornado/test/test.key openssl .key, apple .keychain, etc. \.key$ edd08ff686dabd5f346105c2c43f7ea1df88de8c143adefe212fb7f5ebc3fd40/layer.tar
|Found match localcolabfold-1.0.0/colabfold/colabfold-conda/share/terminfo/p/p12 Potential cryptographic key bundle .p12$ edd08ff686dabd5f346105c2c43f7ea1df88de8c143adefe212fb7f5ebc3fd40/layer.tar
|Found match localcolabfold-1.0.0/colabfold/conda/pkgs/conda-content-trust-0.1.1-pyhd3eb1b0_0/info/test/tests/testdata/test_key_1_268B62D0.pri.asc Potential cryptographic key bundle \.asc$ edd08ff686dabd5f346105c2c43f7ea1df88de8c143adefe212fb7f5ebc3fd40/layer.tar
|Found match localcolabfold-1.0.0/colabfold/conda/pkgs/conda-content-trust-0.1.1-pyhd3eb1b0_0/info/test/tests/testdata/test_key_1_268B62D0.pub.asc Potential cryptographic key bundle \.asc$ edd08ff686dabd5f346105c2c43f7ea1df88de8c143adefe212fb7f5ebc3fd40/layer.tar
|Found match localcolabfold-1.0.0/colabfold/conda/pkgs/conda-content-trust-0.1.1-pyhd3eb1b0_0/info/test/tests/testdata/test_key_2_7DB43643.pri.asc Potential cryptographic key bundle \.asc$ edd08ff686dabd5f346105c2c43f7ea1df88de8c143adefe212fb7f5ebc3fd40/layer.tar
|Found match localcolabfold-1.0.0/colabfold/conda/pkgs/conda-content-trust-0.1.1-pyhd3eb1b0_0/info/test/tests/testdata/test_key_2_7DB43643.pub.asc Potential cryptographic key bundle \.asc$ edd08ff686dabd5f346105c2c43f7ea1df88de8c143adefe212fb7f5ebc3fd40/layer.tar
|Found match localcolabfold-1.0.0/colabfold/conda/pkgs/ncurses-6.3-h7f8727e_2/share/terminfo/p/p12 Potential cryptographic key bundle .p12$ edd08ff686dabd5f346105c2c43f7ea1df88de8c143adefe212fb7f5ebc3fd40/layer.tar
|Found match localcolabfold-1.0.0/colabfold/conda/share/terminfo/p/p12 Potential cryptographic key bundle .p12$ edd08ff686dabd5f346105c2c43f7ea1df88de8c143adefe212fb7f5ebc3fd40/layer.tar
|Found match root/.ipython/profile_default/history.sqlite Database file \.sqlite$ edd08ff686dabd5f346105c2c43f7ea1df88de8c143adefe212fb7f5ebc3fd40/layer.tar
|Found match usr/local/share/cmake-3.23/Templates/Windows/.wh.Windows_TemporaryKey.pfx Private client certificate \.pfx$ feedc66f59fdcfaea678c9597e6dd6da342ee8ccffd12ced1fc5f203cf6377a9/layer.tar
Dockerfile:
CMD ["bash"]
ENV NVARCH=x86_64
ENV NVIDIA_REQUIRE_CUDA=cuda>=11.2 brand=tesla,driver>=418,driver<419
ENV NV_CUDA_CUDART_VERSION=11.2.72-1
ENV NV_CUDA_COMPAT_PACKAGE=cuda-compat-11-2
ARG TARGETARCH
LABEL maintainer=NVIDIA CORPORATION <cudatools@nvidia.com>
RUN |1 TARGETARCH=amd64 RUN apt-get update  \
	&& apt-get install -y --no-install-recommends gnupg2 curl ca-certificates  \
	&& curl -fsSL https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/${NVARCH}/3bf863cc.pub | apt-key add -  \
	&& echo "deb https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/${NVARCH} /" > /etc/apt/sources.list.d/cuda.list  \
	&& apt-get purge --autoremove -y curl  \
	&& rm -rf /var/lib/apt/lists/* # buildkit
ENV CUDA_VERSION=11.2.0
RUN |1 TARGETARCH=amd64 RUN apt-get update  \
	&& apt-get install -y --no-install-recommends cuda-cudart-11-2=${NV_CUDA_CUDART_VERSION} ${NV_CUDA_COMPAT_PACKAGE}  \
	&& ln -s cuda-11.2 /usr/local/cuda  \
	&& rm -rf /var/lib/apt/lists/* # buildkit
RUN |1 TARGETARCH=amd64 RUN echo "/usr/local/nvidia/lib" >> /etc/ld.so.conf.d/nvidia.conf  \
	&& echo "/usr/local/nvidia/lib64" >> /etc/ld.so.conf.d/nvidia.conf # buildkit
ENV PATH=/usr/local/nvidia/bin:/usr/local/cuda/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
ENV LD_LIBRARY_PATH=/usr/local/nvidia/lib:/usr/local/nvidia/lib64
COPY NGC-DL-CONTAINER-LICENSE / # buildkit
	NGC-DL-CONTAINER-LICENSE

ENV NVIDIA_VISIBLE_DEVICES=all
ENV NVIDIA_DRIVER_CAPABILITIES=compute,utility
bash
bash
bash
bash
bash
bash
bash
bash
bash
bash
bash
bash
bash
bash
bash
bash
mkdir -p /colabfold_batch/input_files /colabfold_batch/ouput_files
mv colabfold_batch/ouput_files colabfold_batch/output_files
mv colabfold_batch/output_files colabfold_batch/ouput_files
/bin/bash
mkdir -p DockQ/input_model DockQ/native_model
/bin/bash
/bin/bash
/bin/bash
/colabfold_batch/bin/colabfold_batch --help
/bin/bash
/bin/bash
ost
/bin/bash
/bin/bash

```

## Pushing MultiFOLD Container to the Docker Hub
Chacking first the latest container
```
sudo docker container ls
sudo docker images
```
Tag the image

`sudo docker image tag multifold:latest bsalehe/multifold_test:latest`

Lastly the image was pushed to my docker hub account "bsalehe" using the following command:

`sudo docker image tag multifold:latest bsalehe/multifold_test:latest`

## Running maxit
Installing maxit in the container first required linux homebrew package manager: https://www.how2shout.com/linux/how-to-install-brew-ubuntu-20-04-lts-linux/

Then write the following commands:
```
sudo apt update
sudo apt-get install build-essential
sudo apt install git -y
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
brew install brewsci/bio/maxit
```

### Instructions to run in the host machine (docker client)

Create the container path to maxit as follows:

- `export MAXIT_PATH=”/home/linuxbrew/.linuxbrew/bin/”`

Create in your host machine the paths where input and output files are actually located. Example:
```
MAXIT_INPUT_DIR="/home/intfold/bajuna_test_docker_maxit/inputfile/"
MAXIT_OUTPUT_DIR="/home/intfold/bajuna_test_docker_maxit/outputfile/"
```
In the MAXIT_INPUT_DIR there is a pdb file named "1EXB_r_l_b.model.pdb"

In the MAXIT_INPUT_DIR is where the output file will be stored after the job run. For instance, "1EXB_r_l_b.model.cif"

To run the maxit with the docker container from the host machine type the following command:
```
sudo docker run --rm -it --gpus all -v ${MAXIT_INPUT_DIR}:/maxit/input -v ${MAXIT_OUTPUT_DIR}:/maxit/output/ multifold ${MAXIT_PATH}maxit -input /maxit/input/1EXB_r_l_b.model.pdb -output /maxit/output/1EXB_r_l_b.model.cif -o 1
```

## Running Multifold with script
The simplest way to run MultiFOLD container is to use a script "run_MultiFOLD_docker.sh" as follow
```
sudo docker run -it --gpus all -v /home/intfold/Docker/MultiFOLD_Docker_run/MultiFOLD_input:/MultiFOLD_input -v /home/intfold/Docker/MultiFOLD_Docker_run/MultiFOLD_output:/MultiFOLD_output mcguffin/multifold /MultiFOLD/run_MultiFOLD_docker.sh T1197 T1197.fasta A1
```

