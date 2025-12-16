# Principis SOLID

Guia pràctica amb exemples en Java, JavaScript i Python

---

## Introducció

Els principis SOLID són cinc principis de disseny orientat a objectes que ajuden a crear programari més mantenible, flexible i comprensible. Van ser popularitzats per Robert C. Martin (Uncle Bob) i són fonamentals per a qualsevol desenvolupador.

| Principi | Nom complet | Resum |
|----------|-------------|-------|
| **S** | Single Responsibility | Una classe, una responsabilitat |
| **O** | Open/Closed | Obert a extensió, tancat a modificació |
| **L** | Liskov Substitution | Les subclasses han de ser substituïbles |
| **I** | Interface Segregation | Interfícies petites i específiques |
| **D** | Dependency Inversion | Depèn d'abstraccions, no de concrecions |

---

## S - Single Responsibility Principle (SRP)

> *"Una classe ha de tenir una única raó per canviar."*

Cada classe o mòdul ha de tenir una única responsabilitat ben definida. Si una classe fa massa coses, qualsevol canvi en una d'elles pot afectar les altres.

### Java

```java
// ❌ MALAMENT: Una classe amb múltiples responsabilitats
public class Employee {
    private String name;
    private double salary;
    
    public void calculatePay() {
        // Calcula el salari - lògica de comptabilitat
    }
    
    public void saveToDatabase() {
        // Guarda a la BD - lògica de persistència
    }
    
    public String generateReport() {
        // Genera informe - lògica de presentació
    }
    
    public void sendEmail() {
        // Envia correu - lògica de comunicació
    }
}

// ✅ BÉ: Cada classe té una única responsabilitat
public class Employee {
    private String name;
    private double salary;
    
    // Només getters i setters, representa l'entitat
    public String getName() { return name; }
    public double getSalary() { return salary; }
}

public class PayrollCalculator {
    public double calculatePay(Employee employee) {
        // Lògica de càlcul de nòmina
        return employee.getSalary() * 1.0;
    }
}

public class EmployeeRepository {
    public void save(Employee employee) {
        // Lògica de persistència
    }
    
    public Employee findById(Long id) {
        // Cerca a la BD
        return null;
    }
}

public class EmployeeReportGenerator {
    public String generate(Employee employee) {
        // Genera informe en format específic
        return "Report for: " + employee.getName();
    }
}

public class EmailService {
    public void sendEmail(String to, String subject, String body) {
        // Lògica d'enviament de correus
    }
}
```

### Python

```python
# ❌ MALAMENT: Una classe amb múltiples responsabilitats
class Employee:
    def __init__(self, name: str, salary: float):
        self.name = name
        self.salary = salary
    
    def calculate_pay(self) -> float:
        # Calcula el salari - lògica de comptabilitat
        return self.salary * 1.0
    
    def save_to_database(self):
        # Guarda a la BD - lògica de persistència
        pass
    
    def generate_report(self) -> str:
        # Genera informe - lògica de presentació
        return f"Report for {self.name}"
    
    def send_email(self, message: str):
        # Envia correu - lògica de comunicació
        pass


# ✅ BÉ: Cada classe té una única responsabilitat
from dataclasses import dataclass
from typing import Optional

@dataclass
class Employee:
    """Representa l'entitat empleat - només dades"""
    name: str
    salary: float
    id: Optional[int] = None


class PayrollCalculator:
    """Responsable dels càlculs de nòmina"""
    
    def calculate_pay(self, employee: Employee) -> float:
        return employee.salary * 1.0
    
    def calculate_bonus(self, employee: Employee, percentage: float) -> float:
        return employee.salary * percentage


class EmployeeRepository:
    """Responsable de la persistència d'empleats"""
    
    def __init__(self, db_connection):
        self.db = db_connection
    
    def save(self, employee: Employee) -> Employee:
        # Lògica de persistència
        pass
    
    def find_by_id(self, employee_id: int) -> Optional[Employee]:
        # Cerca a la BD
        pass
    
    def find_all(self) -> list[Employee]:
        pass


class EmployeeReportGenerator:
    """Responsable de generar informes"""
    
    def generate_pdf(self, employee: Employee) -> bytes:
        pass
    
    def generate_csv(self, employees: list[Employee]) -> str:
        pass


class EmailService:
    """Responsable de l'enviament de correus"""
    
    def __init__(self, smtp_config: dict):
        self.config = smtp_config
    
    def send(self, to: str, subject: str, body: str):
        pass
```

### JavaScript

