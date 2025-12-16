# Bones Pràctiques d'Arquitectura d'Aplicacions

Guia de recomanacions per a Java, JavaScript i Python


## Patrons Arquitectònics

### Clean Architecture / Arquitectura Hexagonal

Organitza el codi en capes concèntriques on les dependències apunten cap a dins:

```
┌─────────────────────────────────────────────┐
│  Frameworks & Drivers (BD, Web, UI)         │
│  ┌─────────────────────────────────────┐    │
│  │  Interface Adapters (Controllers,   │    │
│  │  Presenters, Gateways)              │    │
│  │  ┌─────────────────────────────┐    │    │
│  │  │  Use Cases (Application     │    │    │
│  │  │  Business Rules)            │    │    │
│  │  │  ┌─────────────────────┐    │    │    │
│  │  │  │  Entities (Domain)  │    │    │    │
│  │  │  └─────────────────────┘    │    │    │
│  │  └─────────────────────────────┘    │    │
│  └─────────────────────────────────────┘    │
└─────────────────────────────────────────────┘
```

### Model-View-Controller (MVC)

Separa la lògica de negoci (Model), la presentació (View) i el control de flux (Controller).


### Repository Pattern

Abstreu l'accés a dades darrere d'una interfície consistent, facilitant el testing i el canvi de base de dades.

---


## Estructura de Projectes

### Java (Maven/Gradle)

```
src/
├── main/
│   ├── java/com/projecte/
│   │   ├── config/          # Configuració (Spring, etc.)
│   │   ├── controller/      # Endpoints REST
│   │   ├── service/         # Lògica de negoci
│   │   ├── repository/      # Accés a dades (DAO)
│   │   ├── domain/          # Entitats i models
│   │   ├── dto/             # Data Transfer Objects
│   │   └── exception/       # Excepcions personalitzades
│   └── resources/
│       ├── application.yml  # Configuració
│       └── db/migration/    # Migracions Flyway/Liquibase
└── test/
    ├── java/com/projecte/
    │   ├── unit/            # Tests unitaris
    │   └── integration/     # Tests d'integració
    └── resources/
        └── application-test.yml
```

### Python

```
projecte/
├── src/
│   └── projecte/
│       ├── __init__.py
│       ├── api/             # Endpoints (FastAPI/Flask)
│       ├── core/            # Configuració i constants
│       ├── models/          # Models de dades
│       ├── repositories/    # Accés a dades
│       ├── services/        # Lògica de negoci
│       └── utils/           # Utilitats
├── tests/
│   ├── unit/
│   ├── integration/
│   └── conftest.py          # Fixtures pytest
├── pyproject.toml           # Configuració del projecte
└── requirements.txt
```

### JavaScript/TypeScript (Node.js)

```
src/
├── config/                  # Configuració
├── controllers/             # Controladors
├── middleware/              # Middleware Express/Fastify
├── models/                  # Models (Mongoose, Sequelize)
├── routes/                  # Definició de rutes
├── services/                # Lògica de negoci
├── utils/                   # Utilitats
├── validators/              # Validació d'entrada
└── app.js                   # Punt d'entrada
tests/
├── unit/
├── integration/
└── e2e/
```

---

## Testing

### Piràmide de Tests

```
        /\
       /  \     E2E (Cypress, Playwright, Selenium)
      /----\
     /      \   Integració (Testcontainers, Spring Boot Test)
    /--------\
   /          \  Unitaris (JUnit, pytest, Jest)
  /------------\
```

### Frameworks per Llenguatge

| Tipus | Java | Python | JavaScript |
|-------|------|--------|------------|
| Unitaris | JUnit 5 | pytest | Jest, Mocha |
| Mocking | Mockito | unittest.mock, pytest-mock | Jest mocks, Sinon |
| Integració | Testcontainers, Spring Boot Test | pytest + Docker | Supertest |
| E2E | Selenium | Playwright, Selenium | Cypress, Playwright |
| Cobertura | JaCoCo | Coverage.py | Istanbul/nyc |

### Bones Pràctiques de Testing

**Estructura AAA (Arrange-Act-Assert)**

```java
// Java amb JUnit 5
@Test
void sumar_dosPositius_retornaSuma() {
    // Arrange
    Calculadora calc = new Calculadora();
    
    // Act
    int resultat = calc.sumar(2, 3);
    
    // Assert
    assertEquals(5, resultat);
}
```

```python
# Python amb pytest
def test_sumar_dos_positius_retorna_suma():
    # Arrange
    calc = Calculadora()
    
    # Act
    resultat = calc.sumar(2, 3)
    
    # Assert
    assert resultat == 5
```

```javascript
// JavaScript amb Jest
test('sumar dos positius retorna suma', () => {
    // Arrange
    const calc = new Calculadora();
    
    // Act
    const resultat = calc.sumar(2, 3);
    
    // Assert
    expect(resultat).toBe(5);
});
```

**Un concepte per test**: Cada test ha de validar una única funcionalitat.

**Tests independents i aïllats**: Els tests no han de dependre d'altres tests ni d'estat compartit.

**Noms descriptius**: El nom del test ha d'explicar què es prova i quin és el resultat esperat.

---

## Qualitat del Codi

### Linters i Formatadors

| Llenguatge | Linter | Formatador |
|------------|--------|------------|
| Java | Checkstyle, SpotBugs | google-java-format |
| Python | Pylint, Ruff, Flake8 | Black, isort |
| JavaScript | ESLint | Prettier |

### Anàlisi Estàtica

