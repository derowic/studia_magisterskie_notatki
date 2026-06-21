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

# Przykład użycia
service = EmailService()
service.notify("Zamówienie wysłane!")   # nie wiemy, że wewnątrz jest EmailNotifier
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

factory = WindowsFactory() if is_windows else MacFactory()
button = factory.create_button()
checkbox = factory.create_checkbox()
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

house = (HouseBuilder()
         .build_walls("brick")
         .build_roof("tiles")
         .add_windows(8)
         .build())
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

original = Monster("Goblin", 50, 10)
clones = [original.clone() for _ in range(10)]
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

logger = Logger()        # zawsze ta sama instancja
logger2 = Logger()
print(logger is logger2)  # True
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

old_system = OldPaymentSystem()
adapter = PaymentAdapter(old_system)
adapter.make_payment(100)   # używa nowego interfejsu
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

circle = Circle(VectorRenderer())
circle.draw()
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

folder = Composite()
folder.add(Leaf())
folder.add(Composite())   # drzewo
folder.operation()
```

- **Decorator** – dynamiczne dodawanie zachowań
```python
class Coffee:
    def cost(self): 
        return 5
    def description(self): 
        return "Coffee"

# Klasa bazowa dla dodatków (ułatwia tworzenie kolejnych dekoratorów)
class CoffeeDecorator(Coffee):
    def __init__(self, coffee):
        self.coffee = coffee

# Dekorator dodający mleko
class MilkDecorator(CoffeeDecorator):
    def cost(self): 
        return self.coffee.cost() + 2
    def description(self): 
        return self.coffee.description() + " + Milk"

# Dekorator dodający cukier (brakujący element w Twoim kodzie)
class SugarDecorator(CoffeeDecorator):
    def cost(self): 
        return self.coffee.cost() + 1
    def description(self): 
        return self.coffee.description() + " + Sugar"


# Przygotowanie zamówienia: Kawa bazowa -> Cukier -> Mleko
coffee = MilkDecorator(SugarDecorator(Coffee()))

# Wyświetlenie wyników
print(coffee.description())   # Wynik: Coffee + Sugar + Milk
print(coffee.cost())          # Wynik: 8 (5 za kawę + 1 za cukier + 2 za mleko)

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

theater = HomeTheaterFacade(lights, projector, sound)
theater.watch_movie()   # jedna metoda zamiast wielu
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

image = ImageProxy("photo.jpg")
image.display()   # ładuje się dopiero przy pierwszym wywołaniu
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

tree1 = TreeFactory.get_tree("Pine", "green")
tree2 = TreeFactory.get_tree("Pine", "green")   # ten sam obiekt
```

### Behawioralne
- **Chain of Responsibility** – łańcuch handlerów
```python
from abc import ABC, abstractmethod

# 1. Klasa bazowa dla obsługi żądań
class Handler(ABC):
    def __init__(self, next_handler=None):
        self.next = next_handler

    @abstractmethod
    def handle(self, request):
        # Jeśli istnieje kolejny element w łańcuchu, przekaż mu żądanie
        if self.next: 
            return self.next.handle(request)
        return "Żądanie przetworzone pomyślnie!"

# 2. Konkretne klasy Middleware (odpowiedzialności)
class ValidationMiddleware(Handler):
    def handle(self, request):
        print("[Walidacja] Sprawdzanie poprawności danych...")
        if "data" not in request:
            return "Błąd: Brak wymaganych danych w żądaniu."
        return super().handle(request)

class RateLimitMiddleware(Handler):
    def handle(self, request):
        print("[Rate Limit] Sprawdzanie limitu zapytań...")
        if request.get("clicks", 0) > 100:
            return "Błąd: Przekroczono limit zapytań (429 Too Many Requests)."
        return super().handle(request)

class AuthMiddleware(Handler):
    def handle(self, request):
        print("[Autoryzacja] Weryfikacja tokenu użytkownika...")
        if not request.get("is_authenticated", False):
            return "Błąd: Brak autoryzacji (401 Unauthorized)."
        return super().handle(request)

# 3. Przygotowanie przykładowego żądania (request)
request = {
    "is_authenticated": True,
    "clicks": 50,
    "data": "Hello World"
}

# 4. Budowanie i uruchomienie łańcucha (Pipeline)
# Żądanie idzie od lewej do prawej: Auth -> RateLimit -> Validation
pipeline = AuthMiddleware(RateLimitMiddleware(ValidationMiddleware()))
wynik = pipeline.handle(request)

print(f"\nStatus końcowy: {wynik}")
```

- **Command** – akcja jako obiekt (`execute` + `undo`)
```python
from abc import ABC, abstractmethod