```javascript
// ❌ MALAMENT: Una classe amb múltiples responsabilitats
class Employee {
    constructor(name, salary) {
        this.name = name;
        this.salary = salary;
    }
    
    calculatePay() {
        // Calcula el salari - lògica de comptabilitat
        return this.salary * 1.0;
    }
    
    saveToDatabase() {
        // Guarda a la BD - lògica de persistència
    }
    
    generateReport() {
        // Genera informe - lògica de presentació
        return `Report for ${this.name}`;
    }
    
    sendEmail(message) {
        // Envia correu - lògica de comunicació
    }
}


// ✅ BÉ: Cada classe té una única responsabilitat
class Employee {
    constructor(name, salary) {
        this.name = name;
        this.salary = salary;
        this.id = null;
    }
}

class PayrollCalculator {
    calculatePay(employee) {
        return employee.salary * 1.0;
    }
    
    calculateBonus(employee, percentage) {
        return employee.salary * percentage;
    }
}

class EmployeeRepository {
    constructor(database) {
        this.db = database;
    }
    
    async save(employee) {
        // Lògica de persistència
        return await this.db.insert('employees', employee);
    }
    
    async findById(id) {
        return await this.db.findOne('employees', { id });
    }
    
    async findAll() {
        return await this.db.find('employees');
    }
}

class EmployeeReportGenerator {
    generatePdf(employee) {
        // Genera PDF
    }
    
    generateCsv(employees) {
        // Genera CSV
    }
}

class EmailService {
    constructor(smtpConfig) {
        this.config = smtpConfig;
    }
    
    async send(to, subject, body) {
        // Lògica d'enviament
    }
}
```

---

## O - Open/Closed Principle (OCP)

> *"Les entitats de programari han d'estar obertes a l'extensió però tancades a la modificació."*

Hauries de poder afegir noves funcionalitats sense modificar el codi existent. Això s'aconsegueix mitjançant abstraccions i polimorfisme.

### Java

```java
// ❌ MALAMENT: Cal modificar la classe per afegir nous tipus
public class DiscountCalculator {
    public double calculateDiscount(Order order) {
        if (order.getCustomerType().equals("regular")) {
            return order.getTotal() * 0.1;
        } else if (order.getCustomerType().equals("premium")) {
            return order.getTotal() * 0.2;
        } else if (order.getCustomerType().equals("vip")) {
            return order.getTotal() * 0.3;
        }
        // Cada nou tipus requereix modificar aquest codi!
        return 0;
    }
}


// ✅ BÉ: Utilitza abstraccions per permetre extensió
public interface DiscountStrategy {
    double calculate(Order order);
}

public class RegularDiscount implements DiscountStrategy {
    @Override
    public double calculate(Order order) {
        return order.getTotal() * 0.1;
    }
}

public class PremiumDiscount implements DiscountStrategy {
    @Override
    public double calculate(Order order) {
        return order.getTotal() * 0.2;
    }
}

public class VipDiscount implements DiscountStrategy {
    @Override
    public double calculate(Order order) {
        return order.getTotal() * 0.3;
    }
}

// Afegir un nou tipus de descompte és tan fàcil com crear una nova classe
public class BlackFridayDiscount implements DiscountStrategy {
    @Override
    public double calculate(Order order) {
        return order.getTotal() * 0.5;
    }
}

// La classe principal no necessita canvis
public class DiscountCalculator {
    private final DiscountStrategy strategy;
    
    public DiscountCalculator(DiscountStrategy strategy) {
        this.strategy = strategy;
    }
    
    public double calculateDiscount(Order order) {
        return strategy.calculate(order);
    }
}

// Ús
DiscountCalculator calculator = new DiscountCalculator(new VipDiscount());
double discount = calculator.calculateDiscount(order);
```

### Python

```python
# ❌ MALAMENT: Cal modificar la classe per afegir nous tipus
class DiscountCalculator:
    def calculate_discount(self, order) -> float:
        if order.customer_type == "regular":
            return order.total * 0.1
        elif order.customer_type == "premium":
            return order.total * 0.2
        elif order.customer_type == "vip":
            return order.total * 0.3
        # Cada nou tipus requereix modificar aquest codi!
        return 0


# ✅ BÉ: Utilitza abstraccions per permetre extensió
from abc import ABC, abstractmethod

class DiscountStrategy(ABC):
    @abstractmethod
    def calculate(self, order) -> float:
        pass


class RegularDiscount(DiscountStrategy):
    def calculate(self, order) -> float:
        return order.total * 0.1


class PremiumDiscount(DiscountStrategy):
    def calculate(self, order) -> float:
        return order.total * 0.2


class VipDiscount(DiscountStrategy):
    def calculate(self, order) -> float:
        return order.total * 0.3


# Afegir un nou tipus és crear una nova classe sense tocar les existents
class BlackFridayDiscount(DiscountStrategy):
    def calculate(self, order) -> float:
        return order.total * 0.5


class SeasonalDiscount(DiscountStrategy):
    def __init__(self, percentage: float):
        self.percentage = percentage
    
    def calculate(self, order) -> float:
        return order.total * self.percentage


# La classe principal no necessita canvis
class DiscountCalculator:
    def __init__(self, strategy: DiscountStrategy):
        self.strategy = strategy
    
    def calculate_discount(self, order) -> float:
        return self.strategy.calculate(order)


# Ús
calculator = DiscountCalculator(VipDiscount())
discount = calculator.calculate_discount(order)

# O amb el nou descompte
summer_calculator = DiscountCalculator(SeasonalDiscount(0.15))
```

