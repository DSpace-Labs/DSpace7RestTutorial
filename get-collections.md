## /api/core/communities
__If we like this approach, I would like to branch or tag master to keep line numbers stable__

### Find the controller for an object type
[RestResourceController @RequestMapping("/api/{apiCategory}/{model}")](https://github.com/DSpace/DSpace/blob/master/dspace-spring-rest/src/main/java/org/dspace/app/rest/RestResourceController.java#L84-L87")

### With no additional parameters, find all object instances
[RestResourceController.findAll() @RequestMapping(method = RequestMethod.GET)](https://github.com/DSpace/DSpace/blob/master/dspace-spring-rest/src/main/java/org/dspace/app/rest/RestResourceController.java#L769-L787")

- [RestResourceController.findAll() @RequestMapping(method = RequestMethod.GET)](https://github.com/DSpace/DSpace/blob/master/dspace-spring-rest/src/main/java/org/dspace/app/rest/RestResourceController.java#L788")
  - [CommunitiyRestRepository](https://github.com/DSpace/DSpace/blob/master/dspace-spring-rest/src/main/java/org/dspace/app/rest/repository/CommunityRestRepository.java#L37)
- [Call repository.findAll()](https://github.com/DSpace/DSpace/blob/master/dspace-spring-rest/src/main/java/org/dspace/app/rest/RestResourceController.java#L795")
  - [CommunitiyRestRepository.findAll()](https://github.com/DSpace/DSpace/blob/master/dspace-spring-rest/src/main/java/org/dspace/app/rest/repository/CommunityRestRepository.java#L63-L79)
  - note also the lamda call to repository.wrapResource
  - [CommunitiyRestRepository.wrapResource()](https://github.com/DSpace/DSpace/blob/master/dspace-spring-rest/src/main/java/org/dspace/app/rest/repository/CommunityRestRepository.java#L121-L124)
  
