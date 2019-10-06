# Networkd CAD Standard Stencils

This repository contains the collection of standard stencils distributed with
the Network CAD project.  The term *standard* will be used for the following purpose:

   - vendor specific stencils so that there is a single place for these
   - demonstration stencils so that there is a common place for these

The purpose of stencils is to create reusable and inheritable definitions that would be
common for network automation use-cases.  The structure and use of stencils will be discussed
further in this README.

Stencils are generally stored as text files using
[TOML](https://github.com/toml-lang/toml) format; but there is really no
requirement to use TOML since the stencil python library imports the definition
as a dictionary structure.

Stencils are used to define reusable properties of a device (or system) that
can be used to generate specific instances of devices.  An example of a vendor
specific stencil, for example, could be called `DCS-7050X3-S32`, and this
defines the device specific properties and characteristics.  An example of a
demo stencil could be `WAN-Transit` that in turn is based off the
DCS-7050X3-S32 stencil, and the Demo-Spine would be used to further refine and
augment properties so that the resulting stencil could then be used to create
specific device instances of Demo-Spine.  So unpacking the description above,
stencils generally start with a vendor specific mode, are then further refined
into application specific usages, and then finally used to create specific
devices instances.
""   

# Examples

Consider the following an Arista stencil:

````toml
[AristaEOS]
    vendor = 'arista'

    interfaces."Console"                    = {ptype="rj45", role='console',    ctype='console'}
    interfaces."Management1"                = {ptype="rj45", role='management', ctype='cat6'}
    interfaces."Loopback0"                  = {ptype='loopback', role='loopback'}


[DCS-7050X3-S32]

    # The 7050CX3-32S is a 1RU system with 32 100G QSFP ports offering
    # wire speed throughput of up to 6.4 Tbps. Each QSFP port supports
    # a choice of 5 speeds with flexible configuration between 100GbE,
    # 40GbE, 4x10GbE, 4x25GbE or 2x50GbE modes for up to 128 ports of
    # 10GbE and 25GbE or 64 ports of 50GbE. All ports can operate in
    # any supported mode without limitation, allowing easy migration
    # from lower speeds and the flexibility for leaf or spine
    # deployment.

    base = "AristaEOS"
    model = "DCS-7050X3-S32"
    rack_size = 1

    interfaces."Ethernet[1-32]/1"              = {ptype="qsfp"}
    interfaces."Ethernet[33-34]"               = {ptype="sfp"}
````   

And now consider an example of an application stencil that would use the DCS-7050X3-S22:

````toml
[WAN-Transit]
    base = "DCS-7050X3-S32"
    role = 'wan-transit'

    interfaces."Ethernet[21-22]/1"           = {role='transit-wan',     speed=100, optic='QSFP-100G-LR4',     ctype='smf'}
    interfaces."Ethernet[23-30]/1"           = {role='transit-spine',   speed=100, optic='QSFP-100G-CWDM4',   ctype='smf'}
````

Now for a short python code example that loads these stencils into a stencil catalog, which then provides
a registry by name of stencils with fully expanded data dictionaries.

````python
from imnetdb.stencil import Stencils
import toml

stencil_catalog = Stencils()

# load the stencil definitions from files

arista_stencils = toml.load(open('stencils/vendors/arista.toml'))
readme_stencils = toml.load(open('stencils/demos/readme.toml'))

# load the definitions in the stencil catalog instance

stencil_catalog.load(arista_stencils)
stencil_catalog.load(readme_stencils)

# the stencil catalog maintains a registry (dictionary) of all know stencils, we can list
# them out

print(stencil_catalog.registry)
````

Output:
````shell script
['AristaEOS-L2', 'AristaEOS', 'DCS-7050X3-48YC12', 'DCS-7050X3-S32', 'DCS-7020SR-32C2', '7280SR2K-48C6', '7280CR2A-30', 'DCS-7010T-48', 'DCS-7010T-48-R', '720XP-48ZC2', 'WAN-Transit']
````

And we can see the compete `WAN-Transit` stencil dictionary structure

````python
from pprint import pprint
pprint(stencil_catalog.register['WAN-Transit'])
````

Output:
```python
{'interfaces': InterfacesDict([('Console',
                                {'ctype': 'console',
                                 'ptype': 'rj45',
                                 'role': 'console'}),
                               ('Management1',
                                {'ctype': 'cat6',
                                 'ptype': 'rj45',
                                 'role': 'management'}),
                               ('Loopback0',
                                {'ptype': 'loopback', 'role': 'loopback'}),
                               ('Ethernet1/1', {'ptype': 'qsfp'}),
                               ('Ethernet2/1', {'ptype': 'qsfp'}),
                               ('Ethernet3/1', {'ptype': 'qsfp'}),
                               ('Ethernet4/1', {'ptype': 'qsfp'}),
                               ('Ethernet5/1', {'ptype': 'qsfp'}),
                               ('Ethernet6/1', {'ptype': 'qsfp'}),
                               ('Ethernet7/1', {'ptype': 'qsfp'}),
                               ('Ethernet8/1', {'ptype': 'qsfp'}),
                               ('Ethernet9/1', {'ptype': 'qsfp'}),
                               ('Ethernet10/1', {'ptype': 'qsfp'}),
                               ('Ethernet11/1', {'ptype': 'qsfp'}),
                               ('Ethernet12/1', {'ptype': 'qsfp'}),
                               ('Ethernet13/1', {'ptype': 'qsfp'}),
                               ('Ethernet14/1', {'ptype': 'qsfp'}),
                               ('Ethernet15/1', {'ptype': 'qsfp'}),
                               ('Ethernet16/1', {'ptype': 'qsfp'}),
                               ('Ethernet17/1', {'ptype': 'qsfp'}),
                               ('Ethernet18/1', {'ptype': 'qsfp'}),
                               ('Ethernet19/1', {'ptype': 'qsfp'}),
                               ('Ethernet20/1', {'ptype': 'qsfp'}),
                               ('Ethernet21/1',
                                {'ctype': 'smf',
                                 'optic': 'QSFP-100G-LR4',
                                 'ptype': 'qsfp',
                                 'role': 'transit-wan',
                                 'speed': 100}),
                               ('Ethernet22/1',
                                {'ctype': 'smf',
                                 'optic': 'QSFP-100G-LR4',
                                 'ptype': 'qsfp',
                                 'role': 'transit-wan',
                                 'speed': 100}),
                               ('Ethernet23/1',
                                {'ctype': 'smf',
                                 'optic': 'QSFP-100G-CWDM4',
                                 'ptype': 'qsfp',
                                 'role': 'transit-spine',
                                 'speed': 100}),
                               ('Ethernet24/1',
                                {'ctype': 'smf',
                                 'optic': 'QSFP-100G-CWDM4',
                                 'ptype': 'qsfp',
                                 'role': 'transit-spine',
                                 'speed': 100}),
                               ('Ethernet25/1',
                                {'ctype': 'smf',
                                 'optic': 'QSFP-100G-CWDM4',
                                 'ptype': 'qsfp',
                                 'role': 'transit-spine',
                                 'speed': 100}),
                               ('Ethernet26/1',
                                {'ctype': 'smf',
                                 'optic': 'QSFP-100G-CWDM4',
                                 'ptype': 'qsfp',
                                 'role': 'transit-spine',
                                 'speed': 100}),
                               ('Ethernet27/1',
                                {'ctype': 'smf',
                                 'optic': 'QSFP-100G-CWDM4',
                                 'ptype': 'qsfp',
                                 'role': 'transit-spine',
                                 'speed': 100}),
                               ('Ethernet28/1',
                                {'ctype': 'smf',
                                 'optic': 'QSFP-100G-CWDM4',
                                 'ptype': 'qsfp',
                                 'role': 'transit-spine',
                                 'speed': 100}),
                               ('Ethernet29/1',
                                {'ctype': 'smf',
                                 'optic': 'QSFP-100G-CWDM4',
                                 'ptype': 'qsfp',
                                 'role': 'transit-spine',
                                 'speed': 100}),
                               ('Ethernet30/1',
                                {'ctype': 'smf',
                                 'optic': 'QSFP-100G-CWDM4',
                                 'ptype': 'qsfp',
                                 'role': 'transit-spine',
                                 'speed': 100}),
                               ('Ethernet31/1', {'ptype': 'qsfp'}),
                               ('Ethernet32/1', {'ptype': 'qsfp'}),
                               ('Ethernet33', {'ptype': 'sfp'}),
                               ('Ethernet34', {'ptype': 'sfp'})]),
 'model': 'DCS-7050X3-S32',
 'name': 'WAN-Transit',
 'rack_size': 1,
 'role': 'wan-transit',
 'vendor': 'arista'}
```

# Further Reading

The stencils python module is part of the [IMNetDB](https://github.com/imnetdb/imnetdb) repository.  For more information, please refer to that
project documentation.