### JavaScript

```javascript
// ❌ MALAMENT: Cal modificar la classe per afegir nous tipus
class DiscountCalculator {
    calculateDiscount(order) {
        if (order.customerType === 'regular') {
            return order.total * 0.1;
        } else if (order.customerType === 'premium') {
            return order.total * 0.2;
        } else if (order.customerType === 'vip') {
            return order.total * 0.3;
        }
        return 0;
    }
}


// ✅ BÉ: Utilitza abstraccions per permetre extensió
class DiscountStrategy {
    calculate(order) {
        throw new Error('Method calculate() must be implemented');
    }
}

class RegularDiscount extends DiscountStrategy {
    calculate(order) {
        return order.total * 0.1;
    }
}

class PremiumDiscount extends DiscountStrategy {
    calculate(order) {
        return order.total * 0.2;
    }
}

class VipDiscount extends DiscountStrategy {
    calculate(order) {
        return order.total * 0.3;
    }
}

// Afegir nous tipus és trivial
class BlackFridayDiscount extends DiscountStrategy {
    calculate(order) {
        return order.total * 0.5;
    }
}

class SeasonalDiscount extends DiscountStrategy {
    constructor(percentage) {
        super();
        this.percentage = percentage;
    }
    
    calculate(order) {
        return order.total * this.percentage;
    }
}

// La classe principal no canvia
class DiscountCalculator {
    constructor(strategy) {
        this.strategy = strategy;
    }
    
    calculateDiscount(order) {
        return this.strategy.calculate(order);
    }
}

// Ús
const calculator = new DiscountCalculator(new VipDiscount());
const discount = calculator.calculateDiscount(order);
```

---

## L - Liskov Substitution Principle (LSP)

> *"Els objectes d'una superclasse han de poder ser reemplaçats per objectes de les seves subclasses sense trencar l'aplicació."*

Les subclasses han de complir el contracte de la classe pare. Si una subclasse no pot fer tot el que fa la pare, el disseny és incorrecte.

### Java

```java
// ❌ MALAMENT: Rectangle i Square violen LSP
public class Rectangle {
    protected int width;
    protected int height;
    
    public void setWidth(int width) {
        this.width = width;
    }
    
    public void setHeight(int height) {
        this.height = height;
    }
    
    public int getArea() {
        return width * height;
    }
}

public class Square extends Rectangle {
    // Un quadrat ha de tenir width == height, 
    // però això trenca el comportament esperat del Rectangle
    @Override
    public void setWidth(int width) {
        this.width = width;
        this.height = width; // Efecte secundari inesperat!
    }
    
    @Override
    public void setHeight(int height) {
        this.width = height;
        this.height = height; // Efecte secundari inesperat!
    }
}

// Aquest codi falla amb Square
public void testRectangle(Rectangle r) {
    r.setWidth(5);
    r.setHeight(4);
    assert r.getArea() == 20; // Falla amb Square! (àrea seria 16)
}


// ✅ BÉ: Utilitza una abstracció comuna sense herència problemàtica
public interface Shape {
    int getArea();
}

public class Rectangle implements Shape {
    private final int width;
    private final int height;
    
    public Rectangle(int width, int height) {
        this.width = width;
        this.height = height;
    }
    
    @Override
    public int getArea() {
        return width * height;
    }
    
    public int getWidth() { return width; }
    public int getHeight() { return height; }
}

public class Square implements Shape {
    private final int side;
    
    public Square(int side) {
        this.side = side;
    }
    
    @Override
    public int getArea() {
        return side * side;
    }
    
    public int getSide() { return side; }
}

// Ara el codi funciona correctament amb qualsevol Shape
public void printArea(Shape shape) {
    System.out.println("Area: " + shape.getArea());
}
```

### Python

