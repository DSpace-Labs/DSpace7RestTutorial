# Tips for using Postman
{% include navmenu.html %}
This is beyond the scope of the workshop.  The following notes may be helpful in configuring your environment. An export of preconfigured envs to work with the official demo and a collection of useful requests, more than the ones used in the tutorial, are available for download
- [Env Official DSpace 7 Demo - Anonymous user](postman-config/Anonymous.postman_environment.json)
- [Env Official DSpace 7 Demo - Administrator user](postman-config/Administrator.postman_environment.json)
- [Env Official DSpace 7 Demo - Submitter user](postman-config/Submitter.postman_environment.json)
- [Request collection](postman-config/DSpace7Tutorial.postman_collection.json)

## Create a Collection in Postman
- Click "Edit" on the Collection

### Set the following Pre-request Script
Customize the server address, username, and password for your site.
```
const loginRequest = {
  url: pm.environment.get('dspaceresturl') + "/api/authn/login",
  method: 'POST',
  header: 'Content-Type: application/x-www-form-urlencoded',
  body: "user="+pm.environment.get('dspaceuser')+"&password="+pm.environment.get('dspacepwd')
};
pm.sendRequest(loginRequest, function (err, response) {
    var s = response.headers.get("Authorization");
    if (s == null) {
        s = "";
    } else {
        s = s.replace(/Bearer /,"");
    }
    pm.environment.set("auth_token", s);
});
```

### Set the following Environment Variable
Create one or more environments with the following variables

- dspaceresturl: http://localhost:8080/spring-rest
- dspaceuser: dspacedemo%2Badmin@gmail.com
- dspacepwd: dspace

Please note that the dspaceuser and dspacepwd must be URLEncoded (i.e. + become %2B). To setup an env for anonymous user simple keep the dspaceuser and dspacepwd empty

### Set the following Authorization Value

- Type: `Bearer Token`
- Token: `{% raw %}{{auth_token}}{% endraw %}`

## In your requests

Set Authorization to "Inherit from Parent"
{% include nav.html %}