# 1. Interfejs Command
class Command(ABC):
    @abstractmethod
    def execute(self): 
        pass
    @abstractmethod
    def undo(self): 
        pass

# 2. Odbiorca (Receiver) - obiekt, na którym wykonywane są operacje
class Light:
    def on(self): 
        return "Światło zostało włączone."
    def off(self): 
        return "Światło zostało wyłączone."

# 3. Konkretne polecenie (Concrete Command)
class LightOnCommand(Command):
    def __init__(self, light): 
        self.light = light
    def execute(self): 
        return self.light.on()
    def undo(self): 
        return self.light.off()

# 4. Wywoływacz (Invoker) - pilot pamiętający historię poleceń
class RemoteControl:
    def __init__(self):
        self._history = []  # Stos do przechowywania wykonanych poleceń

    def submit(self, command):
        # Wykonaj polecenie i dodaj je do historii
        wynik = command.execute()
        print(f"[AKCJA] {wynik}")
        self._history.append(command)

    def undo_last(self):
        # Jeśli historia nie jest pusta, cofnij ostatnie polecenie
        if self._history:
            ostatnie_polecenie = self._history.pop()
            wynik = ostatnie_polecenie.undo()
            print(f"[COFNIJ] {wynik}")
        else:
            print("[INFO] Brak operacji do cofnięcia.")

# 5. Uruchomienie kodu
light = Light()
remote = RemoteControl()

# Wykonanie polecenia
remote.submit(LightOnCommand(light))  # Wynik: Światło zostało włączone.

# Cofnięcie ostatniego polecenia
remote.undo_last()                    # Wynik: Światło zostało wyłączone.
```

- **Iterator**
```python
from abc import ABC, abstractmethod

# 1. Klasyczny interfejs (opcjonalny w Pythonie, ale definiujący kontrakt)
class Iterator(ABC):
    @abstractmethod
    def __next__(self): 
        """Zwraca kolejny element lub podnosi StopIteration"""
        pass

# 2. Konkretny Iterator dla kolekcji
class RadosnyIterator(Iterator):
    def __init__(self, elementy):
        self._elementy = elementy
        self._indeks = 0

    def __next__(self):
        # Python nie używa has_next(). Brak elementów zgłasza się wyjątkiem StopIteration
        if self._indeks < len(self._elementy):
            wynik = self._elementy[self._indeks]
            self._indeks += 1
            return wynik
        raise StopIteration

# 3. Kolekcja (Iterable) - obiekt, po którym można iterować
class MojaKolekcja:
    def __init__(self):
        self.lista_elementow = ["Kawa", "Kod", "Spanie"]

    def __iter__(self):
        # Metoda __iter__ musi zwrócić obiekt iteratora
        return RadosnyIterator(self.lista_elementow)


# 4. Uruchomienie kodu
my_collection = MojaKolekcja()

# Pętla for automatycznie wywoła __iter__(), a potem __next__() aż do StopIteration
for item in my_collection:
    print(item)

```

- **Mediator**
```python
class ChatRoom:
    def __init__(self):
        self.users = []

    def join(self, user):
        """Dodaje użytkownika do pokoju czatowego"""
        self.users.append(user)
        user.chat_room = self

    def broadcast(self, sender, message):
        """Rozsyła wiadomość do wszystkich oprócz nadawcy"""
        for user in self.users:
            if user != sender:
                user.receive(sender.name, message)

class User:
    def __init__(self, name):
        self.name = name
        self.chat_room = None

    def send(self, message):
        """Użytkownik wysyła wiadomość przez mediatora (pokój)"""
        if self.chat_room:
            print(f"[{self.name} wysyła]: {message}")
            self.chat_room.broadcast(self, message)
        else:
            print(f"{self.name} nie jest w żadnym pokoju!")

    def receive(self, sender_name, message):
        """Metoda wywoływana przez pokój, gdy ktoś inny wyśle wiadomość"""
        print(f"  -> [{self.name} odebrał od {sender_name}]: {message}")


# 1. Tworzenie mediatora i użytkowników
chat = ChatRoom()
user1 = User("Anna")
user2 = User("Bartek")
user3 = User("Cezary")

# 2. Dołączanie użytkowników do czatu
chat.join(user1)
chat.join(user2)
chat.join(user3)

# 3. Wywołanie wysyłania wiadomości
user1.send("Cześć wszystkim!")

