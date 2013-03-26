==============
Liste de Mixin
==============

Voici la partie de la documentation de `SQLAlchemy qui traite des Mixin <http://docs.sqlalchemy.org/en/rel_0_8/orm/extensions/declarative.html#mixin-and-custom-base-classes>`_.


Mixin de Base
=============

Voici une classe Mixin que j'aime utiliser comme base dans mes projets :

::

    import sqalchemy as sa

    class BasicMixin(object):
        @declared_attr
        def id(cls):
            return sa.Column(sa.Integer, primary_key=True)

        def __init__(self, **kwargs):
            for k, v in kwargs.items():
                if hasattr(self, k):
                    setattr(self, k, v) 

        def _get_session(self):
            return Session.object_session(self) 

        def _query(self):
            return Session.object_session(self).query(self.__class__)

        @classmethod
        def query(cls):
            return meta.Session.query(cls)

        @classmethod
        def get_by_id(cls, id):
            return cls.query().filter(cls.id==id).first()


Mixin « UUID »
==============

::

    import sqalchemy as sa

    class UUIDMixin(object):
        @declared_attr
        def uuid(cls):
            return sa.Column(String(50), nullable=False)

        def __init__(self, **kwargs):
            self.uuid = kwargs.get('uuid', str(uuid.uuid4()))

        @classmethod
        def get_by_uuid(cls, uuid):
            return cls.query().filter(cls.uuid==uuid).first()


Mixin « Création, modification »
================================

::

    import sqalchemy as sa

    class CreatedModifiedMixin(object):
        @declared_attr
        def created_at(cls):
            return sa.Column(sa.DateTime(), default=datetime.datetime.now)

        @declared_attr
        def modified_at(cls):
            return sa.Column(sa.DateTime(), default=datetime.datetime.now, onupdate=datetime.datetime.now)
        
        @declared_attr
        def created_by_id(cls):
            return sa.Column(sa.Integer, sa.ForeignKey('users.id'), nullable=True)

        @declared_attr
        def created_by(cls):
            return sa.relationship(
                "User", 
                primaryjoin=cls.__name__ + '.created_by_id==User.id',
                remote_side='User.id',
                uselist=False,
                post_update=True
            )

        @declared_attr
        def updated_by_id(cls):
            return sa.Column(sa.Integer, sa.ForeignKey('users.id'), nullable=True)

        @declared_attr
        def updated_by(cls):
            return relationship(
                "User", 
                primaryjoin=cls.__name__ + '.updated_by_id==User.id',
                remote_side='User.id',
                uselist=False,
                post_update=True
            )
