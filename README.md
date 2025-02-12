OpenModel application ontologies
================================
A few application ontologies have been made publicly available in OpenModel.

## Computation application ontology
The [computation application ontology] ontologises the terms used by the
[ExecFlowAPI], which is a REST API for configuring [ExecFlow] to run
simulations on remote HPC clusters.
## Simulation Software ontology
The [simulation software ontology] describes the Materials Modelling Software used in OpenModel's success stories, and expands the EMMO taxonomy of material relations with classes covering the methods used in the most common software packages. It also defines the following classes:
* *SimulatedMaterial*, assigning semantic meaning to the material taking part in a computer simulation;
*  *InputControlData*, to mark a dataset containing the parameters used in numerical simulations.
## Success story 1 application ontology
The [success story 1 application ontology] describes a multiscale workflow to optimise electronic devices based on phase-changing materials. The example provided covers the much simpler case of a MOSFET transistor based on silicon, resulting in a workflow that can easily be run on a laptop computer. Various ABox of increasing complexity are shipped with this example to test the filtering capability of OpenModel's semantic platorm.
## Success story 3 application ontology
The [success story 3 application ontology] is a simple open application ontology describing inputs and outputs of models to be used  in OpenModel [Success Story 3].
This version depends on a local copy of the [microstructure ontology](./microstructure.ttl), which has been modified to reduce the number of EMMO imports.

[simulation software ontology]: ./simulationsoftware.ttl
[computation application ontology]: ./computation.ttl
[success story 1 application ontology]: ./ss1.ttl
[success story 3 application ontology]: ./ss3.ttl
[ExecFlow]: https://github.com/H2020-OpenModel/ExecFlow
[ExecFlowAPI]: https://github.com/H2020-OpenModel/ExecFlowAPI
[Success Story 1]: https://open-model.eu/success-stories/success-story-1
[Success Story 3]: https://open-model.eu/success-stories/success-story-3/