```

- **Memento** – snapshot stanu
```python
# 1. Pamiątka (Memento) - przechowuje stan obiektu
class Memento:
    def __init__(self, state):
        self._state = state  # Stan jest ukryty (enkapsulacja)

    def get_state(self):
        return self._state


# 2. Inicjator (Originator / TextEditor) - obiekt posiadający stan
class TextEditor:
    def __init__(self):
        self.state = "pusta strona"

    def wpisz_tekst(self, nowy_tekst):
        self.state = nowy_tekst

    def save(self):
        """Tworzy pamiątkę z aktualnym stanem"""
        print(f"[ZAPIS] Zapisuję stan: '{self.state}'")
        return Memento(self.state)

    def restore(self, memento):
        """Przywraca stan z pamiątki"""
        self.state = memento.get_state()
        print(f"[PRZYWRÓCENIE] Stan przywrócony do: '{self.state}'")


# 3. Zarządca (Caretaker) - pilnuje historii pamiątek (np. historia Ctrl+Z)
class History:
    def __init__(self):
        self._snapshots = []

    def push(self, memento):
        self._snapshots.append(memento)

    def pop(self):
        if self._snapshots:
            return self._snapshots.pop()
            
# 4. Uruchomienie kodu
editor = TextEditor()
history = History()

# Modyfikacja stanu i zapis do historii
editor.wpisz_tekst("Napisany artykuł")
history.push(editor.save())  # Zapisujemy stan "Napisany artykuł"

# Kolejna modyfikacja (np. przypadkowe usunięcie tekstu)
editor.wpisz_tekst("Wszystko skasowane!")
print(f"Aktualny stan edytora: '{editor.state}'")

# Cofanie zmian (Ctrl+Z) przy użyciu pamiątki
snapshot = history.pop()
editor.restore(snapshot)

```

- **Observer** – pub/sub
```python
from abc import ABC, abstractmethod

# 1. Interfejs Obserwatora
class Observer(ABC):
    @abstractmethod
    def update(self, event): 
        pass

# 2. Klasa bazowa Podmiotu (Subject)
class Subject:
    def __init__(self):
        self.observers = []

    def subscribe(self, observer):
        """Rejestracja nowego obserwatora"""
        if observer not in self.observers:
            self.observers.append(observer)

    def unsubscribe(self, observer):
        """Wyrejestrowanie obserwatora"""
        self.observers.remove(observer)

    def notify(self, event):
        """Powiadomienie wszystkich obserwatorów o zdarzeniu"""
        for obs in self.observers:
            obs.update(event)

# 3. Konkretny Podmiot - Czujnik Ruchu
class MotionSensor(Subject):
    def detect(self):
        event_message = "Wykryto ruch w salonie!"
        print(f"\n[CZUJNIK] {event_message}")
        # Wywołanie powiadomienia odziedziczonego z klasy Subject
        self.notify(event_message)

# 4. Konkretni Obserwatorzy
class AlarmObserver(Observer):
    def update(self, event):
        print(f"  -> [ALARM] Uruchamiam syrenę! Powód: {event}")

class LightObserver(Observer):
    def update(self, event):
        print(f"  -> [ŚWIATŁO] Włączam oświetlenie awaryjne! Powód: {event}")


# 5. Uruchomienie kodu
sensor = MotionSensor()

# Subskrypcja systemów
sensor.subscribe(AlarmObserver())
sensor.subscribe(LightObserver())

# Wyzwolenie zdarzenia
sensor.detect()

```

- **State** – zachowanie zależne od stanu
```python
from abc import ABC, abstractmethod

# 1. Interfejs Stanu
class State(ABC):
    @abstractmethod
    def press(self, player): 
        """Przyjmuje obiekt player, aby móc zmienić jego stan"""
        pass

# 2. Konkretne Stany
class OffState(State):
    def press(self, player):
        print("[AKCJA] Włączanie urządzenia...")
        player.set_state(OnState())

class OnState(State):
    def press(self, player):
        print("[AKCJA] Przełączanie w tryb czuwania (Standby)...")
        player.set_state(StandbyState())

class StandbyState(State):
    def press(self, player):
        print("[AKCJA] Wyłączanie całkowite...")
        player.set_state(OffState())

# 3. Kontekst (Odtwarzacz)
class Player:
    def __init__(self):
        # Odtwarzacz zaczyna w stanie OffState
        self._state = OffState()
        print(f"Inicjalizacja: Player jest w stanie {type(self._state).__name__}")

    def set_state(self, state):
        """Metoda pozwalająca stanom na zmianę stanu odtwarzacza"""
        self._state = state
        print(f" -> Nowy stan: {type(self._state).__name__}")

    def press(self):
        """Delegowanie zachowania do aktualnego obiektu stanu"""
        self._state.press(self)


