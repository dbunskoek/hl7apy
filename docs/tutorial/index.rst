.. _tutorial:

Getting started
===============

The following tutorial shows you the main features of the library.

Introduction
------------

HL7apy implements classes for messages, groups, segments, fields, components and subcomponents as
defined by the HL7 v2 standard. The elements have a hierarchical relationship and the API gives you the interface
for adding, removing and visiting the tree nodes.

Create a message from scratch
-----------------------------

You can create a new message by instantiating the :class:`hl7apy.core.Message` class:

.. code-block:: python

  from hl7apy.core import Message

  m = Message("ADT_A01")
  m2 = Message()

You can both create a message specifying a structure (e.g. ADT_A01) or create a new message with no predefined structure.

Your new message can be populated as follows:

.. code-block:: python

  pid = Segment("PID")
  patient_group = Group("OML_O33_PATIENT")

  # add a Segment instance
  m.add(pid)

  # add a Group instance
  m2.add(patient_group)

  # create a Segment named MSA and add it to m2
  msa = m2.add_segment('MSA')

  # create a Group named ADT_A01_INSURANCE and add it to m
  g = m.add_group("ADT_A01_INSURANCE")

  # assign a Segment instance
  m.pid = pid

  # assign a string
  m.pid = "PID|1||566-554-3423^^^GHH^MR||EVERYMAN^ADAM^A|||M|||2222 HOME STREET^^ANN ARBOR^MI^^USA||555-555-2004~444-333-222|||M"
  # equivalent to
  m.pid.value = "PID|1||566-554-3423^^^GHH^MR||EVERYMAN^ADAM^A|||M|||2222 HOME STREET^^ANN ARBOR^MI^^USA||555-555-2004~444-333-222|||M"

  # copy from another_message child
  m.pid = m2.oml_o33_patient.pid

You can also populate your message without explicit creation of its children, as in the following example:

.. code-block:: python

  from hl7apy.core import Message

  m = Message("ADT_A01")
  m.pid.pid_5_1 = 'EVERYMAN'
  m.pid.pid_5_2 = 'ADAM'

The ``PID`` segment is created during child traversal, as well as their related fields and components.
The previous snippet of code is equivalent to:

.. code-block:: python

  from hl7apy.core import Message

  m = Message("ADT_A01")
  pid = Segment("PID")
  pid_5 = Field("PID_5")
  pid_5.pid_5_1 = 'EVERYMAN'
  pid_5.pid_5_2 = 'ADAM'
  pid.add(pid_5)
  m.add(pid)


ADT_A01 example
---------------

Suppose you want to create the following ADT_A01 message:

::

  MSH|^~\&|GHH_ADT||||20080115153000||ADT^A01^ADT_A01|0123456789|P|2.5||||AL
  EVN||20080115153000||AAA|AAA|20080114003000
  PID|1||566-554-3423^^^GHH^MR||EVERYMAN^ADAM^A|||M|||2222 HOME STREET^^ANN ARBOR^MI^^USA||555-555-2004~444-333-222|||M
  NK1|1|NUCLEAR^NELDA^W|SPO|2222 HOME STREET^^ANN ARBOR^MI^^USA

You can create it from scratch by using the core classes, or by using the :func:`hl7apy.parser.parse_message` function;
in the following snippet of code, we show you a way to create it from scratch:

.. code-block:: python

  from hl7apy.core import Message

  m = Message("ADT_A01", version="2.5")
  m.msh.msh_3 = 'GHH_ADT'
  m.msh.msh_7 = '20080115153000'
  m.msh.msh_9 = 'ADT^A01^ADT_A01'
  m.msh.msh_10 = "0123456789"
  m.msh.msh_11 = "P"
  m.msh.msh_16 = "AL"
  m.evn.evn_2 = m.msh.msh_7
  m.evn.evn_4 = "AAA"
  m.evn.evn_5 = m.evn.evn_4
  m.evn.evn_6 = '20080114003000'
  m.pid = "PID|1||566-554-3423^^^GHH^MR||EVERYMAN^ADAM^A|||M|||2222 HOME STREET^^ANN ARBOR^MI^^USA||555-555-2004~444-333-222|||M"
  m.nk1.nk1_1 = '1'
  m.nk1.nk1_2 = 'NUCLEAR^NELDA^W'
  m.nk1.nk1_3 = 'SPO'
  m.nk1.nk1_4 = '2222 HOME STREET^^ANN ARBOR^MI^^USA'

.. _parsing:

Parsing
-------

You can use the provided ER7 parsers to parse a message string:

.. code-block:: python

  from hl7apy.parser import parse_message

  msh = "MSH|^~\&|GHH_ADT||||20080115153000||ADT^A01^ADT_A01|0123456789|P|2.5||||AL\r"
  evn = "EVN||20080115153000||AAA|AAA|20080114003000\r"
  pid = "PID|1||566-554-3423^^^GHH^MR||EVERYMAN^ADAM^A|||M|||2222 HOME STREET^^ANN ARBOR^MI^^USA||555-555-2004~444-333-222|||M\r"
  nk1 = "NK1|1|NUCLEAR^NELDA^W|SPO|2222 HOME STREET^^ANN ARBOR^MI^^USA\r"
  pv1 = "PV1|1|I|GHH PATIENT WARD|U||||^SENDER^SAM^^MD|^PUMP^PATRICK^P|CAR||||2|A0|||||||||||||||||||||||||||||2008\r"
  in1 = "IN1|1|HCID-GL^GLOBAL|HCID-23432|HC PAYOR, INC.|5555 INSURERS CIRCLE^^ANN ARBOR^MI^99999^USA||||||||||||||||||||||||||||||||||||||||||||444-33-3333"

  s = msh + evn + pid + nk1 + pv1 + in1
  message = parse_message(s)

By default, :func:`hl7apy.parser.parse_message` assigns the segments found to the relevant HL7 group.
You can disable this behaviour by passing ``find_groups=False`` to the function. In this case, the segments found are assigned as direct children of the :class:`hl7apy.core.Message` instance.

ER7 parsers for segments, fields and components are also provided:

.. code-block:: python

  from hl7apy.parser import parse_segment, parse_field, parse_component

  pid = "PID|1||566-554-3423^^^GHH^MR||EVERYMAN^ADAM^A|||M|||2222 HOME STREET^^ANN ARBOR^MI^^USA||555-555-2004~444-333-222|||M\r"
  segment = parse_segment(pid)
  field = parse_field("EVERYMAN^ADAM^A") # it will return an instance of Field()
  component = parse_component("ID&TEST&TEST2") # it will return an instance of Component()

Each parser will return an instance of the corresponding core class (e.g. :func:`hl7apy.parser.parse_field` will return a :class:`hl7apy.core.Field` instance).

You can pass the ``name`` argument to both :func:`hl7apy.parser.parse_field` and :func:`hl7apy.parser.parse_component`
functions to assign the name of the corresponding :class:`hl7apy.core.Field` and :class:`hl7apy.core.Component` instances returned by the functions, since it is
not possible to infer their names by simply parsing the input strings:

.. code-block:: python

  from hl7apy.parser import parse_field, parse_component

  field = parse_field("EVERYMAN^ADAM^A", name="PID_5") # it will return an instance of Field("PID_5")
  component = parse_component("AUTH&1.3.6.1.4.1.21367.2011.2.5.17&ISO", name="CX_4") # it will return an instance of Component("CX_4")

ER7 encoding
------------

You can get the ER7-encoded string of ``Message``, ``Group``, ``Segment``, ``Field``, ``Component`` instances by simply calling the :meth:`hl7apy.Element.to_er7` method:

.. code-block:: python

  from hl7apy.parser import parse_segment

  pid = "PID|1||566-554-3423^^^GHH^MR||EVERYMAN^ADAM^A|||M|||2222 HOME STREET^^ANN ARBOR^MI^^USA||555-555-2004~444-333-222|||M\r"
  segment = parse_segment(pid)
  print segment.to_er7()

You can also use custom encoding chars:

.. code-block:: python

  from hl7apy.parser import parse_segment

  custom_chars = {'FIELD': '!', 'COMPONENT': '@', 'SUBCOMPONENT': '%', 'REPETITION': '~', 'ESCAPE': '$'}
  pid = "PID|1||566-554-3423^^^GHH^MR||EVERYMAN^ADAM^A|||M|||2222 HOME STREET^^ANN ARBOR^MI^^USA||555-555-2004~444-333-222|||M\r"
  segment = parse_segment(pid)
  print segment.to_er7(encoding_chars=custom_chars)

Datatypes
---------

Library supports both base and complex datatypes according to standard specifications.
Elements that can have a datatype are Field, Component and SubComponent, the latter supports only base datatypes.
Components and SubComponents name are defined as follows:

  * If the name is specified it must be <complex_datatype>_<position>
  * If the name is not specified it is the name of the datatype

.. code-block:: python

  f = Field('PID_1')
  f.datatype # it prints 'SI'
  f = Field('PID_3')
  f.datatype # it prints 'CX'
  c = Component('CX_10') # the component is part of a complex datatype (CX)
  s = SubComponent('CWE_1') # the subcomponent is part of a complex datatype (CWE)
  c = Component(datatype='CWE') # the name is 'CWE'
  s = SubComponent(datatype='ST') # the name is 'ST'

The library implements base datatypes classes and validation of their values