```python
# ❌ MALAMENT: Ocell i Pingüí violen LSP
class Bird:
    def fly(self):
        return "Flying high!"
    
    def eat(self):
        return "Eating seeds"


class Penguin(Bird):
    def fly(self):
        # Els pingüins no poden volar! Això trenca el contracte
        raise NotImplementedError("Penguins can't fly!")


# Aquest codi falla amb Penguin
def make_bird_fly(bird: Bird):
    return bird.fly()  # Llança excepció amb Penguin!


# ✅ BÉ: Separa les capacitats en interfícies diferents
from abc import ABC, abstractmethod

class Bird(ABC):
    @abstractmethod
    def eat(self) -> str:
        pass


class FlyingBird(Bird):
    @abstractmethod
    def fly(self) -> str:
        pass


class SwimmingBird(Bird):
    @abstractmethod
    def swim(self) -> str:
        pass


class Sparrow(FlyingBird):
    def eat(self) -> str:
        return "Eating seeds"
    
    def fly(self) -> str:
        return "Flying high!"


class Penguin(SwimmingBird):
    def eat(self) -> str:
        return "Eating fish"
    
    def swim(self) -> str:
        return "Swimming fast!"


class Duck(FlyingBird, SwimmingBird):
    def eat(self) -> str:
        return "Eating bread"
    
    def fly(self) -> str:
        return "Flying not so high"
    
    def swim(self) -> str:
        return "Swimming in the pond"


# Ara el codi és segur
def make_bird_fly(bird: FlyingBird) -> str:
    return bird.fly()  # Només accepta ocells que poden volar

def make_bird_swim(bird: SwimmingBird) -> str:
    return bird.swim()  # Només accepta ocells que poden nedar
```

### JavaScript

```javascript
// ❌ MALAMENT: Vehicle i Bicycle violen LSP
class Vehicle {
    startEngine() {
        return 'Engine started';
    }
    
    accelerate() {
        return 'Accelerating...';
    }
}

class Car extends Vehicle {
    startEngine() {
        return 'Car engine started';
    }
}

class Bicycle extends Vehicle {
    startEngine() {
        // Les bicicletes no tenen motor!
        throw new Error("Bicycles don't have engines!");
    }
}

// Aquest codi falla amb Bicycle
function startVehicle(vehicle) {
    return vehicle.startEngine(); // Llança excepció amb Bicycle!
}


// ✅ BÉ: Separa les capacitats
class Vehicle {
    move() {
        throw new Error('Method move() must be implemented');
    }
}

class MotorizedVehicle extends Vehicle {
    startEngine() {
        throw new Error('Method startEngine() must be implemented');
    }
    
    stopEngine() {
        throw new Error('Method stopEngine() must be implemented');
    }
}

class HumanPoweredVehicle extends Vehicle {
    pedal() {
        throw new Error('Method pedal() must be implemented');
    }
}

class Car extends MotorizedVehicle {
    startEngine() {
        return 'Car engine started';
    }
    
    stopEngine() {
        return 'Car engine stopped';
    }
    
    move() {
        return 'Car is moving';
    }
}

class Bicycle extends HumanPoweredVehicle {
    pedal() {
        return 'Pedaling...';
    }
    
    move() {
        return 'Bicycle is moving';
    }
}

// Ara cada funció treballa amb el tipus correcte
function startMotorizedVehicle(vehicle) {
    if (vehicle instanceof MotorizedVehicle) {
        return vehicle.startEngine();
    }
    throw new Error('Vehicle is not motorized');
}

function moveVehicle(vehicle) {
    return vehicle.move(); // Funciona amb qualsevol Vehicle
}
```

---

## I - Interface Segregation Principle (ISP)

> *"Els clients no haurien de ser forçats a dependre d'interfícies que no utilitzen."*

És millor tenir moltes interfícies petites i específiques que una de gran i general. Cada classe només ha d'implementar els mètodes que realment necessita.

### Java

```java
// ❌ MALAMENT: Interfície massa gran
public interface Worker {
    void work();
    void eat();
    void sleep();
    void attendMeeting();
    void writeReport();
    void manageTeam();
}

// Un robot ha d'implementar mètodes que no té sentit
public class Robot implements Worker {
    @Override
    public void work() { /* OK */ }
    
    @Override
    public void eat() { 
        throw new UnsupportedOperationException(); // Els robots no mengen!
    }
    
    @Override
    public void sleep() {
        throw new UnsupportedOperationException(); // Els robots no dormen!
    }
    
    @Override
    public void attendMeeting() {
        throw new UnsupportedOperationException();
    }
    
    @Override
    public void writeReport() { /* OK */ }
    
    @Override
    public void manageTeam() {
        throw new UnsupportedOperationException();
    }
}


// ✅ BÉ: Interfícies petites i específiques
public interface Workable {
    void work();
}

public interface Feedable {
    void eat();
    void drink();
}

public interface Sleepable {
    void sleep();
}

public interface Reportable {
    void writeReport();
}

public interface Manageable {
    void manageTeam();
    void attendMeeting();
}

// Cada classe només implementa el que necessita
public class HumanEmployee implements Workable, Feedable, Sleepable, Reportable {
    @Override
    public void work() { System.out.println("Working..."); }
    
    @Override
    public void eat() { System.out.println("Eating lunch"); }
    
    @Override
    public void drink() { System.out.println("Drinking coffee"); }
    
    @Override
    public void sleep() { System.out.println("Sleeping at night"); }
    
    @Override
    public void writeReport() { System.out.println("Writing report"); }
}

public class Manager extends HumanEmployee implements Manageable {
    @Override
    public void manageTeam() { System.out.println("Managing team"); }
    
    @Override
    public void attendMeeting() { System.out.println("In a meeting"); }
}

public class Robot implements Workable, Reportable {
    @Override
    public void work() { System.out.println("Robot working 24/7"); }
    
    @Override
    public void writeReport() { System.out.println("Generating automated report"); }
}
```

