The wrapper for the Ginestra task:

```bash
ginestra-core-sim.exe -l “ginestra.lic” -i “input.ini” -p “input_parameter.tsv”`
```

depends on the specific OTEAPI pipelines providing the relevant data nodes:

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

This is an example of a dataset for SS1 written as a single-file in YAML format. It needs a pipeline to pass the file location (a string) into a corresponding AiiDA datanode.

```yaml
---
uFDE13MVbM:
  meta: http://ontotrans.eu/meta/0.1/InputGinestraSimulation
  dimensions: {}
  properties:
     executable: /tmp/ss1/dummy/ginestra-core-sim.exe
     licence: /tmp/ss1/Inputs/ginestra.lic
     g_input: /tmp/ss1/Inputs/input.ini
     g_parameters: /tmp/ss1/Inputs/input_parameter.tsv
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
  (GSET.g_parameters,    MAP.mapsTo,            SIMSOFT:TOMLFile),
  (GSET.m_n_machines,    EMMO.isDescriptionFor, SIMSOFT:MPIComputerNode),
  (GSET.m_n_mpiprocs_pm, MAP.mapsTo,            SIMSOFT:MPIProcsPerNode)
]
```



To increase the level of detail in the description of the input dataset, the `g_parameters` section is provided as a separate input file, with its internal structure.

* GSetup1

  ```yaml
  ---
  egbRESXwrn:
    meta: http://ontotrans.eu/meta/0.1/InputGinestraSimulation
    dimensions: {}
    properties:
       executable: /tmp/ss1/dummy/ginestra-core-sim.exe
       licence: /tmp/ss1/Inputs/ginestra.lic
       g_input: /tmp/ss1/Inputs/input.ini
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
       id_region: 3
       bandgap: 5.0
       effective_mass: 0.5
  ...
  ```

The datamodel for GParameters:

```python
 mappings = [
  (GPAR,                EMMO.isDescriptionFor, EMMO.Material),
  (GPAR.id_region,      MAP.mapsTo,            SS1:RegionID),
  (GPAR.bandgap,        MAP.mapsTo,            SS1:BandGap),
  (GPAR.bandgap,        EMMO.isDescriptionFor, SS1:VirtualMaterial),
  (GPAR.effective_mass, MAP.mapsTo,            EMMO:EffectiveMass),
  (GPAR.effective_mass, EMMO.isDescriptionFor, SS1:VirtualMaterial)
]
```

In the most expressive representation of the input datasets, the `g_input` section is expanded into a device and test datasets. The other parameters in the file `input.ini` are added to `GSetup`.

* GSetup2 (needs a pipeline for defining the datanodes for the wrapper, and one to serialise the file `input.ini`).

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
       state: /tmp/ss1/Inputs/state.hdf5
       out_folder: ./
       output: sim_device.hdf5
  ...
  ```

* GParameters remains the same.

* GDevice

  ```yaml
  ---
  4I4DsZ4Kjj:
    meta: http://ontotrans.eu/meta/0.1/GinestraDevice
    dimensions: {}
    properties:
       g_device: /tmp/ss1/Inputs/device.xml
  ...
  ```

* GTestOperation

  ```yaml
  ---
  nThlFwkFpI:
    meta: http://ontotrans.eu/meta/0.1/GinestraTest
    dimensions: {}
    properties:
       g_device: /tmp/ss1/Inputs/test.xml
  ...
  ```

