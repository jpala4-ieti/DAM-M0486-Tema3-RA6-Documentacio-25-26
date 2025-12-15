# Tutorial JUnit 5: Bones Pràctiques i Recomanacions

## Índex

1. [Introducció a JUnit 5](#1-introducció-a-junit-5)
2. [Configuració del Projecte](#2-configuració-del-projecte)
3. [Anatomia d'un Test](#3-anatomia-dun-test)
4. [Anotacions Principals](#4-anotacions-principals)
5. [Assertions](#5-assertions)
6. [Cicle de Vida dels Tests](#6-cicle-de-vida-dels-tests)
7. [Tests Parametritzats](#7-tests-parametritzats)
8. [Tests Niats i Agrupació](#8-tests-niats-i-agrupació)
9. [Mockito: Simulació de Dependències](#9-mockito-simulació-de-dependències)
10. [AssertJ: Assertions Fluides](#10-assertj-assertions-fluides)
11. [Bones Pràctiques](#11-bones-pràctiques)
12. [Patrons de Testing](#12-patrons-de-testing)

---

## 1. Introducció a JUnit 5

JUnit 5 és l'evolució del framework de testing més utilitzat a l'ecosistema Java. A diferència de JUnit 4, està compost per tres mòduls:

| Mòdul | Descripció |
|-------|------------|
| **JUnit Platform** | Fundació per llançar frameworks de testing sobre la JVM |
| **JUnit Jupiter** | Model de programació i extensió per escriure tests |
| **JUnit Vintage** | Compatibilitat amb JUnit 3 i 4 |

### Requisits

- Java 8 o superior (recomanat Java 11+)
- Maven o Gradle com a gestor de dependències

---

## 2. Configuració del Projecte

### Maven (`pom.xml`)

```xml
<dependencies>
    <!-- JUnit 5 -->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>5.10.2</version>
        <scope>test</scope>
    </dependency>
    
    <!-- Mockito -->
    <dependency>
        <groupId>org.mockito</groupId>
        <artifactId>mockito-junit-jupiter</artifactId>
        <version>5.11.0</version>
        <scope>test</scope>
    </dependency>
    
    <!-- AssertJ -->
    <dependency>
        <groupId>org.assertj</groupId>
        <artifactId>assertj-core</artifactId>
        <version>3.25.3</version>
        <scope>test</scope>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.2.5</version>
        </plugin>
    </plugins>
</build>
```

### Gradle (`build.gradle`)

```groovy
dependencies {
    testImplementation 'org.junit.jupiter:junit-jupiter:5.10.2'
    testImplementation 'org.mockito:mockito-junit-jupiter:5.11.0'
    testImplementation 'org.assertj:assertj-core:3.25.3'
}

test {
    useJUnitPlatform()
}
```

### Estructura de Directoris

```
projecte/
├── src/
│   ├── main/
│   │   └── java/
│   │       └── com/exemple/
│   │           └── Calculadora.java
│   └── test/
│       └── java/
│           └── com/exemple/
│               └── CalculadoraTest.java
└── pom.xml
```

---

## 3. Anatomia d'un Test

### Classe a Testar

```java
package com.exemple;

public class Calculadora {
    
    public int sumar(int a, int b) {
        return a + b;
    }
    
    public int dividir(int dividend, int divisor) {
        if (divisor == 0) {
            throw new ArithmeticException("No es pot dividir per zero");
        }
        return dividend / divisor;
    }
}
```

### Test Bàsic

```java
package com.exemple;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.DisplayName;
import static org.junit.jupiter.api.Assertions.*;

class CalculadoraTest {

    @Test
    @DisplayName("Suma de dos nombres positius retorna el resultat correcte")
    void sumarDosNombresPositius() {
        // Arrange (Preparació)
        Calculadora calc = new Calculadora();
        
        // Act (Execució)
        int resultat = calc.sumar(2, 3);
        
        // Assert (Verificació)
        assertEquals(5, resultat);
    }
}
```

### Convenció de Nomenclatura

```java
// Opció 1: metodeBaixTest_escenari_resultatEsperat
void dividir_perZero_llancaExcepcio()

// Opció 2: should_resultat_when_condicio
void should_throwException_when_dividingByZero()

// Opció 3: given_when_then
void givenDivisorZero_whenDividir_thenThrowsException()
```

---

## 4. Anotacions Principals

### Anotacions de Test

| Anotació | Descripció |
|----------|------------|
| `@Test` | Marca un mètode com a test |
| `@DisplayName` | Nom descriptiu per al test |
| `@Disabled` | Desactiva temporalment un test |
| `@Tag` | Etiqueta tests per filtrar-los |
| `@Timeout` | Límit de temps d'execució |

### Exemples d'Ús

```java
import org.junit.jupiter.api.*;
import java.time.Duration;
import static org.junit.jupiter.api.Assertions.*;

class AnotacionsTest {

    @Test
    @DisplayName("Test amb nom llegible als informes")
    void testAmbDisplayName() {
        assertTrue(true);
    }

    @Test
    @Disabled("Pendent d'implementar la funcionalitat X")
    void testDesactivat() {
        fail("Aquest test no s'executarà");
    }

    @Test
    @Tag("integracio")
    void testEtiquetat() {
        // Executa amb: mvn test -Dgroups=integracio
        assertTrue(true);
    }

    @Test
    @Timeout(value = 500, unit = TimeUnit.MILLISECONDS)
    void testAmbTimeout() {
        // Falla si triga més de 500ms
        assertTrue(true);
    }
    
    @Test
    void testAmbAssertTimeout() {
        assertTimeout(Duration.ofMillis(100), () -> {
            // Codi que ha d'executar-se en menys de 100ms
            Thread.sleep(50);
        });
    }
}
```

---

## 5. Assertions

### Assertions Bàsiques de JUnit

```java
import static org.junit.jupiter.api.Assertions.*;

class AssertionsTest {

    @Test
    void assertionsBasiques() {
        // Igualtat
        assertEquals(4, 2 + 2);
        assertEquals(3.14, Math.PI, 0.01); // amb delta per doubles
        
        // Booleans
        assertTrue(5 > 3);
        assertFalse(5 < 3);
        
        // Null checks
        assertNull(null);
        assertNotNull(new Object());
        
        // Referències
        String a = "test";
        String b = a;
        assertSame(a, b);      // mateixa referència
        assertNotSame(a, new String("test")); // diferent referència
        
        // Arrays
        assertArrayEquals(
            new int[]{1, 2, 3}, 
            new int[]{1, 2, 3}
        );
    }

    @Test
    void assertionsAmbMissatge() {
        int edat = 15;
        assertTrue(
            edat >= 18, 
            () -> "L'edat " + edat + " no és major d'edat"
        );
    }

    @Test
    void assertionsAgrupades() {
        Persona p = new Persona("Joan", 25);
        
        // Executa TOTES les assertions i reporta tots els errors
        assertAll("Verificació de Persona",
            () -> assertEquals("Joan", p.getNom()),
            () -> assertEquals(25, p.getEdat()),
            () -> assertNotNull(p.getNom())
        );
    }

    @Test
    void assertionsExcepcions() {
        Calculadora calc = new Calculadora();
        
        // Verifica que es llança l'excepció
        ArithmeticException ex = assertThrows(
            ArithmeticException.class,
            () -> calc.dividir(10, 0)
        );
        
        // Verifica el missatge
        assertEquals("No es pot dividir per zero", ex.getMessage());
    }
    
    @Test
    void assertionsNoLlancaExcepcio() {
        assertDoesNotThrow(() -> {
            new Calculadora().sumar(1, 2);
        });
    }
}
```

---

## 6. Cicle de Vida dels Tests

### Anotacions de Cicle de Vida

```java
import org.junit.jupiter.api.*;

class CicleVidaTest {

    private Calculadora calc;
    
    @BeforeAll
    static void initGlobal() {
        // Executa UNA VEGADA abans de TOTS els tests
        // Ha de ser static (per defecte)
        System.out.println("Inicialització global");
    }

    @BeforeEach
    void setUp() {
        // Executa ABANS de CADA test
        calc = new Calculadora();
        System.out.println("Preparant test...");
    }

    @AfterEach
    void tearDown() {
        // Executa DESPRÉS de CADA test
        calc = null;
        System.out.println("Netejant després del test...");
    }

    @AfterAll
    static void cleanUpGlobal() {
        // Executa UNA VEGADA després de TOTS els tests
        System.out.println("Neteja global");
    }

    @Test
    void test1() {
        assertEquals(5, calc.sumar(2, 3));
    }

    @Test
    void test2() {
        assertEquals(7, calc.sumar(3, 4));
    }
}
```

### Cicle de Vida per Instància

```java
// Per defecte: una instància nova per cada test (PER_METHOD)
// Alternativa: una instància compartida (PER_CLASS)

@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class TestAmbInstanciaCompartida {

    private int comptador = 0;

    @BeforeAll
    void init() {
        // Ara NO cal que sigui static
        System.out.println("Init");
    }

    @Test
    void test1() {
        comptador++;
        assertEquals(1, comptador);
    }

    @Test
    void test2() {
        comptador++;
        // COMPTE: l'ordre d'execució no està garantit!
        assertTrue(comptador >= 1);
    }
}
```

---

## 7. Tests Parametritzats

### Dependència Addicional (si cal)

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-params</artifactId>
    <version>5.10.2</version>
    <scope>test</scope>
</dependency>
```

### Fonts de Paràmetres

```java
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.*;

class TestsParametritzats {

    // @ValueSource: valors simples
    @ParameterizedTest
    @ValueSource(ints = {1, 2, 3, 4, 5})
    void nombrePositiu(int nombre) {
        assertTrue(nombre > 0);
    }

    @ParameterizedTest
    @ValueSource(strings = {"hello", "world", "test"})
    void stringNoEsBuit(String text) {
        assertFalse(text.isEmpty());
    }

    // @NullSource, @EmptySource, @NullAndEmptySource
    @ParameterizedTest
    @NullAndEmptySource
    @ValueSource(strings = {"  ", "\t", "\n"})
    void stringsBuitsONuls(String text) {
        assertTrue(text == null || text.trim().isEmpty());
    }

    // @EnumSource
    @ParameterizedTest
    @EnumSource(DiaSetmana.class)
    void totsElsDies(DiaSetmana dia) {
        assertNotNull(dia);
    }

    @ParameterizedTest
    @EnumSource(value = DiaSetmana.class, names = {"DISSABTE", "DIUMENGE"})
    void nomesCapSetmana(DiaSetmana dia) {
        assertTrue(dia == DiaSetmana.DISSABTE || dia == DiaSetmana.DIUMENGE);
    }

    // @CsvSource: múltiples paràmetres
    @ParameterizedTest
    @CsvSource({
        "1, 2, 3",
        "0, 0, 0",
        "-1, 1, 0",
        "10, -5, 5"
    })
    void testSuma(int a, int b, int resultatEsperat) {
        Calculadora calc = new Calculadora();
        assertEquals(resultatEsperat, calc.sumar(a, b));
    }

    // @CsvFileSource: dades des d'un fitxer CSV
    @ParameterizedTest
    @CsvFileSource(resources = "/test-data.csv", numLinesToSkip = 1)
    void testDesFitxer(int a, int b, int resultat) {
        assertEquals(resultat, new Calculadora().sumar(a, b));
    }

    // @MethodSource: mètode que proporciona arguments
    @ParameterizedTest
    @MethodSource("proveeixArguments")
    void testAmbMethodSource(int a, int b, int resultatEsperat) {
        assertEquals(resultatEsperat, new Calculadora().sumar(a, b));
    }

    static Stream<Arguments> proveeixArguments() {
        return Stream.of(
            Arguments.of(1, 1, 2),
            Arguments.of(2, 3, 5),
            Arguments.of(-1, -1, -2)
        );
    }

    // @ArgumentsSource: proveïdor personalitzat
    @ParameterizedTest
    @ArgumentsSource(CustomArgumentsProvider.class)
    void testAmbProveidorCustom(int valor) {
        assertTrue(valor > 0);
    }
}

// Proveïdor personalitzat
class CustomArgumentsProvider implements ArgumentsProvider {
    @Override
    public Stream<? extends Arguments> provideArguments(ExtensionContext context) {
        return Stream.of(1, 2, 3, 4, 5).map(Arguments::of);
    }
}

enum DiaSetmana {
    DILLUNS, DIMARTS, DIMECRES, DIJOUS, DIVENDRES, DISSABTE, DIUMENGE
}
```

---

## 8. Tests Niats i Agrupació

### @Nested per Organitzar Tests

```java
import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;

@DisplayName("Tests de Calculadora")
class CalculadoraNiatsTest {

    private Calculadora calc;

    @BeforeEach
    void setUp() {
        calc = new Calculadora();
    }

    @Nested
    @DisplayName("Operacions de Suma")
    class SumaTests {

        @Test
        @DisplayName("suma nombres positius")
        void sumaPositius() {
            assertEquals(5, calc.sumar(2, 3));
        }

        @Test
        @DisplayName("suma amb zero")
        void sumaAmbZero() {
            assertEquals(5, calc.sumar(5, 0));
        }

        @Test
        @DisplayName("suma nombres negatius")
        void sumaNegatius() {
            assertEquals(-5, calc.sumar(-2, -3));
        }
    }

    @Nested
    @DisplayName("Operacions de Divisió")
    class DivisioTests {

        @Test
        @DisplayName("divisió exacta")
        void divisioExacta() {
            assertEquals(5, calc.dividir(10, 2));
        }

        @Nested
        @DisplayName("Casos d'Error")
        class ErrorTests {

            @Test
            @DisplayName("divisió per zero llança excepció")
            void divisioPerZero() {
                assertThrows(
                    ArithmeticException.class,
                    () -> calc.dividir(10, 0)
                );
            }
        }
    }
}
```

### @TestMethodOrder per Controlar l'Ordre

```java
import org.junit.jupiter.api.*;
import org.junit.jupiter.api.MethodOrderer.*;

@TestMethodOrder(OrderAnnotation.class)
class TestsOrdenats {

    @Test
    @Order(1)
    void primerTest() {
        System.out.println("Primer");
    }

    @Test
    @Order(2)
    void segonTest() {
        System.out.println("Segon");
    }

    @Test
    @Order(3)
    void tercerTest() {
        System.out.println("Tercer");
    }
}

// Altres ordres disponibles:
// @TestMethodOrder(MethodName.class) - per nom del mètode
// @TestMethodOrder(Random.class) - aleatori
// @TestMethodOrder(DisplayName.class) - per DisplayName
```

---

## 9. Mockito: Simulació de Dependències

### Conceptes Clau

- **Mock**: Objecte simulat que substitueix una dependència real
- **Stub**: Definició del comportament d'un mock
- **Verify**: Comprovació que s'han cridat certs mètodes

### Exemple Pràctic

```java
// Classe a testar
public class ServeiComandes {
    
    private final RepositoriComandes repositori;
    private final ServeiNotificacions notificacions;
    
    public ServeiComandes(RepositoriComandes repositori, 
                          ServeiNotificacions notificacions) {
        this.repositori = repositori;
        this.notificacions = notificacions;
    }
    
    public Comanda crearComanda(String producte, int quantitat) {
        Comanda comanda = new Comanda(producte, quantitat);
        repositori.guardar(comanda);
        notificacions.enviarConfirmacio(comanda);
        return comanda;
    }
    
    public Comanda obtenirComanda(Long id) {
        return repositori.trobarPerId(id)
            .orElseThrow(() -> new ComandaNoTrobadaException(id));
    }
}
```

### Tests amb Mockito

```java
import org.junit.jupiter.api.*;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.*;
import org.mockito.junit.jupiter.MockitoExtension;
import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;

@ExtendWith(MockitoExtension.class)
class ServeiComandesTest {

    @Mock
    private RepositoriComandes repositori;

    @Mock
    private ServeiNotificacions notificacions;

    @InjectMocks
    private ServeiComandes servei;

    @Captor
    private ArgumentCaptor<Comanda> comandaCaptor;

    @Test
    @DisplayName("Crear comanda guarda i notifica")
    void crearComanda_guardaINotifica() {
        // Act
        Comanda resultat = servei.crearComanda("Laptop", 2);

        // Assert
        assertNotNull(resultat);
        assertEquals("Laptop", resultat.getProducte());
        
        // Verify: comprova que s'han cridat els mètodes
        verify(repositori).guardar(any(Comanda.class));
        verify(notificacions).enviarConfirmacio(any(Comanda.class));
    }

    @Test
    @DisplayName("Obtenir comanda existent")
    void obtenirComanda_quanExisteix_retornaComanda() {
        // Arrange: definim el comportament del mock
        Comanda comandaEsperada = new Comanda("Teclat", 1);
        when(repositori.trobarPerId(1L))
            .thenReturn(Optional.of(comandaEsperada));

        // Act
        Comanda resultat = servei.obtenirComanda(1L);

        // Assert
        assertEquals(comandaEsperada, resultat);
        verify(repositori).trobarPerId(1L);
    }

    @Test
    @DisplayName("Obtenir comanda inexistent llança excepció")
    void obtenirComanda_quanNoExisteix_llancaExcepcio() {
        // Arrange
        when(repositori.trobarPerId(anyLong()))
            .thenReturn(Optional.empty());

        // Act & Assert
        assertThrows(
            ComandaNoTrobadaException.class,
            () -> servei.obtenirComanda(999L)
        );
    }

    @Test
    @DisplayName("Captura arguments passats al mock")
    void crearComanda_capturaArguments() {
        // Act
        servei.crearComanda("Monitor", 3);

        // Captura l'argument passat
        verify(repositori).guardar(comandaCaptor.capture());
        Comanda comandaCapturada = comandaCaptor.getValue();

        assertEquals("Monitor", comandaCapturada.getProducte());
        assertEquals(3, comandaCapturada.getQuantitat());
    }
}
```

### Stubbing Avançat

```java
@Test
void stubbingAvancat() {
    // Retornar valors diferents en crides successives
    when(mock.metode())
        .thenReturn("primera")
        .thenReturn("segona")
        .thenReturn("tercera");

    // Llançar excepcions
    when(mock.metodePerillos())
        .thenThrow(new RuntimeException("Error!"));

    // Resposta dinàmica basada en l'argument
    when(repositori.trobarPerId(anyLong()))
        .thenAnswer(invocation -> {
            Long id = invocation.getArgument(0);
            return Optional.of(new Comanda("Producte-" + id, 1));
        });

    // doReturn per mètodes void o spies
    doNothing().when(notificacions).enviarConfirmacio(any());
    doThrow(new RuntimeException()).when(mock).metodeVoid();
}
```

### Spy: Mock Parcial

```java
@Test
void spyExemple() {
    // Spy: objecte real amb capacitat de stubbing
    List<String> llistaReal = new ArrayList<>();
    List<String> spy = spy(llistaReal);

    // Mètodes reals s'executen per defecte
    spy.add("element");
    assertEquals(1, spy.size());

    // Podem fer stub de mètodes específics
    doReturn(100).when(spy).size();
    assertEquals(100, spy.size());
}
```

---

## 10. AssertJ: Assertions Fluides

AssertJ ofereix una API fluida i llegible per escriure assertions.

### Assertions Bàsiques

```java
import static org.assertj.core.api.Assertions.*;

class AssertJTest {

    @Test
    void assertionsStrings() {
        String text = "Hola, món!";

        assertThat(text)
            .isNotNull()
            .isNotEmpty()
            .startsWith("Hola")
            .endsWith("!")
            .contains("món")
            .hasSize(10)
            .matches("Hola.*!");
    }

    @Test
    void assertionsNombres() {
        int valor = 42;

        assertThat(valor)
            .isPositive()
            .isGreaterThan(40)
            .isLessThanOrEqualTo(42)
            .isBetween(40, 45);
    }

    @Test
    void assertionsColleccions() {
        List<String> fruites = List.of("poma", "pera", "taronja");

        assertThat(fruites)
            .hasSize(3)
            .contains("poma", "pera")
            .doesNotContain("plàtan")
            .containsExactly("poma", "pera", "taronja")
            .containsExactlyInAnyOrder("taronja", "poma", "pera")
            .allMatch(f -> f.length() > 3)
            .anyMatch(f -> f.startsWith("p"))
            .noneMatch(f -> f.isEmpty());
    }

    @Test
    void assertionsObjectes() {
        Persona persona = new Persona("Joan", 30);

        assertThat(persona)
            .isNotNull()
            .hasFieldOrProperty("nom")
            .hasFieldOrPropertyWithValue("nom", "Joan")
            .extracting(Persona::getEdat)
            .isEqualTo(30);
    }

    @Test
    void assertionsExcepcions() {
        assertThatThrownBy(() -> {
            throw new IllegalArgumentException("Argument invàlid");
        })
            .isInstanceOf(IllegalArgumentException.class)
            .hasMessage("Argument invàlid")
            .hasMessageContaining("invàlid");

        // Alternativa més explícita
        assertThatExceptionOfType(ArithmeticException.class)
            .isThrownBy(() -> new Calculadora().dividir(1, 0))
            .withMessageContaining("zero");
    }

    @Test
    void assertionsSoftAssertions() {
        // Executa totes les assertions i reporta tots els errors junts
        SoftAssertions soft = new SoftAssertions();
        
        Persona p = new Persona("Joan", 25);
        
        soft.assertThat(p.getNom()).isEqualTo("Joan");
        soft.assertThat(p.getEdat()).isGreaterThan(18);
        soft.assertThat(p.getEdat()).isLessThan(30);
        
        soft.assertAll(); // Llança errors acumulats
    }
}
```

### Comparació d'Objectes Complexos

```java
@Test
void comparacioObjectes() {
    Persona p1 = new Persona("Joan", 30);
    Persona p2 = new Persona("Joan", 30);

    // Comparació per camps (ignorant equals)
    assertThat(p1)
        .usingRecursiveComparison()
        .isEqualTo(p2);

    // Ignorant certs camps
    assertThat(p1)
        .usingRecursiveComparison()
        .ignoringFields("id", "dataCreacio")
        .isEqualTo(p2);
}

@Test
void extraccioDeCamps() {
    List<Persona> persones = List.of(
        new Persona("Joan", 30),
        new Persona("Maria", 25),
        new Persona("Pere", 35)
    );

    // Extracció de propietats
    assertThat(persones)
        .extracting(Persona::getNom)
        .containsExactly("Joan", "Maria", "Pere");

    // Extracció múltiple
    assertThat(persones)
        .extracting(Persona::getNom, Persona::getEdat)
        .containsExactly(
            tuple("Joan", 30),
            tuple("Maria", 25),
            tuple("Pere", 35)
        );

    // Filtrat
    assertThat(persones)
        .filteredOn(p -> p.getEdat() > 28)
        .hasSize(2)
        .extracting(Persona::getNom)
        .containsOnly("Joan", "Pere");
}
```

---

## 11. Bones Pràctiques

### 1. Estructura AAA (Arrange-Act-Assert)

```java
@Test
void exempleBenEstructurat() {
    // Arrange: prepara les dades i dependències
    Calculadora calc = new Calculadora();
    int a = 5;
    int b = 3;

    // Act: executa l'acció a testar
    int resultat = calc.sumar(a, b);

    // Assert: verifica el resultat
    assertEquals(8, resultat);
}
```

### 2. Un Concepte per Test

```java
// MALAMENT: múltiples conceptes
@Test
void testCalculadora() {
    Calculadora calc = new Calculadora();
    assertEquals(5, calc.sumar(2, 3));
    assertEquals(2, calc.restar(5, 3));
    assertEquals(6, calc.multiplicar(2, 3));
}

// BÉ: un concepte per test
@Test
void sumar_dosPositius_retornaSuma() {
    assertEquals(5, new Calculadora().sumar(2, 3));
}

@Test
void restar_dosPositius_retornaDiferencia() {
    assertEquals(2, new Calculadora().restar(5, 3));
}
```

### 3. Tests Independents i Aïllats

```java
// MALAMENT: tests que depenen d'altres
class TestsDependents {
    static List<String> llistaCompartida = new ArrayList<>();

    @Test
    void test1_afegirElement() {
        llistaCompartida.add("A");
        assertEquals(1, llistaCompartida.size());
    }

    @Test
    void test2_comprovarElement() {
        // PERILL: depèn que test1 s'executi primer!
        assertTrue(llistaCompartida.contains("A"));
    }
}

// BÉ: tests independents
class TestsIndependents {
    private List<String> llista;

    @BeforeEach
    void setUp() {
        llista = new ArrayList<>();
    }

    @Test
    void afegirElement_incrementaMida() {
        llista.add("A");
        assertEquals(1, llista.size());
    }

    @Test
    void conteElement_retornaTrue() {
        llista.add("A");
        assertTrue(llista.contains("A"));
    }
}
```

### 4. Noms Descriptius

```java
// MALAMENT
@Test
void test1() { }

@Test  
void testDividir() { }

// BÉ
@Test
@DisplayName("Dividir per zero llança ArithmeticException")
void dividir_perZero_llancaArithmeticException() { }

@Test
@DisplayName("Usuari amb email vàlid es registra correctament")
void registrarUsuari_emailValid_usuariCreat() { }
```

### 5. Evitar Lògica en Tests

```java
// MALAMENT: lògica complexa en el test
@Test
void testAmbLogica() {
    List<Integer> nombres = List.of(1, 2, 3, 4, 5);
    int suma = 0;
    for (int n : nombres) {
        if (n % 2 == 0) {
            suma += n;
        }
    }
    assertEquals(6, suma);
}

// BÉ: dades i resultat explícits
@Test
void sumarParells_retornaSumaCorrecta() {
    List<Integer> nombres = List.of(1, 2, 3, 4, 5);
    
    int resultat = calculador.sumarParells(nombres);
    
    assertEquals(6, resultat); // 2 + 4 = 6
}
```

### 6. Usar Constants per Valors Màgics

```java
// MALAMENT
@Test
void validarEdat() {
    assertFalse(validador.esMajorEdat(17));
    assertTrue(validador.esMajorEdat(18));
}

// BÉ
private static final int EDAT_MINIMA_ADULT = 18;

@Test
void validarEdat_menorDeMinima_retornaFalse() {
    assertFalse(validador.esMajorEdat(EDAT_MINIMA_ADULT - 1));
}

@Test
void validarEdat_igualAMinima_retornaTrue() {
    assertTrue(validador.esMajorEdat(EDAT_MINIMA_ADULT));
}
```

### 7. Test Builders i Object Mothers

```java
// Object Mother: fàbrica d'objectes per tests
class PersonaMother {
    
    public static Persona unAdult() {
        return new Persona("Joan Adult", 30, "joan@email.com");
    }
    
    public static Persona unMenor() {
        return new Persona("Petit", 10, "petit@email.com");
    }
    
    public static Persona senseEmail() {
        return new Persona("Sense Email", 25, null);
    }
}

// Builder per tests
class PersonaTestBuilder {
    private String nom = "Nom per defecte";
    private int edat = 25;
    private String email = "default@email.com";

    public static PersonaTestBuilder unaPersona() {
        return new PersonaTestBuilder();
    }

    public PersonaTestBuilder ambNom(String nom) {
        this.nom = nom;
        return this;
    }

    public PersonaTestBuilder ambEdat(int edat) {
        this.edat = edat;
        return this;
    }

    public PersonaTestBuilder menor() {
        this.edat = 15;
        return this;
    }

    public Persona build() {
        return new Persona(nom, edat, email);
    }
}

// Ús en tests
@Test
void exempleAmbBuilder() {
    Persona menor = PersonaTestBuilder.unaPersona()
        .ambNom("Joan")
        .menor()
        .build();

    assertFalse(servei.potVotar(menor));
}
```

---

## 12. Patrons de Testing

### Patró Given-When-Then (BDD)

```java
import org.junit.jupiter.api.*;

class CompteBancariTest {

    @Nested
    @DisplayName("Given un compte amb saldo inicial de 100€")
    class CompteAmbSaldo {
        
        private CompteBancari compte;

        @BeforeEach
        void setUp() {
            compte = new CompteBancari(100);
        }

        @Nested
        @DisplayName("When es retiren 50€")
        class QuanRetirar50 {

            @BeforeEach
            void retirar() {
                compte.retirar(50);
            }

            @Test
            @DisplayName("Then el saldo és 50€")
            void saldoEs50() {
                assertEquals(50, compte.getSaldo());
            }
        }

        @Nested
        @DisplayName("When es retiren 150€")
        class QuanRetirar150 {

            @Test
            @DisplayName("Then es llança SaldoInsuficientException")
            void llancaExcepcio() {
                assertThrows(
                    SaldoInsuficientException.class,
                    () -> compte.retirar(150)
                );
            }
        }
    }
}
```

### Patró de Test de Contracte

```java
// Test abstracte per verificar contracte d'una interfície
abstract class RepositoriContracteTest<T extends Repositori<Entitat>> {

    protected abstract T crearRepositori();
    protected abstract Entitat crearEntitat();

    private T repositori;

    @BeforeEach
    void setUp() {
        repositori = crearRepositori();
    }

    @Test
    void guardar_iRecuperar_retornaMateixaEntitat() {
        Entitat entitat = crearEntitat();
        
        repositori.guardar(entitat);
        Optional<Entitat> recuperada = repositori.trobarPerId(entitat.getId());
        
        assertTrue(recuperada.isPresent());
        assertEquals(entitat, recuperada.get());
    }

    @Test
    void eliminar_entityNoExisteix() {
        Optional<Entitat> resultat = repositori.trobarPerId(999L);
        assertTrue(resultat.isEmpty());
    }
}

// Implementació concreta del test
class RepositoriMemoriaTest extends RepositoriContracteTest<RepositoriMemoria> {

    @Override
    protected RepositoriMemoria crearRepositori() {
        return new RepositoriMemoria();
    }

    @Override
    protected Entitat crearEntitat() {
        return new Entitat(1L, "Test");
    }
}
```

### Extensions Personalitzades

```java
// Extensió per mesurar temps d'execució
public class TimingExtension implements BeforeTestExecutionCallback, 
                                        AfterTestExecutionCallback {

    private static final String START_TIME = "start_time";

    @Override
    public void beforeTestExecution(ExtensionContext context) {
        getStore(context).put(START_TIME, System.currentTimeMillis());
    }

    @Override
    public void afterTestExecution(ExtensionContext context) {
        long startTime = getStore(context).remove(START_TIME, long.class);
        long duration = System.currentTimeMillis() - startTime;
        System.out.println(
            context.getDisplayName() + " va trigar " + duration + "ms"
        );
    }

    private ExtensionContext.Store getStore(ExtensionContext context) {
        return context.getStore(
            ExtensionContext.Namespace.create(getClass(), context.getRequiredTestMethod())
        );
    }
}

// Ús
@ExtendWith(TimingExtension.class)
class TestsAmbTiming {
    @Test
    void testLent() throws InterruptedException {
        Thread.sleep(100);
    }
}
```

---

## Resum Ràpid

| Categoria | Recomanació |
|-----------|-------------|
| **Estructura** | Usar AAA (Arrange-Act-Assert) |
| **Nomenclatura** | Noms descriptius que indiquin escenari i resultat |
| **Independència** | Tests aïllats, sense dependencies entre ells |
| **Focalització** | Un concepte/comportament per test |
| **Mocking** | Mockito per simular dependències externes |
| **Assertions** | AssertJ per assertions llegibles i fluides |
| **Organització** | @Nested per agrupar tests relacionats |
| **Parametrització** | @ParameterizedTest per evitar duplicació |
| **Mantenibilitat** | Builders i Object Mothers per crear dades de test |

---

## Referències

- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [Mockito Documentation](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html)
- [AssertJ Documentation](https://assertj.github.io/doc/)
- [Effective Unit Testing - Lasse Koskela](https://www.manning.com/books/effective-unit-testing)
