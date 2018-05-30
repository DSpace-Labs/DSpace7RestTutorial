# Tips for using Postman
This is beyond the scope of the workshop.  The following notes may be helpful in configuring your environment.

## Create a Collection in Postman
- Click "Edit" on the Collection

### Set the following Pre-request Script

```
var pass = "admin";
pm.sendRequest("https://[your-server]/spring-rest/api/authn/login?user=user@test.edu&password="+pass, function (err, response) {
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
- Token: ```{{rest7bearer}}```

## In your requests

Set Authorization to "Inherit from Parent"