# 4. Uruchomienie kodu (zgodnie z Twoim scenariuszem)
player = Player()   # Zaczyna w OffState

player.press()      # OffState -> OnState
player.press()      # OnState -> StandbyState
player.press()      # StandbyState -> OffState (zamknięcie pętli stanów)

```

- **Strategy** – wymienny algorytm
```python
from abc import ABC, abstractmethod

# 1. Interfejs Strategii Grzewczej
class HeatingStrategy(ABC):
    @abstractmethod
    def calculate_target_temp(self, current_temp):
        pass

# 2. Konkretne Strategie
class EcoStrategy(HeatingStrategy):
    def calculate_target_temp(self, current_temp):
        print("[Strategia: ECO] Oszczędzanie energii.")
        return 19.0  # Niska, ekonomiczna temperatura docelowa

class ComfortStrategy(HeatingStrategy):
    def calculate_target_temp(self, current_temp):
        print("[Strategia: COMFORT] Maksymalny komfort domowników.")
        return 22.5  # Wyższa, komfortowa temperatura docelowa

# 3. Kontekst (Grzejnik / Heater)
class Heater:
    def __init__(self, strategy: HeatingStrategy):
        self._strategy = strategy  # Wstrzyknięcie domyślnej strategii

    def set_strategy(self, strategy: HeatingStrategy):
        """Dynamiczna zmiana strategii w czasie działania programu"""
        print(f"\n[SYSTEM] Zmiana trybu pracy na {type(strategy).__name__}")
        self._strategy = strategy

    def run(self, current_temp):
        """Wykonanie algorytmu wybranej strategii"""
        target_temp = self._strategy.calculate_target_temp(current_temp)
        print(f"Aktualna temperatura: {current_temp}°C")
        print(f"Temperatura docelowa: {target_temp}°C")
        
        if current_temp < target_temp:
            print(">> STATUS: Grzejnik WŁĄCZONY (Grzanie...)")
        else:
            print(">> STATUS: Grzejnik WYŁĄCZONY (Temperatura osiągnięta)")


# 4. Uruchomienie kodu (zgodnie z Twoim scenariuszem)
current_temp = 20.0

# Inicjalizacja grzejnika ze strategią Eco
heater = Heater(EcoStrategy())
heater.run(current_temp)

# Dynamiczna zmiana strategii na Comfort w trakcie pracy
heater.set_strategy(ComfortStrategy())
heater.run(current_temp)

```

- **Template Method**
```python
from abc import ABC, abstractmethod

# 1. Klasa bazowa definiująca szkielet algorytmu
class BaseEtlJob(ABC):
    def run(self):
        """Metoda Szablonowa - definiuje niezmienny szkielet procesu"""
        print(f"=== Uruchamianie procesu ETL: {self.__class__.__name__} ===")
        self.extract()
        self.transform()
        self.load()
        print("=== Proces zakończony sukcesem ===\n")

    @abstractmethod
    def extract(self):
        pass

    @abstractmethod
    def transform(self):
        pass

    @abstractmethod
    def load(self):
        pass


# 2. Konkretna implementacja dla danych sprzedażowych
class SalesEtlJob(BaseEtlJob):
    def extract(self):
        print("[Krok 1/3] Pobieranie danych sprzedażowych z bazy SQL...")

    def transform(self):
        print("[Krok 2/3] Czyszczenie danych: konwersja walut i usuwanie duplikatów...")

    def load(self):
        print("[Krok 3/3] Ładowanie przetworzonych danych do Hurtowni Danych...")


# 3. Inna przykładowa implementacja (pokazuje elastyczność wzorca)
class MarketingEtlJob(BaseEtlJob):
    def extract(self):
        print("[Krok 1/3] Pobieranie logów z API Google Analytics...")

    def transform(self):
        print("[Krok 2/3] Agregacja kliknięć i obliczanie współczynnika konwersji...")

    def load(self):
        print("[Krok 3/3] Zapisywanie raportu do pliku CSV na serwerze S3...")


# 4. Uruchomienie kodu (zgodnie z Twoim scenariuszem)
etl_job = SalesEtlJob()
etl_job.run()

# Testowanie drugiego zadania dla porównania
marketing_job = MarketingEtlJob()
marketing_job.run()

```

- **Visitor**
```python
from abc import ABC, abstractmethod

# 1. Interfejs Wizytatora (Visitor)
class Visitor(ABC):
    @abstractmethod
    def visit_circle(self, circle): pass

    @abstractmethod
    def visit_square(self, square): pass