### Python

```python
# ❌ MALAMENT: Interfície massa gran
from abc import ABC, abstractmethod

class Printer(ABC):
    @abstractmethod
    def print_document(self, document): pass
    
    @abstractmethod
    def scan_document(self): pass
    
    @abstractmethod
    def fax_document(self, document): pass
    
    @abstractmethod
    def staple_document(self, document): pass


# Una impressora bàsica ha d'implementar mètodes que no suporta
class BasicPrinter(Printer):
    def print_document(self, document):
        print(f"Printing: {document}")
    
    def scan_document(self):
        raise NotImplementedError("Basic printer can't scan")
    
    def fax_document(self, document):
        raise NotImplementedError("Basic printer can't fax")
    
    def staple_document(self, document):
        raise NotImplementedError("Basic printer can't staple")


# ✅ BÉ: Interfícies petites i específiques
class Printable(ABC):
    @abstractmethod
    def print_document(self, document) -> None:
        pass


class Scannable(ABC):
    @abstractmethod
    def scan_document(self) -> bytes:
        pass


class Faxable(ABC):
    @abstractmethod
    def fax_document(self, document, number: str) -> bool:
        pass


class Stapleable(ABC):
    @abstractmethod
    def staple_document(self, document) -> None:
        pass


# Cada classe implementa només el que necessita
class BasicPrinter(Printable):
    def print_document(self, document) -> None:
        print(f"Printing: {document}")


class MultifunctionPrinter(Printable, Scannable, Faxable):
    def print_document(self, document) -> None:
        print(f"Printing: {document}")
    
    def scan_document(self) -> bytes:
        print("Scanning...")
        return b"scanned_data"
    
    def fax_document(self, document, number: str) -> bool:
        print(f"Faxing to {number}")
        return True


class ProfessionalPrinter(Printable, Scannable, Stapleable):
    def print_document(self, document) -> None:
        print(f"High quality printing: {document}")
    
    def scan_document(self) -> bytes:
        print("High resolution scanning...")
        return b"hq_scanned_data"
    
    def staple_document(self, document) -> None:
        print("Stapling document")


# Les funcions poden demanar interfícies específiques
def print_all(printers: list[Printable], document):
    for printer in printers:
        printer.print_document(document)

def scan_all(scanners: list[Scannable]) -> list[bytes]:
    return [scanner.scan_document() for scanner in scanners]
```

### JavaScript

```javascript
// ❌ MALAMENT: Interfície massa gran (simulada amb classe base)
class DataStore {
    save(data) { throw new Error('Not implemented'); }
    update(id, data) { throw new Error('Not implemented'); }
    delete(id) { throw new Error('Not implemented'); }
    find(id) { throw new Error('Not implemented'); }
    findAll() { throw new Error('Not implemented'); }
    query(criteria) { throw new Error('Not implemented'); }
    transaction(callback) { throw new Error('Not implemented'); }
    backup() { throw new Error('Not implemented'); }
    restore(backup) { throw new Error('Not implemented'); }
}

// Un cache simple ha d'implementar mètodes que no suporta
class SimpleCache extends DataStore {
    constructor() {
        super();
        this.cache = new Map();
    }
    
    save(data) { this.cache.set(data.id, data); }
    find(id) { return this.cache.get(id); }
    delete(id) { this.cache.delete(id); }
    
    // Aquests mètodes no tenen sentit per a un cache
    update(id, data) { throw new Error('Cache does not support update'); }
    findAll() { throw new Error('Cache does not support findAll'); }
    query(criteria) { throw new Error('Cache does not support queries'); }
    transaction(callback) { throw new Error('Cache does not support transactions'); }
    backup() { throw new Error('Cache does not support backup'); }
    restore(backup) { throw new Error('Cache does not support restore'); }
}


// ✅ BÉ: Interfícies petites i específiques
// Readable
class Readable {
    find(id) { throw new Error('Not implemented'); }
}

// Writable
class Writable {
    save(data) { throw new Error('Not implemented'); }
    delete(id) { throw new Error('Not implemented'); }
}

// Queryable
class Queryable {
    findAll() { throw new Error('Not implemented'); }
    query(criteria) { throw new Error('Not implemented'); }
}

// Transactional
class Transactional {
    beginTransaction() { throw new Error('Not implemented'); }
    commit() { throw new Error('Not implemented'); }
    rollback() { throw new Error('Not implemented'); }
}

// Backupable
class Backupable {
    backup() { throw new Error('Not implemented'); }
    restore(backup) { throw new Error('Not implemented'); }
}

// Implementacions específiques
class SimpleCache {
    constructor() {
        this.cache = new Map();
    }
    
    // Només implementa Readable i Writable bàsic
    find(id) {
        return this.cache.get(id);
    }
    
    save(data) {
        this.cache.set(data.id, data);
    }
    
    delete(id) {
        this.cache.delete(id);
    }
}

class Database {
    // Implementa totes les interfícies que necessita
    find(id) { /* ... */ }
    save(data) { /* ... */ }
    delete(id) { /* ... */ }
    findAll() { /* ... */ }
    query(criteria) { /* ... */ }
    beginTransaction() { /* ... */ }
    commit() { /* ... */ }
    rollback() { /* ... */ }
    backup() { /* ... */ }
    restore(backup) { /* ... */ }
}

// Funcions que demanen capacitats específiques
function readData(store, id) {
    if (typeof store.find !== 'function') {
        throw new Error('Store must be Readable');
    }
    return store.find(id);
}

function queryData(store, criteria) {
    if (typeof store.query !== 'function') {
        throw new Error('Store must be Queryable');
    }
    return store.query(criteria);
}
```

