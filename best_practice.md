# OpenShift migration best practice

Steps:

* Get Compatible Image
* get deployment config

This document summarizes best practices for migrating dockerised applications to
OpenShift. Most of the content is taken form a series of blog posts on
[puzzle.ch](https://www.puzzle.ch/de/blog/articles/2019/10/31/migration-einer-dockerisierten-anwendung-nach-openshift-teil-1-3)

Die *Security Context Constraints* kontrollieren die Berechtigungen für laufende
Container (Pods) im OpenShift. Dies soll verhindern, dass Container Zugriff auf
den Host und das Cluster erlangen. Per default wird Containern die restricted SCC zugewiesen. Das bedeutet unter anderem:

* kein root or privileged Container
* kein anbinden von host directory volumes
* Container User UIDs müssen in einer bestimmten range sein (Support Arbitrary User IDs)
* SELinux (Security-Enhanced Linux) ist aktiviert

[SCC Dokumentation](https://docs.openshift.com/container-platform/4.4/authentication/managing-security-context-constraints.html), [OpenShift Image Guidelines](https://docs.openshift.com/container-platform/4.4/openshift_images/create-images.html#images-create-guide-openshift_create-images)

Container Images, welche dies nicht erfüllen müssen neu gebaut werden ([Beispiel](https://www.puzzle.ch/de/blog/articles/2019/10/31/migration-einer-dockerisierten-anwendung-nach-openshift-teil-1-3)). Eine solche
Inkompatibilität würde spätestens beim Deployment erkannt. Unter Umständen
könne man sie auch mit einem Anchore scan erkennen.  

OpenShift Guidelines[v3](https://docs.openshift.com/container-platform/3.11/creating_images/guidelines.html) [v4](https://docs.openshift.com/container-platform/4.4/openshift_images/create-images.html)
[Dockerfile best practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
