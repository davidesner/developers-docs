---
title: Architecture
permalink: /architecture/docker-bundle/
---

### Common Custom applications properties
Our Docker component takes care of some things, which means that the Custom application itself is simpler and generally more secure.
- authentication - The docker component makes sure that the application is run by authorized users/tokens, It is not possible to run an application anonymously. The application will does not have access to the KBC token itself, and only limited information about the project or end-user.
- starting and stopping the application - The docker component will boot a docker container which contains the application. This ensures that the applications runs in precisely defined environment which is guaranteed to be same for each application run (no application state is presrved)
- reading and writing data to KBC storage - The docker component ensures that the application will only receive the input mapping defined by end-user, and that it will write to the project only those outputs defined in output mappping by the end-user. This makes sure that a custom application cannot access arbitrary data in project.
- application isolation - each application is run in it's own docker container which is isolated from other containers, the application cannot be affected by other running applications. The application may also be limited to have no network access.