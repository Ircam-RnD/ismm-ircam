# ISMM - C++ Libraries

## Signal processing

### PiPo

- sources: https://github.com/Ircam-RnD/pipo
- documentation: http://recherche.ircam.fr/equipes/temps-reel/mubu/pipo/sdk-doc-v0.1/

#### Description

PiPo (for Plug In Plug Out / Plugin Interface for Processing Objects) is a C++ library that has been developed by Norbert Schnell, Diemo Schwarz, Riccardo Borghesi et al. in the ISMM team at IRCAM since 2013.
It is designed as a modular signal processing plugin API for real-time and offline processing, and is distributed with its own collection of modules, which are mostly based on the [rta](https://github.com/Ircam-RnD/rta-lib) library.

#### SDK

The PiPo SDK allows for easy development of new custom modules. It has its own github repository, https://github.com/Ircam-RnD/pipo-sdk, which is distributed as a git submodule of the PiPo library. New modules created with the SDK can directly be used in combination with the existing ones.

#### Host

To use PiPo modules, one needs a PiPo host. A basic one is provided in the SDK and can be extended to suit various use cases. Basic classes to build a custom host from scratch are also available in the SDK.

#### Max/MSP

PiPo lives as Max/MSP externals in the [MuBu](http://forumnet.ircam.fr/fr/produit/mubu/) package. The Max PiPo hosts are the **pipo**, **pipo~** (real-time) and **mubu.process** (offline) objects. The SDK also provides a basic example of how to build a PiPo module for Max.

#### Basic PiPo principles

The signals that PiPo modules are able to process are streams of bidimensional matrices of any (reasonable) size and frame rate. The input and output sizes and frame rates can be different. The drawback of this flexibility is that once plugged together, they have to be initialized before they can be used, because the properties of the signal they send depend on the properties of the signal they receive.
They also have to be reinitialized each time the input signal attributes change.

The typical use of PiPo is the following :

- plug some modules together into a chain or a graph
- initialize the modules with the streamAttributes function
- send some signal into the chain or the graph and get the processed signal at the output

#### Current status / recent improvements

The most recent paper about PiPo at the time of this writing is an [ISMIR 2017 article](https://ismir2017.smcnus.org/wp-content/uploads/2017/10/125_Paper.pdf).

It describes the latest developments of the API, some of which are:

- the possibility to create graphs of modules thanks to the **PiPoSequence** and **PiPoParallel** classes
- the possibility to instantiate such graphs using a simple syntax similar to FAUST (see the article for details), thanks to the **PiPoGraph** class

A notable constraint on the graph feature is that each class deriving from PiPo is a PiPo, which means PiPoSequence, PiPoParallel and PiPoGraph are also PiPos, which means they must have a single input and a single output, which means a PiPoGraph must always merge all of its branches at the end of the graph.

To allow the instantiation from a graph string description, a PiPo factory is needed. There’s a class implementing a generic factory in the library, named **PiPoCollection**. Its header is located in the SDK and its implementation (defining the base list of available modules) is located in the library. PiPoCollection can also add new user defined PiPo classes to its list to create more fancy graphs.

The PiPoGraph class functions in the following way:

- parse the description string to create an internal graph representation
- use the PiPoCollection class to instantiate individual PiPo operators and fill the internal representation
- plug the PiPo operator instances together using the PiPoSequence and PiPoParallel classes

#### Integration into the C++ RAPID-MIX API

To provide the most possible user friendly API for PiPo through the [RAPID-MIX API](http://gitlab.doc.gold.ac.uk/rapid-mix/RAPID-MIX_API), the chosen solution was to only expose a host class. Basically, this class allows to do everything the base PiPo host class can do, and is also able to serialize / deserialize the whole state of a graph in JSON notation, by storing the graph description, the streamAttributes, and the parameter types, names and values of each operator.

##### known issues / problems

The PiPoParallel operator doesn’t know how to merge heterogeneous frame rates, so its output can be an unusable mess. The only solution for now would be to improve the PiPoMerge private class in PiPoParallel, and allow it to choose between various heterogeneous streams merging policies:

- dumb policy (as it is right now): consider all streams to have the same frame rate
- have a master branch that duplicates (or better : interpolates if possible) last stored values from other branches
- have each branch triggering the output with the last stored values of the other branches
- … better ideas ?

The PiPoHost base class has to be overridden, no elegant solution was found for the moment to pass callbacks to PiPo operators, as it can be done with waves-lfo. Is there any C++ pattern that could allow to do this ?

##### @todo

- implement merging policies in PiPoMerge
- think of a way to implement callbacks on individual PiPo operators.

## Machine learning

### XMM

- sources: https://github.com/Ircam-RnD/xmm
- documentation: http://ircam-rnd.github.io/xmm/

#### Description

XMM is a portable, cross-platform C++ library that implements Gaussian Mixture Models and Hidden Markov Models for recognition and regression. The XMM library was developed for movement interaction in creative applications and implements an interactive machine learning workflow with fast training and continuous, real-time inference.
It was developed by Jules Françoise during his PhD thesis in the ISMM team at IRCAM.
