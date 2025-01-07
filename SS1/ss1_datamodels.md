The wrapper for the Ginestra task:

```bash
ginestra-core-sim.exe -l "ginestra.lic" -i "input.ini" -p "input_parameter.tsv"
```

It depends on the specific OTEAPI pipelines providing the relevant data nodes:

```yaml
  - workflow: execflow.exec_wrapper
    inputs:
      shelljob:
       metadata:
         options:
           resources:
             num_machines: "{{ ctx.ginestra.m_n_machines }}"
             num_mpiprocs_per_machine: "{{ ctx.ginestra.m_n_mpiprocs_pm }}"
      command: "{{ ctx.ginestra.executable }}"
      arguments:
        - "-l"
        - "{{ ctx.ginestra.licence }}"
        - "-i"
        - "{{ ctx.ginestra.g_setup }}"
        - "-p"
        - "{parameters}"
      files:
        parameters:
          filename: "input_parameters.in"
          node: "{{ ctx.g_parameters }}"
      outputs:
        - test.log
    postprocess:
      - "{{ ctx.current.outputs['test_log']|to_ctx('g_output') }}"
```

This is an example of a dataset for SS1 written as a single-file in YAML format. It needs a pipeline to pass the file locations (each, a string) into a corresponding AiiDA datanode.

```yaml
---
uFDE13MVbM:
  meta: http://ontotrans.eu/meta/0.1/InputGinestraSimulation
  dimensions: {}
  properties:
     executable: /tmp/ss1/dummy/ginestra-core-sim.exe
     licence: /tmp/ss1/Inputs/ginestra.lic
     g_input: /tmp/ss1/Inputs/input1.ini
     g_parameters: /tmp/ss1/Inputs/input_parameter1.tsv
     m_n_machines: 1
     m_n_mpiprocs_pm: 1
...
```

The datamodel for GSetup0:

```python
 mappings = [
  (GSET,                 EMMO.isDescriptionFor, EMMO.ManufacturedProduct),
  (GSET.executable,      MAP.mapsTo,            SIMSOFT:Ginestra),
  (GSET.licence,         MAP.mapsTo,            SIMSOFT:LicenceFile),
  (GSET.g_input,         MAP.mapsTo,            SIMSOFT:TOMLFile),
  (GSET.g_input,         EMMO.isDescriptionFor, EMMO.ManufacturedProduct),
  (GSET.g_parameters,    MAP.mapsTo,            SIMSOFT:TOMLFile),
  (GSET.g_parameters,    EMMO.isDescriptionFor, EMMO.ManufacturedProduct),
  (GSET.m_n_machines,    MAP.mapsTo,            SIMSOFT:MPIComputerNode),
  (GSET.m_n_mpiprocs_pm, MAP.mapsTo,            SIMSOFT:MPIProcsPerNode)
]
```



To increase the level of detail in the description of the input dataset, the `g_parameters` section is provided as a separate input file, exposing its internal structure.

* GSetup1

  ```yaml
  ---
  egbRESXwrn:
    meta: http://ontotrans.eu/meta/0.1/InputGinestraSimulation
    dimensions: {}
    properties:
       executable: /tmp/ss1/dummy/ginestra-core-sim.exe
       licence: /tmp/ss1/Inputs/ginestra.lic
       g_input: /tmp/ss1/Inputs/input1.ini
       m_n_machines: 1
       m_n_mpiprocs_pm: 1
  ...
  ```
  
* GParameters (it needs a dedicated pipeline and template file for serialisation).
  ```yaml
  ---
  VHNl24cU4M:
    meta: http://ontotrans.eu/meta/0.1/GinestraParameters
    dimensions: {}
    properties:
       bandgap: 5.0
       effective_mass: 0.5
       temperature: 298.5
       grid_size: 100
  ...
  ```

The datamodel for GParameters:

```python
 mappings = [
  (GPAR,                EMMO.isDescriptionFor, EMMO.ManufacturedProduct),
  (GPAR.bandgap,        MAP.mapsTo,            SS1:BandGap),
  (GPAR.bandgap,        EMMO.isPropertyFor,    SS1:VirtualMaterial),
  (GPAR.effective_mass, MAP.mapsTo,            EMMO:EffectiveMass),
  (GPAR.effective_mass, EMMO.isPropertyFor,    SS1:VirtualMaterial),
  (GPAR.temperature,    MAP.mapsTo,            EMMO.ThermodynamicTemperature),
  (GPAR.temperature,    EMMO.isDescriptionFor, EMMO.ManufacturedProduct),
  (GPAR.grid_size,      MAP.mapsTo,            MICROSTR.Grid)
]
```

However, the user-defined parameters in `GParameters` refer both to the device and the material it is composed of. To enable proper filtering of this dataset, we need separate the properties related to the device and those related to the material, while the system-independent numerical parameters are added to `GSetup2`. The `g_input` section too is expanded and its content split between the `GDevice` and `GSetup`.

* GSetup (needs a pipeline for defining the datanodes for the wrapper, and one to serialise the file `input.ini`).

  ```yaml
  ---
  Tf0GCu7eH3:
    meta: http://ontotrans.eu/meta/0.1/InputGinestraSimulation
    dimensions: {}
    properties:
       executable: /tmp/ss1/dummy/ginestra-core-sim.exe
       licence: /tmp/ss1/Inputs/ginestra.lic
       m_n_machines: 1
       m_n_mpiprocs_pm: 1
       input_template: /tmp/ss1/Inputs/ginestra_input.template # generic
       state: /tmp/ss1/Inputs/state.hdf5
       out_folder: ./
       output: sim_device.hdf5
  ...
  ```

* GDevice: this dataset contains all the data referring to a device, including the user-controlled parameters for the virtual experiment. In this case, the oxide layer (HfO<sub>2</sub>) is never changed, but the semiconductor material can be overridden with the parameter file, whose template is also part of the input dataset[^1].

  ```yaml
  ---
  4I4DsZ4Kjj:
    meta: http://ontotrans.eu/meta/0.1/GinestraDevice
    dimensions: {}
    properties:
       g_device: /tmp/ss1/Inputs/device1.xml
       g_test: /tmp/ss1/Inputs/test1.xml
       param_template: /tmp/ss1/Inputs/ginestra_input_parameters1.template # test1
       temperature: 298.5
       grid_size: 100
  ...
  ```

  and its mapping:

  ```python
  mappings = [
    (GDEV,                EMMO.isDescriptionFor, EMMO.ManufacturedProduct),
    (GDEV,                EMMO.hasProperPart,    SS1.VirtualMaterial),
    (GDEV.g_device,       MAP.mapsTo,            SIMSOFT:XMLFile),
    (GDEV.g_device,       EMMO.isDescriptionFor, EMMO.ManufacturedProduct),
    (GDEV.g_test,         MAP.mapsTo,            SIMSOFT:XMLFile),
    (GDEV.param_template, MAP.mapsTo,            SIMSOFT:Jinja2File),
    (GDEV.temperature,    MAP.mapsTo,            EMMO.ThermodynamicTemperature),
    (GDEV.temperature,    EMMO.isDescriptionFor, EMMO.ManufacturedProduct),
    (GDEV.grid_size,      MAP.mapsTo,            MICROSTR.Grid)
  ]
  ```

* GMaterial

  ```yaml
  ---
  nThlFwkFpI:
    meta: http://ontotrans.eu/meta/0.1/GinestraMaterial
    dimensions: {}
    properties:
       bandgap: 5.0
       effective_mass: 0.5
  ...
  ```

  and its mapping:

  ```python
  mappings = [
    (GMAT,                EMMO.isDescriptionFor, SS1.VirtualMaterial),
    (GMAT.bandgap,        MAP.mapsTo,            SS1:BandGap),
    (GMAT.bandgap,        EMMO.isPropertyFor,    SS1:VirtualMaterial),
    (GMAT.effective_mass, MAP.mapsTo,            EMMO:EffectiveMass),
    (GMAT.effective_mass, EMMO.isDescriptionFor, SS1:VirtualMaterial)
  ]
  ```

[^1]: Verify that a data pipeline can accept this parameter as input, or that OntoConv can write a DLite function using this parameter from the dataset. In the latter case, the suffix `_template` should be reserved for this function only.
