# Tutorial: Testing amb JUnit 5 per Aplicacions Hibernate 6

## Índex
1. [Configuració del Projecte](#1-configuració-del-projecte)
2. [Arquitectura de Tests](#2-arquitectura-de-tests)
3. [Cicle de Vida dels Tests](#3-cicle-de-vida-dels-tests)
4. [Testing de la Capa DAO](#4-testing-de-la-capa-dao)
5. [Testing d'Entitats i Relacions](#5-testing-dentitats-i-relacions)
6. [Aïllament i Base de Dades de Test](#6-aïllament-i-base-de-dades-de-test)
7. [Assertions Avançades amb AssertJ](#7-assertions-avançades-amb-assertj)
8. [Tests Parametritzats](#8-tests-parametritzats)
9. [Gestió de Transaccions en Tests](#9-gestió-de-transaccions-en-tests)
10. [Patrons i Antipatrons](#10-patrons-i-antipatrons)

---

## 1. Configuració del Projecte

### 1.1 Dependències Maven

```xml
<dependencies>
    <!-- JUnit 5 - Jupiter API -->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-api</artifactId>
        <version>5.11.3</version>
        <scope>test</scope>
    </dependency>
    
    <!-- JUnit 5 - Engine (necessari per executar tests) -->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-engine</artifactId>
        <version>5.11.3</version>
        <scope>test</scope>
    </dependency>
    
    <!-- JUnit 5 - Params (tests parametritzats) -->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-params</artifactId>
        <version>5.11.3</version>
        <scope>test</scope>
    </dependency>
    
    <!-- AssertJ - Assertions fluides i llegibles -->
    <dependency>
        <groupId>org.assertj</groupId>
        <artifactId>assertj-core</artifactId>
        <version>3.26.3</version>
        <scope>test</scope>
    </dependency>
    
    <!-- H2 Database - BD en memòria per tests -->
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <version>2.2.224</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

### 1.2 Estructura de Directoris

```
src/
├── main/
│   ├── java/com/project/
│   │   ├── dao/Manager.java
│   │   └── domain/{Employee,Contact,Project}.java
│   └── resources/
│       └── hibernate.properties          # Configuració producció (SQLite)
└── test/
    ├── java/com/project/
    │   ├── dao/ManagerTest.java
    │   ├── domain/
    │   │   ├── EmployeeTest.java
    │   │   ├── ContactTest.java
    │   │   └── ProjectTest.java
    │   └── integration/
    │       └── EmployeeProjectIntegrationTest.java
    └── resources/
        └── hibernate-test.properties     # Configuració test (H2 en memòria)
```

### 1.3 Fitxer de Propietats per Tests

**`src/test/resources/hibernate-test.properties`**
```properties
# H2 Database en memòria - Ideal per tests
hibernate.connection.driver_class=org.h2.Driver
hibernate.connection.url=jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1;MODE=MySQL
hibernate.connection.username=sa
hibernate.connection.password=

# Dialecte H2
hibernate.dialect=org.hibernate.dialect.H2Dialect

# Mostra SQL per depuració de tests
hibernate.show_sql=true
hibernate.format_sql=true

# IMPORTANT: create-drop per aïllament entre tests
hibernate.hbm2ddl.auto=create-drop

# Estadístiques útils per tests de rendiment
hibernate.generate_statistics=true
```

---

## 2. Arquitectura de Tests

### 2.1 Principis FIRST

| Principi | Significat | Aplicació a Hibernate |
|----------|------------|----------------------|
| **F**ast | Tests ràpids | Usar BD en memòria (H2), evitar I/O disc |
| **I**ndependent | Tests aïllats | Cada test té el seu estat, `create-drop` |
| **R**epeatable | Resultats consistents | No dependre de dades externes |
| **S**elf-validating | Pass/Fail automàtic | Assertions clares i completes |
| **T**imely | Escrits a temps | TDD o simultani al codi |

### 2.2 Piràmide de Tests per DAO/Hibernate

```
            /\
           /  \         <- Tests E2E (pocs, lents)
          /    \           Verifiquen flux complet
         /------\
        /        \      <- Tests d'Integració
       /          \        Verifiquen DAO + BD real
      /------------\
     /              \   <- Tests Unitaris (molts, ràpids)
    /                \     Verifiquen lògica entitats
   /------------------\
```

---

## 3. Cicle de Vida dels Tests

### 3.1 Anotacions de Cicle de Vida JUnit 5

```java
package com.project.dao;

import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;

@TestInstance(TestInstance.Lifecycle.PER_CLASS)  // Una instància per tots els tests
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)  // Ordre determinista
class ManagerTest {

    // S'executa UNA vegada abans de TOTS els tests de la classe
    @BeforeAll
    static void initAll() {
        // Crear SessionFactory amb configuració de test
        Manager.createSessionFactory("hibernate-test.properties");
    }

    // S'executa UNA vegada després de TOTS els tests
    @AfterAll
    static void tearDownAll() {
        Manager.close();
    }

    // S'executa ABANS de cada test individual
    @BeforeEach
    void setUp() {
        // Netejar dades o preparar estat inicial
        clearDatabase();
    }

    // S'executa DESPRÉS de cada test individual
    @AfterEach
    void tearDown() {
        // Alliberar recursos si cal
    }

    private void clearDatabase() {
        // Implementar neteja de taules
    }
}
```

### 3.2 Patró de Base Test per Hibernate

```java
package com.project.test;

import com.project.dao.Manager;
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.junit.jupiter.api.*;

/**
 * Classe base per tots els tests que necessiten accés a Hibernate.
 * Proporciona setup/teardown comú i utilitats.
 */
public abstract class HibernateTestBase {
    
    protected static SessionFactory sessionFactory;
    
    @BeforeAll
    static void initHibernate() {
        Manager.createSessionFactory("hibernate-test.properties");
    }
    
    @AfterAll
    static void closeHibernate() {
        Manager.close();
    }
    
    @BeforeEach
    void cleanDatabase() {
        // Neteja totes les taules en ordre invers a les dependències FK
        try (Session session = Manager.getSessionFactory().openSession()) {
            session.beginTransaction();
            // Primer la taula pont (sense FK cap a ella)
            session.createNativeQuery("DELETE FROM employee_project").executeUpdate();
            // Després Contact (FK a Employee)
            session.createNativeQuery("DELETE FROM contacts").executeUpdate();
            // Després les principals
            session.createNativeQuery("DELETE FROM projects").executeUpdate();
            session.createNativeQuery("DELETE FROM employees").executeUpdate();
            session.getTransaction().commit();
        }
    }
    
    /**
     * Helper per executar codi dins d'una transacció.
     * Útil per tests que necessiten control fi de la sessió.
     */
    protected <T> T executeInTransaction(TransactionCallback<T> callback) {
        try (Session session = Manager.getSessionFactory().openSession()) {
            session.beginTransaction();
            T result = callback.execute(session);
            session.getTransaction().commit();
            return result;
        }
    }
    
    @FunctionalInterface
    protected interface TransactionCallback<T> {
        T execute(Session session);
    }
}
```

---

## 4. Testing de la Capa DAO

### 4.1 Test CRUD Complet del Manager

```java
package com.project.dao;

import com.project.domain.*;
import com.project.test.HibernateTestBase;
import org.junit.jupiter.api.*;

import java.util.Collection;
import java.util.Set;

import static org.junit.jupiter.api.Assertions.*;

@DisplayName("Manager DAO Tests")
class ManagerTest extends HibernateTestBase {

    // ========== TESTS CREATE ==========
    
    @Nested
    @DisplayName("Operacions CREATE")
    class CreateTests {
        
        @Test
        @DisplayName("addEmployee retorna entitat amb ID generat")
        void addEmployee_ShouldReturnEntityWithGeneratedId() {
            // Arrange - res a preparar
            
            // Act
            Employee result = Manager.addEmployee("Joan", "Garcia", 35000);
            
            // Assert
            assertAll("Employee creat correctament",
                () -> assertNotNull(result, "L'empleat no hauria de ser null"),
                () -> assertNotNull(result.getEmployeeId(), "L'ID hauria d'estar assignat"),
                () -> assertTrue(result.getEmployeeId() > 0, "L'ID ha de ser positiu"),
                () -> assertEquals("Joan", result.getFirstName()),
                () -> assertEquals("Garcia", result.getLastName()),
                () -> assertEquals(35000, result.getSalary())
            );
        }
        
        @Test
        @DisplayName("addProject crea projecte amb estat inicial")
        void addProject_ShouldCreateProjectWithStatus() {
            // Act
            Project result = Manager.addProject("Web App", "Descripció", "ACTIU");
            
            // Assert
            assertNotNull(result.getProjectId());
            assertEquals("ACTIU", result.getStatus());
        }
        
        @Test
        @DisplayName("addContactToEmployee vincula correctament")
        void addContactToEmployee_ShouldLinkContactToEmployee() {
            // Arrange
            Employee emp = Manager.addEmployee("Test", "User", 30000);
            
            // Act
            Contact contact = Manager.addContactToEmployee(
                emp.getEmployeeId(), 
                "EMAIL", 
                "test@example.com", 
                "Email de test"
            );
            
            // Assert
            assertNotNull(contact.getContactId());
            assertEquals("EMAIL", contact.getContactType());
            
            // Verificar la relació bidireccional
            Employee refreshed = Manager.getById(Employee.class, emp.getEmployeeId());
            assertTrue(refreshed.getContacts().contains(contact));
        }
    }
    
    // ========== TESTS READ ==========
    
    @Nested
    @DisplayName("Operacions READ")
    class ReadTests {
        
        private Employee testEmployee;
        private Project testProject;
        
        @BeforeEach
        void setUpTestData() {
            testEmployee = Manager.addEmployee("Maria", "Test", 40000);
            testProject = Manager.addProject("Test Project", "Desc", "ACTIU");
            Manager.addContactToEmployee(
                testEmployee.getEmployeeId(), "PHONE", "666123456", "Mòbil"
            );
        }
        
        @Test
        @DisplayName("getById retorna entitat existent")
        void getById_WhenExists_ShouldReturnEntity() {
            Employee result = Manager.getById(
                Employee.class, 
                testEmployee.getEmployeeId()
            );
            
            assertNotNull(result);
            assertEquals(testEmployee.getEmployeeId(), result.getEmployeeId());
        }
        
        @Test
        @DisplayName("getById retorna null si no existeix")
        void getById_WhenNotExists_ShouldReturnNull() {
            Employee result = Manager.getById(Employee.class, 99999L);
            
            assertNull(result);
        }
        
        @Test
        @DisplayName("listCollection retorna tots els elements")
        void listCollection_ShouldReturnAllEntities() {
            // Afegir un segon empleat
            Manager.addEmployee("Segon", "Empleat", 25000);
            
            Collection<Employee> result = Manager.listCollection(Employee.class);
            
            assertEquals(2, result.size());
        }
        
        @Test
        @DisplayName("findEmployeesByContactType retorna empleats filtrats")
        void findEmployeesByContactType_ShouldReturnFilteredEmployees() {
            // Afegir empleat sense telèfon
            Manager.addEmployee("Sense", "Telefon", 20000);
            
            Collection<Employee> result = Manager.findEmployeesByContactType("PHONE");
            
            assertEquals(1, result.size());
            assertTrue(result.stream()
                .anyMatch(e -> e.getEmployeeId().equals(testEmployee.getEmployeeId())));
        }
    }
    
    // ========== TESTS UPDATE ==========
    
    @Nested
    @DisplayName("Operacions UPDATE")
    class UpdateTests {
        
        @Test
        @DisplayName("updateEmployee modifica tots els camps")
        void updateEmployee_ShouldModifyAllFields() {
            // Arrange
            Employee emp = Manager.addEmployee("Original", "Name", 30000);
            Long id = emp.getEmployeeId();
            
            // Act
            Manager.updateEmployee(id, "Nou", "Nom", 50000);
            
            // Assert
            Employee updated = Manager.getById(Employee.class, id);
            assertAll(
                () -> assertEquals("Nou", updated.getFirstName()),
                () -> assertEquals("Nom", updated.getLastName()),
                () -> assertEquals(50000, updated.getSalary())
            );
        }
        
        @Test
        @DisplayName("updateEmployeeProjects assigna projectes correctament")
        void updateEmployeeProjects_ShouldAssignProjects() {
            // Arrange
            Employee emp = Manager.addEmployee("Test", "Employee", 30000);
            Project p1 = Manager.addProject("Project 1", "Desc", "ACTIU");
            Project p2 = Manager.addProject("Project 2", "Desc", "PLANIFICAT");
            
            // Act
            Manager.updateEmployeeProjects(emp.getEmployeeId(), Set.of(p1, p2));
            
            // Assert
            Employee updated = Manager.getById(Employee.class, emp.getEmployeeId());
            assertEquals(2, updated.getProjects().size());
        }
    }
    
    // ========== TESTS DELETE ==========
    
    @Nested
    @DisplayName("Operacions DELETE")
    class DeleteTests {
        
        @Test
        @DisplayName("delete elimina entitat de la BD")
        void delete_ShouldRemoveEntity() {
            // Arrange
            Project project = Manager.addProject("ToDelete", "Desc", "ACTIU");
            Long id = project.getProjectId();
            
            // Act
            Manager.delete(Project.class, id);
            
            // Assert
            assertNull(Manager.getById(Project.class, id));
        }
        
        @Test
        @DisplayName("delete Employee elimina contacts en cascada (orphanRemoval)")
        void deleteEmployee_ShouldCascadeDeleteContacts() {
            // Arrange
            Employee emp = Manager.addEmployee("With", "Contacts", 30000);
            Contact c1 = Manager.addContactToEmployee(
                emp.getEmployeeId(), "EMAIL", "test@test.com", "Test"
            );
            Long contactId = c1.getContactId();
            
            // Act
            Manager.delete(Employee.class, emp.getEmployeeId());
            
            // Assert - El contact també s'ha d'haver eliminat
            assertNull(Manager.getById(Contact.class, contactId));
        }
    }
}
```

### 4.2 Test de Consultes HQL

```java
package com.project.dao;

import com.project.domain.*;
import com.project.test.HibernateTestBase;
import org.junit.jupiter.api.*;
import java.util.Collection;

import static org.junit.jupiter.api.Assertions.*;
import static org.assertj.core.api.Assertions.*;  // AssertJ

@DisplayName("Tests de Consultes HQL")
class ManagerQueryTest extends HibernateTestBase {
    
    @BeforeEach
    void setUpTestData() {
        // Crear estructura de dades de prova
        Employee joan = Manager.addEmployee("Joan", "Garcia", 35000);
        Employee marta = Manager.addEmployee("Marta", "Ferrer", 42000);
        Employee pere = Manager.addEmployee("Pere", "Soler", 38000);
        
        // Afegir contactes
        Manager.addContactToEmployee(joan.getEmployeeId(), "EMAIL", "joan@test.com", "Email");
        Manager.addContactToEmployee(joan.getEmployeeId(), "PHONE", "666111222", "Mòbil");
        Manager.addContactToEmployee(marta.getEmployeeId(), "EMAIL", "marta@test.com", "Email");
        Manager.addContactToEmployee(pere.getEmployeeId(), "PHONE", "666333444", "Mòbil");
        
        // Crear projectes i assignar
        Project web = Manager.addProject("Web", "Desc", "ACTIU");
        Project app = Manager.addProject("App", "Desc", "ACTIU");
        
        Manager.updateEmployeeProjects(joan.getEmployeeId(), Set.of(web));
        Manager.updateEmployeeProjects(marta.getEmployeeId(), Set.of(web, app));
        Manager.updateEmployeeProjects(pere.getEmployeeId(), Set.of(app));
    }
    
    @Test
    @DisplayName("Cerca empleats per tipus de contacte EMAIL")
    void findEmployeesByContactType_Email() {
        Collection<Employee> result = Manager.findEmployeesByContactType("EMAIL");
        
        // AssertJ proporciona assertions més llegibles
        assertThat(result)
            .hasSize(2)
            .extracting(Employee::getFirstName)
            .containsExactlyInAnyOrder("Joan", "Marta");
    }
    
    @Test
    @DisplayName("Cerca empleats per projecte retorna els correctes")
    void findEmployeesByProject_ShouldReturnCorrectEmployees() {
        // Obtenir el projecte "App"
        Collection<Project> projects = Manager.listCollection(Project.class);
        Project app = projects.stream()
            .filter(p -> p.getName().equals("App"))
            .findFirst()
            .orElseThrow();
        
        Collection<Employee> result = Manager.findEmployeesByProject(app.getProjectId());
        
        assertThat(result)
            .hasSize(2)
            .extracting(Employee::getFirstName)
            .containsExactlyInAnyOrder("Marta", "Pere");
    }
    
    @Test
    @DisplayName("listCollection amb filtre HQL")
    void listCollection_WithFilter_ShouldReturnFiltered() {
        Collection<Employee> highSalary = Manager.listCollection(
            Employee.class, 
            "salary > 37000"
        );
        
        assertThat(highSalary)
            .hasSize(2)
            .allMatch(e -> e.getSalary() > 37000);
    }
}
```

---

## 5. Testing d'Entitats i Relacions

### 5.1 Test de l'Entitat Employee

```java
package com.project.domain;

import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;

@DisplayName("Employee Entity Tests")
class EmployeeTest {
    
    private Employee employee;
    
    @BeforeEach
    void setUp() {
        employee = new Employee("Joan", "Garcia", 35000);
    }
    
    @Nested
    @DisplayName("Constructor i Getters")
    class ConstructorTests {
        
        @Test
        @DisplayName("Constructor inicialitza correctament")
        void constructor_ShouldInitializeAllFields() {
            assertAll(
                () -> assertEquals("Joan", employee.getFirstName()),
                () -> assertEquals("Garcia", employee.getLastName()),
                () -> assertEquals(35000, employee.getSalary()),
                () -> assertNotNull(employee.getContacts()),
                () -> assertTrue(employee.getContacts().isEmpty()),
                () -> assertNotNull(employee.getProjects()),
                () -> assertTrue(employee.getProjects().isEmpty())
            );
        }
        
        @Test
        @DisplayName("Constructor per defecte crea col·leccions buides")
        void defaultConstructor_ShouldCreateEmptyCollections() {
            Employee emp = new Employee();
            
            assertNotNull(emp.getContacts());
            assertNotNull(emp.getProjects());
        }
    }
    
    @Nested
    @DisplayName("Mètodes Helper de Relacions")
    class RelationshipHelperTests {
        
        @Test
        @DisplayName("addContact manté consistència bidireccional")
        void addContact_ShouldMaintainBidirectionalConsistency() {
            Contact contact = new Contact("EMAIL", "test@test.com", "Test");
            
            employee.addContact(contact);
            
            assertAll(
                () -> assertTrue(employee.getContacts().contains(contact)),
                () -> assertEquals(employee, contact.getEmployee())
            );
        }
        
        @Test
        @DisplayName("removeContact trenca la relació bidireccional")
        void removeContact_ShouldBreakBidirectionalLink() {
            Contact contact = new Contact("EMAIL", "test@test.com", "Test");
            employee.addContact(contact);
            
            employee.removeContact(contact);
            
            assertAll(
                () -> assertFalse(employee.getContacts().contains(contact)),
                () -> assertNull(contact.getEmployee())
            );
        }
        
        @Test
        @DisplayName("addProject estableix relació ManyToMany")
        void addProject_ShouldEstablishManyToMany() {
            Project project = new Project("Test Project", "Desc", "ACTIU");
            
            employee.addProject(project);
            
            assertAll(
                () -> assertTrue(employee.getProjects().contains(project)),
                () -> assertTrue(project.getEmployees().contains(employee))
            );
        }
        
        @Test
        @DisplayName("removeProject elimina de tots dos costats")
        void removeProject_ShouldRemoveFromBothSides() {
            Project project = new Project("Test Project", "Desc", "ACTIU");
            employee.addProject(project);
            
            employee.removeProject(project);
            
            assertAll(
                () -> assertFalse(employee.getProjects().contains(project)),
                () -> assertFalse(project.getEmployees().contains(employee))
            );
        }
    }
    
    @Nested
    @DisplayName("equals() i hashCode()")
    class EqualsHashCodeTests {
        
        @Test
        @DisplayName("equals amb mateixa instància retorna true")
        void equals_SameInstance_ReturnsTrue() {
            assertEquals(employee, employee);
        }
        
        @Test
        @DisplayName("equals amb null retorna false")
        void equals_Null_ReturnsFalse() {
            assertNotEquals(null, employee);
        }
        
        @Test
        @DisplayName("equals amb diferent classe retorna false")
        void equals_DifferentClass_ReturnsFalse() {
            assertNotEquals(employee, "String");
        }
        
        @Test
        @DisplayName("hashCode és consistent")
        void hashCode_IsConsistent() {
            int hash1 = employee.hashCode();
            int hash2 = employee.hashCode();
            
            assertEquals(hash1, hash2);
        }
        
        @Test
        @DisplayName("equals i hashCode són consistents")
        void equalsAndHashCode_AreConsistent() {
            Employee emp1 = new Employee("Test", "User", 30000);
            Employee emp2 = new Employee("Test", "User", 30000);
            
            // Si equals és true, hashCode ha de ser igual
            if (emp1.equals(emp2)) {
                assertEquals(emp1.hashCode(), emp2.hashCode());
            }
        }
    }
}
```

### 5.2 Test de Relacions ManyToMany amb Persistència

```java
package com.project.integration;

import com.project.dao.Manager;
import com.project.domain.*;
import com.project.test.HibernateTestBase;
import org.junit.jupiter.api.*;

import java.util.Set;

import static org.assertj.core.api.Assertions.*;

@DisplayName("Tests d'Integració Employee-Project (ManyToMany)")
class EmployeeProjectIntegrationTest extends HibernateTestBase {
    
    @Test
    @DisplayName("Assignar múltiples projectes a un empleat")
    void assignMultipleProjectsToEmployee() {
        // Arrange
        Employee emp = Manager.addEmployee("Multi", "Project", 40000);
        Project p1 = Manager.addProject("Project 1", "Desc", "ACTIU");
        Project p2 = Manager.addProject("Project 2", "Desc", "ACTIU");
        Project p3 = Manager.addProject("Project 3", "Desc", "PLANIFICAT");
        
        // Act
        Manager.updateEmployeeProjects(emp.getEmployeeId(), Set.of(p1, p2, p3));
        
        // Assert - Verificar des de l'empleat
        Employee refreshedEmp = Manager.getById(Employee.class, emp.getEmployeeId());
        assertThat(refreshedEmp.getProjects()).hasSize(3);
        
        // Assert - Verificar des dels projectes
        Project refreshedP1 = Manager.getById(Project.class, p1.getProjectId());
        assertThat(refreshedP1.getEmployees())
            .extracting(Employee::getEmployeeId)
            .contains(emp.getEmployeeId());
    }
    
    @Test
    @DisplayName("Reassignar projectes reemplaça els anteriors")
    void reassignProjects_ShouldReplaceExisting() {
        // Arrange
        Employee emp = Manager.addEmployee("Reassign", "Test", 40000);
        Project oldProject = Manager.addProject("Old", "Desc", "ACTIU");
        Project newProject = Manager.addProject("New", "Desc", "ACTIU");
        
        Manager.updateEmployeeProjects(emp.getEmployeeId(), Set.of(oldProject));
        
        // Act - Reassignar amb projecte diferent
        Manager.updateEmployeeProjects(emp.getEmployeeId(), Set.of(newProject));
        
        // Assert
        Employee refreshed = Manager.getById(Employee.class, emp.getEmployeeId());
        assertThat(refreshed.getProjects())
            .hasSize(1)
            .extracting(Project::getName)
            .containsOnly("New");
    }
    
    @Test
    @DisplayName("Eliminar empleat no elimina projectes")
    void deleteEmployee_ShouldNotDeleteProjects() {
        // Arrange
        Employee emp = Manager.addEmployee("Delete", "Me", 30000);
        Project project = Manager.addProject("Survive", "Desc", "ACTIU");
        Manager.updateEmployeeProjects(emp.getEmployeeId(), Set.of(project));
        Long projectId = project.getProjectId();
        
        // Act
        Manager.delete(Employee.class, emp.getEmployeeId());
        
        // Assert - El projecte ha de sobreviure
        Project survived = Manager.getById(Project.class, projectId);
        assertThat(survived).isNotNull();
        assertThat(survived.getEmployees()).isEmpty();
    }
}
```

---

## 6. Aïllament i Base de Dades de Test

### 6.1 Configuració H2 per Aïllament Total

```java
package com.project.test;

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;
import org.junit.jupiter.api.*;

/**
 * Test base que crea una BD H2 fresca per cada classe de test.
 * Màxim aïllament però més lent (crear esquema cada vegada).
 */
public abstract class IsolatedHibernateTest {
    
    protected SessionFactory sessionFactory;
    
    @BeforeEach
    void setUpDatabase() {
        Configuration config = new Configuration();
        config.setProperty("hibernate.connection.driver_class", "org.h2.Driver");
        // Cada test obté una BD única amb UUID
        config.setProperty("hibernate.connection.url", 
            "jdbc:h2:mem:test_" + System.nanoTime() + ";DB_CLOSE_DELAY=-1");
        config.setProperty("hibernate.dialect", "org.hibernate.dialect.H2Dialect");
        config.setProperty("hibernate.hbm2ddl.auto", "create-drop");
        
        // Registrar entitats
        config.addAnnotatedClass(com.project.domain.Employee.class);
        config.addAnnotatedClass(com.project.domain.Contact.class);
        config.addAnnotatedClass(com.project.domain.Project.class);
        
        sessionFactory = config.buildSessionFactory();
    }
    
    @AfterEach
    void tearDownDatabase() {
        if (sessionFactory != null) {
            sessionFactory.close();
        }
    }
    
    protected Session openSession() {
        return sessionFactory.openSession();
    }
}
```

### 6.2 Execució Paral·lela de Tests

**`src/test/resources/junit-platform.properties`**
```properties
# Execució paral·lela de tests
junit.jupiter.execution.parallel.enabled=true
junit.jupiter.execution.parallel.mode.default=concurrent
junit.jupiter.execution.parallel.mode.classes.default=concurrent

# Nombre de threads (cores disponibles)
junit.jupiter.execution.parallel.config.strategy=dynamic
junit.jupiter.execution.parallel.config.dynamic.factor=1
```

**Ús amb `@Isolated`** per tests que no poden ser paral·lels:

```java
@Isolated  // Aquest test s'executa sol
@DisplayName("Test que modifica estat global")
void testQueNecessitaIsolament() {
    // ...
}
```

---

## 7. Assertions Avançades amb AssertJ

### 7.1 Assertions per Col·leccions

```java
import static org.assertj.core.api.Assertions.*;

@Test
@DisplayName("Verificar col·lecció d'empleats amb AssertJ")
void verifyEmployeeCollection() {
    Collection<Employee> employees = Manager.listCollection(Employee.class);
    
    assertThat(employees)
        .isNotEmpty()
        .hasSize(3)
        .extracting(Employee::getFirstName, Employee::getSalary)
        .contains(
            tuple("Joan", 35000),
            tuple("Marta", 42000)
        );
}

@Test
@DisplayName("Verificar que tots compleixen una condició")
void verifyAllMatchCondition() {
    Collection<Employee> employees = Manager.listCollection(Employee.class);
    
    assertThat(employees)
        .allSatisfy(emp -> {
            assertThat(emp.getEmployeeId()).isNotNull();
            assertThat(emp.getSalary()).isPositive();
        });
}

@Test
@DisplayName("Filtrar i verificar")
void filterAndVerify() {
    Collection<Employee> employees = Manager.listCollection(Employee.class);
    
    assertThat(employees)
        .filteredOn(emp -> emp.getSalary() > 40000)
        .hasSize(1)
        .first()
        .extracting(Employee::getFirstName)
        .isEqualTo("Marta");
}
```

### 7.2 Assertions per Excepcions

```java
@Test
@DisplayName("Verificar que llança excepció correcta")
void shouldThrowExceptionForInvalidOperation() {
    assertThatThrownBy(() -> Manager.getById(Employee.class, null))
        .isInstanceOf(IllegalArgumentException.class)
        .hasMessageContaining("id")
        .hasNoCause();
}

@Test
@DisplayName("Verificar que NO llança excepció")
void shouldNotThrowException() {
    assertThatCode(() -> Manager.addEmployee("Valid", "User", 30000))
        .doesNotThrowAnyException();
}
```

---

## 8. Tests Parametritzats

### 8.1 Paràmetres amb `@ValueSource`

```java
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.*;

@ParameterizedTest
@ValueSource(ints = {25000, 35000, 50000, 75000})
@DisplayName("Crear empleat amb diferents salaris")
void createEmployee_WithDifferentSalaries(int salary) {
    Employee emp = Manager.addEmployee("Test", "User", salary);
    
    assertThat(emp.getSalary()).isEqualTo(salary);
}
```

### 8.2 Paràmetres amb `@CsvSource`

```java
@ParameterizedTest
@CsvSource({
    "Joan, Garcia, 35000",
    "Marta, Ferrer, 42000",
    "Pere, Soler, 38000"
})
@DisplayName("Crear empleats amb dades CSV")
void createEmployee_WithCsvData(String firstName, String lastName, int salary) {
    Employee emp = Manager.addEmployee(firstName, lastName, salary);
    
    assertAll(
        () -> assertThat(emp.getFirstName()).isEqualTo(firstName),
        () -> assertThat(emp.getLastName()).isEqualTo(lastName),
        () -> assertThat(emp.getSalary()).isEqualTo(salary)
    );
}
```

### 8.3 Paràmetres amb `@MethodSource`

```java
@ParameterizedTest
@MethodSource("provideContactTypes")
@DisplayName("Afegir contactes de diferents tipus")
void addContact_WithDifferentTypes(String type, String value) {
    Employee emp = Manager.addEmployee("Test", "User", 30000);
    
    Contact contact = Manager.addContactToEmployee(
        emp.getEmployeeId(), type, value, "Description"
    );
    
    assertThat(contact.getContactType()).isEqualTo(type);
}

static Stream<Arguments> provideContactTypes() {
    return Stream.of(
        Arguments.of("EMAIL", "test@example.com"),
        Arguments.of("PHONE", "666123456"),
        Arguments.of("ADDRESS", "Carrer Test 1, Barcelona")
    );
}
```

### 8.4 Tests Repetits

```java
@RepeatedTest(value = 5, name = "Repetició {currentRepetition} de {totalRepetitions}")
@DisplayName("Verificar consistència d'IDs generats")
void verifyIdGeneration_IsConsistent(RepetitionInfo info) {
    Employee emp = Manager.addEmployee(
        "Test" + info.getCurrentRepetition(), 
        "User", 
        30000
    );
    
    assertThat(emp.getEmployeeId())
        .isNotNull()
        .isPositive();
}
```

---

## 9. Gestió de Transaccions en Tests

### 9.1 Test de Rollback Manual

```java
@Test
@DisplayName("Verificar que rollback desfà canvis")
void testRollback_ShouldUndoChanges() {
    try (Session session = Manager.getSessionFactory().openSession()) {
        Transaction tx = session.beginTransaction();
        
        // Crear empleat dins la transacció
        Employee emp = new Employee("Rollback", "Test", 30000);
        session.persist(emp);
        
        // Verificar que existeix dins la transacció
        assertNotNull(session.get(Employee.class, emp.getEmployeeId()));
        
        // Fer rollback
        tx.rollback();
    }
    
    // Verificar que NO existeix fora de la transacció
    Collection<Employee> employees = Manager.listCollection(Employee.class);
    assertThat(employees).isEmpty();
}
```

### 9.2 Test de LazyInitializationException

```java
@Test
@DisplayName("Accedir a col·lecció LAZY fora de sessió llança excepció")
void accessLazyCollection_OutsideSession_ShouldThrowException() {
    // Arrange - Crear dades
    Employee emp = Manager.addEmployee("Lazy", "Test", 30000);
    Manager.addContactToEmployee(emp.getEmployeeId(), "EMAIL", "test@test.com", "Test");
    
    // Act - Obtenir empleat amb col·lecció LAZY (si no inicialitzem)
    Employee detached;
    try (Session session = Manager.getSessionFactory().openSession()) {
        detached = session.get(Employee.class, emp.getEmployeeId());
        // NO inicialitzem la col·lecció aquí
    }
    
    // Assert - Accedir fora de sessió hauria de fallar
    // NOTA: Això depèn de la configuració. Si usem EAGER, no fallarà.
    // Aquest test documenta el comportament esperat.
    assertThatThrownBy(() -> detached.getContacts().size())
        .isInstanceOf(org.hibernate.LazyInitializationException.class);
}
```

---

## 10. Patrons i Antipatrons

### 10.1 ✅ Patrons Recomanats

| Patró | Descripció | Exemple |
|-------|------------|---------|
| **Arrange-Act-Assert** | Estructura clara del test | Veure exemples anteriors |
| **One Assert per Test** | Tests atòmics (excepte `assertAll`) | Tests individuals per cada comportament |
| **Meaningful Names** | Noms descriptius: `methodName_StateUnderTest_ExpectedBehavior` | `addEmployee_ValidData_ReturnsEntityWithId` |
| **Test Data Builders** | Crear dades de test fàcilment | Veure secció 10.2 |
| **BD en Memòria** | Velocitat i aïllament | H2 amb `create-drop` |

### 10.2 Test Data Builder Pattern

```java
package com.project.test.builders;

import com.project.domain.Employee;

public class EmployeeBuilder {
    private String firstName = "Default";
    private String lastName = "User";
    private int salary = 30000;
    
    public static EmployeeBuilder anEmployee() {
        return new EmployeeBuilder();
    }
    
    public EmployeeBuilder withFirstName(String firstName) {
        this.firstName = firstName;
        return this;
    }
    
    public EmployeeBuilder withLastName(String lastName) {
        this.lastName = lastName;
        return this;
    }
    
    public EmployeeBuilder withSalary(int salary) {
        this.salary = salary;
        return this;
    }
    
    public EmployeeBuilder withHighSalary() {
        this.salary = 100000;
        return this;
    }
    
    public Employee build() {
        return new Employee(firstName, lastName, salary);
    }
    
    // Per persistir directament
    public Employee buildAndSave() {
        return Manager.addEmployee(firstName, lastName, salary);
    }
}

// Ús en tests
@Test
void testWithBuilder() {
    Employee emp = EmployeeBuilder.anEmployee()
        .withFirstName("Custom")
        .withHighSalary()
        .buildAndSave();
    
    assertThat(emp.getSalary()).isEqualTo(100000);
}
```

### 10.3 ❌ Antipatrons a Evitar

```java
// ❌ MALAMENT: Test dependent d'ordre d'execució
@Test
@Order(1)
void createData() {
    Manager.addEmployee("Test", "User", 30000);
}

@Test
@Order(2)
void useDataFromPreviousTest() {  // ❌ Depèn del test anterior!
    Collection<Employee> emps = Manager.listCollection(Employee.class);
    assertEquals(1, emps.size());
}

// ✅ BENE: Cada test és independent
@Test
void testOne() {
    Employee emp = Manager.addEmployee("Test", "User", 30000);
    assertNotNull(emp);
}

@Test
void testTwo() {
    Employee emp = Manager.addEmployee("Another", "User", 40000);
    Collection<Employee> emps = Manager.listCollection(Employee.class);
    assertEquals(1, emps.size());  // Només veu el que ha creat
}
```

```java
// ❌ MALAMENT: Masses assertions sense relació
@Test
void testEverything() {
    Employee emp = Manager.addEmployee("Test", "User", 30000);
    assertNotNull(emp);
    assertEquals("Test", emp.getFirstName());
    
    Project p = Manager.addProject("P", "D", "ACTIU");  // ❌ Altra cosa!
    assertNotNull(p);
}

// ✅ BÉ: Tests focalizats
@Test
void addEmployee_ShouldSetFirstName() {
    Employee emp = Manager.addEmployee("Test", "User", 30000);
    assertEquals("Test", emp.getFirstName());
}

@Test
void addProject_ShouldReturnNotNull() {
    Project p = Manager.addProject("P", "D", "ACTIU");
    assertNotNull(p);
}
```

---

## Resum Final

| Aspecte | Recomanació |
|---------|-------------|
| **Base de dades** | H2 en memòria amb `create-drop` |
| **Aïllament** | Netejar BD abans de cada test (`@BeforeEach`) |
| **Estructura** | Arrange-Act-Assert amb noms descriptius |
| **Assertions** | AssertJ per llegibilitat, `assertAll` per agrupacions |
| **Cicle de vida** | `@BeforeAll` per SessionFactory, `@BeforeEach` per dades |
| **Cobertura** | Tests per CRUD, relacions, consultes HQL, i casos límit |
| **Velocitat** | Tests paral·lels quan sigui possible |
| **Mantenibilitat** | Builders per dades de test, classe base comuna |

---

> **Nota**: Aquest tutorial està adaptat al projecte `javahibernatejpamanytomany` amb entitats Employee, Contact i Project, i el DAO Manager amb mètodes estàtics.
