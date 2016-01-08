+++
date = "2016-01-08T16:01:47-05:00"
title = "Ionic IO app name"

+++


A good question came up in the Ionic Slack the today. A dev was looking to change the name of an app that was already uploaded to the Ionic platform portal.

When you start an ionic project from the CLI, along with all the html/css/js that you need, it creats an `ionic.project` file. In here, you get your apps name and app_id
set, which gets used by the CLI


```json

{
  "name": "tmp",
  "app_id": "9cf2c890"
}


```

Now when I login to the Platform portal and upload my app, I'll see it appear on my dash.


 [![too-many-tmps](/img/too-many-tmps.png)](/img/too-many-tmps.png)


 As you can see, it's not an original name and could be quite confusing.


 So to change the name, I can just edit the name property in my `ionic.project` file.


 ```json

{
  "name": "newName",
  "app_id": "9cf2c890"
}

```

When I upload the app again, it will change the displayed name, but keep everything else the same.


 [![new-name](/img/new-name.png)](/img/new-name.png)

