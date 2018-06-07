## /api/core/communities
__If we like this approach, I would like to branch or tag master to keep line numbers stable__

### Find the controller for an object type
[RestResourceController @RequestMapping("/api/{apiCategory}/{model}")](https://github.com/DSpace/DSpace/blob/master/dspace-spring-rest/src/main/java/org/dspace/app/rest/RestResourceController.java#L84-L87")

### With no additional parameters, find all object instances
[RestResourceController.findAll() @RequestMapping(method = RequestMethod.GET)](https://github.com/DSpace/DSpace/blob/master/dspace-spring-rest/src/main/java/org/dspace/app/rest/RestResourceController.java#L769-L787")
