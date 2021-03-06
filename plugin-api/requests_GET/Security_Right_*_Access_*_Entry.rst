.. _reuqest-GET-Security/Right/*/Access/*/Entry:

**Security/Right/*/Access/*/Entry**
==========================================================

* Use the HTTP **GET** method
* The request is defined by API call ``security.api.right.IRightService.getEntriesFiltered``

* The request will GET a collection of references to models :ref:`entity-acl.api.access.Entry`

::

   Provides the filtered entries position for the ACL object identifier and access id.
   
   @param identifier: object
       The ACL object identifier to provide the filtered entries for.
   @param accessId: integer
       The access id.
   @param options: key arguments
       The result iteration options.
   @return: Iterable(Entry.Position)
       An iterator containing the entries positions that are filtered.


URL parameters
-------------------------------------
Each entry matches a **\*** in their respective order.

* The unique identifier **Right.Id** from :ref:`entity-security.api.right.Right`.
* The unique identifier **Access.Id** from :ref:`entity-acl.api.access.Access`.


Query parameters
-------------------------------------
There are no query parameters available for this request except for global :ref:`headers_parameters`.
