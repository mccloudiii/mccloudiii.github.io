---
layout: post
title:  "Simplifying Network Configs: A Jinja2 Guide"
date: 2020-04-22
categories: [Automation]
tags: [automation, Jinja ]
description: Simplifying Network Configs A Jinja2 Guide
image:
 path: /assets_2/jinja2.png
---

# Introduction  
Jinja2 is a Python-based templating language that is inspired by the Django templating system. It's widely used across various projects as a templating engine, with some well-known examples including Ansible, Salt, and Flask.  

The goal of Jinja2 is to integrate common Python functionalities directly into the templating engine, enabling the dynamic creation of static content.

# Installation  
Jinja2 is available on PyPi, which means it can be installed easily via pip. For managing Python projects, I recommend using virtual environments.  

```bash
# Create a virtual environment
python3.10 -m venv ~/envs/jinja2-env/

# Activate the virtual environment
source ~/envs/jinja2-env/bin/activate

# Install Jinja2
pip install jinja2
```

The versions used in this tutorial are:

- Jinja2: 2.9.6  
- Python: 3.6.1  

# Delimiters  
Jinja2 templates utilize specific delimiters to define operations executed by the templating engine:

- `{{ ... }}` for variables and expressions  
- `{% ... %}` for statements like `for`, `if`, and `include`  
- `{# ... #}` for comments  

# Variables  
In Jinja2, variables are represented with the `{{ some_variable }}` format.

```jinja
{{ some_variable }}
```

# Looping  
A `for` loop allows you to repeat blocks of text, reducing the need for manual copying and pasting.

```jinja
# Example
{% for item in collection %}
    {{ item }}
{% endfor %}
```
Note: `for` loops must always be properly closed with `{% endfor %}`.

# Conditionals  
Conditionals enable rendering content based on whether a specific condition evaluates to `True` or `False`.

```jinja
# Example
{% if item in collection %}
    {{ item }}
{% endif %}  # Ensure the if statement is properly closed

{% if item not in collection %}
    No item found here
{% endif %}
```
Like loops, `if` statements require an `{% endif %}` to close them.

# Usage  
Working with Jinja2 generally involves three main steps:

1. Define variables  
2. Create a template  
3. Render the template using the defined variables  

Let's explore with a basic example. Start by defining some interface variables in a Python dictionary.

```python
# Example for Cisco
interface = {
    'name': 'gigabitethernet0/0',
    'description': 'Uplink to WAN',
    'ip_address': '10.10.10.1 255.255.255.0',
}
```

Next, create a template where variables will be substituted with values from the `interface` dictionary.

```jinja
# Cisco interface template
interface_template = '''
interface {{ interface.name }}
 description {{ interface.description }}
 ip address {{ interface.ip_address }}
 no shutdown
!
'''
```

Note: The `.` syntax in Jinja2 is used to access dictionary keys. For example, `interface.name` accesses the `name` key in the `interface` dictionary. Alternatively, you can use Python-style dictionary access: `interface['name']`.  

Rendering the template with the `interface` data:

```python
from jinja2 import Template

template = Template(interface_template)
template.render(interface=interface)

# Output
interface gigabitethernet0/0
 description Uplink to WAN
 ip address 10.10.10.1 255.255.255.0
 no shutdown
!
```

This generates a configuration for a single interface. Let's now expand this to handle multiple interfaces using a `for` loop.

Define a list of dictionaries, each representing a different interface.

```python
interfaces = [
    {'description': 'Uplink to WAN',
     'ip_address': '10.10.10.1 255.255.255.0',
     'name': 'gigabitethernet0/0'},
    {'description': 'Crosslink to R2',
     'ip_address': '10.10.20.1 255.255.255.0',
     'name': 'gigabitethernet0/1'}
]
```

Next, modify the template to loop through these interfaces.

```jinja
# Cisco interface template with loop
interfaces_template = '''
{% for interface in interfaces %}
interface {{ interface.name }}
 description {{ interface.description }}
 ip address {{ interface.ip_address }}
 no shutdown
!
{% endfor %}
```

Rendering the template:

```python
template = Template(interfaces_template)
template.render(interfaces=interfaces)

# Output
interface gigabitethernet0/0
 description Uplink to WAN
 ip address 10.10.10.1 255.255.255.0
 no shutdown
!
interface gigabitethernet0/1
 description Crosslink to R2
 ip address 10.10.20.1 255.255.255.0
 no shutdown
!
```

Now, imagine you want to switch to a Juniper router. This demonstrates the true power of templating: easily adapting configurations to different devices.

```python
# Juniper interface template
interfaces_template = '''
interfaces {
{% for interface in interfaces %}
    {{ interface.name }} {
    description {{ interface.description }} ;
    unit 0 {
        family inet {
            address {{ interface.ip_address }} ;
            }
        }
    }
{% endfor %}
}
```

For this example, change the interface names and IP address formats.

```python
interfaces = [
    {'description': 'Uplink to WAN',
     'ip_address': '10.10.10.1/24',
     'name': 'ge-0/0/0'},
    {'description': 'Crosslink to R2',
     'ip_address': '10.10.20.1/24',
     'name': 'ge-0/0/1'}
]
```

Rendering the Juniper template:

```python
template = Template(interfaces_template)
template.render(interfaces=interfaces)

# Output
interfaces {
    ge-0/0/0 {
    description Uplink to WAN ;
    unit 0 {
        family inet {
            address 10.10.10.1/24 ;
            }
        }
    }
    ge-0/0/1 {
    description Crosslink to R2 ;
    unit 0 {
        family inet {
            address 10.10.20.1/24 ;
            }
        }
    }
}
```

# Conclusion  
Jinja2 is an excellent tool for creating reusable configuration templates. While this guide only scratches the surface, Jinja2 provides many other features that make it highly suitable for managing device configurations.

# Additional Example with Python and YAML Integration  

If you'd like to integrate Jinja2 with other Python-based tools for network automation, you can render the template with Python directly. This approach is similar to Ansible but uses Python to load the template and variable data.

Here's an example template in Jinja2:

```jinja
# template.j2
{% for interface in interfaces %}
interface {{ interface.name }}
 description {{ interface.description }}
 ip address {{ interface.p2p }}
 no switchport
 no shut
!
{% endfor %}
```

### Variables in YAML  
You can define your variables in a YAML file, as shown below:

```yaml
# vars.yaml
---
interfaces:
  - name: Eth01
    p2p: 10.10.10.1/30
    description: UPLINK_TO_CORE

  - name: Eth02
    p2p: 10.10.20.1/30
    description: TO_ACCESS

  - name: Eth03
    p2p: 10.10.30.1/30
    description: INTERNET
```

### Python Script to Render Template  
You can write a Python script to load the YAML file and render the template.

```python
# template_script.py
import yaml
from jinja2 import Environment, FileSystemLoader

env = Environment(loader=FileSystemLoader('.'),
                  trim_blocks=True,
                  lstrip_blocks=True)

template = env.get_template('template.j2')

with open('vars.yaml', 'r') as f:
    data = yaml.safe_load(f)

config = template.render(data)
with open('config.txt', 'w') as fw:
    fw.write(config)
```

This script loads the `vars.yaml` file, renders the `template.j2`, and writes the result to `config.txt`.
