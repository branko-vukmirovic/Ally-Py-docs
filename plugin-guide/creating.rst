.. _Creating:

Creating a plugin
=================

In this guide we will create a simple plugin called ``sample_plugin`` with the structure described in :ref:`Structure`.

Describing the API
------------------

The first example is based on a model of a user, with a name and an id number.  Create the module ``user`` in ``sample_plugin.api.user.py`` containing:

.. code-block:: python

    class User:
        '''
        The user model.
        '''
        Id = int
        Name = str


Add the Ally.py egg to the python path.               

add to python path
        ``distribution/components/ally-api.1.0.egg``

To make the class recognizable by the Ally.py framework, edit the previous code to import the Ally.py model, specify the attribute to use as a unique id by passing it as a keyed argument to the model decorator (this is required for all Ally.py models) and specify a domain (similar to a namespace) ``sample_plugin.api.user.py``.

.. code-block:: python

   from ally.api.config import model

   @model(id='Id', domain='Sample')
   class User:
        '''
        The user model.
        '''
        Id = int
        Name = str

Without a domain, the model is accessible at `User <http://localhost/resources/User>`_ , but with domain set to 'Sample' the domain is accessible at `Sample/User <http://localhost/resources/Sample/User>`_.

To reuse a domain definition in various modules or plugins, define a domain model decorator in the ``sample_plugin.api.__init__.py`` module:

.. code-block:: python

   from functools import partial
   from ally.api.config import model
   
   # --------------------------------------------------------------------

   modelSample = partial(model, domain='Sample')

And edit the decorated user model in ``sample_plugin.api.user.py``:

.. code-block:: python

    from sample_plugin.api import modelSample

    # --------------------------------------------------------------------

    @modelSample(id='Id')
    class User:
        '''
        The user model.
        '''
        Id = int
        Name = str
        

To complement the model, we need to add a service to deliver instances of the model as a REST response in ``sample_plugin.api.user.py``: 

.. code-block:: python

    from ally.api.config import service, call
    from ally.api.type import Iter
    from sample_plugin.api import modelSample

    # --------------------------------------------------------------------

    @modelSample(id='Id')
    class User:
        '''
        The user mode
        '''
        Id = int
        Name = str

    # --------------------------------------------------------------------

    @service
    class IUserService:
        '''
        The user service.
        '''
        @call
        def getUsers(self) -> Iter(User):
            '''
            Provides all the users.
            '''

All service interface names start with a capital 'I' followed by the service name and end in 'Service' and are decorated with ``@service``. This convention simplifies inversion of control and aspect oriented programming configuration. Each method that exposes a response model is decorated with ``@call`` and annotated with the return type. In the previous example, the return type is an iterable collection of User models.  The Ally.py framework maps annotated return and input types to a path invoking the corresponding service method. All service methods, even those not exposed using the ``@call`` decorator must have input and return types annotated. 

Implementing the API
----------------------------------------------

After defining the API, create an implementation based upon it, returning a list with one user model at the address `Sample/User <http://localhost/resources/Sample/User>`_:

.. code-block:: xml

    <UserList>
        <User>
            <Name>User 1</Name>
            <Id>1</Id>
        </User>
    </UserList>

Edit the user implementation in ``sample_plugin.impl.user.py`` :

.. code-block:: python 

    from sample_plugin.api.user import IUserService
    from ally.container.ioc import injected
    from ally.container.support import setup

    # --------------------------------------------------------------------

    @injected
    @setup(IUserService, name='userService')
    class UserService(IUserService):
        '''
        Implementation for @see: IUserService
        '''
        
        def getUsers(self):
    	'''
    	@see: IUserService.getUsers
    	'''
    	return []

Creating the configuration
--------------------------

After defining the API and the implementation of the API, use the dependency injection container to expose the API through the HTTP REST interface of the Ally.py framework. Create the module ``service`` in package ``__plugin__.sample_plugin.service.py`` 

The setup function decorated with ``@ioc.entity`` delivers the implementation instance of ``UserService`` for the ``IUserService`` api.  Because this function is decorated with the ``ioc.entity`` decorator it will be used as a entity source by the dependency injection container. 