**SonarQube/SonarCloud** identifica:
- Code smells
- Complexitat ciclomàtica excessiva
- Duplicacions
- Vulnerabilitats de seguretat

Pot bloquejar merges si el codi no supera els quality gates definits (cobertura mínima del 80%, zero vulnerabilitats crítiques).


### Pre-commit Hooks

Configura hooks per executar linters i formatadors automàticament abans de cada commit:

```yaml
# .pre-commit-config.yaml (Python)
repos:
  - repo: https://github.com/psf/black
    rev: 23.12.1
    hooks:
      - id: black
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.1.8
    hooks:
      - id: ruff
```

```json
// package.json (JavaScript amb Husky)
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "*.js": ["eslint --fix", "prettier --write"]
  }
}
```

---

## Gestió de Dependències

### Principis

- **Minimitza dependències**: Cada dependència és un risc potencial de seguretat i manteniment.
- **Fixa versions**: Utilitza versions exactes o rangs estrictes per evitar sorpreses.
- **Actualitza regularment**: Utilitza Dependabot, Renovate o Snyk per detectar vulnerabilitats.

### Per Llenguatge

| Llenguatge | Gestor | Fitxer de lock |
|------------|--------|----------------|
| Java | Maven, Gradle | pom.xml, build.gradle.kts |
| Python | pip, Poetry, uv | requirements.txt, poetry.lock |
| JavaScript | npm, yarn, pnpm | package-lock.json, yarn.lock |

---

## Patrons de Disseny Essencials

### Creacionals

- **Factory**: Encapsula la creació d'objectes
- **Builder**: Construeix objectes complexos pas a pas
- **Singleton**: Garanteix una única instància (usar amb precaució)

### Estructurals

- **Adapter**: Converteix interfícies incompatibles
- **Decorator**: Afegeix funcionalitat dinàmicament
- **Facade**: Simplifica interfícies complexes

### Comportamentals

- **Strategy**: Encapsula algoritmes intercanviables
- **Observer**: Notifica canvis a múltiples objectes
- **Command**: Encapsula peticions com a objectes

---

## Injecció de Dependències

### Java (Spring)

```java
@Service
public class UserService {
    private final UserRepository repository;
    private final EmailService emailService;
    
    // Constructor injection (recomanat)
    public UserService(UserRepository repository, EmailService emailService) {
        this.repository = repository;
        this.emailService = emailService;
    }
}
```

### Python (dependency-injector o manual)

```python
class UserService:
    def __init__(self, repository: UserRepository, email_service: EmailService):
        self.repository = repository
        self.email_service = email_service
```

### JavaScript/TypeScript (inversify o manual)

```typescript
class UserService {
    constructor(
        private readonly repository: UserRepository,
        private readonly emailService: EmailService
    ) {}
}
```

---

## Documentació

### Documentació de Codi

| Llenguatge | Eina | Format |
|------------|------|--------|
| Java | Javadoc | `/** ... */` |
| Python | Sphinx, docstrings | `"""..."""` |
| JavaScript | JSDoc, TypeDoc | `/** ... */` |

### APIs

- **REST**: OpenAPI/Swagger per documentació interactiva
- **GraphQL**: GraphQL Playground, Apollo Studio

### Architecture Decision Records (ADRs)

Documenta decisions arquitectòniques importants:
- Quin problema resolien
- Quines alternatives es van considerar
- Per què es va escollir una opció

---

## Seguretat

### Bones Pràctiques

- **Secrets fora del codi**: Utilitza HashiCorp Vault, AWS Secrets Manager o variables d'entorn
- **Validació d'entrada**: Mai confiar en dades de l'usuari
- **Escapament de sortida**: Prevenir XSS i injeccions
- **Principi de mínim privilegi**: Només els permisos necessaris

### Eines d'Anàlisi

| Tipus | Eines |
|-------|-------|
| Dependències | Dependabot, Snyk, OWASP Dependency-Check |
| SAST (codi) | SonarQube, Semgrep, CodeQL |
| DAST (runtime) | OWASP ZAP, Burp Suite |
| Contenidors | Trivy, Clair |

---

## Observabilitat

### Els Tres Pilars

1. **Logs**: ELK Stack (Elasticsearch, Logstash, Kibana), Loki + Grafana
2. **Mètriques**: Prometheus + Grafana, Datadog
3. **Traces**: OpenTelemetry, Jaeger, Zipkin

### Error Tracking

- **Sentry**: Captura excepcions amb context complet
- **Rollbar**, **Bugsnag**: Alternatives populars

---

## CI/CD

### Pipeline Típic

```yaml
# Exemple GitHub Actions
name: CI/CD

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Lint
        run: npm run lint  # o ./gradlew checkstyleMain, ruff check
      
      - name: Test
        run: npm test      # o ./gradlew test, pytest
      
      - name: Build
        run: npm run build # o ./gradlew build, python -m build
      
      - name: Security Scan
        run: npm audit     # o snyk test, safety check
```

### Infrastructure as Code

- **Terraform**, **Pulumi**: Defineix infraestructura versionada
- **Docker**: Encapsula aplicacions amb dependències
- **Kubernetes**: Orquestració per producció

---

## Principi Transversal

Totes aquestes pràctiques comparteixen un objectiu comú: **automatitzar processos repetitius**, **detectar errors el més aviat possible** i crear sistemes **mantenibles, segurs i fiables** a llarg termini.

No cal implementar-ho tot de cop; l'important és avançar progressivament cap a un ecosistema de desenvolupament més madur, prioritzant les pràctiques que aportin més valor al context específic de cada equip i projecte.
