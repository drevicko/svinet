INSTALLATION
============

See file INSTALL for further details.

On Linux/Unix run

 ./configure 
 make; make install

On Mac OS, the required gsl, gslblas and pthread libraries may be in a
different location. Specify the location of these libraries to
configure:

 ./configure LDFLAGS="-L/opt/local/lib" CPPFLAGS="-I/opt/local/include"
 make; make install

The binary 'svinet' will be installed in /usr/local/bin unless a
different prefix is provided to configure. (See INSTALL.)


SVINET: fast stochastic variational inference of undirected networks

**svinet** [OPTIONS]

        -help           usage

        -file <name>    input tab-separated file with a list of undirected links

        -n <N>          number of nodes in network

        -k <K>          number of communities

        -batch          run batch variational inference

        -stratified     use stratified sampling
         * use with rpair or rnode options

        -rnode          inference using random node sampling

        -rpair          inference using random pair sampling

        -label          tag output directory

        -link-sampling  inference using link sampling 
			*recommended*

        -infset         inference using informative set sampling

        -preprocess     preprocess to run informative set sampling

        -rfreq          set the frequency at which
         * convergence is estimated
         * statistics, e.g., heldout likelihood are computed

        -max-iterations         maximum number of iterations (use with -no-stop to avoid stopping in an earlier iteration)

        -no-stop                disable stopping criteria

        -seed           set GSL random generator seed

        -gml            generate a GML format file that visualizes link communities



INPUT FILE FORMAT FOR UNDIRECTED GRAPHS
=======================================

See the example included file ./example/assort-75-4.txt.

Each line contains a tab-separated pair of node IDs corresponding to a
link. The file can contain duplicate links or directed links, but the
graph will be treated as undirected.

INFERENCE
=========

help:
svinet -help

*RECOMMENDED*
link sampling inference:
svinet -file ca-AstroPh.csv -n 17903 -k 20 -link-sampling

*OTHER OPTIONS*
batch inference:
svinet -file ca-AstroPh.csv -n 17903 -k 20 -batch

inference with random pair sampling:
svinet -file ca-AstroPh.csv -n 17903 -k 20 -rpair

inference with random node sampling:
svinet -file ca-AstroPh.csv -n 17903 -k 20 -rnode

inference with stratified random pair sampling:
svinet -file ca-AstroPh.csv -n 17903 -k 20 -rpair -stratified

inference with stratified random node sampling:
svinet -file ca-AstroPh.csv -n 17903 -k 20 -rnode -stratified

For other useful options, run

   svinet -help

Preprocessing for the informative set sampling option
=====================================================

This sampling option is meant to be used on large, sparse networks.

1. preprocess the network

svinet -file ca-AstroPh.csv -n 17903 -k 20 -preprocess

2. move the generated "informative set" file to the location where you
   will run svinet

mv n17903-k20-mmsb-massive/neighbors.bin .

3. run inference

svinet -file ca-AstroPh.csv -n 17903 -k 20 -massive

SAVING MODEL STATE
==================

At any point during inference you can send a signal to the running
svinet process to save results including the model state and
discovered groups. The signal is sent as follows. 

Find the process ID using the command "ps -ax | grep svinet" (on unix
systems). Then,

kill -TERM <process ID>

The results are also saved when the inference converges and the
process terminates.

STOPPING CRITERIA 
=================

The avg. heldout log likelihood at network sparsity is used to assess
convergence. (Details in the paper.)

The stopping criteria may need to be adjusted for best results. The
default settings are set as a function of the network size. 

For help with adjusting the stopping criteria please contact the
authors.

PARAMETERS
==========

If parameters or options cannot be set through the documented command
line options, you may be able to set them in env.hh. In this case,
recompile to create a new svinet binary. 

Most parameters used in inference are written to param.txt in the
output directory.

OUTPUT FILES
============

- For each inference run a directory is created, e.g., 

  n17903-k20-mmsb-linksampling (link sampling)
  n17903-k20-mmsb-rnode        (random node sampling)
  n17903-k20-mmsb-Srnode       (stratified random node sampling)
  n17903-k20-mmsb-rpair        (random pair sampling)
  n17903-k20-mmsb-Srpair       (stratified random pair sampling)
  n17903-k20-mmsb-infset       (informative set sampling)

- The files heldout.txt and validation.txt in the directory contain
  the heldout log likelihood on heldout sets.

  The columns are as follows: 

  iteration #, 
  cum. duration (secs), 
  avg. heldout log likelihood
  # samples
  avg. heldout log likelihood (link)
  # link samples
  avg. heldout log likelihood (non link)
  # nonlink samples
  avg. heldout log likelihood (link) at network sparsity
  avg. heldout log likelihood (nonlink) at network sparsity
  avg. heldout log likelihood at network sparsity 

  See the paper for how we compute log likelihood at network sparsity.

- groups.txt : mixed-memberships of nodes
  	       NOTE: the first column is a sequence
	       	     the second column is the node ID as in the input file
		     the last column is the most likely group membership
		     the columns in between are the expected
		     membership probabilities

- communities.txt : Each node is bucketed into a "link community"
  		    based on whether it exhibited at least one link
  		    that belongs to that community.

		    
- gamma.txt: posterior variational mixed-membership parameter
  	     (Dirichlet parameters)

- lambda.txt: inferred posterior community strengths (Beta parameters)


VISUALIZE LINK COMMUNITIES
==========================

After the model has been saved, i.e., gamma.txt and lambda.txt exist,
run the following command to visualize the link communities.

svinet -file ca-AstroPh.csv -n 17903 -k 20 -gml

The output file is written to network.gml. 

Load the file in Gephi to visualize it. Use each node's "group" member
and each edge's "color" member for "partitioning".  Use the bridgeness
to size each node. (First, copy bridgeness to a new column with column
type BigDecimal.)