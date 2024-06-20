# SD Events (SDE)

## Description

SD Events (SDE) is an experimental proposal for defining event sheets/macros to operate Stable Diffusion in a structured and efficient manner. This documentation outlines the Domain-Specific Language (DSL) syntax and core objects available for creating event sheets.

## DSL

Event sheets/macros are written using a dedicated DSL. The SDE DSL is a subset of Python. This means that any valid Python code is also valid SDE code. However, SDE code is not guaranteed to be valid Python code, as it may contain additional syntax and constructs specific to the SDE DSL. Here's an overview of the key components:

### Event sheets

A script can contain one or multiple event sheets. When multiple event sheets are present, they execute in a top-down order, from the first sheet to the last. Event sheets are defined in `.sheet` files. Sheet files are text files that contain event sheets/macros written in the SDE DSL. The file extension for sheet files is `.sheet`. Sheet files can be executed using the sde command-line tool.

```
Sheet EventSheet:
  ...
```

```
Sheet EventSheet1:
  ...

Sheet EventSheet2:
  ...
```

### Order of operation

Within an event sheet, operations are executed sequentially from top to bottom. This allows for a clear and predictable flow of actions.

### Core objects

SDE introduces a set of built-in core objects that represent fundamental entities in Stable Diffusion tasks:

- **Prompt:** An image prompt. Can be `positive` or `negative`.
- **Generator:** An image generator. Has most of the generation parameters.
- **Concept:** An image concept, i.e. LoRA, LyCORIS, DORA, TI, etc.
- **Effect:** An image effect or transformation.
- **Image:** A resulting image.
- **Control:** Controlnet-related parameters.

The core objects are interconnected and work together to produce the desired image:

```
Prompt → Generator → Control → Effect → Image
```

## Defining Core Objects

Objects in SDE can be defined **outside of Sheet blocks** in sheet files using the obj keyword. This allows for reusable object definitions that can be used across multiple sheets. Object definitions must be valid Python code and must follow the syntax and conventions of the SDE DSL.

```
obj MyGenerator:
  props:
    model: "stable_diffusion_v1"
  func:
    custom_method: lambda self: None

Sheet MySheet:
  # your actions
```

Core objects can be called using three different approaches:

1. **Calls:** Core objects can be instantiated by calling their constructors directly.

   ```
   Prompt();
   Generator();
   ...
   ```

2. **Method Calls:** Core objects can also be defined by calling the `set` method on the corresponding object.

   ```
   Prompt.set();
   Generator.set();
   ...
   ```

3. **Inline Definition:** Core objects can be defined inline by providing their properties as an object literal.

   ```
   Generator.set({
     "model": "stable_diffusion_v1",
     "concepts": [
       Concept({
         "path": "my/lora.safetensors",
         "strength": 1
       }),
       Concept({
         "path": "my/lyco.safetensors",
         "strength": 1
       })
     ],
     "steps": 50
   });
   ```

## Common Methods

To interact with core objects, SDE provides a set of common methods:

| Method | Type | Value | Description |
|--------|------|-------|-------------|
| set | Function | n/a | Sets a property value |
| get | Function | n/a | Retrieves a property value |

## Prompt Object

| Method | Type | Value | Description |
|--------|------|-------|-------------|
| isPositive | Function | n/a | Checks if the prompt is positive |
| isNegative | Function | n/a | Checks if the prompt is negative |
| toggle | Function | n/a | Toggles the prompt polarity |
| positive | Property | \<text> | Sets the positive image prompt |
| negative | Property | \<text> | Sets the negative image prompt |

```
prompt.set({"positive":"A beautiful sunset"})
prompt.toggle() // Switches to negative prompt
```

## Generator Object

| Method | Type | Value | Description |
|--------|------|-------|-------------|
| seed | Property | \<number> | Generation seed |
| model | Property | \<Model/array> | Stable Diffusion model(s) to use |
| concepts | Property | \<array> | Array of image concepts |
| scheduler | Property | \<text/enum> | Scheduler type |
| steps | Property | \<number> | Number of generation steps |
| CFG | Property | \<number> | Strictness of prompt adherence |
| VAE | Property | \<text> | Stable Diffusion VAE |
| VAE.precision | Property | \<text/enum> | VAE precision level |

```
generator.model("stable_diffusion_v1")
generator.concepts = [Concept(
      "path":"my/lora.safetensors"
      "strength":1
),
    Concept(
      "path":"my/lyco.safetensors"
      "strength":1
    )];
generator.steps = 50
- or -
generator.set({
  "model":"stable_diffusion_v1",
  "concepts":[
    Concept(
      "path":"my/lora.safetensors"
      "strength":1
    ),
    Concept(
      "path":"my/lyco.safetensors"
      "strength":1
    )
  ],
  "steps":50,
})
```

