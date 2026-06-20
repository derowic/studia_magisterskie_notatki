# Notatki na egzamin – Architektury Oprogramowania i Wzorce Projektowe

**© Błażej Strus 2026**  
**Przygotowane na podstawie materiałów wykładowych, laboratoriów i notebooków**

---

## Spis treści

1. [Najważniejsza filozofia](#0-najważniejsza-filozofia)
2. [Architektury jednej aplikacji](#1-architektury-jednej-aplikacji)
3. [Inne wymiary architektury](#2-inne-wymiary-architektury)
4. [Zasady SOLID](#3-zasady-solid)
5. [Wzorce Projektowe GoF](#4-wzorce-projektowe-gof)
6. [Przykłady z laboratoriów](#5-przykłady-z-laboratoriów)
7. [Najczęstsze pytania egzaminacyjne](#6-najczęstsze-pytania-egzaminacyjne)

---

## 0. Najważniejsza filozofia

**Te sześć architektur to NIE jest sześć opcji „albo-albo”**.

One odpowiadają na **zupełnie różne pytania**:

- **Layered / Clean / Hexagonal** → Jak zorganizować **wnętrze jednej aplikacji** (ochrona logiki biznesowej).
- **Microservices** → Czy to jedna duża aplikacja, czy wiele niezależnych serwisów.
- **Event-Driven** → W jaki sposób komponenty mają się komunikować.
- **CQRS** → Czy operacje odczytu i zapisu powinny mieć osobne modele.

**Można je dowolnie łączyć.**  
Przykład idealny: Mikroserwis napisany w **Clean Architecture**, komunikujący się przez zdarzenia (**Event-Driven**) i stosujący **CQRS** wewnątrz.

**Analogia wspólna dla wszystkich**: Restauracja (kuchnia = logika biznesowa, kelner = adapter, goście = klienci, dostawcy = zewnętrzne systemy).

---

## 1. Architektury jednej aplikacji

### 1.1 Layered / N-Tier (Warstwowa)

- Najprostsza i najstarsza.
- Klasyczne warstwy: **Presentation → Business Logic → Data**.
- Często dodaje się warstwy: Application, Infrastructure.

**Zalety**: łatwa do nauki, dobra separacja na początku projektu.  
**Wady**: zależności często idą w obie strony, logika biznesowa „przecieka” do warstw zewnętrznych, słaba odporność na zmiany technologii.

### 1.2 Clean Architecture (Robert C. Martin)

- Koncentryczne kręgi, zależności **zawsze skierowane do środka**.
- Najważniejsze elementy w centrum:
  1. **Entities** (reguły biznesowe, najczystsza logika)
  2. **Use Cases / Interactors**
  3. **Interface Adapters** (kontrolery, repozytoria)
  4. **Frameworks & Drivers** (baza, framework webowy, UI)

**Kluczowa zasada**: Logika biznesowa **nie zależy** od szczegółów implementacyjnych (bazy, frameworka, UI).

### 1.3 Hexagonal Architecture (Ports & Adapters)

- Aplikacja jako **heksagon**.
- **Ports** = interfejsy (kontrakty) – co aplikacja oferuje i czego potrzebuje.
- **Adapters** = konkretne implementacje (REST adapter, Database adapter, itd.).

**Driving Ports** (wejściowe) i **Driven Ports** (wyjściowe).  
Najwyższy poziom izolacji logiki biznesowej.

**Porównanie**:

| Aspekt                  | Layered       | Clean             | Hexagonal          |
|-------------------------|---------------|-------------------|--------------------|
| Kierunek zależności     | Dowolny       | Do środka         | Do środka          |
| Ochrona logiki          | Słaba         | Bardzo dobra      | Najlepsza          |
| Wymiana technologii     | Trudna        | Łatwa             | Najłatwiejsza      |
| Złożoność               | Niska         | Średnia           | Średnia+           |

---

## 2. Inne wymiary architektury

- **Microservices** – dekompozycja na małe, niezależne serwisy (własna baza, deployment, zespół).
- **Event-Driven** – komunikacja przez zdarzenia (Kafka, RabbitMQ) zamiast bezpośrednich wywołań.
- **CQRS** – rozdzielenie modelu zapisu (Command) od modelu odczytu (Query). Często łączone z Event Sourcing.

---

## 3. Zasady SOLID

- **SRP** – jedna odpowiedzialność
❌ Single Responsibility Principle Violation
```python
public class OrderManager
{
  public void ProcessOrder(Order order)
  {
      // Logic to process the order
  }

  public void SaveOrderToDatabase(Order order)
  {
      // Logic to save order to the database
  }

  public void SendOrderConfirmationEmail(Order order)
  {
      // Logic to send confirmation email
  }
}
```

✔️ Single Responsibility Principle Compliant
```python
public class OrderProcessor
{
  public void ProcessOrder(Order order)
  {
      // Logic to process the order
  }
}

public class OrderRepository
{
  public void SaveOrder(Order order)
  {
      // Logic to save order to the database
  }
}

public class EmailService
{
  public void SendOrderConfirmationEmail(Order order)
  {
      // Logic to send confirmation email
  }
}
```
- **OCP** – otwarte na rozszerzanie, zamknięte na modyfikację (**klucz do wzorców**)
❌ Open/Closed Principle Violation
```python
public class DiscountCalculator
{
    public double CalculateDiscount(Order order, string customerType)
    {
        if (customerType == "Regular")
        {
            return order.Amount * 0.05;
        }
        else if (customerType == "Premium")
        {
            return order.Amount * 0.10;
        }
        return 0;
    }
}
```
✔️ Open/Closed Principle Compliant
```python
public interface IDiscountStrategy
{
    double CalculateDiscount(Order order);
}

public class RegularCustomerDiscount : IDiscountStrategy
{
    public double CalculateDiscount(Order order)
    {
        return order.Amount * 0.05;
    }
}

public class PremiumCustomerDiscount : IDiscountStrategy
{
    public double CalculateDiscount(Order order)
    {
        return order.Amount * 0.10;
    }
}

public class DiscountCalculator
{
    public double CalculateDiscount(Order order, IDiscountStrategy discountStrategy)
    {
        return discountStrategy.CalculateDiscount(order);
    }
}
```
- **LSP** – podtypy zastępowalne bazowym
❌ Liskov Substitution Principle Violation
```python
public class Bird
{
    public virtual void Fly() { }
}

public class Penguin : Bird
{
    public override void Fly()
    {
        throw new NotImplementedException();
    }
}
```
✔️ Liskov Substitution Principle Compliant
```python
public class Bird { }

public class FlyingBird : Bird
{
    public virtual void Fly() { }
}

public class Sparrow : FlyingBird
{
    public override void Fly() { }
}

public class Penguin : Bird
{
    // Penguins don't fly, no Fly method needed
}
```
- **ISP** – małe interfejsy zamiast „grubych”

❌ Interface Segregation Principle Violation
```python
public interface IWorker
{
    void Work();
    void Eat();
}

public class RobotWorker : IWorker
{
    public void Work() { }
    public void Eat() { throw new NotImplementedException(); }
}
public class HumanWorker : IWorker
{
    public void Work() { }
    public void Eat() { }
}
```

✔️ Interface Segregation Principle Compliant
```python
public interface IWorkable
{
    void Work();
}

public interface IFeedable
{
    void Eat();
}

public class HumanWorker : IWorkable, IFeedable
{
    public void Work() { }
    public void Eat() { }
}

public class RobotWorker : IWorkable
{
    public void Work() { }
}
```
- **DIP** – zależ od abstrakcji, nie od konkretów
❌ Dependency Inversion Principle Violation
```python
public class Database
{
    public void Save() { }
}

public class EmployeeManager
{
    private Database _database;

    public EmployeeManager()
    {
        _database = new Database();
    }

    public void SaveEmployee()
    {
        _database.Save();
    }
}
```

✔️ Dependency Inversion Principle Compliant
```python
public interface IDataStore
{
    void Save();
}

public class Database : IDataStore
{
    public void Save() { }
}

public class EmployeeManager
{
    private IDataStore _dataStore;

    public EmployeeManager(IDataStore dataStore)
    {
        _dataStore = dataStore;
    }

    public void SaveEmployee()
    {
        _dataStore.Save();
    }
}
```
---

## 4. Wzorce Projektowe GoF (szczegółowo)

### Kreacyjne
- **Factory Method** – podklasa decyduje jaki obiekt stworzyć
```python
from abc import ABC, abstractmethod

class Notifier(ABC):
    @abstractmethod
    def send(self, message: str): ...

class EmailNotifier(Notifier):
    def send(self, message): return f"Email: {message}"

class SmsNotifier(Notifier):
    def send(self, message): return f"SMS: {message}"

# Factory Method
class NotificationService(ABC):
    @abstractmethod
    def create_notifier(self) -> Notifier: ...

    def notify(self, message):
        notifier = self.create_notifier()
        return notifier.send(message)

class EmailService(NotificationService):
    def create_notifier(self): return EmailNotifier()

class SmsService(NotificationService):
    def create_notifier(self): return SmsNotifier()
```
- **Abstract Factory** – fabryka rodzin obiektów
```python
class UIFactory(ABC):
    @abstractmethod
    def create_button(self): ...
    @abstractmethod
    def create_checkbox(self): ...

class WindowsFactory(UIFactory):
    def create_button(self): return "Windows Button"
    def create_checkbox(self): return "Windows Checkbox"

class MacFactory(UIFactory):
    def create_button(self): return "Mac Button"
    def create_checkbox(self): return "Mac Checkbox"
```

- **Builder** – krokowe budowanie złożonego obiektu
```python
class House:
    def __init__(self):
        self.walls = None
        self.roof = None
        self.windows = 0

class HouseBuilder:
    def __init__(self):
        self.house = House()
    
    def build_walls(self, material):
        self.house.walls = material
        return self
    
    def build_roof(self, material):
        self.house.roof = material
        return self
    
    def add_windows(self, count):
        self.house.windows = count
        return self
    
    def build(self):
        return self.house
```

- **Prototype** – klonowanie
```python
import copy

class Monster:
    def __init__(self, name, health, damage):
        self.name = name
        self.health = health
        self.damage = damage
    
    def clone(self):
        return copy.deepcopy(self)

goblin = Monster("Goblin", 50, 10)
goblin_army = [goblin.clone() for _ in range(5)]
```

- **Singleton** – jedna instancja (często anti-pattern)
```python
class Logger:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance.logs = []
        return cls._instance
```

### Strukturalne
- **Adapter** – tłumaczenie interfejsu
```python
class OldPaymentSystem:
    def pay(self, amount): return f"Old system paid {amount}"

class NewPaymentInterface(ABC):
    @abstractmethod
    def make_payment(self, amount): ...

class PaymentAdapter(NewPaymentInterface):
    def __init__(self, old_system):
        self.old = old_system
    
    def make_payment(self, amount):
        return self.old.pay(amount)
```

- **Bridge** – oddzielenie abstrakcji od implementacji
```python
class Renderer(ABC):
    @abstractmethod
    def render(self, shape): ...

class VectorRenderer(Renderer):
    def render(self, shape): return f"Vector {shape}"

class Shape(ABC):
    def __init__(self, renderer):
        self.renderer = renderer
    def draw(self): ...

class Circle(Shape):
    def draw(self): return self.renderer.render("Circle")
```

- **Composite** – drzewo (liść = kompozyt)
```python
class Component(ABC):
    def operation(self): ...

class Leaf(Component):
    def operation(self): return "Leaf"

class Composite(Component):
    def __init__(self):
        self.children = []
    def add(self, child): self.children.append(child)
    def operation(self):
        return f"Composite: {[c.operation() for c in self.children]}"
```

- **Decorator** – dynamiczne dodawanie zachowań
```python
class Coffee:
    def cost(self): return 5
    def description(self): return "Coffee"

class MilkDecorator(Coffee):
    def __init__(self, coffee):
        self.coffee = coffee
    def cost(self): return self.coffee.cost() + 2
    def description(self): return self.coffee.description() + " + Milk"
```

- **Facade** – uproszczenie podsystemu
```python
class HomeTheaterFacade:
    def __init__(self, lights, projector, sound):
        self.lights = lights
        self.projector = projector
        self.sound = sound
    
    def watch_movie(self):
        self.lights.dim()
        self.projector.on()
        self.sound.on()
        print("Film started!")
```

- **Proxy** – kontrola dostępu / lazy
```python
class RealImage:
    def display(self): print("Displaying image")

class ImageProxy:
    def __init__(self):
        self.real = None
    def display(self):
        if not self.real:
            self.real = RealImage()
        self.real.display()
```

- **Flyweight** – oszczędność pamięci
```python
class TreeType:
    def __init__(self, name, color):
        self.name = name
        self.color = color

class TreeFactory:
    _trees = {}
    @staticmethod
    def get_tree(name, color):
        key = (name, color)
        if key not in TreeFactory._trees:
            TreeFactory._trees[key] = TreeType(name, color)
        return TreeFactory._trees[key]
```

### Behawioralne
- **Chain of Responsibility** – łańcuch handlerów
```python
class Handler(ABC):
    def __init__(self, next=None):
        self.next = next
    def handle(self, request):
        if self.next: return self.next.handle(request)
```

- **Command** – akcja jako obiekt (`execute` + `undo`)
```python
class Command(ABC):
    @abstractmethod
    def execute(self): ...
    @abstractmethod
    def undo(self): ...

class LightOnCommand(Command):
    def __init__(self, light): self.light = light
    def execute(self): return self.light.on()
    def undo(self): return self.light.off()
```

- **Iterator**
```python
class Iterator(ABC):
    @abstractmethod
    def next(self): ...
    @abstractmethod
    def has_next(self): ...
```

- **Mediator**
```python
class ChatRoom:
    def __init__(self):
        self.users = []
    def broadcast(self, sender, message):
        for user in self.users:
            if user != sender:
                user.receive(message)
```

- **Memento** – snapshot stanu
```python
class Memento:
    def __init__(self, state): self.state = state

class Originator:
    def __init__(self): self.state = "initial"
    def save(self): return Memento(self.state)
    def restore(self, memento): self.state = memento.state
```

- **Observer** – pub/sub
```python
class Observer(ABC):
    @abstractmethod
    def update(self, event): ...

class Subject:
    def __init__(self):
        self.observers = []
    def notify(self, event):
        for obs in self.observers:
            obs.update(event)
```

- **State** – zachowanie zależne od stanu
```python
class State(ABC):
    @abstractmethod
    def press(self): ...

class OffState(State):
    def press(self): return OnState(), "turning on"
```

- **Strategy** – wymienny algorytm
```python
class SortStrategy(ABC):
    @abstractmethod
    def sort(self, data): ...

class QuickSort(SortStrategy):
    def sort(self, data): return sorted(data)
```

- **Template Method**
```python
class Algorithm(ABC):
    def template_method(self):
        self.step1()
        self.step2()
        self.step3()
    
    @abstractmethod
    def step1(self): ...
```

- **Visitor**
```python
class Visitor(ABC):
    def visit_circle(self, circle): ...
    def visit_square(self, square): ...

class Circle:
    def accept(self, visitor): visitor.visit_circle(self)
```

- **Interpreter**
```python
class Expression(ABC):
    @abstractmethod
    def interpret(self, context): ...

class Number(Expression):
    def __init__(self, value): self.value = value
    def interpret(self, context): return self.value
```

---

## 5. Przykłady z laboratoriów (kluczowe)

- **Factory Method** – tworzenie urządzeń w inteligentnym domu
- **Strategy** – różne tryby ogrzewania
- **Observer** – czujnik ruchu
- **State** – odtwarzacz
- **Adapter** – integracja z legacy
- **Decorator** – dodawanie funkcjonalności (logging, metering…)
- **Command** – pilot z undo
- **Facade** – sterowanie kinem domowym

---

## 6. Najczęstsze pytania egzaminacyjne + wskazówki

1. Porównaj **Layered vs Clean vs Hexagonal** (zawsze rysuj diagram zależności).
2. Co to jest **Ports & Adapters** i dlaczego jest potężne?
3. Jak **OCP** i **DIP** realizują się we wzorcach?
4. Zalety i wady **Microservices**.
5. Kiedy stosować **CQRS**?
6. Różnica między **Decorator** a dziedziczeniem.
7. Jak **Command** umożliwia undo i makra?

**Złota rada na egzamin**: Zawsze zaczynaj od analogii restauracji i podkreślaj kierunek zależności (do środka).

---

**Koniec notatek**