---

## D - Dependency Inversion Principle (DIP)

> *"Els mòduls d'alt nivell no han de dependre de mòduls de baix nivell. Ambdós han de dependre d'abstraccions."*

En lloc de que les classes depenguin directament d'implementacions concretes, han de dependre d'interfícies o classes abstractes. Això permet canviar implementacions fàcilment.

### Java

```java
// ❌ MALAMENT: Dependència directa d'implementacions concretes
public class OrderService {
    // Dependències concretes - difícil de testar i canviar
    private MySQLDatabase database = new MySQLDatabase();
    private SmtpEmailSender emailSender = new SmtpEmailSender();
    private StripePaymentProcessor paymentProcessor = new StripePaymentProcessor();
    
    public void processOrder(Order order) {
        paymentProcessor.charge(order.getAmount());
        database.save(order);
        emailSender.send(order.getCustomerEmail(), "Order confirmed");
    }
}


// ✅ BÉ: Depèn d'abstraccions, no de concrecions
// Definim les abstraccions
public interface OrderRepository {
    void save(Order order);
    Order findById(Long id);
}

public interface EmailService {
    void send(String to, String subject, String body);
}

public interface PaymentProcessor {
    PaymentResult charge(Money amount, PaymentDetails details);
}

// Implementacions concretes
public class MySQLOrderRepository implements OrderRepository {
    @Override
    public void save(Order order) {
        // Lògica MySQL
    }
    
    @Override
    public Order findById(Long id) {
        // Lògica MySQL
        return null;
    }
}

public class SmtpEmailService implements EmailService {
    @Override
    public void send(String to, String subject, String body) {
        // Lògica SMTP
    }
}

public class StripePaymentProcessor implements PaymentProcessor {
    @Override
    public PaymentResult charge(Money amount, PaymentDetails details) {
        // Lògica Stripe
        return new PaymentResult(true);
    }
}

// El servei depèn d'abstraccions (injectades)
public class OrderService {
    private final OrderRepository repository;
    private final EmailService emailService;
    private final PaymentProcessor paymentProcessor;
    
    // Injecció de dependències via constructor
    public OrderService(
            OrderRepository repository,
            EmailService emailService,
            PaymentProcessor paymentProcessor) {
        this.repository = repository;
        this.emailService = emailService;
        this.paymentProcessor = paymentProcessor;
    }
    
    public void processOrder(Order order) {
        PaymentResult result = paymentProcessor.charge(
            order.getAmount(), 
            order.getPaymentDetails()
        );
        
        if (result.isSuccessful()) {
            repository.save(order);
            emailService.send(
                order.getCustomerEmail(),
                "Order Confirmed",
                "Your order has been processed."
            );
        }
    }
}

// Configuració (Spring Boot)
@Configuration
public class AppConfig {
    @Bean
    public OrderRepository orderRepository() {
        return new MySQLOrderRepository();
    }
    
    @Bean
    public EmailService emailService() {
        return new SmtpEmailService();
    }
    
    @Bean
    public PaymentProcessor paymentProcessor() {
        return new StripePaymentProcessor();
    }
}

// Per testing - podem injectar mocks fàcilment
@Test
void processOrder_sendsConfirmationEmail() {
    OrderRepository mockRepo = mock(OrderRepository.class);
    EmailService mockEmail = mock(EmailService.class);
    PaymentProcessor mockPayment = mock(PaymentProcessor.class);
    
    when(mockPayment.charge(any(), any()))
        .thenReturn(new PaymentResult(true));
    
    OrderService service = new OrderService(mockRepo, mockEmail, mockPayment);
    service.processOrder(testOrder);
    
    verify(mockEmail).send(eq(testOrder.getCustomerEmail()), any(), any());
}
```