### Model Object

| Method | Type | Value | Description |
|--------|------|-------|-------------|
| set | Function | n/a | Sets the Stable Diffusion model |
| get | Function | n/a | Retrieves the Stable Diffusion model |
| type | Property | n/a | Model type (huggingface, web, local) |
| path | Property | \<text> | Path to the model file |

```
generator.set({"model":{"type":"huggingface","path":"stable_diffusion_v2"}});
- or -
model.type = "huggingface";
model.path = "stable_diffusion_v2";
```

### Concept Object

| Method | Type | Value | Description |
|--------|------|-------|-------------|
| set | Function | n/a | Sets a property value |
| get | Function | n/a | Retrieves a property value |
| path | Property | \<text> | Path to the concept file |
| strength | Property | \<number> | Strength of the concept |

```
generator.set({"concepts":[{"path":"path/to/concept.safetensor",strength:1.0}]});
- or -
concept.path = "path/to/concept.safetensor";
concept.strength = 0.8;
```

## Control Object

| Method | Type | Value | Description |
|--------|------|-------|-------------|
| type | Property | \<text/enum> | Controlnet type (IP-adapter/controlnet) |
| model | Property | \<text/enum> | Controlnet model |
| control | Property | \<number/enum> | Level of control |
| weight | Property | \<number> | Controlnet weight |
| begin | Property | \<number> | Beginning step |
| end | Property | \<number> | Ending step |
| reference | Property | \<text> | Reference image path |

```
control.type = "controlnet";
control.model = "controlnet_v1";
control.weight = 0.7;
```

## Effect Object

| Method | Type | Value | Description |
|--------|------|-------|-------------|
| list | Function | n/a | Lists available effects |
| ? | ? | \<text> | Placeholder for effect definition |

## Image Object

| Method | Type | Value | Description |
|--------|------|-------|-------------|
| width | Property | \<number> | Width of the resulting image |
| height | Property | \<number> | Height of the resulting image |

```
image.width = 1024;
image.height = 768;
```

## YAML Format

SDE sheets can also be defined using a YAML format. The YAML format allows for more flexibility and customization in the definition of objects and actions in the sheet. The YAML format consists of the following sections:

- `dependencies`: A list of Python packages that the sheet depends on.
- `objects`: A dictionary of object definitions, where each object definition consists of a `name` field and a `props` field. The `props` field is a dictionary of property definitions, where each property definition consists of a `type` field, a `value` field, and a `required` field.
- `sheets`: A dictionary of sheet definitions, where each sheet definition consists of a list of actions. Each action is a dictionary that consists of an `action` field and a `props` field. The `action` field specifies the type of action, and the `props` field specifies the properties of the action.

Here's an example of a YAML file that defines a single sheet:

```yaml
dependencies:
  - os
  - crypto

objects:
  MyGenerator:
    props:
      model:
        type: str
        value: "stable_diffusion_v1"
        required: true
      concepts:
        type: list[Concept]
        value: []
        required: false
      steps:
        type: int
        value: 50
        required: false

sheets:
  mySheet:
    - action: prompt
      props:
        positive: "A beautiful sunset"
    - action: create
      type: MyGenerator
      props:
        model: "stable_diffusion_v2"
        concepts:
          - path: "my/lora.safetensors"
            strength: 1.0
        steps: 75
    - action: image
      props:
        width: 1024
        height: 768
        path: "output.png"
```

## CLI Commands

The `sde.py` script provides the following CLI commands:

- `sde.py <sheet_file>`: Executes the specified SDE sheet file.
- `sde.py --from-yaml <yaml_file> <sheet_file>`: Converts the specified YAML file to a SDE sheet file, and saves the result to the specified sheet file.

Here's an example of how to use the `sde.py` script to execute a SDE sheet file:

```
python sde.py mySheet.sheet
```

Here's an example of how to use the `sde.py` script to convert a YAML file to a SDE sheet file:

```
python sde.py --from-yaml myYaml.yaml mySheet.sheet
```

This example demonstrates how to define sheets in a YAML format, and how to use the `sde.py` script to convert YAML files to SDE sheet files and to execute SDE sheet files.


## Examples

```
prompt.set({"positive":"Generate a cityscape"})
generator.model = "art_generator.safetensors"
generator.concepts = [
    Concept(
      "path":"my/lora.safetensors"
      "strength":1
    ),
    Concept(
      "path":"my/lyco.safetensors"
      "strength":1
    )
  ]
image.width = 800;
image.height = 600;
```
