# Tips for using Postman
This is beyond the scope of the workshop.  The following notes may be helpful in configuring your environment.

## Create a Collection in Postman
- Click "Edit" on the Collection

### Set the following Pre-request Script

```
const loginRequest = {
  url: 'http://localhost:8080/spring-rest/api/authn/login',
  method: 'POST',
  header: 'Content-Type: application/x-www-form-urlencoded',
  body: "user=test@test.edu&password=admin"
};
pm.sendRequest(loginRequest, function (err, response) {
    var s = response.headers.get("Authorization");
    if (s == null) {
        s = "";
    } else {
        s = s.replace(/Bearer /,"");
    }
    pm.environment.set("rest7bearer", s);
});
```

### Set the following Authorization Value

- Type: `Bearer Token`
- Token: `{% raw %}{{rest7bearer}}{% endraw %}`

## In your requests

Set Authorization to "Inherit from Parent"
