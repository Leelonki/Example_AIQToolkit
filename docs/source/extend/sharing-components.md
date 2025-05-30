<!--
SPDX-FileCopyrightText: Copyright (c) 2024-2025, NVIDIA CORPORATION & AFFILIATES. All rights reserved.
SPDX-License-Identifier: Apache-2.0

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->

# Sharing NVIDIA Agent Intelligence Toolkit Components

Every AIQ toolkit component is packaged inside of an AIQ toolkit plugin and is designed to be sharable with the community of AIQ toolkit  developers. Functions are by far the most common AIQ toolkit component type. In fact, AIQ components include all pieces that leverage an AIQ toolkit registration decorator (e.g. `register_function`, `register_llm_client`, `register_evaluator`, etc.). This guide will discuss the requirements for developing registered components that can be shared, discovered, and integrated leveraged with any AIQ toolkit application.

## Enabling Local and Remote Discovery
To begin building a sharable component, do the following:
* Define a configuration object as described in [Customizing the Configuration Object](../workflows/workflow-configuration.md)
* Define a function as described in [Creating a New Tool and Workflow](../tutorials/create-a-new-workflow.md).

This section emphasizes the details of configuration objects that facilitate component discovery.

After installing the AIQ toolkit library, and potentially other AIQ toolkit plugin packages, a developer may want to know what
components are available for workflow development or evaluation. A great tool for this is the `aiq info components` CLI
utility described in [Components Information](../reference/cli.md#components-information). This command produces a
table containing information dynamically accumulated from each AIQ toolkit component. The `details` column is sourced from
each configuration object's docstring and field descriptions. Behind the scenes, these data (and others) are aggregated
into a component's `DiscoveryMetadata` to enable local and remote discovery. This object includes the following key
fields:

- `package`: The name of the package containing the AIQ toolkit component.
- `version`: The version number of the package containing the AIQ toolkit component.
- `component_type`: The type of AIQ toolkit component this metadata represents (e.g. `function`, `llm`, `embedder`, etc.)
- `component_name`: The registered name of the AIQ toolkit component to be used in the `_type` field when configuring a
workflow configuration object.
- `description`: Description of the AIQ toolkit component pulled from its config objects docstrings and field metadata.
- `developer_notes`: Other notes to a developers to aid in the use of the component.

For this feature to provide useful information, there are a few hygiene requirements placed on AIQ toolkit component configuration object implementations.

* Specify a name: This will be pulled into the `component_name` column and will be used in the `_type` field of a
workflow's configuration object.
* Include a Docstring: This information is pulled into the `description` column to describe the functionality of the
component.
* Annotate fields with [`pydantic.Field`](https://docs.pydantic.dev/2.9/api/fields/#pydantic.fields.Field): This
information is pulled into the `description` and provides developers with documentation on each configurable field,
including `dtype`, field description, and any default values.

The code sample below provides a notional registered function's configuration object that satisfies with these
requirements.

```python
from pydantic import Field

from aiq.data_models.function import FunctionBaseConfig

class MyFnConfig(FunctionBaseConfig, name="my_fn_name"):  # includes a name
    """The docstring should provide a description of the components utility."""  # includes a docstring

    a: str = Field(default="my_default_value", description="Notational description of what this field represents")  # includes a field description
```

By incorporating these elements, the `description` field in the `aiq info components` provides the following
information:

```bash
                                                                                        AIQ toolkit Search Results
┏━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ package                ┃ version                ┃ component_type ┃ component_name          ┃ description                                                                                        ┃
┡━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┩
│ aiq_notional_pkg_name  │ 0.1.1                  │ function       │ my_fn_name              │ The docstring should provide a description of the components utility.                              │
│                        │                        │                │                         │                                                                                                    │
│                        │                        │                │                         │   Args:                                                                                            │
│                        │                        │                │                         │     _type (str): The type of the object.                                                           │
│                        │                        │                │                         │     a (str): Notational description of what this field represents. Defaults to "my_default_value". │
└────────────────────────┴────────────────────────┴────────────────┴─────────────────────────┴────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

Without satisfying these requirements, a developer would need to inspect the each component's source code to identify
when it should be used and its configuration options. This significantly reduces developer velocity.

## Package Distribution

After completing AIQ toolkit development of component plugin, the next step is to create a package that will allow the
plugin to be installed and registered with the AIQ toolkit environment. Because each AIQ toolkit plugin package is a pip
installable package, this process it is straightforward, and follows standard Python `pyproject.toml` packaging steps.
If you are unfamiliar with this process, consider reviewing the [Python Packaging User Guide](https://packaging.python.org/en/latest/guides/writing-pyproject-toml/).

When building the `pyproject.toml` file, there are two critical sections:

* Dependencies: Ensure you include the necessary AIQ toolkit dependencies. An example is provided below:

    ```
    dependencies = [
    "aiq[langchain]",
    ]
    ```
* Entrypoints: Provide the path to your plugins so they are registered with AIQ toolkit when installed.
An example is provided below:
    ```
    [project.entry-points.'aiq.components']
    aiq_notional_pkg_name = "aiq_notional_pkg_name.register"
    ```

### Building a Wheel Package

After completing development and creating a `pyproject.toml` file that includes the necessary sections, the simplest
distribution path is to generate a Python wheel. This wheel can be distributed manually or published to a package repository such as [PyPI](https://pypi.org/).
The standard process for generating a Python wheel can be followed as outlined in the
[Packaging Python Projects] (https://packaging.python.org/en/latest/tutorials/packaging-projects/) guide.

While simple, this process does not take advantage of the `DiscoveryMetadata` to enable remote component discovery.

### Publish to a Remote Registry

Alternatively, AIQ toolkit provides an extensible interface that allows developers to publish packages and their
`DiscoveryMetadata`  arbitrary remote registries. The benefit of this approach comes from improved utilization of
captured `DiscoveryMetadata` to improve discovery of useful components.

By including this additional metadata, registry owners are empowered to extend their search interface and accelerate the
process of discovering useful components and development of AIQ toolkit based applications.

### Share Source Code

The last option for distribution is through source code. Since each AIQ toolkit package is a pip installable Python package,
each can be installed directly from source. Examples of this installation path are provided in the
[Get Started](../quick-start/installing.md) guide.

## Summary

There are several methods for component distribution, each of which depends on constructing a pip installable Python
packages that point to the hygienic implementations of component plugins. This lightweight, but extensible approach
provides a straightforward path for distributing AIQ toolkit agentic applications and their components to the developer
community.
