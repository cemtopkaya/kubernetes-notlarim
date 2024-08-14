# Container Networking with Links

## Introduction

Welcome aboard. It's all about networking in this activity. We've got to create a network for transporting secure information between the official `SpaceBones` website and the backend database. We're going to do it using legacy _Docker links_.

Let's get logged into the server, using the credentials provided in the lab page. Once we're in, make sure Docker is running:

```
docker ps
```

If we see the `ps` column headers, we're good to go. There won't be any data probably, because there aren't any containers running.

## Create the SpaceBones Container

We need a container to run the website itself, so lets spin one up with this:

```
docker run -d -p 80:80 --name spacebones spacebones/spacebones:thewebsite
```

Now, we we run `docker ps` again, we'll see that `spacebones` is running.

## Create the Treatlist Container

Now we're going to create the container that will hold the database, and make it link the `spacebones` and `postgres` (an image that was already sitting there in our home directory).

```
docker run -d -P --name treatlist --link spacebones:spacebones spacebones/postgres
```

If we run `docker ps` again, we'll see `treatlist` is running. But does that mean the connection is actually made? Who knows? We'd better check:

If not already running, create a new container from the `spacebones/postgres` image named `treatlist`.

## Verify That the Link Works

```
docker inspect -f "{{ .HostConfig.Links }}" treatlist
```
