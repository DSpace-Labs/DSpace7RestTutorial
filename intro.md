
{% include nav.html %}

## Introduction to the DSpace 7 REST API

### What is a REST API?

- [What is REST](https://restfulapi.net/) - I just found this link, we should review and decide if it is a good overview

### History of the REST API and DSpace
- REST API in prior versions of DSpace
  - DSpace 4, read only - no access to non public objects
  - DSpace 5, password authentication only, some write operations
  - DSpace 6, password and shibboleth, some write operations
- [Tour of the old REST API](https://demo.dspace.org/rest/) on demo.dspace.org (DSpace 6)
- Limitations of the old REST API
  - Not full featured to support a DSpace front end 
    - No search/discovery
    - Incomplete authentication access
    - Limited objects available with read/write access
  - Limited configuration options conveying how a client should behave

### Goals of the new REST API in DSpace 7
- Go between for any and all UI functions
- permit plugin development
- allow the development of custom integrations/apps at adopting institutions


{% include nav.html %}