.. code-block:: python

  from hl7apy.v2_4 import get_base_datatypes #...the list of datatypes depends on the version
  ST = get_base_datatypes()['ST']
  NM = get_base_datatypes()['NM']
  DTM = get_base_datatypes()['DTM']
  s = ST('some information')
  s = ST(1000*'a') # it raises an exceptions since the given value exceeds the max length for an ST datatype
  n = NM(111)
  n = NM(11111) # it raises an exceptions since the given value exceeds the max length for a NM datatype
  d = DTM('20131010')
  d = DTM('10102013') # it raises an exceptions since the given value is not a valid DTM value

In the case of SubComponent the :attr:`value` can also be an instance of a base datatype

.. code-block:: python

  s = SubComponent(datatype="FT")
  s.value = FT('some information')

Elements manipulation
---------------------

You can visit an element's children in different ways:

    * by name
    * by long name (as defined in HL7 official structures)
    * by position

.. code-block:: python

    s = Segment('PID')
    s.pid_5 # by name
    s.patient_name # by long name
    s.pid_5.pid_5_1 # by position

Please note that child traversal is case insensitive (e.g. s.PATIENT_NAME is the same as s.patient_name)

By default the returned child is always the first, because usually an element have only one instance for a child.
If you want to access to another child you have to specify the index

.. code-block:: python

    s.pid_13 # it is the same as s.pid_13[0]
    s.pid_13[1] # it returns the second instance of pid_13 (if it exists)

If you want to access to a Field's children you can also use the following syntax:

.. code-block:: python

    org_5 = Field('org_5') # the datatype is CX
    org_5.org_5_10 # it returns the tenth component of the field. It is the same as org_5.cx_10
    org_5.org_5_10_3 # it returns the third subcomponent of the tenth component of the field. It is the same as org_5.cx_10.cwe_3

    org_4 = Field('ORG_4') # the datatype is ID
    org_4.org_4_1_1 # it raises an exception since org_4_1 is a base_datatype and doesn't have a subcomponent

If you want to iterate over an element's children

.. code-block:: python

    m = Message()
    for child in m.children:
        # do something useful with child

You can also iterate over all the repetitions of a given child

.. code-block:: python

    m = Message('OML_O33')
    for spm in m.spm: # in this case returns all the children named spm, not just the first one
        # do something useful with spm

You can delete a child from an elements


.. code-block:: python

    m = Message('OML_O33')
    del m.MSA # it deletes the first msa
    del m.spm[1].spm_1 # it deletes the spm_1 field of the second spm segment

During children traversal if you try to access to an element which has not been created yet, it returns an empty list (if the child is valid)

.. code-block:: python

    f = Field('PID_3')
    f.cx_10 # it returns []
    f.cx_30 # it raise an exception since cx_30 does not exist
    f.cx_10 = Component('CX_10')
    f.cx_10 # it returns [<Component CX_10>]



Validation
----------

The library supports 2 levels of validation: ``STRICT`` and ``QUIET``.

In ``STRICT`` mode, the elements should completely adhere to the structures defined by HL7. In particular, the library checks:
    * children name (e.g. a segment is not a valid child of a message according to the message's structure)
    * children cardinality (e.g. a segment is mandatory and it is missing in the message)
    * datatype constraints (e.g. a field of datatype ST exceeds 200 chars)

Moreover, when using ``STRICT`` validation it is not possible to instantiate an unknown element - instantiating a ``Message``,
``Group``, ``Field``, ``Component`` with ``name=None`` is not allowed.

The following examples will raise an exception in case of ``STRICT`` validation:

.. code-block:: python

  from hl7apy.core import Message
  from hl7apy.consts import VALIDATION_LEVEL

  m = Message("ADT_A01", validation_level=VALIDATION_LEVEL.STRICT) # note that the MSH segment is automatically created when instantiating a Message
  m.add_segment('MSH') # a Message cannot have more than 1 MSH segment
  #Traceback (most recent call last):
  #...
  #MaxChildLimitReached: Cannot add <Segment MSH>: max limit (1) reached for <Message ADT_A01>

  m.msh.pid_1 = Field('PID_1')
  #Traceback (most recent call last):
  #...
  #ChildNotValid: <Field PID_1 (SET_ID_PID) of type SI> is not a valid child for <Segment MSH>

  m.msh.msh_7 = 'abcde' # its value should be a valid DTM value (e.g. 20130101)
  #Traceback (most recent call last):
  #...
  #ValueError: abcde is not an HL7 valid date value

In ``QUIET`` mode, the library does not perform the checks listed above, but you can still verify if an element created with ``QUIET``
validation is compliant to the standard by calling the :func:`hl7apy.core.Element.validate` method:

.. code-block:: python

  from hl7apy.core import Message

  m = Message("ADT_A01")
  m.validate()