### Python

```python
# ❌ MALAMENT: Dependència directa d'implementacions concretes
class OrderService:
    def __init__(self):
        # Dependències concretes - difícil de testar i canviar
        self.database = MySQLDatabase()
        self.email_sender = SmtpEmailSender()
        self.payment_processor = StripePaymentProcessor()
    
    def process_order(self, order):
        self.payment_processor.charge(order.amount)
        self.database.save(order)
        self.email_sender.send(order.customer_email, "Order confirmed")


# ✅ BÉ: Depèn d'abstraccions mitjançant protocols o ABC
from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import Protocol


# Definim les abstraccions amb Protocol (duck typing estructural)
class OrderRepository(Protocol):
    def save(self, order) -> None: ...
    def find_by_id(self, order_id: int): ...


class EmailService(Protocol):
    def send(self, to: str, subject: str, body: str) -> None: ...


class PaymentProcessor(Protocol):
    def charge(self, amount: float, details: dict) -> bool: ...


# Implementacions concretes
class MySQLOrderRepository:
    def __init__(self, connection_string: str):
        self.connection_string = connection_string
    
    def save(self, order) -> None:
        print(f"Saving to MySQL: {order}")
    
    def find_by_id(self, order_id: int):
        print(f"Finding order {order_id} in MySQL")


class PostgresOrderRepository:
    def __init__(self, connection_string: str):
        self.connection_string = connection_string
    
    def save(self, order) -> None:
        print(f"Saving to PostgreSQL: {order}")
    
    def find_by_id(self, order_id: int):
        print(f"Finding order {order_id} in PostgreSQL")


class SmtpEmailService:
    def __init__(self, smtp_host: str, smtp_port: int):
        self.host = smtp_host
        self.port = smtp_port
    
    def send(self, to: str, subject: str, body: str) -> None:
        print(f"Sending email via SMTP to {to}")


class SendGridEmailService:
    def __init__(self, api_key: str):
        self.api_key = api_key
    
    def send(self, to: str, subject: str, body: str) -> None:
        print(f"Sending email via SendGrid to {to}")


class StripePaymentProcessor:
    def __init__(self, api_key: str):
        self.api_key = api_key
    
    def charge(self, amount: float, details: dict) -> bool:
        print(f"Charging {amount} via Stripe")
        return True


class PayPalPaymentProcessor:
    def __init__(self, client_id: str, client_secret: str):
        self.client_id = client_id
        self.client_secret = client_secret
    
    def charge(self, amount: float, details: dict) -> bool:
        print(f"Charging {amount} via PayPal")
        return True


# El servei depèn d'abstraccions (injectades)
class OrderService:
    def __init__(
        self,
        repository: OrderRepository,
        email_service: EmailService,
        payment_processor: PaymentProcessor
    ):
        self.repository = repository
        self.email_service = email_service
        self.payment_processor = payment_processor
    
    def process_order(self, order) -> bool:
        if self.payment_processor.charge(order.amount, order.payment_details):
            self.repository.save(order)
            self.email_service.send(
                order.customer_email,
                "Order Confirmed",
                f"Your order #{order.id} has been processed."
            )
            return True
        return False


# Configuració - podem canviar implementacions fàcilment
def create_order_service(environment: str) -> OrderService:
    if environment == "production":
        return OrderService(
            repository=PostgresOrderRepository("postgresql://prod-db"),
            email_service=SendGridEmailService("sg_api_key"),
            payment_processor=StripePaymentProcessor("stripe_key")
        )
    else:  # development/testing
        return OrderService(
            repository=MySQLOrderRepository("mysql://localhost"),
            email_service=SmtpEmailService("localhost", 1025),
            payment_processor=StripePaymentProcessor("stripe_test_key")
        )


# Per testing - podem injectar mocks fàcilment
def test_process_order_sends_email():
    # Mocks
    mock_repo = Mock(spec=OrderRepository)
    mock_email = Mock(spec=EmailService)
    mock_payment = Mock(spec=PaymentProcessor)
    mock_payment.charge.return_value = True
    
    service = OrderService(mock_repo, mock_email, mock_payment)
    order = Order(id=1, amount=100, customer_email="test@example.com")
    
    service.process_order(order)
    
    mock_email.send.assert_called_once()
```

### JavaScript

