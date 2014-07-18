# Dry Dock
Dry Dock is a full-production network application integration test system, inspired by Twilio's [Shadow](https://github.com/twilio/shadow) by @kelvl. It allows you to specify two versions of a docker container running an application you would like to test, one stable version being run in production and one version you would like to test before deploying. It then allows you to test how the new version behaves in relative to the stable version and detect any errors or anomalies using actual production traffic, safely. It can facillitate any kind of IP traffic, but is better at automatically analyzing HTTP traffic (such as HTTP responses).

# Design
Dry Dock runs both versions in containers and intercepts all incoming and outgoing network traffic for these containers by simulating the network topology in it's own container (which runs both containers). It then routes the incoming requests to both containers and compares the outputs. The key is that only the stable version's output will be forwarded to its actual destination, meaning as far as any clients or backend services are concerned, only the stable version is running.

This transparent-proxy design allows anyone to test any application that can be containerized (with Docker) without modifying the application code or the application even being aware. It also offers a few advantages over Shadow, mainly that it not only observes HTTP responses, but it observes backend service interactions (and intercepts them). This means any operation can be run against Dry Dock without risking production data, including destructive or non-idempotent database operations.

Dry Dock is designed to be the ultimate way to truly do a full integration test of an application deployment candidate against actual production traffic and be entirely confident of its stability in a completely automated fashion that cannot affect end-users.

# Usage
Dry Dock is packaged as a Docker container for easy use and deployment. If I wanted to test two (Docker containerized) versions of a web application against each other, `carlosdp/web:stable` and `carlosdp/web:latest`, I would run this on a machine in my cluster with Docker running:

```bash
docker run --privileged=true -p 7070:7070 carlosdp/dry-dock -stable carlosdp/web:stable -test carlosdp/web:latest -p 80:80
```

**Note:** Dry Dock *must* be run with `--privileged=true`. This is because it needs to manipulate the network stack in order to perform the network interception and re-routing.
