# Pràctiques Indispensables d'Enginyeria del Software

---

## Control de Versions

- **Git** com a estàndard universal
- Estratègies de branching: GitFlow, trunk-based development
- Plataformes: **GitHub**, **GitLab**, **Bitbucket**, **Azure DevOps**
- Pull/Merge Requests amb revisió de codi obligatòria

El control de versions amb Git s'ha convertit en l'estàndard universal per gestionar el codi font. Permet que múltiples desenvolupadors treballin simultàniament sense trepitjar-se, manté un historial complet de canvis i facilita la reversió d'errors. Les estratègies de branching com GitFlow (amb branques de feature, develop i release) o trunk-based development (commits freqüents a main) ajuden a organitzar el flux de treball segons les necessitats de l'equip.

Els Pull Requests o Merge Requests són el mecanisme per integrar canvis de forma controlada. Abans de fusionar codi a la branca principal, un o més companys revisen els canvis, detecten errors, suggereixen millores i asseguren que el codi compleix els estàndards de l'equip. Plataformes com GitHub ofereixen funcionalitats com CODEOWNERS per assignar revisors automàticament segons els fitxers modificats.

---

## CI/CD (Integració i Desplegament Continus)

- Eines CI/CD: **GitHub Actions**, **GitLab CI**, **Jenkins**, **CircleCI**, **Azure Pipelines**
- Infrastructure as Code: **Terraform**, **Pulumi**, **AWS CloudFormation**
- Gestió de configuració: **Ansible**, **Chef**, **Puppet**
- Artefactes: **Nexus**, **Artifactory**, **GitHub Packages**

La integració contínua (CI) consisteix a fusionar freqüentment el codi de tots els desenvolupadors i executar automàticament una bateria de tests i validacions. Per exemple, un pipeline típic amb GitHub Actions pot incloure etapes de lint, test, build i anàlisi de seguretat que s'executen a cada push. Això detecta problemes d'integració immediatament, quan són més fàcils de solucionar.

El desplegament continu (CD) automatitza el procés de portar el codi validat fins a producció. Amb Terraform o Pulumi, la infraestructura es defineix com a codi versionat, garantint que els entorns siguin reproduïbles i auditables. Un pipeline madur pot desplegar automàticament a staging després de cada merge a main, i a producció amb un sol clic o després d'aprovació manual.

---

## Testing

- Tests unitaris: **JUnit** (Java), **pytest** (Python), **Jest** (JavaScript), **xUnit** (.NET)
- Tests d'integració: **Testcontainers**, **Spring Boot Test**
- Tests E2E: **Cypress**, **Playwright**, **Selenium**
- Cobertura: **JaCoCo**, **Istanbul/nyc**, **Coverage.py**
- BDD: **Cucumber**, **SpecFlow**, **Behave**

Una suite de tests ben dissenyada és la xarxa de seguretat que permet fer canvis amb confiança. Els tests unitaris amb frameworks com Jest o pytest validen components aïllats de forma ràpida. Els tests d'integració, sovint utilitzant Testcontainers per aixecar bases de dades o serveis en contenidors, comproven que les peces funcionen juntes. Els tests end-to-end amb Playwright o Cypress simulen el comportament real de l'usuari en un navegador.

Pràctiques com TDD (Test-Driven Development) proposen escriure els tests abans del codi, forçant un disseny més net i modular. BDD (Behavior-Driven Development) amb Cucumber permet escriure tests en llenguatge natural (Gherkin) que serveixen alhora com a documentació viva. La cobertura de codi mesurada amb JaCoCo o Istanbul és útil, però cal interpretar-la amb criteri: un 80% amb tests significatius val més que un 100% amb tests superficials.

---

## Qualitat del Codi

- Linters: **ESLint** (JS/TS), **Pylint/Ruff** (Python), **RuboCop** (Ruby), **Checkstyle** (Java)
- Formatadors: **Prettier**, **Black**, **gofmt**
- Anàlisi estàtica: **SonarQube**, **SonarCloud**, **CodeClimate**
- Pre-commit hooks: **Husky**, **pre-commit**

Els linters i formatadors automàtics eliminen discussions estèrils sobre estil i detecten errors comuns abans que arribin a revisió. Configurar Prettier i ESLint amb Husky per executar-se a cada commit garanteix que tot el codi segueix el mateix estil. Eines com Ruff (un linter de Python extremadament ràpid escrit en Rust) o gofmt (que és canònic en Go) s'integren fàcilment al workflow.

L'anàlisi estàtica amb SonarQube identifica code smells, complexitat ciclomàtica excessiva, duplicacions i vulnerabilitats potencials. Pot bloquejar un merge si el codi no supera els quality gates definits (per exemple, cobertura mínima del 80% o zero vulnerabilitats crítiques). Les revisions de codi humanes complementen això detectant problemes de disseny o lògica de negoci que cap eina automatitzada captaria.

---

## Documentació

- Documentació de codi: **Javadoc**, **Sphinx**, **JSDoc**, **TypeDoc**
- APIs REST: **OpenAPI/Swagger**, **Postman Collections**
- APIs GraphQL: **GraphQL Playground**, **Apollo Studio**
- Wikis i docs: **Notion**, **Confluence**, **GitBook**, **Docusaurus**
- Diagrames: **Mermaid**, **PlantUML**, **Draw.io**

La documentació és sovint la germana pobra del desenvolupament, però és essencial per l'onboarding i la mantenibilitat. Un README clar amb instruccions d'instal·lació i exemples d'ús és el mínim. Per APIs, OpenAPI/Swagger genera documentació interactiva que permet provar endpoints directament des del navegador. Docusaurus o GitBook permeten crear portals de documentació elegants versionats amb el codi.

