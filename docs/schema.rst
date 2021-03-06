Schemas and validation
======================

Form schemas are high-level declarative descriptions of forms. They describe
every field of the form, how it should rendered, what's the shape of the data
and how it should be validated.

React Forms provides ``Mapping``, ``List`` and ``Scalar`` schema nodes to
describe mapping, list and scalar values correspondingly::

  var Mapping = ReactForms.schema.Mapping
  var List    = ReactForms.schema.List
  var Scalar  = ReactForms.schema.Scalar

  var PersonSchema = Mapping(
    Scalar({name: 'name', label: 'Name'}),
    Scalar({name: 'dob', label: 'Date Of Birth'}))

Object ``PersonSchema`` is a schema for objects which represent persons, every
person have ``name`` and ``dob`` properties with labels ``Name`` and ``Date Of
Birth`` correspondingly. Attribute ``name`` is required for schema nodes which
are defined as a part of ``Mapping`` declarations.

Forms schema API is designed to be compatible with JSX. The schema above can be
specified using JSX syntax::

  var PersonSchema = Mapping({
    name: Scalar({label: 'Name'}),
    dob: Scalar({label: 'Date Of Birth'})
  })

Reusable schemas
----------------

Schemas are immutable values and can be reused as parts of more sophisticated
schemas as much as needed.

Also it is possible to define parametrized schemas as functions which construct
schema nodes based on arguments passed::

  function Name(props) {
    props = props || {}
    return Scalar({label: 'Name'})
  }

Schema metadata
---------------

There are a couple of schema metadata (alongside ``name``) supported by form
components out of the box.

Schema metadata related to validation:

  * ``type`` property can be used to specify type of the schema which defines how
    object is serialized to/deserialize from DOM value. You can read more about
    schema types below.
  * ``validate`` property is used to specify validators for
    values corresponding to schema.
  * ``defaultValue`` is used to define a value which will be used when a
    corresponding value for schema node is absent
  * ``onUpdate`` is a callback which is fires during an update to form value, it
    can be used to rewrite parts of form value based on some criteria.

Schema metadata related to presentation:

  * ``label`` property is used by form components to render ``<label />``
    elements for form fields
  * ``hint`` property if specified

Example of schema declarations which define all the available metadata
properties::

  Scalar({
    name: 'age',
    type: 'number',
    defaultValue: 27,
    validate: function(v) { return v > 0 },
    label: 'Age',
    hint: 'How old are you?',
    onUpdate: function(value) {
      if (value < 0) {
        return -value;
      } else {
        return value;
      }
    }
  })

Types
-----

Scalars can specify type of the value they represent.

Type defines how value is serialized to/deserialized from its DOM
representation. For example if you work with dates you would want to define type
which would marshal strings in format ``"YYYY-MM-DD"`` into ``Date`` objects and
vice versa. Fortunately there's built-in ``date`` type for that::

  Mapping({
    ...
    birthday: Scalar({type: 'date'}),
    ...
  })

You can refer to built-in types by specifying a ``type`` property which has type
name as its string value. Currently React Forms provide a limited set of
built-in types: ``date``, ``number`` and ``string`` (used by default if no type
is specified). But you can create a custom one easily.

Schema types are simply objects with ``serialize`` and ``deserialize`` methods.
Method ``serialize`` is called before value is sent to input component and
``deserialize`` is called on change before validation occurs.

Method ``deserialize`` could also throw an exception in case it cannot
deserialize a passed value::

  var MyType = new ReactForms.type.Type({

    name: 'MyType',

    serialize: function(value) {
      // return a value which will be passed
      // to input component, probably a string
    },

    deserialize: function(value) {
      // return a value which will be passed
      // through validators and stored as a part
      // of the form value
    }
  })

  var schema = Scalar({type: MyType})

Validation
----------

Schema is used by form components to validate form value. Basic validation is
done by schema types. But to specify more sophisticated validation rules one can
attach custom validators to each schema node.

Validators are functions which can return a boolean value: ``true`` corresponds
to validation success and ``false`` to validation failure.

For example one can define a reusable schema node for positive numbers which
validates only if corresponding value is a number and is greater than zero::

  function PositiveNumber(props) {
    props = props || {}
    return Scalar({
      name: props.name,
      type: 'number',
      validate: function(v) { return v > 0; }
    })
  }
