# OpenShift migration best practice

Dieses Dokument gibt eine kurze Übersicht über die Schritte einer migration von Docker zu OpenShift und was dabei zu beachten ist.

Schritte:

* OpenShift kompatibles Image
  Prüfen of das Container Image kompatibel mit OpenShift is und gegebenenfalls anpassen.
* Deployment Konfiguration erstellen
* ggf. Deployment Automatisierung

Weiterführende Blogposts zu den drei Schritten: [Schritt 1](https://www.puzzle.ch/de/blog/articles/2019/10/31/migration-einer-dockerisierten-anwendung-nach-openshift-teil-1-3),[2](https://www.puzzle.ch/de/blog/articles/2019/11/21/ausrollen-einer-dockerisierten-anwendung-auf-openshift-teil-2-3) & [3](https://www.puzzle.ch/de/blog/articles/2019/12/23/ci-cd-pipelin-integrationstests-auf-openshift-teil-3-3).

## Kompatibles Image

Die *Security Context Constraints* kontrollieren die Berechtigungen für laufende
Container (Pods) im OpenShift. Dies soll verhindern, dass Container Zugriff auf
den Host und das Cluster erlangen. Per default wird Containern die restricted SCC zugewiesen. Das bedeutet unter anderem:

* kein root or privileged Container
* Arbitrary User IDs (Container werden mit einer Zufälligen UID gestartet)
* SELinux ist aktiviert

[SCC Dokumentation](https://docs.openshift.com/container-platform/4.4/authentication/managing-security-context-constraints.html), [OpenShift Image Guidelines](https://docs.openshift.com/container-platform/4.4/openshift_images/create-images.html#images-create-guide-openshift_create-images)

Die häufigsten Probleme sind hier:

* Container process hat kein zugriff auf seine Dateien (permission denied)
* Container process benötigt root/privileged (z.B. für docker-in-docker)

Die Lösung für das erste Problem ist im Dockerfile der Root-Gruppe rechte auf die Dateien zu geben. Um zu sehen welche Permissions geändert werden müssen ist es hilfreich erst im Dockerfile des Images nachzuschauen was wo installiert wird. In einem zweiten Schritt kann man dann das Image im OpenShift starten und in den logs schauen ob noch weitere Permissions geändert werden müssen.

```
FROM image-to-be-migrated
RUN chgrp -R 0 /some/directory && \
    chmod -R g=u /some/directory
```

[OpenShift Guidelines](https://docs.openshift.com/container-platform/4.4/openshift_images/create-images.html)
[Dockerfile best practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

Wenn der Container zwingend root/privileged benötigt muss eine alternative Architekturen in Betracht gezogen werden. Grundsätzlich kann ihm die `privileged` SCC zugewiesen werden dies ist aber mit enormen Sicherheitsrisiken verbunden und könnte das ganze Cluster beeinträchtigen

# Deployment Konfiguration

Für die Erstellung der yaml oder json Dateien, welche die Ressourcen Definitionen enthalten empfiehlt es sich zuerst:
* Beim Hersteller nach Konfigurationen zu suchen
* weitere Suche im Internet (evtl. gibt es nicht-offizielle Beispiele bzw. Anleitungen)

Falls die Suche geglückt ist, sollte es ausprobiert und wenn nötig angepasst werden.

Für K8S erstellte Helm charts sind leider meistens nicht mit OpenShift kompatibel. Sie können aber mit dem `helm template` Befahl lokal gerendert werden und dann als Vorlage dienen.

Wenn keine Konfigurationsvorlage gefunden wurde kann man mit dem CLI (z.B. `oc new-app`) und GUI tools von OpenShift ein Deployment engineeren und dieses dann exportieren. Hierfür sind die flags `--dry-run` und `-o yaml` nützlich. Ersteres erstellt keine Resources sondern gibt nur die Befahle aus, Zweitieres kann verwendet werden um Ressourcen als `.yaml` zu exportieren.

Einige Deployment best practices:

* [Templates](https://docs.openshift.com/container-platform/4.4/openshift_images/using-templates.html)
  Ermöglicht die einfache Parametrisierung des Deployments für unterschiedliche Environments
* [Liveness & Readiness probes](https://docs.openshift.com/container-platform/4.4/applications/application-health.html)
  Verringern downtime indem sie OpenShift ermöglichen zu sehen wann ein Container up ist und hängende Container neu zu starten.
* [CPU & Memory Request und Limits](https://docs.openshift.com/enterprise/3.1/dev_guide/compute_resources.html)
  Ermöglichen OpenShift die Container effizient zu platzieren und verbessern und vereinheitlichen so die performance der Applikation.
