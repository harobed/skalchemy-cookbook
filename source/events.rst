====================
Extension événements
====================

::

    from sqlalchemy.orm.interfaces import MapperExtension, SessionExtension
    from sqlalchemy.orm.collections import InstrumentedList

    class EventsOnInstance(MapperExtension):
        def before_insert(self, mapper, connection, instance): 
            if hasattr(instance, '_before_insert'):
                instance._before_insert()

        def before_update(self, mapper, connection, instance): 
            if hasattr(instance, '_before_update'): 
                instance._before_update()

        def before_delete(self, mapper, connection, instance): 
            if hasattr(instance, '_before_delete'): 
                instance._before_delete()

        def after_insert(self, mapper, connection, instance): 
            if hasattr(instance, '_after_insert'):
                instance._after_insert()

        def after_update(self, mapper, connection, instance): 
            if hasattr(instance, '_after_update'): 
                instance._after_update()

        def after_delete(self, mapper, connection, instance): 
            if hasattr(instance, '_after_delete'): 
                instance._after_delete()

    class SessionEvent(SessionExtension):
        def before_flush(self, session, flush_context, instances):
            for obj in session.new:
                if hasattr(obj, '_new_before_flush'):
                    obj._new_before_flush(session, flush_context)

        def after_flush(self, session, flush_context):
            for obj in session.new:
                if hasattr(obj, '_new_after_flush'):
                    obj._new_after_flush(session, flush_context)

            for obj in session.dirty:
                if hasattr(obj, '_dirty_after_flush'):
                    obj._dirty_after_flush(session, flush_context)

            for obj in session.deleted:
                if hasattr(obj, '_deleted_after_flush'):
                    obj._deleted_after_flush(session, flush_context)