The ``register`` method registers the user service implementation instance which is obtained by invoking the dependency injection function ``userService``.

``__plugin__.sample_plugin.service.py``:

.. code-block:: python

    from __plugin__.plugin.registry import registerService
    from ally.container import ioc
    from sample_plugin.api.user import IUserService
    from sample_plugin.impl.user import UserService

    # --------------------------------------------------------------------

    @ioc.entity
    def userService() -> IUserService:
        b = UserService()
        return b

    @ioc.start
    def register():
        registerService(userService())


Packaging and Deploying
-----------------------

The plugin is now fully functional. Package it for deployment to the application using ``build-plugin.ant``.  The ant script creates ``02 - plugin sample.1.0.egg`` in the plugin ``source`` folder. To deploy the plugin, copy the file into the ``distribution/plugin/`` folder of your application. If the application is running on localhost port 80, access your plugin at http://localhost/resources and verify that the resource SampleUser exists:

.. code-block:: xml

    <Resources>
        <SampleUser href="http://localhost/resources/Sample/User/" />
        ...
    </Resources>

Querying and Filtering
------------------------

To filter the list of users use ``@query`` as shown in ``sample_plugin.api.user.py``:

.. code-block:: python

    from ally.api.config import service, call, query
    from ally.api.criteria import AsLikeOrdered
    from ally.api.type import Iter
    from sample_plugin.api import modelSample

    # --------------------------------------------------------------------

    @modelSample(id='Id')
    class User:
        '''
        The user model.
        '''
        Id = int
        Name = str

    # --------------------------------------------------------------------

    @query(User)
    class QUser:
        '''
        The user model query object.
        '''
        name = AsLikeOrdered

    @service
    class IUserService:
        '''
        The user service.
        '''
        
        @call
        def getUsers(self, q:QUser=None) -> Iter(User):
       	    '''
    	    Provides all the users.
    	    '''

Query objects are like models that contains data used for filtering. Queries have the name of the model and are prefixed with 'Q', and attributes are lower case to avoid confusion with the model attributes. Query attribute values are the criteria class of the model attribute. ``AsLike`` enables filtering and ordering on an attribute.

The ``getUsers`` method now takes an query object instance as an optional parameter (with a default value of None). It is good practice to specify a default value for all query objects used in service methods, as queries are optional. In ``sample_plugin.impl.user.py``:

.. code-block:: python 

    from sample_plugin.api.user import IUserService, User, QUser
    from ally.support.api.util_service import likeAsRegex
    from ally.container.ioc import injected
    from ally.container.support import setup

    # --------------------------------------------------------------------

    @injected
    @setup(IUserService, name='userService')
    class UserService(IUserService):
        '''
        Implementation for @see: IUserService
        '''
        
        def getUsers(self, q=None):
    	'''
    	@see: IUserService.getUsers
    	'''
    	users = []
    	for k in range(1, 10):
    	    user = User()
    	    user.Id = k
    	    user.Name = 'User %s' % k
    	    users.append(user)
    	    
    	if q:
    	    assert isinstance(q, QUser)
    	    if QUser.name.like in q:
    		nameRegex = likeAsRegex(q.name.like)
    		users = [user for user in users if nameRegex.match(user.Name)]
    	    if QUser.name.ascending in q:
    		users.sort(key=lambda user: user.Name, reverse=not q.name.ascending)
    	    
    	return users

``getUsers`` now returns 10 users, and checks if query object exists. If a query object exists and has a specified like value in the name criteria, we generate a regular expression and filter the user list accordingly. If the ascending flag exists, we sort the user list in ascending order. 

Redeploy the plugin then view all ten users at `/Sample/User <http://localhost/resources/Sample/User>`_. View only the seventh user at `/Sample/User?name=%7 <http://localhost/resources/Sample/User?name=%7>`_ and sort the user list at `/Sample/User?asc=name <http://localhost/resources/Sample/User?asc=name>`_.

Download the `example egg <https://github.com/sourcefabric/Ally-Py-docs/blob/master/plugin-guide/source_code/02_-_query_plugin_sample/sample_plugin-1.0.dev-py3.2.egg>`_