```javascript
// ❌ MALAMENT: Dependència directa d'implementacions concretes
class OrderService {
    constructor() {
        // Dependències concretes - difícil de testar i canviar
        this.database = new MySQLDatabase();
        this.emailSender = new SmtpEmailSender();
        this.paymentProcessor = new StripePaymentProcessor();
    }
    
    async processOrder(order) {
        await this.paymentProcessor.charge(order.amount);
        await this.database.save(order);
        await this.emailSender.send(order.customerEmail, 'Order confirmed');
    }
}


// ✅ BÉ: Depèn d'abstraccions (injectades)
// En JavaScript no hi ha interfícies, però podem documentar el contracte

/**
 * @typedef {Object} OrderRepository
 * @property {function(Order): Promise<void>} save
 * @property {function(number): Promise<Order>} findById
 */

/**
 * @typedef {Object} EmailService
 * @property {function(string, string, string): Promise<void>} send
 */

/**
 * @typedef {Object} PaymentProcessor
 * @property {function(number, Object): Promise<boolean>} charge
 */

// Implementacions concretes
class MySQLOrderRepository {
    constructor(connectionString) {
        this.connectionString = connectionString;
    }
    
    async save(order) {
        console.log(`Saving to MySQL: ${order.id}`);
    }
    
    async findById(orderId) {
        console.log(`Finding order ${orderId} in MySQL`);
    }
}

class MongoOrderRepository {
    constructor(connectionString) {
        this.connectionString = connectionString;
    }
    
    async save(order) {
        console.log(`Saving to MongoDB: ${order.id}`);
    }
    
    async findById(orderId) {
        console.log(`Finding order ${orderId} in MongoDB`);
    }
}

class SmtpEmailService {
    constructor(smtpConfig) {
        this.config = smtpConfig;
    }
    
    async send(to, subject, body) {
        console.log(`Sending email via SMTP to ${to}`);
    }
}

class SendGridEmailService {
    constructor(apiKey) {
        this.apiKey = apiKey;
    }
    
    async send(to, subject, body) {
        console.log(`Sending email via SendGrid to ${to}`);
    }
}

class StripePaymentProcessor {
    constructor(apiKey) {
        this.apiKey = apiKey;
    }
    
    async charge(amount, details) {
        console.log(`Charging ${amount} via Stripe`);
        return true;
    }
}

class PayPalPaymentProcessor {
    constructor(clientId, clientSecret) {
        this.clientId = clientId;
        this.clientSecret = clientSecret;
    }
    
    async charge(amount, details) {
        console.log(`Charging ${amount} via PayPal`);
        return true;
    }
}

// El servei depèn d'abstraccions (injectades via constructor)
class OrderService {
    /**
     * @param {OrderRepository} repository
     * @param {EmailService} emailService
     * @param {PaymentProcessor} paymentProcessor
     */
    constructor(repository, emailService, paymentProcessor) {
        this.repository = repository;
        this.emailService = emailService;
        this.paymentProcessor = paymentProcessor;
    }
    
    async processOrder(order) {
        const paymentSuccess = await this.paymentProcessor.charge(
            order.amount,
            order.paymentDetails
        );
        
        if (paymentSuccess) {
            await this.repository.save(order);
            await this.emailService.send(
                order.customerEmail,
                'Order Confirmed',
                `Your order #${order.id} has been processed.`
            );
            return true;
        }
        return false;
    }
}

// Factory per crear el servei amb les dependències adequades
function createOrderService(environment) {
    if (environment === 'production') {
        return new OrderService(
            new MongoOrderRepository(process.env.MONGO_URI),
            new SendGridEmailService(process.env.SENDGRID_KEY),
            new StripePaymentProcessor(process.env.STRIPE_KEY)
        );
    } else {
        return new OrderService(
            new MySQLOrderRepository('mysql://localhost'),
            new SmtpEmailService({ host: 'localhost', port: 1025 }),
            new StripePaymentProcessor('stripe_test_key')
        );
    }
}

// Per testing amb Jest
describe('OrderService', () => {
    it('sends confirmation email on successful payment', async () => {
        const mockRepo = { save: jest.fn() };
        const mockEmail = { send: jest.fn() };
        const mockPayment = { charge: jest.fn().mockResolvedValue(true) };
        
        const service = new OrderService(mockRepo, mockEmail, mockPayment);
        const order = { id: 1, amount: 100, customerEmail: 'test@test.com' };
        
        await service.processOrder(order);
        
        expect(mockEmail.send).toHaveBeenCalledWith(
            'test@test.com',
            'Order Confirmed',
            expect.any(String)
        );
    });
});
```

---

## Resum

| Principi | Problema que resol | Solució |
|----------|-------------------|---------|
| **SRP** | Classes amb massa responsabilitats | Dividir en classes més petites i enfocades |
| **OCP** | Modificar codi existent per afegir funcionalitat | Utilitzar abstraccions i polimorfisme |
| **LSP** | Subclasses que trenquen el comportament esperat | Dissenyar jerarquies coherents |
| **ISP** | Interfícies massa grans | Crear interfícies petites i específiques |
| **DIP** | Acoblament fort a implementacions | Dependre d'abstraccions, injectar dependències |

Aplicar aquests principis de manera consistent produeix codi més mantenible, testejable i flexible davant els canvis.