Els ADRs (Architecture Decision Records) documenten decisions arquitectòniques importants: quin problema resolien, quines alternatives es van considerar i per què es va escollir una opció. Eines com Mermaid permeten incloure diagrames de seqüència o arquitectura directament al Markdown, mantenint-los actualitzats amb el codi. Quan algú pregunta "per què fem servir Kafka i no RabbitMQ?", la resposta està documentada.

---

## Seguretat

- Anàlisi de dependències: **Dependabot**, **Snyk**, **OWASP Dependency-Check**, **Renovate**
- Gestió de secrets: **HashiCorp Vault**, **AWS Secrets Manager**, **Azure Key Vault**, **doppler**
- SAST: **SonarQube**, **Semgrep**, **CodeQL**, **Checkmarx**
- DAST: **OWASP ZAP**, **Burp Suite**
- Escaneig de contenidors: **Trivy**, **Clair**, **Aqua Security**

La seguretat ha de ser integrada des del principi (shift-left security). Dependabot o Renovate obren PRs automàticament quan es detecten vulnerabilitats en dependències, i Trivy escaneja imatges Docker per trobar paquets vulnerables. Snyk pot integrar-se al pipeline per bloquejar builds amb vulnerabilitats crítiques.

Els secrets mai han d'estar al codi font. HashiCorp Vault o AWS Secrets Manager els gestionen de forma segura amb rotació automàtica i control d'accés granular. CodeQL (integrat gratuïtament a GitHub) i Semgrep detecten patrons de codi vulnerables com SQL injection o XSS. OWASP ZAP pot executar-se al pipeline per fer anàlisi dinàmica de l'aplicació desplegada a un entorn de test.

---

## Observabilitat

- Logging: **ELK Stack** (Elasticsearch, Logstash, Kibana), **Loki + Grafana**, **Datadog**
- Mètriques: **Prometheus + Grafana**, **Datadog**, **New Relic**, **CloudWatch**
- Tracing: **Jaeger**, **Zipkin**, **OpenTelemetry**, **Datadog APM**
- Alerting: **PagerDuty**, **Opsgenie**, **Prometheus Alertmanager**
- Error tracking: **Sentry**, **Rollbar**, **Bugsnag**

Quan una aplicació falla en producció, necessitem visibilitat per diagnosticar el problema ràpidament. L'stack ELK o Loki+Grafana permeten centralitzar logs de tots els serveis i fer cerques eficients. Sentry captura excepcions amb context complet (stack trace, request, usuari) i agrupa errors similars, facilitant la priorització.

El tracing distribuït amb OpenTelemetry (l'estàndard emergent) o Jaeger és essencial en microserveis. Permet seguir una petició a través de múltiples serveis i identificar exactament on es produeixen latències. Prometheus recull mètriques que alimenten dashboards a Grafana i alertes que notifiquen via PagerDuty quan els SLOs estan en risc.

---

## Metodologies Àgils

- Frameworks: **Scrum**, **Kanban**, **SAFe** (per escalar)
- Gestió de projectes: **Jira**, **Linear**, **Asana**, **Trello**, **Azure Boards**
- Retrospectives: **Miro**, **FunRetro**, **EasyRetro**
- Documentació àgil: **Notion**, **Confluence**

Les metodologies àgils prioritzen iteracions curtes (sprints de 1-2 setmanes en Scrum) amb lliuraments freqüents i feedback constant. Eines com Jira o Linear permeten gestionar el backlog, planificar sprints i visualitzar el flux de treball amb taulers Kanban. Linear ha guanyat popularitat per la seva velocitat i experiència d'usuari moderna.

Les retrospectives amb Miro o EasyRetro són sessions on l'equip reflexiona sobre què ha funcionat bé i què cal millorar. Aquesta pràctica de millora contínua és potser l'element més valuós de l'agilitat. Un equip que aprèn dels seus errors i optimitza els seus processos constantment esdevé progressivament més eficient i cohesionat.

---

## Contenidors i Orquestració

- Contenidors: **Docker**, **Podman**
- Orquestració: **Kubernetes**, **Docker Swarm**, **Amazon ECS**, **Nomad**
- Gestió K8s: **Helm**, **Kustomize**, **ArgoCD**, **Flux**
- Service mesh: **Istio**, **Linkerd**, **Cilium**
- Desenvolupament local: **Docker Compose**, **Minikube**, **Kind**, **Tilt**

Docker ha revolucionat el desplegament encapsulant l'aplicació amb totes les seves dependències. Elimina el clàssic "a la meva màquina funciona": si funciona en un contenidor, funcionarà igual a qualsevol entorn. Docker Compose permet definir entorns multi-contenidor per desenvolupament local, mentre que Podman ofereix una alternativa sense daemon i més segura.

Kubernetes s'ha convertit en l'estàndard d'orquestració per producció. Gestiona automàticament el desplegament, l'escalat horitzontal, el balanceig de càrrega i la recuperació de fallades. Helm empaqueta aplicacions K8s en charts reutilitzables, i ArgoCD implementa GitOps: l'estat desitjat del cluster es defineix a Git, i ArgoCD s'encarrega de sincronitzar-ho automàticament. Per desenvolupament local, Tilt accelera el cicle de feedback reconstruint i redesplegant automàticament quan el codi canvia.

---

## Principi Transversal

Totes aquestes pràctiques comparteixen un objectiu comú: **automatitzar processos repetitius**, **detectar errors el més aviat possible** i crear sistemes **mantenibles, segurs i fiables** a llarg termini. No cal implementar-ho tot de cop; l'important és avançar progressivament cap a un ecosistema de desenvolupament més madur, prioritzant les pràctiques que aportin més valor al context específic de cada equip i projecte.
