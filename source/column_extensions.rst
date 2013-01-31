==========================
Types de colonnes avancées
==========================

Colonne de type Json
====================

| Très souvent, j'aime avoir des champs de type Json dans mes entités.
| Par exemple, pour des champs de type ``metadata`` ou ``extra`` comme dans l'exemple suivant :

::

    import sqlalchemy as sa
    from skalchemy import JSONEncodedDict, MutationDict

    class Foobar(Base):
        
        id = sa.Column(sa.Integer, primary_key=True)
        ...
        extra = Column(MutationDict.as_mutable(JSONEncodedDict))

Cette déclaration du champ ``extra`` me permet d'enregistrer des objets complexes (liste, dictionnaire…) dans
un champ de type texte, sans me soucier de l'encodage et du décodage Json :

::

    >>> obj = Foobar()
    >>> obj.id
    1

    >>> obj.extra = {'foo': 1, 'bar': [1, 2, 3]}
    >>> DBSession.add(obj)
    >>> DBSession.commit()
    >>> obj = None

    >>> obj = DBSession.query(Foobar).filter(Foobar.id==1).first()
    >>> print(obj.extra)
    {'foo': 1, 'bar': [1, 2, 3]}



Voici le détail des deux classes JSONEncodedDict, MutationDict

.. code-block:: python

    from datetime import datetime, date

    import jsonpublish
    import json
    import sqalchemy as sa

    @jsonpublish.register_adapter(date)
    def adapt_date(d):
        return d.strftime('%Y-%m-%d')


    @jsonpublish.register_adapter(datetime)
    def adapt_datetime(d):
        return d.strftime('%Y-%m-%d %H:%M:%S')

    class JSONEncodedDict(sa.types.TypeDecorator):
        "Represents an immutable structure as a json-encoded string."

        impl = sa.String

        def process_bind_param(self, value, dialect):
            if value is not None:
                value = jsonpublish.dumps(value)
            return value

        def process_result_value(self, value, dialect):
            if value is not None:
                value = json.loads(value)
            return value


    class MutationDict(sa.ext.mutable.Mutable, dict):
        @classmethod
        def coerce(cls, key, value):
            "Convert plain dictionaries to MutationDict."

            if not isinstance(value, MutationDict):
                if isinstance(value, dict):
                    return MutationDict(value)

                # this call will raise ValueError
                return sa.ext.mutable.Mutable.coerce(key, value)
            else:
                return value

        def __setitem__(self, key, value):
            "Detect dictionary set events and emit change events."

            dict.__setitem__(self, key, value)
            self.changed()

        def __delitem__(self, key):
            "Detect dictionary del events and emit change events."

            dict.__delitem__(self, key)
            self.changed()
                if value is not None:
                    value = jsonpublish.dumps(value)
                return value

            def process_result_value(self, value, dialect):
                if value is not None:
                    value = json.loads(value)
                return value
