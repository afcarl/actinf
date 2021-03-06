# actinf - Active Inference

A set of sensorimotor learning experiments using the active inference
and predictive coding approach. Inspired by the input from Bruno Lara
and Guido Schillaci.

This just came out of my main work repo as a submodule for easier
sharing. This module is work in progress so please expect the
interface to be changing while integration is happening. If you want
to make changes, fork and send a pull request.

**Update 20170607** I'm freezing this repository in favor of
  integrating the experiments and models into
  [https://github.com/x75/smp_graphs](smp_graphs).

## Dependencies

Apart from the base dependencies numpy, scipy, sklearn, and matplotlib
which you can install via distro mechanisms or pip, we need additionally:
 - explauto for the environments, simple_arm and pointmass. You will
   need to use my fork of explauto at https://github.com/x75/explauto
   and checkout the "smp" branch
 - PyPR: pip install pypr for gaussian mixtures with conditional inference
 - kohonen, SOM library, also need to use my fork at
   https://github.com/x75/kohonen, branch master in this case. We need this in active_inference_hebbsom.py
 - otl: Harold Soh's online temporal learning library, use my github
   import from https://github.com/x75/otl and follow build instruction
   there, call cmake to build _with_ python wrappers

I just build these locally and export each directory into my PYTHONPATH.

## Files

 - actinf_models.py: knn, soesgp and storkgp based online learning
   models
 - active_inference_basic.py: basic experiments, here we use explauto's
   simple_arm in low-dimensional configuration (3 joints) to learn to
   control the arm under dynamic online goals. Several program
   execution modes are available:
  - m1_goal_error_nd: most basic proprioceptive only model
  - m2_error_nd: most basic proprioceptive only model
  - m1_goal_error_nd_e2p: this introduces an e2p map that's built with a
    gaussian mixture model using PyPR lib so we can pass down
    exteroceptive goals to the proprioceptive layer
  - test_models: basic model test
  - plot_system: plot the system response as scattermatrix
  - m1_goal_error_1d: demonstrate the basic operation of model
    type M1 on a one-dimensional system
  - m2_error_nd_ext: analyze prediction errors on model type M2
 - active_inference_naoqi.py: base proprio only learning using webots
   and naoqi on a simulated nao
 - active_inference_hebbsom.py: this is just replicating the gaussian
   mixture model functionality using SOMs with Hebbian associative
   connections and conditional inference

## Examples

Run it like

    python active_inference_basic.py --mode m1_goal_error_nd

which will run the basic proprioceptive only learning scenario.

This will also run a p-only learning but different setup, FWDp only get's the error
as an input.

    python active_inference_basic.py --mode m2_error_nd --numsteps 2000 --model knn

This command

    python active_inference_basic.py --mode m1_goal_error_nd_e2p --model knn --numsteps 1000

which will run the combined etxtero/proprio learning scenario for 1000
timesteps (the default anyway) and produce various plots along the way.

Runs of active_inference_basic.py will produce a file "EP.npy" in the
current directory. This needs to be passed to
active_inference_hebbsom.py as a datafile to load for testing only the
e2p prbabilistic mapping part using the Hebbian SOMs.

I run it like

    python active_inference_hebbsom.py --datafile data/simplearm_n3000/EP.npy

which contains data from a 3000 timesteps run of
active_inference_basic.py. on the first run, hebbsom will first learn
the SOMs for E and P, then learn the Hebbian connections (I separated
it for debugging, rejoining the processes is TODO, considers large
initial neighborhood_size and decreasing it with time) and finally
evaluate the learnt mapping by feeding extero signal to the E map,
activating P map via Hebbian links, sample from P map joint density,
feed sampled P_ into the real system and compute the end effector
position.
