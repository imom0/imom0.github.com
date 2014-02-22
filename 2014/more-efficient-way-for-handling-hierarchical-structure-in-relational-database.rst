=============================================================================
More Efficient Way for Handling Hierarchical Structure in Relational Database
=============================================================================

.. sectionauthor:: momo <mobeiheart@gmail.com>

.. feed-entry::
   :date: 2014-02-23


When developing an employee management system, a common and typical problem is finding all subordinates of somebody, direct and indirect.It can be done using recursive way::

    def expand_subordinates(employee):
        direct_subordinates = Employee.objects.filter(superior=employee)
        indirect_subordinates = []
        for subordinate in direct_subordinates:
            direct, indirect = expand_subordinates(subordinate)
            indirect_subordinates.extend(direct)
            indirect_subordinates.extend(indirect)
        return direct_subordinates, indirect_subordinates

It must be slow in relational database, the higher level the employee is in, the slower the query is.

But there are efficient way of handling this hierarchical data structure, the well-known `Django mptt <https://github.com/django-mptt/django-mptt>`_ and others like `sqlamp <http://sqlamp.angri.ru/>`_  or `sqlalchemy nested set example <https://bitbucket.org/zzzeek/sqlalchemy/src/ceaa6047ef8bc3916ffdda1924844cbf233dfd94/examples/nested_sets/nested_sets.py>`_.They add additional two fields to implement modified preorder tree traversal.