# 2. Interfejs Elementu akceptującego wizytę
class Element(ABC):
    @abstractmethod
    def accept(self, visitor: Visitor): pass


# 3. Konkretne Elementy struktury danych (np. figury geometryczne jako węzły AST)
class Circle(Element):
    def __init__(self, radius):
        self.radius = radius

    def accept(self, visitor: Visitor):
        # Podwójne rozproszenie: element wywołuje dedykowaną metodę wizytatora
        visitor.visit_circle(self)


class Square(Element):
    def __init__(self, side):
        self.side = side

    def accept(self, visitor: Visitor):
        visitor.visit_square(self)


# 4. Konkretni Wizytatorzy (Operacje wykonywane na strukturze)
class CodeGeneratorVisitor(Visitor):
    def visit_circle(self, circle):
        print(f"[Generator] Generuję kod maszynowy dla koła o promieniu {circle.radius}")

    def visit_square(self, square):
        print(f"[Generator] Generuję kod maszynowy dla kwadratu o boku {square.side}")


class PrettyPrinterVisitor(Visitor):
    def visit_circle(self, circle):
        print(f"[PrettyPrinter] ◯ Koło (r = {circle.radius})")

    def visit_square(self, square):
        print(f"[PrettyPrinter] ☐ Kwadrat (a = {square.side})")


# 5. Uruchomienie kodu (Przetwarzanie węzła drzewa AST)
# Tworzymy przykładowy węzeł (strukturę)
ast_node = Circle(radius=5)

# Przekazujemy strukturę do pierwszego wizytatora
ast_node.accept(CodeGeneratorVisitor())

# Przekazujemy tę samą strukturę do drugiego wizytatora
ast_node.accept(PrettyPrinterVisitor())

```

- **Interpreter**
```python
from abc import ABC, abstractmethod

# 1. Interfejs Wyrażenia (Expression)
class Expression(ABC):
    @abstractmethod
    def interpret(self, context: dict) -> bool:
        """Interpretuje wyrażenie w odniesieniu do podanego kontekstu (wiersza danych)"""
        pass

    def matches(self, row: dict) -> bool:
        """Wygodny alias zgodny z Twoim wywołaniem"""
        return self.interpret(row)


# 2. Wyrażenia Terminalne (Terminal Expressions) - liście drzewa składniowego
class Variable(Expression):
    """Reprezentuje nazwę kolumny/klucza w słowniku (np. 'status', 'total')"""
    def __init__(self, name):
        self.name = name

    def interpret(self, context: dict):
        return context.get(self.name)


class Constant(Expression):
    """Reprezentuje stałą wartość (np. 'paid', 100)"""
    def __init__(self, value):
        self.value = value

    def interpret(self, context: dict):
        return self.value


# 3. Wyrażenia Nieterminalne (Non-terminal Expressions) - węzły łączące inne wyrażenia
class Equals(Expression):
    """Sprawdza, czy wartość zmiennej jest równa stałej"""
    def __init__(self, variable_name: str, constant_value):
        self.variable = Variable(variable_name)
        self.constant = Constant(constant_value)

    def interpret(self, context: dict) -> bool:
        return self.variable.interpret(context) == self.constant.interpret(context)


class GreaterThan(Expression):
    """Sprawdza, czy wartość zmiennej jest większa od stałej"""
    def __init__(self, variable_name: str, constant_value):
        self.variable = Variable(variable_name)
        self.constant = Constant(constant_value)

    def interpret(self, context: dict) -> bool:
        try:
            return self.variable.interpret(context) > self.constant.interpret(context)
        except TypeError:
            return False


class And(Expression):
    """Koniunkcja logiczna - oba warunki muszą być prawdziwe"""
    def __init__(self, left: Expression, right: Expression):
        self.left = left
        self.right = right

    def interpret(self, context: dict) -> bool:
        return self.left.interpret(context) and self.right.interpret(context)


# 4. Budowanie zapytania (Zgodnie z Twoim scenariuszem)
query = And(Equals("status", "paid"), GreaterThan("total", 100))

# 5. Przygotowanie przykładowych danych (Kontekst)
row_valid = {"status": "paid", "total": 250, "user": "Jan"}
row_invalid = {"status": "pending", "total": 150, "user": "Anna"}

# 6. Testowanie interpretera
print(f"Czy row_valid spełnia warunki? -> {query.matches(row_valid)}")
# Wynik: True (status == paid ORAZ total > 100)

print(f"Czy row_invalid spełnia warunki? -> {query.matches(row_invalid)}")
# Wynik: False (status != paid)

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