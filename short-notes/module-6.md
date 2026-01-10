# Module 6: Design Patterns & Clean Code

## SOLID Principles

### Single Responsibility Principle (SRP)

**What It Means**

A class or module should have one, and only one, reason to change. This means it should have a single responsibility or job. When a class has multiple responsibilities, it becomes harder to maintain, test, and understand. SRP promotes separation of concerns and makes code more modular and easier to refactor.

```javascript
// Violation: Multiple responsibilities
class User {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }

  save() {
    // Database logic
    console.log(`Saving ${this.name} to database`);
  }

  sendEmail() {
    // Email logic
    console.log(`Sending email to ${this.email}`);
  }

  validate() {
    // Validation logic
    return this.name.length > 0 && this.email.includes('@');
  }
}

// Corrected: Single responsibility per class
class User {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }

  validate() {
    return this.name.length > 0 && this.email.includes('@');
  }
}

class UserRepository {
  save(user) {
    console.log(`Saving ${user.name} to database`);
  }
}

class EmailService {
  sendEmail(user, message) {
    console.log(`Sending email to ${user.email}: ${message}`);
  }
}
```

**Practical Tips**

- Identify different reasons a class might change and separate them
- Name classes based on their single responsibility
- Use composition to combine small, focused classes
- Apply SRP at multiple levels: functions, classes, modules

---

### Open/Closed Principle (OCP)

**What It Means**

Software entities should be open for extension but closed for modification. This means you should be able to add new functionality without changing existing code. Achieve this through abstraction, polymorphism, and dependency injection. OCP reduces the risk of introducing bugs when adding features.

```javascript
// Violation: Must modify to add new shapes
class AreaCalculator {
  calculate(shape) {
    if (shape.type === 'circle') {
      return Math.PI * shape.radius * shape.radius;
    } else if (shape.type === 'rectangle') {
      return shape.width * shape.height;
    }
    // Must add more if-else for new shapes
  }
}

// Corrected: Open for extension, closed for modification
class Shape {
  area() {
    throw new Error('Must implement area()');
  }
}

class Circle extends Shape {
  constructor(radius) {
    super();
    this.radius = radius;
  }

  area() {
    return Math.PI * this.radius * this.radius;
  }
}

class Rectangle extends Shape {
  constructor(width, height) {
    super();
    this.width = width;
    this.height = height;
  }

  area() {
    return this.width * this.height;
  }
}

class AreaCalculator {
  calculate(shapes) {
    return shapes.reduce((total, shape) => total + shape.area(), 0);
  }
}

// Can add new shapes without modifying AreaCalculator
class Triangle extends Shape {
  constructor(base, height) {
    super();
    this.base = base;
    this.height = height;
  }

  area() {
    return 0.5 * this.base * this.height;
  }
}
```

**Practical Tips**

- Use abstract base classes or interfaces
- Leverage polymorphism for extensibility
- Apply the Strategy pattern for interchangeable behaviors
- Use dependency injection to swap implementations

---

### Liskov Substitution Principle (LSP)

**What It Means**

Objects of a superclass should be replaceable with objects of a subclass without affecting the correctness of the program. Subclasses must honor the contract established by their parent class. Violations occur when subclasses weaken preconditions, strengthen postconditions, or throw unexpected exceptions.

```javascript
// Violation: Square breaks Rectangle contract
class Rectangle {
  constructor(width, height) {
    this.width = width;
    this.height = height;
  }

  setWidth(width) {
    this.width = width;
  }

  setHeight(height) {
    this.height = height;
  }

  area() {
    return this.width * this.height;
  }
}

class Square extends Rectangle {
  constructor(side) {
    super(side, side);
  }

  setWidth(width) {
    this.width = width;
    this.height = width; // Breaks Rectangle contract
  }

  setHeight(height) {
    this.height = height;
    this.width = height; // Breaks Rectangle contract
  }
}

// Corrected: Separate classes with common interface
class Shape {
  area() {
    throw new Error('Must implement area()');
  }
}

class Rectangle extends Shape {
  constructor(width, height) {
    super();
    this.width = width;
    this.height = height;
  }

  area() {
    return this.width * this.height;
  }
}

class Square extends Shape {
  constructor(side) {
    super();
    this.side = side;
  }

  area() {
    return this.side * this.side;
  }
}
```

**Practical Tips**

- Ensure subclasses don't violate parent class invariants
- Avoid overriding methods to change behavior fundamentally
- Use composition over inheritance when appropriate
- Write tests that verify LSP compliance

---

### Interface Segregation Principle (ISP)

**What It Means**

Clients should not be forced to depend on interfaces they don't use. Create small, focused interfaces rather than large, general-purpose ones. ISP reduces coupling and makes code more flexible by preventing clients from depending on unused methods.

```javascript
// Violation: Fat interface with unused methods
class Worker {
  work() {
    throw new Error('Must implement work()');
  }

  eat() {
    throw new Error('Must implement eat()');
  }

  sleep() {
    throw new Error('Must implement sleep()');
  }
}

class Robot extends Worker {
  work() {
    console.log('Robot working');
  }

  eat() {
    // Robot doesn't eat, forced to implement
    throw new Error('Robots don\'t eat');
  }

  sleep() {
    // Robot doesn't sleep, forced to implement
    throw new Error('Robots don\'t sleep');
  }
}

// Corrected: Segregated interfaces
class Workable {
  work() {
    throw new Error('Must implement work()');
  }
}

class Eatable {
  eat() {
    throw new Error('Must implement eat()');
  }
}

class Sleepable {
  sleep() {
    throw new Error('Must implement sleep()');
  }
}

class Human {
  work() {
    console.log('Human working');
  }

  eat() {
    console.log('Human eating');
  }

  sleep() {
    console.log('Human sleeping');
  }
}

class Robot {
  work() {
    console.log('Robot working');
  }
}
```

**Practical Tips**

- Split large interfaces into smaller, cohesive ones
- Use composition to combine multiple interfaces
- Apply ISP at the function level too (avoid passing unused parameters)
- Prefer role-based interfaces over implementation-based ones

---

### Dependency Inversion Principle (DIP)

**What It Means**

High-level modules should not depend on low-level modules. Both should depend on abstractions. Abstractions should not depend on details. Details should depend on abstractions. This principle promotes loose coupling and makes systems easier to test and maintain.

```javascript
// Violation: High-level depends on low-level
class LightBulb {
  turnOn() {
    console.log('Light bulb on');
  }

  turnOff() {
    console.log('Light bulb off');
  }
}

class Switch {
  constructor(bulb) {
    this.bulb = bulb;
    this.on = false;
  }

  toggle() {
    if (this.on) {
      this.bulb.turnOff();
      this.on = false;
    } else {
      this.bulb.turnOn();
      this.on = true;
    }
  }
}

// Corrected: Depend on abstraction
class Switchable {
  turnOn() {
    throw new Error('Must implement turnOn()');
  }

  turnOff() {
    throw new Error('Must implement turnOff()');
  }
}

class LightBulb extends Switchable {
  turnOn() {
    console.log('Light bulb on');
  }

  turnOff() {
    console.log('Light bulb off');
  }
}

class Fan extends Switchable {
  turnOn() {
    console.log('Fan on');
  }

  turnOff() {
    console.log('Fan off');
  }
}

class Switch {
  constructor(device) {
    this.device = device;
    this.on = false;
  }

  toggle() {
    if (this.on) {
      this.device.turnOff();
      this.on = false;
    } else {
      this.device.turnOn();
      this.on = true;
    }
  }
}

// Can switch any Switchable device
const lightSwitch = new Switch(new LightBulb());
const fanSwitch = new Switch(new Fan());
```

**Practical Tips**

- Depend on interfaces or abstract classes, not concrete implementations
- Use dependency injection to provide dependencies
- Apply the Factory pattern for creating dependencies
- Write tests using mock implementations of abstractions

---

## Creational Patterns

### Singleton Pattern

**Problem It Solves**

Ensures a class has only one instance and provides a global point of access to it. Useful for shared resources like database connections, configuration objects, logging services, and caches.

```javascript
// Basic Singleton implementation
class Database {
  constructor() {
    if (Database.instance) {
      return Database.instance;
    }
    this.connection = 'Connected to database';
    Database.instance = this;
  }

  static getInstance() {
    if (!Database.instance) {
      Database.instance = new Database();
    }
    return Database.instance;
  }

  query(sql) {
    console.log(`Executing: ${sql}`);
  }
}

const db1 = Database.getInstance();
const db2 = Database.getInstance();
console.log(db1 === db2); // true

// Module pattern for Singleton (Node.js)
const Logger = (() => {
  let instance;

  function createInstance() {
    return {
      logs: [],
      log(message) {
        this.logs.push({ timestamp: Date.now(), message });
        console.log(`[LOG] ${message}`);
      },
      getLogs() {
        return this.logs;
      }
    };
  }

  return {
    getInstance() {
      if (!instance) {
        instance = createInstance();
      }
      return instance;
    }
  };
})();

const logger1 = Logger.getInstance();
const logger2 = Logger.getInstance();
logger1.log('First message');
logger2.log('Second message');
console.log(logger1.getLogs()); // Both messages
```

**When to Use**

- Shared resource management (database connections, thread pools)
- Configuration or settings objects
- Logging services
- Caching mechanisms
- Hardware interface access

**Pros and Cons**

| Pros | Cons |
|------|------|
| Controlled access to sole instance | Global state can make testing difficult |
| Lazy initialization (only when needed) | Violates Single Responsibility Principle |
| Reduced namespace pollution | Can hide dependencies |
| Easy to implement | Not thread-safe by default in some languages |

---

### Factory Pattern

**Problem It Solves**

Provides an interface for creating objects without specifying their exact classes. Delegates object creation to subclasses or factory methods. Useful when you need to create objects based on runtime conditions or when object creation logic is complex.

```javascript
// Simple Factory
class Car {
  constructor(model) {
    this.model = model;
  }

  drive() {
    console.log(`${this.model} is driving`);
  }
}

class Truck {
  constructor(model) {
    this.model = model;
  }

  drive() {
    console.log(`${this.model} is driving`);
  }
}

class VehicleFactory {
  static createVehicle(type, model) {
    switch (type) {
      case 'car':
        return new Car(model);
      case 'truck':
        return new Truck(model);
      default:
        throw new Error('Unknown vehicle type');
    }
  }
}

const car = VehicleFactory.createVehicle('car', 'Toyota');
const truck = VehicleFactory.createVehicle('truck', 'Ford');
car.drive(); // Toyota is driving
truck.drive(); // Ford is driving

// Factory Method Pattern
class Document {
  constructor() {
    this.content = '';
  }

  createPage() {
    throw new Error('Must implement createPage()');
  }

  addContent(text) {
    this.content += text;
  }
}

class Resume extends Document {
  createPage() {
    return new ResumePage();
  }
}

class Report extends Document {
  createPage() {
    return new ReportPage();
  }
}

class Page {
  render() {
    throw new Error('Must implement render()');
  }
}

class ResumePage extends Page {
  render() {
    return '<div class="resume-page"></div>';
  }
}

class ReportPage extends Page {
  render() {
    return '<div class="report-page"></div>';
  }
}

// Abstract Factory Pattern
class GUIFactory {
  createButton() {
    throw new Error('Must implement createButton()');
  }

  createCheckbox() {
    throw new Error('Must implement createCheckbox()');
  }
}

class WindowsFactory extends GUIFactory {
  createButton() {
    return new WindowsButton();
  }

  createCheckbox() {
    return new WindowsCheckbox();
  }
}

class MacFactory extends GUIFactory {
  createButton() {
    return new MacButton();
  }

  createCheckbox() {
    return new MacCheckbox();
  }
}

class Button {
  render() {
    throw new Error('Must implement render()');
  }
}

class Checkbox {
  render() {
    throw new Error('Must implement render()');
  }
}

class WindowsButton extends Button {
  render() {
    return '<button style="windows">Click</button>';
  }
}

class MacButton extends Button {
  render() {
    return '<button style="mac">Click</button>';
  }
}

class WindowsCheckbox extends Checkbox {
  render() {
    return '<input type="checkbox" style="windows" />';
  }
}

class MacCheckbox extends Checkbox {
  render() {
    return '<input type="checkbox" style="mac" />';
  }
}

// Usage
function createUI(factory) {
  const button = factory.createButton();
  const checkbox = factory.createCheckbox();
  return button.render() + checkbox.render();
}

const windowsUI = createUI(new WindowsFactory());
const macUI = createUI(new MacFactory());
```

**When to Use**

- Object creation logic is complex or should be centralized
- You need to create objects based on runtime conditions
- You want to decouple object creation from usage
- You need to support multiple product families (Abstract Factory)

**Pros and Cons**

| Pros | Cons |
|------|------|
| Decouples creation from usage | Increases complexity with more classes |
| Open/Closed Principle - easy to add new types | Can lead to many small classes |
| Centralized object creation logic | May require type casting |
| Supports dependency injection | Can be overkill for simple cases |

---

### Builder Pattern

**Problem It Solves**

Separates the construction of complex objects from their representation. Allows you to construct objects step-by-step with different configurations. Useful for objects with many optional parameters or complex initialization logic.

```javascript
// Builder Pattern for complex object construction
class House {
  constructor() {
    this.walls = 0;
    this.doors = 0;
    this.windows = 0;
    this.hasGarage = false;
    this.hasPool = false;
    this.hasGarden = false;
  }
}

class HouseBuilder {
  constructor() {
    this.house = new House();
  }

  buildWalls(count) {
    this.house.walls = count;
    return this;
  }

  buildDoors(count) {
    this.house.doors = count;
    return this;
  }

  buildWindows(count) {
    this.house.windows = count;
    return this;
  }

  addGarage() {
    this.house.hasGarage = true;
    return this;
  }

  addPool() {
    this.house.hasPool = true;
    return this;
  }

  addGarden() {
    this.house.hasGarden = true;
    return this;
  }

  build() {
    return this.house;
  }
}

// Usage
const house = new HouseBuilder()
  .buildWalls(4)
  .buildDoors(2)
  .buildWindows(6)
  .addGarage()
  .addGarden()
  .build();

// Builder for SQL queries
class SQLQueryBuilder {
  constructor(table) {
    this.table = table;
    this.select = ['*'];
    this.where = [];
    this.orderBy = null;
    this.limit = null;
  }

  selectFields(fields) {
    this.select = fields;
    return this;
  }

  whereClause(condition) {
    this.where.push(condition);
    return this;
  }

  orderByField(field, direction = 'ASC') {
    this.orderBy = `${field} ${direction}`;
    return this;
  }

  limitResults(count) {
    this.limit = count;
    return this;
  }

  build() {
    let query = `SELECT ${this.select.join(', ')} FROM ${this.table}`;

    if (this.where.length > 0) {
      query += ` WHERE ${this.where.join(' AND ')}`;
    }

    if (this.orderBy) {
      query += ` ORDER BY ${this.orderBy}`;
    }

    if (this.limit) {
      query += ` LIMIT ${this.limit}`;
    }

    return query;
  }
}

const query = new SQLQueryBuilder('users')
  .selectFields(['id', 'name', 'email'])
  .whereClause('age > 18')
  .whereClause('status = "active"')
  .orderByField('name', 'ASC')
  .limitResults(10)
  .build();

console.log(query);
// SELECT id, name, email FROM users WHERE age > 18 AND status = "active" ORDER BY name ASC LIMIT 10
```

**When to Use**

- Objects have many optional parameters
- Object construction involves multiple steps
- You need different representations of the same construction process
- You want to separate construction logic from the object's representation

**Pros and Cons**

| Pros | Cons |
|------|------|
| Clear, readable construction code | Increases number of classes |
| Supports step-by-step construction | Can be overkill for simple objects |
| Encapsulates complex construction logic | Builder must be mutable |
| Can create multiple representations | May violate immutability |

---

### Prototype Pattern

**Problem It Solves**

Creates new objects by cloning existing ones instead of creating from scratch. Useful when object creation is expensive or when you need to create objects with similar initial states. Avoids the overhead of subclassing.

```javascript
// Prototype Pattern implementation
class Prototype {
  clone() {
    throw new Error('Must implement clone()');
  }
}

class Shape extends Prototype {
  constructor(color) {
    super();
    this.color = color;
  }

  clone() {
    return new Shape(this.color);
  }
}

class Circle extends Shape {
  constructor(color, radius) {
    super(color);
    this.radius = radius;
  }

  clone() {
    return new Circle(this.color, this.radius);
  }

  draw() {
    console.log(`Drawing ${this.color} circle with radius ${this.radius}`);
  }
}

class Rectangle extends Shape {
  constructor(color, width, height) {
    super(color);
    this.width = width;
    this.height = height;
  }

  clone() {
    return new Rectangle(this.color, this.width, this.height);
  }

  draw() {
    console.log(`Drawing ${this.color} rectangle ${this.width}x${this.height}`);
  }
}

// Prototype registry
class ShapeRegistry {
  constructor() {
    this.shapes = new Map();
  }

  addShape(key, shape) {
    this.shapes.set(key, shape);
  }

  getShape(key) {
    const shape = this.shapes.get(key);
    return shape ? shape.clone() : null;
  }
}

const registry = new ShapeRegistry();
registry.addShape('red-circle', new Circle('red', 5));
registry.addShape('blue-rectangle', new Rectangle('blue', 10, 20));

const circle1 = registry.getShape('red-circle');
const circle2 = registry.getShape('red-circle');
const rect1 = registry.getShape('blue-rectangle');

circle1.draw(); // Drawing red circle with radius 5
rect1.draw(); // Drawing blue rectangle 10x20

// Deep clone for complex objects
function deepClone(obj) {
  if (obj === null || typeof obj !== 'object') {
    return obj;
  }

  if (obj instanceof Date) {
    return new Date(obj.getTime());
  }

  if (obj instanceof Array) {
    return obj.map(item => deepClone(item));
  }

  if (obj instanceof Object) {
    const clonedObj = {};
    for (const key in obj) {
      if (obj.hasOwnProperty(key)) {
        clonedObj[key] = deepClone(obj[key]);
      }
    }
    return clonedObj;
  }
}

const original = {
  name: 'John',
  age: 30,
  address: {
    city: 'New York',
    country: 'USA'
  },
  hobbies: ['reading', 'coding']
};

const cloned = deepClone(original);
cloned.address.city = 'Boston';
cloned.hobbies.push('gaming');

console.log(original.address.city); // New York
console.log(cloned.address.city); // Boston
```

**When to Use**

- Object creation is expensive (database queries, file I/O)
- You need to create objects with similar initial states
- You want to avoid subclassing for object creation
- You need to hide concrete classes from clients

**Pros and Cons**

| Pros | Cons |
|------|------|
| Avoids expensive object creation | Deep cloning can be complex |
| Supports dynamic object creation | Can be difficult to implement for circular references |
| Reduces subclassing | May violate immutability |
| Preserves initial state | Clone methods can become complex |

---

## Structural Patterns

### Adapter Pattern

**Problem It Solves**

Allows incompatible interfaces to work together. Wraps an object with a new interface that clients expect. Useful when integrating third-party libraries, legacy code, or systems with incompatible interfaces.

```javascript
// Adapter Pattern for incompatible interfaces
class MediaPlayer {
  play(audioType, filename) {
    throw new Error('Must implement play()');
  }
}

class AdvancedMediaPlayer {
  playVlc(filename) {
    console.log(`Playing vlc file: ${filename}`);
  }

  playMp4(filename) {
    console.log(`Playing mp4 file: ${filename}`);
  }
}

class MediaAdapter extends AdvancedMediaPlayer {
  constructor(audioType) {
    super();
    this.audioType = audioType;
  }

  play(audioType, filename) {
    if (audioType === 'vlc') {
      this.playVlc(filename);
    } else if (audioType === 'mp4') {
      this.playMp4(filename);
    }
  }
}

class AudioPlayer extends MediaPlayer {
  constructor() {
    super();
    this.mediaAdapter = null;
  }

  play(audioType, filename) {
    if (audioType === 'mp3') {
      console.log(`Playing mp3 file: ${filename}`);
    } else if (audioType === 'vlc' || audioType === 'mp4') {
      this.mediaAdapter = new MediaAdapter(audioType);
      this.mediaAdapter.play(audioType, filename);
    } else {
      console.log(`Invalid media. ${audioType} format not supported`);
    }
  }
}

const player = new AudioPlayer();
player.play('mp3', 'song.mp3');
player.play('vlc', 'movie.vlc');
player.play('mp4', 'video.mp4');

// Real-world example: Payment gateway adapter
class StripePaymentGateway {
  processPayment(amount, currency, token) {
    console.log(`Stripe: Processing $${amount} ${currency} with token ${token}`);
    return { success: true, transactionId: 'stripe_123' };
  }
}

class PayPalPaymentGateway {
  makePayment(amount, currency, email) {
    console.log(`PayPal: Processing $${amount} ${currency} for ${email}`);
    return { success: true, transactionId: 'paypal_456' };
  }
}

class PaymentAdapter {
  constructor(gateway) {
    this.gateway = gateway;
  }

  pay(paymentDetails) {
    if (this.gateway instanceof StripePaymentGateway) {
      return this.gateway.processPayment(
        paymentDetails.amount,
        paymentDetails.currency,
        paymentDetails.token
      );
    } else if (this.gateway instanceof PayPalPaymentGateway) {
      return this.gateway.makePayment(
        paymentDetails.amount,
        paymentDetails.currency,
        paymentDetails.email
      );
    }
  }
}

// Usage
const stripeAdapter = new PaymentAdapter(new StripePaymentGateway());
stripeAdapter.pay({ amount: 100, currency: 'USD', token: 'tok_123' });

const paypalAdapter = new PaymentAdapter(new PayPalPaymentGateway());
paypalAdapter.pay({ amount: 50, currency: 'EUR', email: 'user@example.com' });
```

**When to Use**

- Integrating third-party libraries with incompatible interfaces
- Working with legacy code that cannot be modified
- Creating reusable classes that cooperate with unrelated classes
- Needing to use multiple existing subclasses but requiring incompatible interfaces

**Pros and Cons**

| Pros | Cons |
|------|------|
| Allows incompatible interfaces to work together | Increases complexity |
| Promotes reusability of existing functionality | Can introduce many adapter classes |
| Decouples client from implementation | May hide implementation details |
| Follows Open/Closed Principle | Can make code harder to debug |

---

### Decorator Pattern

**Problem It Solves**

Adds behavior to objects dynamically without affecting other objects of the same class. Provides a flexible alternative to subclassing for extending functionality. Useful for adding responsibilities to individual objects at runtime.

```javascript
// Decorator Pattern for adding behavior dynamically
class Coffee {
  cost() {
    return 0;
  }

  description() {
    return '';
  }
}

class SimpleCoffee extends Coffee {
  cost() {
    return 2;
  }

  description() {
    return 'Simple Coffee';
  }
}

class CoffeeDecorator extends Coffee {
  constructor(coffee) {
    super();
    this.coffee = coffee;
  }

  cost() {
    return this.coffee.cost();
  }

  description() {
    return this.coffee.description();
  }
}

class MilkDecorator extends CoffeeDecorator {
  constructor(coffee) {
    super(coffee);
  }

  cost() {
    return this.coffee.cost() + 0.5;
  }

  description() {
    return `${this.coffee.description()}, Milk`;
  }
}

class SugarDecorator extends CoffeeDecorator {
  constructor(coffee) {
    super(coffee);
  }

  cost() {
    return this.coffee.cost() + 0.2;
  }

  description() {
    return `${this.coffee.description()}, Sugar`;
  }
}

class WhippedCreamDecorator extends CoffeeDecorator {
  constructor(coffee) {
    super(coffee);
  }

  cost() {
    return this.coffee.cost() + 0.7;
  }

  description() {
    return `${this.coffee.description()}, Whipped Cream`;
  }
}

// Usage
let coffee = new SimpleCoffee();
console.log(`${coffee.description()}: $${coffee.cost()}`);

coffee = new MilkDecorator(coffee);
console.log(`${coffee.description()}: $${coffee.cost()}`);

coffee = new SugarDecorator(coffee);
console.log(`${coffee.description()}: $${coffee.cost()}`);

coffee = new WhippedCreamDecorator(coffee);
console.log(`${coffee.description()}: $${coffee.cost()}`);

// Real-world example: HTTP request decorators
class HttpRequest {
  constructor(url) {
    this.url = url;
    this.headers = {};
    this.body = null;
  }

  send() {
    console.log(`Sending request to ${this.url}`);
    console.log(`Headers: ${JSON.stringify(this.headers)}`);
    if (this.body) {
      console.log(`Body: ${JSON.stringify(this.body)}`);
    }
  }
}

class RequestDecorator {
  constructor(request) {
    this.request = request;
  }

  send() {
    this.request.send();
  }
}

class AuthDecorator extends RequestDecorator {
  constructor(request, token) {
    super(request);
    this.token = token;
  }

  send() {
    this.request.headers['Authorization'] = `Bearer ${this.token}`;
    this.request.send();
  }
}

class LoggingDecorator extends RequestDecorator {
  constructor(request) {
    super(request);
  }

  send() {
    console.log(`[LOG] Sending request to ${this.request.url}`);
    this.request.send();
    console.log('[LOG] Request sent');
  }
}

class CacheDecorator extends RequestDecorator {
  constructor(request, cache) {
    super(request);
    this.cache = cache;
  }

  send() {
    const cached = this.cache.get(this.request.url);
    if (cached) {
      console.log(`[CACHE] Returning cached response for ${this.request.url}`);
      return;
    }
    this.request.send();
    this.cache.set(this.request.url, 'response');
  }
}

// Usage
const cache = new Map();
let request = new HttpRequest('https://api.example.com/data');
request = new AuthDecorator(request, 'secret-token');
request = new LoggingDecorator(request);
request = new CacheDecorator(request, cache);
request.send();
```

**When to Use**

- Adding responsibilities to individual objects dynamically
- Extending functionality without subclassing
- When subclassing would create an explosion of subclasses
- When you need to add/remove responsibilities at runtime

**Pros and Cons**

| Pros | Cons |
|------|------|
| Adds behavior without modifying original class | Can result in many small objects |
| More flexible than static inheritance | Harder to configure and debug |
| Follows Single Responsibility Principle | Can make code harder to understand |
| Supports Open/Closed Principle | May affect performance |

---

### Facade Pattern

**Problem It Solves**

Provides a simplified interface to a complex subsystem. Hides complexity from clients and provides a single entry point. Useful for working with complex libraries, APIs, or legacy systems.

```javascript
// Facade Pattern for complex subsystem
class CPU {
  freeze() {
    console.log('CPU: Freezing');
  }

  jump(position) {
    console.log(`CPU: Jumping to ${position}`);
  }

  execute() {
    console.log('CPU: Executing');
  }
}

class Memory {
  load(position, data) {
    console.log(`Memory: Loading ${data} at ${position}`);
  }
}

class HardDrive {
  read(lba, size) {
    console.log(`HardDrive: Reading ${size} bytes from LBA ${lba}`);
    return 'data';
  }
}

class ComputerFacade {
  constructor() {
    this.cpu = new CPU();
    this.memory = new Memory();
    this.hardDrive = new HardDrive();
  }

  start() {
    console.log('Starting computer...');
    this.cpu.freeze();
    this.memory.load(0x00, this.hardDrive.read(0, 1024));
    this.cpu.jump(0x00);
    this.cpu.execute();
    console.log('Computer started!');
  }
}

// Usage
const computer = new ComputerFacade();
computer.start();

// Real-world example: Database connection facade
class MySqlConnection {
  connect() {
    console.log('MySQL: Connecting...');
  }

  query(sql) {
    console.log(`MySQL: ${sql}`);
    return [{ id: 1, name: 'John' }];
  }

  close() {
    console.log('MySQL: Closing connection');
  }
}

class PostgresConnection {
  connect() {
    console.log('PostgreSQL: Connecting...');
  }

  query(sql) {
    console.log(`PostgreSQL: ${sql}`);
    return [{ id: 1, name: 'Jane' }];
  }

  close() {
    console.log('PostgreSQL: Closing connection');
  }
}

class DatabaseFacade {
  constructor(type) {
    this.type = type;
    this.connection = null;
  }

  connect() {
    if (this.type === 'mysql') {
      this.connection = new MySqlConnection();
    } else if (this.type === 'postgres') {
      this.connection = new PostgresConnection();
    }
    this.connection.connect();
  }

  query(sql) {
    return this.connection.query(sql);
  }

  close() {
    this.connection.close();
  }
}

// Usage
const db = new DatabaseFacade('mysql');
db.connect();
const users = db.query('SELECT * FROM users');
db.close();

// Real-world example: API client facade
class HttpClient {
  get(url) {
    console.log(`GET ${url}`);
    return { data: 'response' };
  }

  post(url, body) {
    console.log(`POST ${url}`, body);
    return { data: 'created' };
  }

  put(url, body) {
    console.log(`PUT ${url}`, body);
    return { data: 'updated' };
  }

  delete(url) {
    console.log(`DELETE ${url}`);
    return { data: 'deleted' };
  }
}

class AuthClient {
  login(credentials) {
    console.log('Auth: Logging in', credentials);
    return { token: 'jwt-token' };
  }

  logout() {
    console.log('Auth: Logging out');
  }
}

class ApiFacade {
  constructor() {
    this.http = new HttpClient();
    this.auth = new AuthClient();
    this.baseUrl = 'https://api.example.com';
    this.token = null;
  }

  async login(email, password) {
    const result = this.auth.login({ email, password });
    this.token = result.token;
    return result;
  }

  async getUsers() {
    return this.http.get(`${this.baseUrl}/users`);
  }

  async createUser(userData) {
    return this.http.post(`${this.baseUrl}/users`, userData);
  }

  async updateUser(id, userData) {
    return this.http.put(`${this.baseUrl}/users/${id}`, userData);
  }

  async deleteUser(id) {
    return this.http.delete(`${this.baseUrl}/users/${id}`);
  }

  logout() {
    this.auth.logout();
    this.token = null;
  }
}

// Usage
const api = new ApiFacade();
api.login('user@example.com', 'password');
api.getUsers();
api.createUser({ name: 'John' });
api.logout();
```

**When to Use**

- Providing a simple interface to a complex subsystem
- Decoupling clients from subsystem implementation
- Layering your application with facades between layers
- Working with complex libraries or legacy code

**Pros and Cons**

| Pros | Cons |
|------|------|
| Simplifies complex subsystems | Can hide too much functionality |
| Reduces coupling between client and subsystem | May become a "god object" |
| Promotes loose coupling | Can limit flexibility |
| Easier to use and understand | May introduce performance overhead |

---

### Proxy Pattern

**Problem It Solves**

Provides a surrogate or placeholder to control access to another object. Useful for lazy initialization, access control, logging, caching, and remote object access. The proxy has the same interface as the real object.

```javascript
// Proxy Pattern for lazy loading
class Image {
  constructor(filename) {
    this.filename = filename;
  }

  display() {
    console.log(`Displaying ${this.filename}`);
  }
}

class ProxyImage {
  constructor(filename) {
    this.filename = filename;
    this.image = null;
  }

  display() {
    if (!this.image) {
      this.image = new Image(this.filename);
      console.log(`Loading ${this.filename} from disk`);
    }
    this.image.display();
  }
}

// Usage
const image1 = new ProxyImage('photo1.jpg');
const image2 = new ProxyImage('photo2.jpg');

image1.display(); // Loads and displays
image1.display(); // Already loaded, just displays
image2.display(); // Loads and displays

// Protection Proxy for access control
class BankAccount {
  constructor(owner, balance) {
    this.owner = owner;
    this.balance = balance;
  }

  withdraw(amount) {
    if (amount <= this.balance) {
      this.balance -= amount;
      console.log(`Withdrew $${amount}. New balance: $${this.balance}`);
      return true;
    }
    console.log('Insufficient funds');
    return false;
  }
}

class BankAccountProxy {
  constructor(account, owner) {
    this.account = account;
    this.owner = owner;
  }

  withdraw(amount) {
    if (this.account.owner !== this.owner) {
      console.log('Access denied: Not the account owner');
      return false;
    }
    return this.account.withdraw(amount);
  }
}

// Usage
const account = new BankAccount('John', 1000);
const proxy = new BankAccountProxy(account, 'John');
proxy.withdraw(500); // Success

const unauthorizedProxy = new BankAccountProxy(account, 'Jane');
unauthorizedProxy.withdraw(100); // Access denied

// Caching Proxy for expensive operations
class ExpensiveOperation {
  perform(input) {
    console.log(`Performing expensive operation on ${input}`);
    // Simulate expensive computation
    return input * 2;
  }
}

class CachedOperationProxy {
  constructor(operation) {
    this.operation = operation;
    this.cache = new Map();
  }

  perform(input) {
    if (this.cache.has(input)) {
      console.log(`Returning cached result for ${input}`);
      return this.cache.get(input);
    }
    const result = this.operation.perform(input);
    this.cache.set(input, result);
    return result;
  }
}

// Usage
const operation = new ExpensiveOperation();
const cachedOperation = new CachedOperationProxy(operation);

cachedOperation.perform(5); // Computes
cachedOperation.perform(5); // Returns cached
cachedOperation.perform(10); // Computes
cachedOperation.perform(10); // Returns cached
```

**When to Use**

- Lazy initialization of expensive objects
- Access control to objects
- Logging requests to objects
- Caching expensive operations
- Remote object access (remote proxy)

**Pros and Cons**

| Pros | Cons |
|------|------|
| Controls access to real object | Adds indirection layer |
| Supports lazy initialization | Can increase complexity |
| Enables caching and optimization | May impact performance |
| Provides access control | Can hide real object behavior |

---

## Behavioral Patterns

### Observer Pattern

**Problem It Solves**

Defines a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically. Useful for event handling systems, pub/sub messaging, and reactive programming.

```javascript
// Observer Pattern implementation
class Observer {
  update(data) {
    throw new Error('Must implement update()');
  }
}

class Subject {
  constructor() {
    this.observers = [];
  }

  attach(observer) {
    this.observers.push(observer);
  }

  detach(observer) {
    const index = this.observers.indexOf(observer);
    if (index > -1) {
      this.observers.splice(index, 1);
    }
  }

  notify(data) {
    this.observers.forEach(observer => observer.update(data));
  }
}

class NewsPublisher extends Subject {
  constructor() {
    super();
    this.news = [];
  }

  addNews(article) {
    this.news.push(article);
    this.notify(article);
  }
}

class NewsSubscriber extends Observer {
  constructor(name) {
    super();
    this.name = name;
  }

  update(article) {
    console.log(`${this.name} received: ${article}`);
  }
}

// Usage
const publisher = new NewsPublisher();
const subscriber1 = new NewsSubscriber('Alice');
const subscriber2 = new NewsSubscriber('Bob');

publisher.attach(subscriber1);
publisher.attach(subscriber2);

publisher.addNews('Breaking: JavaScript is awesome!');
publisher.addNews('New framework released!');

// Real-world example: Event emitter
class EventEmitter {
  constructor() {
    this.events = {};
  }

  on(event, listener) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    this.events[event].push(listener);
  }

  off(event, listener) {
    if (!this.events[event]) return;
    this.events[event] = this.events[event].filter(l => l !== listener);
  }

  emit(event, data) {
    if (!this.events[event]) return;
    this.events[event].forEach(listener => listener(data));
  }
}

// Usage
const emitter = new EventEmitter();

emitter.on('user-login', (user) => {
  console.log(`User logged in: ${user.name}`);
});

emitter.on('user-login', (user) => {
  console.log(`Sending welcome email to ${user.email}`);
});

emitter.emit('user-login', { name: 'John', email: 'john@example.com' });

// Real-world example: Reactive state management
class Store {
  constructor(initialState = {}) {
    this.state = initialState;
    this.listeners = [];
  }

  getState() {
    return this.state;
  }

  setState(newState) {
    this.state = { ...this.state, ...newState };
    this.notify();
  }

  subscribe(listener) {
    this.listeners.push(listener);
    return () => this.unsubscribe(listener);
  }

  unsubscribe(listener) {
    this.listeners = this.listeners.filter(l => l !== listener);
  }

  notify() {
    this.listeners.forEach(listener => listener(this.state));
  }
}

// Usage
const store = new Store({ count: 0, user: null });

store.subscribe((state) => {
  console.log('State updated:', state);
});

store.setState({ count: 1 });
store.setState({ user: { name: 'John' } });
```

**When to Use**

- When one object change should notify multiple others
- For event handling systems
- In pub/sub messaging systems
- For reactive programming and state management
- When you need loose coupling between objects

**Pros and Cons**

| Pros | Cons |
|------|------|
| Loose coupling between subject and observers | Can cause performance issues with many observers |
| Supports broadcast communication | Debugging can be difficult |
| Open/Closed Principle - easy to add observers | Unexpected updates can cause bugs |
| Supports dynamic relationships | Memory leaks if observers not detached |

---

### Strategy Pattern

**Problem It Solves**

Defines a family of algorithms, encapsulates each one, and makes them interchangeable. Lets the algorithm vary independently from clients that use it. Useful for different ways to accomplish a task, such as sorting, compression, or payment processing.

```javascript
// Strategy Pattern for sorting algorithms
class SortStrategy {
  sort(array) {
    throw new Error('Must implement sort()');
  }
}

class BubbleSort extends SortStrategy {
  sort(array) {
    console.log('Using Bubble Sort');
    const arr = [...array];
    for (let i = 0; i < arr.length - 1; i++) {
      for (let j = 0; j < arr.length - i - 1; j++) {
        if (arr[j] > arr[j + 1]) {
          [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]];
        }
      }
    }
    return arr;
  }
}

class QuickSort extends SortStrategy {
  sort(array) {
    console.log('Using Quick Sort');
    const arr = [...array];
    if (arr.length <= 1) return arr;
    const pivot = arr[Math.floor(arr.length / 2)];
    const left = arr.filter(x => x < pivot);
    const middle = arr.filter(x => x === pivot);
    const right = arr.filter(x => x > pivot);
    return [...this.sort(left), ...middle, ...this.sort(right)];
  }
}

class Sorter {
  constructor(strategy) {
    this.strategy = strategy;
  }

  setStrategy(strategy) {
    this.strategy = strategy;
  }

  sort(array) {
    return this.strategy.sort(array);
  }
}

// Usage
const sorter = new Sorter(new BubbleSort());
console.log(sorter.sort([64, 34, 25, 12, 22, 11, 90]));

sorter.setStrategy(new QuickSort());
console.log(sorter.sort([64, 34, 25, 12, 22, 11, 90]));

// Real-world example: Payment processing
class PaymentStrategy {
  process(amount) {
    throw new Error('Must implement process()');
  }
}

class CreditCardPayment extends PaymentStrategy {
  constructor(cardNumber, cvv, expiry) {
    super();
    this.cardNumber = cardNumber;
    this.cvv = cvv;
    this.expiry = expiry;
  }

  process(amount) {
    console.log(`Processing $${amount} via Credit Card ending in ${this.cardNumber.slice(-4)}`);
    return { success: true, transactionId: 'cc_' + Date.now() };
  }
}

class PayPalPayment extends PaymentStrategy {
  constructor(email) {
    super();
    this.email = email;
  }

  process(amount) {
    console.log(`Processing $${amount} via PayPal for ${this.email}`);
    return { success: true, transactionId: 'pp_' + Date.now() };
  }
}

class CryptoPayment extends PaymentStrategy {
  constructor(walletAddress) {
    super();
    this.walletAddress = walletAddress;
  }

  process(amount) {
    console.log(`Processing $${amount} via Crypto from ${this.walletAddress}`);
    return { success: true, transactionId: 'crypto_' + Date.now() };
  }
}

class PaymentProcessor {
  constructor(strategy) {
    this.strategy = strategy;
  }

  setStrategy(strategy) {
    this.strategy = strategy;
  }

  processPayment(amount) {
    return this.strategy.process(amount);
  }
}

// Usage
const processor = new PaymentProcessor(new CreditCardPayment('4111111111111111', '123', '12/25'));
processor.processPayment(100);

processor.setStrategy(new PayPalPayment('user@example.com'));
processor.processPayment(50);

processor.setStrategy(new CryptoPayment('0x123...abc'));
processor.processPayment(75);

// Real-world example: Compression strategies
class CompressionStrategy {
  compress(data) {
    throw new Error('Must implement compress()');
  }

  decompress(data) {
    throw new Error('Must implement decompress()');
  }
}

class GzipCompression extends CompressionStrategy {
  compress(data) {
    console.log('Compressing with Gzip');
    return Buffer.from(data).toString('base64') + '_gz';
  }

  decompress(data) {
    console.log('Decompressing Gzip');
    return Buffer.from(data.replace('_gz', ''), 'base64').toString();
  }
}

class ZipCompression extends CompressionStrategy {
  compress(data) {
    console.log('Compressing with Zip');
    return Buffer.from(data).toString('base64') + '_zip';
  }

  decompress(data) {
    console.log('Decompressing Zip');
    return Buffer.from(data.replace('_zip', ''), 'base64').toString();
  }
}

class Compressor {
  constructor(strategy) {
    this.strategy = strategy;
  }

  setStrategy(strategy) {
    this.strategy = strategy;
  }

  compress(data) {
    return this.strategy.compress(data);
  }

  decompress(data) {
    return this.strategy.decompress(data);
  }
}

// Usage
const compressor = new Compressor(new GzipCompression());
const compressed = compressor.compress('Hello, World!');
const decompressed = compressor.decompress(compressed);
```

**When to Use**

- Multiple ways to accomplish a task
- Need to switch algorithms at runtime
- Want to avoid conditional statements for algorithm selection
- Similar classes differ only in behavior
- Need to hide algorithm implementation details

**Pros and Cons**

| Pros | Cons |
|------|------|
| Open/Closed Principle - easy to add strategies | Clients must know about strategies |
| Avoids conditional statements | Increases number of classes |
| Encapsulates algorithm implementation | Strategy selection can be complex |
| Allows runtime algorithm switching | May violate DIP if not careful |

---

### Command Pattern

**Problem It Solves**

Encapsulates a request as an object, thereby letting you parameterize clients with different requests, queue or log requests, and support undoable operations. Useful for implementing undo/redo, macro operations, and task queues.

```javascript
// Command Pattern for remote control
class Command {
  execute() {
    throw new Error('Must implement execute()');
  }

  undo() {
    throw new Error('Must implement undo()');
  }
}

class Light {
  on() {
    console.log('Light is ON');
  }

  off() {
    console.log('Light is OFF');
  }
}

class LightOnCommand extends Command {
  constructor(light) {
    super();
    this.light = light;
  }

  execute() {
    this.light.on();
  }

  undo() {
    this.light.off();
  }
}

class LightOffCommand extends Command {
  constructor(light) {
    super();
    this.light = light;
  }

  execute() {
    this.light.off();
  }

  undo() {
    this.light.on();
  }
}

class RemoteControl {
  constructor() {
    this.commands = [];
    this.undoStack = [];
  }

  setCommand(command) {
    this.commands.push(command);
  }

  executeCommands() {
    this.commands.forEach(command => {
      command.execute();
      this.undoStack.push(command);
    });
    this.commands = [];
  }

  undo() {
    const command = this.undoStack.pop();
    if (command) {
      command.undo();
    }
  }
}

// Usage
const remote = new RemoteControl();
const light = new Light();

remote.setCommand(new LightOnCommand(light));
remote.setCommand(new LightOffCommand(light));
remote.executeCommands();
remote.undo();

// Real-world example: Text editor undo/redo
class TextEditor {
  constructor() {
    this.content = '';
    this.history = [];
    this.historyIndex = -1;
  }

  write(text) {
    this.content += text;
    this.saveState();
  }

  delete(count) {
    this.content = this.content.slice(0, -count);
    this.saveState();
  }

  saveState() {
    this.history = this.history.slice(0, this.historyIndex + 1);
    this.history.push(this.content);
    this.historyIndex++;
  }

  undo() {
    if (this.historyIndex > 0) {
      this.historyIndex--;
      this.content = this.history[this.historyIndex];
    }
  }

  redo() {
    if (this.historyIndex < this.history.length - 1) {
      this.historyIndex++;
      this.content = this.history[this.historyIndex];
    }
  }

  getContent() {
    return this.content;
  }
}

// Usage
const editor = new TextEditor();
editor.write('Hello ');
editor.write('World');
console.log(editor.getContent()); // Hello World

editor.undo();
console.log(editor.getContent()); // Hello

editor.redo();
console.log(editor.getContent()); // Hello World

// Real-world example: Macro commands
class MacroCommand extends Command {
  constructor(commands) {
    super();
    this.commands = commands;
  }

  execute() {
    this.commands.forEach(command => command.execute());
  }

  undo() {
    [...this.commands].reverse().forEach(command => command.undo());
  }
}

// Usage
const livingRoomLight = new Light();
const kitchenLight = new Light();

const allLightsOn = new MacroCommand([
  new LightOnCommand(livingRoomLight),
  new LightOnCommand(kitchenLight)
]);

const allLightsOff = new MacroCommand([
  new LightOffCommand(livingRoomLight),
  new LightOffCommand(kitchenLight)
]);

allLightsOn.execute();
allLightsOff.execute();
allLightsOff.undo();
```

**When to Use**

- Parameterizing objects with operations
- Queuing operations and executing them later
- Implementing undo/redo functionality
- Logging and transactional operations
- Macro operations (composite commands)

**Pros and Cons**

| Pros | Cons |
|------|------|
| Decouples invoker from receiver | Can create many command classes |
| Supports undo/redo operations | Increases complexity |
| Easy to add new commands | May impact performance |
| Supports composite commands | Command objects can become large |

---

### Iterator Pattern

**Problem It Solves**

Provides a way to access elements of an aggregate object sequentially without exposing its underlying representation. Useful for traversing collections, implementing custom iteration logic, and providing uniform interfaces for different collection types.

```javascript
// Iterator Pattern implementation
class Iterator {
  hasNext() {
    throw new Error('Must implement hasNext()');
  }

  next() {
    throw new Error('Must implement next()');
  }
}

class Aggregate {
  createIterator() {
    throw new Error('Must implement createIterator()');
  }
}

class ArrayIterator extends Iterator {
  constructor(array) {
    super();
    this.array = array;
    this.index = 0;
  }

  hasNext() {
    return this.index < this.array.length;
  }

  next() {
    if (!this.hasNext()) {
      throw new Error('No more elements');
    }
    return this.array[this.index++];
  }
}

class ArrayCollection extends Aggregate {
  constructor(array) {
    super();
    this.array = array;
  }

  createIterator() {
    return new ArrayIterator(this.array);
  }
}

// Usage
const collection = new ArrayCollection([1, 2, 3, 4, 5]);
const iterator = collection.createIterator();

while (iterator.hasNext()) {
  console.log(iterator.next());
}

// Real-world example: Tree traversal iterator
class TreeNode {
  constructor(value) {
    this.value = value;
    this.left = null;
    this.right = null;
  }
}

class TreeIterator extends Iterator {
  constructor(root) {
    super();
    this.stack = [];
    this.pushLeft(root);
  }

  pushLeft(node) {
    while (node) {
      this.stack.push(node);
      node = node.left;
    }
  }

  hasNext() {
    return this.stack.length > 0;
  }

  next() {
    if (!this.hasNext()) {
      throw new Error('No more elements');
    }
    const node = this.stack.pop();
    this.pushLeft(node.right);
    return node.value;
  }
}

// Usage
const root = new TreeNode(4);
root.left = new TreeNode(2);
root.right = new TreeNode(6);
root.left.left = new TreeNode(1);
root.left.right = new TreeNode(3);
root.right.left = new TreeNode(5);
root.right.right = new TreeNode(7);

const treeIterator = new TreeIterator(root);
while (treeIterator.hasNext()) {
  console.log(treeIterator.next()); // 1, 2, 3, 4, 5, 6, 7
}

// Real-world example: Paginated API iterator
class PaginatedAPIIterator extends Iterator {
  constructor(fetchPage) {
    super();
    this.fetchPage = fetchPage;
    this.currentPage = 0;
    this.items = [];
    this.currentIndex = 0;
    this.hasMore = true;
  }

  async hasNext() {
    if (this.currentIndex < this.items.length) {
      return true;
    }
    if (!this.hasMore) {
      return false;
    }
    await this.fetchNextPage();
    return this.items.length > 0;
  }

  async next() {
    if (!(await this.hasNext())) {
      throw new Error('No more elements');
    }
    return this.items[this.currentIndex++];
  }

  async fetchNextPage() {
    const result = await this.fetchPage(this.currentPage);
    this.items = result.items;
    this.hasMore = result.hasMore;
    this.currentIndex = 0;
    this.currentPage++;
  }
}

// Usage
async function fetchAPIPage(page) {
  // Simulate API call
  console.log(`Fetching page ${page}`);
  const items = Array.from({ length: 10 }, (_, i) => `item-${page * 10 + i}`);
  return { items, hasMore: page < 2 };
}

const apiIterator = new PaginatedAPIIterator(fetchAPIPage);
while (await apiIterator.hasNext()) {
  console.log(await apiIterator.next());
}
```

**When to Use**

- Accessing collection elements without exposing internal structure
- Traversing different collection structures uniformly
- Implementing custom iteration logic
- Providing multiple ways to traverse a collection

**Pros and Cons**

| Pros | Cons |
|------|------|
| Uniform interface for traversal | Can increase complexity |
| Hides collection implementation | May be overkill for simple arrays |
| Supports multiple traversal methods | Iterator state management |
| Follows Single Responsibility Principle | Can impact performance |

---

### Mediator Pattern

**Problem It Solves**

Defines an object that encapsulates how a set of objects interact. Promotes loose coupling by keeping objects from referring to each other explicitly. Useful for complex communication between multiple objects, UI components, and chat systems.

```javascript
// Mediator Pattern for chat room
class Mediator {
  sendMessage(message, sender) {
    throw new Error('Must implement sendMessage()');
  }
}

class ChatRoom extends Mediator {
  constructor() {
    this.participants = [];
  }

  register(participant) {
    this.participants.push(participant);
    participant.setMediator(this);
  }

  sendMessage(message, sender) {
    this.participants.forEach(participant => {
      if (participant !== sender) {
        participant.receive(message);
      }
    });
  }
}

class Participant {
  constructor(name) {
    this.name = name;
    this.mediator = null;
  }

  setMediator(mediator) {
    this.mediator = mediator;
  }

  send(message) {
    console.log(`${this.name} sends: ${message}`);
    this.mediator.sendMessage(message, this);
  }

  receive(message) {
    console.log(`${this.name} receives: ${message}`);
  }
}

// Usage
const chatRoom = new ChatRoom();
const alice = new Participant('Alice');
const bob = new Participant('Bob');
const charlie = new Participant('Charlie');

chatRoom.register(alice);
chatRoom.register(bob);
chatRoom.register(charlie);

alice.send('Hello everyone!');
bob.send('Hi Alice!');
charlie.send('Hey all!');

// Real-world example: Form validation mediator
class FormMediator {
  constructor() {
    this.fields = [];
    this.errors = {};
  }

  registerField(field) {
    this.fields.push(field);
    field.setMediator(this);
  }

  validate() {
    this.errors = {};
    let isValid = true;

    this.fields.forEach(field => {
      if (!field.validate()) {
        isValid = false;
        this.errors[field.name] = field.getError();
      }
    });

    return isValid;
  }

  showError(fieldName, error) {
    this.errors[fieldName] = error;
  }

  clearError(fieldName) {
    delete this.errors[fieldName];
  }

  getErrors() {
    return this.errors;
  }
}

class FormField {
  constructor(name, value, validators = []) {
    this.name = name;
    this.value = value;
    this.validators = validators;
    this.mediator = null;
    this.error = null;
  }

  setMediator(mediator) {
    this.mediator = mediator;
  }

  setValue(value) {
    this.value = value;
    this.validate();
  }

  validate() {
    for (const validator of this.validators) {
      const result = validator(this.value);
      if (!result.valid) {
        this.error = result.message;
        this.mediator?.showError(this.name, this.error);
        return false;
      }
    }
    this.error = null;
    this.mediator?.clearError(this.name);
    return true;
  }

  getError() {
    return this.error;
  }
}

// Usage
const formMediator = new FormMediator();

const emailField = new FormField('email', '', [
  (value) => ({ valid: value.includes('@'), message: 'Invalid email' }),
  (value) => ({ valid: value.length > 5, message: 'Email too short' })
]);

const passwordField = new FormField('password', '', [
  (value) => ({ valid: value.length >= 8, message: 'Password too short' })
]);

formMediator.registerField(emailField);
formMediator.registerField(passwordField);

emailField.setValue('test@test.com');
passwordField.setValue('short');

console.log(formMediator.getErrors()); // { password: 'Password too short' }

passwordField.setValue('longpassword');
console.log(formMediator.getErrors()); // {}

// Real-world example: Air traffic control mediator
class AirTrafficControlMediator {
  constructor() {
    this.aircrafts = [];
    this.runways = new Map();
  }

  registerAircraft(aircraft) {
    this.aircrafts.push(aircraft);
    aircraft.setMediator(this);
  }

  requestLanding(aircraft) {
    const availableRunway = this.findAvailableRunway();
    if (availableRunway) {
      this.assignRunway(aircraft, availableRunway);
      return true;
    }
    this.addToQueue(aircraft);
    return false;
  }

  findAvailableRunway() {
    for (const [runway, status] of this.runways) {
      if (status === 'available') {
        return runway;
      }
    }
    return null;
  }

  assignRunway(aircraft, runway) {
    this.runways.set(runway, 'occupied');
    console.log(`${aircraft.id} cleared to land on runway ${runway}`);
  }

  addToQueue(aircraft) {
    console.log(`${aircraft.id} added to landing queue`);
  }

  releaseRunway(runway) {
    this.runways.set(runway, 'available');
  }
}

class Aircraft {
  constructor(id) {
    this.id = id;
    this.mediator = null;
  }

  setMediator(mediator) {
    this.mediator = mediator;
  }

  requestLanding() {
    console.log(`${this.id} requesting landing`);
    return this.mediator.requestLanding(this);
  }
}

// Usage
const atc = new AirTrafficControlMediator();
atc.runways.set('RW1', 'available');
atc.runways.set('RW2', 'available');

const plane1 = new Aircraft('FL123');
const plane2 = new Aircraft('FL456');
const plane3 = new Aircraft('FL789');

atc.registerAircraft(plane1);
atc.registerAircraft(plane2);
atc.registerAircraft(plane3);

plane1.requestLanding();
plane2.requestLanding();
plane3.requestLanding();
```

**When to Use**

- Complex communication between multiple objects
- Objects need to communicate but should be loosely coupled
- Reusable components that need to interact with many others
- Centralizing complex communication logic

**Pros and Cons**

| Pros | Cons |
|------|------|
| Reduces coupling between objects | Mediator can become complex |
| Centralizes communication logic | Single point of failure |
| Simplifies object interactions | Can become a "god object" |
| Easier to maintain and extend | May reduce performance |

---

## Architectural Patterns

### Repository Pattern

**Problem It Solves**

Mediates between the domain and data mapping layers, acting like an in-memory domain object collection. Provides abstraction over data storage, making it easier to switch data sources and test code. Useful for database operations, API calls, and file system access.

```javascript
// Repository Pattern implementation
class Repository {
  findById(id) {
    throw new Error('Must implement findById()');
  }

  findAll() {
    throw new Error('Must implement findAll()');
  }

  save(entity) {
    throw new Error('Must implement save()');
  }

  delete(id) {
    throw new Error('Must implement delete()');
  }
}

class UserRepository extends Repository {
  constructor() {
    super();
    this.users = new Map();
    this.nextId = 1;
  }

  findById(id) {
    return this.users.get(id);
  }

  findAll() {
    return Array.from(this.users.values());
  }

  save(user) {
    if (!user.id) {
      user.id = this.nextId++;
    }
    this.users.set(user.id, user);
    return user;
  }

  delete(id) {
    return this.users.delete(id);
  }

  findByEmail(email) {
    return Array.from(this.users.values()).find(u => u.email === email);
  }
}

// Usage
const userRepo = new UserRepository();
const user1 = userRepo.save({ name: 'John', email: 'john@example.com' });
const user2 = userRepo.save({ name: 'Jane', email: 'jane@example.com' });

console.log(userRepo.findById(user1.id));
console.log(userRepo.findAll());
console.log(userRepo.findByEmail('john@example.com'));

// Real-world example: Database repository with async operations
class DatabaseRepository extends Repository {
  constructor(db) {
    super();
    this.db = db;
    this.tableName = '';
  }

  async findById(id) {
    const result = await this.db.query(
      `SELECT * FROM ${this.tableName} WHERE id = ?`,
      [id]
    );
    return result[0];
  }

  async findAll() {
    return await this.db.query(`SELECT * FROM ${this.tableName}`);
  }

  async save(entity) {
    if (entity.id) {
      await this.db.query(
        `UPDATE ${this.tableName} SET ? WHERE id = ?`,
        [entity, entity.id]
      );
      return entity;
    } else {
      const result = await this.db.query(
        `INSERT INTO ${this.tableName} SET ?`,
        [entity]
      );
      entity.id = result.insertId;
      return entity;
    }
  }

  async delete(id) {
    await this.db.query(`DELETE FROM ${this.tableName} WHERE id = ?`, [id]);
  }
}

class PostRepository extends DatabaseRepository {
  constructor(db) {
    super(db);
    this.tableName = 'posts';
  }

  async findByAuthor(authorId) {
    return await this.db.query(
      `SELECT * FROM ${this.tableName} WHERE author_id = ?`,
      [authorId]
    );
  }

  async findPublished() {
    return await this.db.query(
      `SELECT * FROM ${this.tableName} WHERE published = ?`,
      [true]
    );
  }
}

// Real-world example: Repository with caching
class CachedRepository extends Repository {
  constructor(repository, cache) {
    super();
    this.repository = repository;
    this.cache = cache;
    this.cachePrefix = 'repo:';
  }

  async findById(id) {
    const cacheKey = `${this.cachePrefix}${id}`;
    const cached = await this.cache.get(cacheKey);

    if (cached) {
      return JSON.parse(cached);
    }

    const entity = await this.repository.findById(id);
    if (entity) {
      await this.cache.set(cacheKey, JSON.stringify(entity), 3600);
    }

    return entity;
  }

  async findAll() {
    return await this.repository.findAll();
  }

  async save(entity) {
    const saved = await this.repository.save(entity);
    const cacheKey = `${this.cachePrefix}${saved.id}`;
    await this.cache.set(cacheKey, JSON.stringify(saved), 3600);
    return saved;
  }

  async delete(id) {
    await this.repository.delete(id);
    const cacheKey = `${this.cachePrefix}${id}`;
    await this.cache.del(cacheKey);
  }
}

// Usage
const db = { /* database connection */ };
const cache = { /* cache client */ };

const postRepo = new PostRepository(db);
const cachedPostRepo = new CachedRepository(postRepo, cache);

const post = await cachedPostRepo.save({ title: 'Hello', content: 'World' });
const foundPost = await cachedPostRepo.findById(post.id); // From cache
```

**When to Use**

- Abstracting data access logic
- Switching between data sources (database, API, file)
- Implementing caching strategies
- Testing with mock repositories
- Centralizing data access code

**Pros and Cons**

| Pros | Cons |
|------|------|
| Separates domain logic from data access | Can add abstraction overhead |
| Easy to switch data sources | May hide performance issues |
| Supports caching and optimization | Can lead to generic repositories |
| Improves testability | May not fit all use cases |

---

### MVC (Model-View-Controller)

**Problem It Solves**

Separates application logic into three interconnected components: Model (data), View (presentation), and Controller (logic). Promotes separation of concerns, making code more maintainable and testable. Widely used in web frameworks and desktop applications.

```javascript
// Simple MVC implementation for a todo app

// Model - Data and business logic
class TodoModel {
  constructor() {
    this.todos = [];
    this.listeners = [];
  }

  addTodo(text) {
    const todo = {
      id: Date.now(),
      text,
      completed: false
    };
    this.todos.push(todo);
    this.notify();
    return todo;
  }

  toggleTodo(id) {
    const todo = this.todos.find(t => t.id === id);
    if (todo) {
      todo.completed = !todo.completed;
      this.notify();
    }
  }

  deleteTodo(id) {
    this.todos = this.todos.filter(t => t.id !== id);
    this.notify();
  }

  getTodos() {
    return this.todos;
  }

  subscribe(listener) {
    this.listeners.push(listener);
  }

  notify() {
    this.listeners.forEach(listener => listener(this.todos));
  }
}

// View - Presentation logic
class TodoView {
  constructor() {
    this.container = document.createElement('div');
    this.form = this.createForm();
    this.list = this.createList();
    this.container.appendChild(this.form);
    this.container.appendChild(this.list);
  }

  createForm() {
    const form = document.createElement('form');
    const input = document.createElement('input');
    input.type = 'text';
    input.placeholder = 'Add a todo...';
    const button = document.createElement('button');
    button.type = 'submit';
    button.textContent = 'Add';
    form.appendChild(input);
    form.appendChild(button);
    return form;
  }

  createList() {
    return document.createElement('ul');
  }

  render(todos) {
    this.list.innerHTML = '';
    todos.forEach(todo => {
      const li = document.createElement('li');
      li.textContent = todo.text;
      li.style.textDecoration = todo.completed ? 'line-through' : 'none';

      const toggleBtn = document.createElement('button');
      toggleBtn.textContent = 'Toggle';
      toggleBtn.onclick = () => this.onToggle(todo.id);

      const deleteBtn = document.createElement('button');
      deleteBtn.textContent = 'Delete';
      deleteBtn.onclick = () => this.onDelete(todo.id);

      li.appendChild(toggleBtn);
      li.appendChild(deleteBtn);
      this.list.appendChild(li);
    });
  }

  onAddTodo(callback) {
    this.form.onsubmit = (e) => {
      e.preventDefault();
      const input = this.form.querySelector('input');
      const text = input.value.trim();
      if (text) {
        callback(text);
        input.value = '';
      }
    };
  }

  onToggle(callback) {
    this.onToggle = callback;
  }

  onDelete(callback) {
    this.onDelete = callback;
  }

  mount(parent) {
    parent.appendChild(this.container);
  }
}

// Controller - Application logic
class TodoController {
  constructor(model, view) {
    this.model = model;
    this.view = view;

    this.view.onAddTodo((text) => this.model.addTodo(text));
    this.view.onToggle((id) => this.model.toggleTodo(id));
    this.view.onDelete((id) => this.model.deleteTodo(id));

    this.model.subscribe((todos) => this.view.render(todos));
  }
}

// Usage
const model = new TodoModel();
const view = new TodoView();
const controller = new TodoController(model, view);

view.mount(document.body);

// Real-world example: Express.js MVC structure
// models/User.js
class User {
  static async findById(id) {
    // Database query
    return { id, name: 'John', email: 'john@example.com' };
  }

  static async create(data) {
    // Create user in database
    return { id: Date.now(), ...data };
  }
}

// controllers/userController.js
class UserController {
  async getUser(req, res) {
    const { id } = req.params;
    const user = await User.findById(id);
    res.json(user);
  }

  async createUser(req, res) {
    const user = await User.create(req.body);
    res.status(201).json(user);
  }
}

// views/userView.js
class UserView {
  static renderUser(user) {
    return `
      <div class="user">
        <h2>${user.name}</h2>
        <p>${user.email}</p>
      </div>
    `;
  }
}

// routes/userRoutes.js
const express = require('express');
const router = express.Router();
const userController = new UserController();

router.get('/users/:id', (req, res) => userController.getUser(req, res));
router.post('/users', (req, res) => userController.createUser(req, res));

module.exports = router;
```

**When to Use**

- Building web applications
- Separating concerns in large applications
- Multiple views for the same model
- Complex user interfaces
- Team development (different developers work on different layers)

**Pros and Cons**

| Pros | Cons |
|------|------|
| Clear separation of concerns | Can be overkill for simple apps |
| Parallel development | Learning curve |
| Reusable models and views | Can lead to many small files |
| Easier testing and maintenance | Communication overhead between layers |

---

### Dependency Injection

**Problem It Solves**

Inverts control of object creation and dependency management. Instead of objects creating their dependencies, dependencies are provided from outside. Promotes loose coupling, testability, and flexibility. Essential for modern application architecture.

```javascript
// Dependency Injection implementation

// Without DI - Tight coupling
class UserService {
  constructor() {
    this.database = new Database(); // Hard dependency
    this.emailService = new EmailService(); // Hard dependency
  }

  createUser(userData) {
    const user = this.database.save(userData);
    this.emailService.sendWelcomeEmail(user.email);
    return user;
  }
}

// With DI - Loose coupling
class UserService {
  constructor(database, emailService) {
    this.database = database;
    this.emailService = emailService;
  }

  createUser(userData) {
    const user = this.database.save(userData);
    this.emailService.sendWelcomeEmail(user.email);
    return user;
  }
}

// Dependency Container
class Container {
  constructor() {
    this.services = new Map();
    this.factories = new Map();
  }

  register(name, factory) {
    this.factories.set(name, factory);
  }

  registerInstance(name, instance) {
    this.services.set(name, instance);
  }

  get(name) {
    if (this.services.has(name)) {
      return this.services.get(name);
    }

    if (this.factories.has(name)) {
      const instance = this.factories.get(name)(this);
      this.services.set(name, instance);
      return instance;
    }

    throw new Error(`Service not found: ${name}`);
  }
}

// Usage
const container = new Container();

// Register services
container.register('database', (c) => new Database());
container.register('emailService', (c) => new EmailService());
container.register('userService', (c) =>
  new UserService(c.get('database'), c.get('emailService'))
);

// Get service with all dependencies injected
const userService = container.get('userService');
userService.createUser({ name: 'John', email: 'john@example.com' });

// Real-world example: Constructor injection
class OrderService {
  constructor(
    private orderRepository: OrderRepository,
    private paymentService: PaymentService,
    private notificationService: NotificationService
  ) {
    this.orderRepository = orderRepository;
    this.paymentService = paymentService;
    this.notificationService = notificationService;
  }

  async createOrder(orderData) {
    const order = await this.orderRepository.save(orderData);
    await this.paymentService.processPayment(order);
    await this.notificationService.sendOrderConfirmation(order);
    return order;
  }
}

// Real-world example: Property injection
class ReportService {
  constructor() {
    this.logger = null;
    this.cache = null;
  }

  setLogger(logger) {
    this.logger = logger;
  }

  setCache(cache) {
    this.cache = cache;
  }

  async generateReport(reportId) {
    this.logger?.log(`Generating report ${reportId}`);

    const cached = await this.cache?.get(`report:${reportId}`);
    if (cached) {
      return cached;
    }

    const report = await this.fetchReportData(reportId);
    await this.cache?.set(`report:${reportId}`, report);
    return report;
  }
}

// Real-world example: Method injection
class TaskExecutor {
  async executeTask(task, dependencies) {
    const { logger, metrics, cache } = dependencies;

    logger?.log(`Executing task: ${task.id}`);
    const start = Date.now();

    try {
      const result = await this.runTask(task, cache);
      metrics?.record('task.success', Date.now() - start);
      return result;
    } catch (error) {
      metrics?.record('task.error', Date.now() - start);
      logger?.error(`Task failed: ${error.message}`);
      throw error;
    }
  }

  async runTask(task, cache) {
    // Task execution logic
  }
}

// Real-world example: DI with decorators (TypeScript-style)
function Injectable(dependencies = []) {
  return function(target) {
    target.dependencies = dependencies;
    return target;
  };
}

@Injectable(['database', 'cache'])
class ProductService {
  constructor(database, cache) {
    this.database = database;
    this.cache = cache;
  }
}

class DIContainer {
  constructor() {
    this.services = new Map();
  }

  resolve(target) {
    if (this.services.has(target)) {
      return this.services.get(target);
    }

    const dependencies = (target.dependencies || []).map(dep => this.resolve(dep));
    const instance = new target(...dependencies);
    this.services.set(target, instance);
    return instance;
  }
}
```

**When to Use**

- Large applications with many dependencies
- Testing with mock implementations
- Swapping implementations (database, API, cache)
- Following SOLID principles
- Building maintainable, testable code

**Pros and Cons**

| Pros | Cons |
|------|------|
| Loose coupling between components | Increases complexity |
| Easy testing with mocks | Learning curve |
| Flexible implementation swapping | Can be overkill for small apps |
| Follows SOLID principles | Container can become complex |

---

## Clean Code Practices

### Code Quality Principles

**DRY (Don't Repeat Yourself)**

Avoid duplication by abstracting common logic into reusable functions, classes, or modules. Duplicated code leads to maintenance issues—if you fix a bug in one place, you must fix it everywhere.

```javascript
// Violation: Duplicated logic
function calculateAreaCircle(radius) {
  return Math.PI * radius * radius;
}

function calculateCircumferenceCircle(radius) {
  return 2 * Math.PI * radius;
}

function calculateAreaSphere(radius) {
  return 4 * Math.PI * radius * radius;
}

// Corrected: Extract common logic
const PI = Math.PI;

function multiplyByPi(value) {
  return PI * value;
}

function calculateAreaCircle(radius) {
  return multiplyByPi(radius * radius);
}

function calculateCircumferenceCircle(radius) {
  return multiplyByPi(2 * radius);
}

function calculateAreaSphere(radius) {
  return multiplyByPi(4 * radius * radius);
}

// Better: Use constants and helper functions
const Geometry = {
  PI: Math.PI,

  circleArea(radius) {
    return this.PI * radius * radius;
  },

  circleCircumference(radius) {
    return 2 * this.PI * radius;
  },

  sphereArea(radius) {
    return 4 * this.PI * radius * radius;
  }
};
```

**KISS (Keep It Simple, Stupid)**

Write simple, straightforward code that's easy to understand. Avoid clever tricks, over-engineering, and unnecessary complexity. Simple code is easier to maintain, debug, and extend.

```javascript
// Violation: Overly complex
function isLeapYear(year) {
  return (year % 4 === 0 && year % 100 !== 0) || (year % 400 === 0);
}

// Actually, this is fine - but here's a more readable version
function isLeapYear(year) {
  const divisibleBy4 = year % 4 === 0;
  const divisibleBy100 = year % 100 === 0;
  const divisibleBy400 = year % 400 === 0;

  return divisibleBy4 && (!divisibleBy100 || divisibleBy400);
}

// Violation: Over-engineered
class Calculator {
  constructor() {
    this.operations = {
      add: (a, b) => a + b,
      subtract: (a, b) => a - b,
      multiply: (a, b) => a * b,
      divide: (a, b) => a / b
    };
  }

  calculate(operation, a, b) {
    const op = this.operations[operation];
    if (!op) {
      throw new Error(`Unknown operation: ${operation}`);
    }
    return op(a, b);
  }
}

// Corrected: Simple and direct
const add = (a, b) => a + b;
const subtract = (a, b) => a - b;
const multiply = (a, b) => a * b;
const divide = (a, b) => a / b;
```

**YAGNI (You Aren't Gonna Need It)**

Don't implement features or abstractions you don't need right now. Build only what's required for current requirements. Future-proofing often leads to unused code and unnecessary complexity.

```javascript
// Violation: Implementing features not needed yet
class UserService {
  constructor() {
    this.users = [];
  }

  addUser(user) {
    this.users.push(user);
  }

  // Not needed yet
  updateUser(id, updates) {
    const index = this.users.findIndex(u => u.id === id);
    if (index !== -1) {
      this.users[index] = { ...this.users[index], ...updates };
    }
  }

  // Not needed yet
  deleteUser(id) {
    this.users = this.users.filter(u => u.id !== id);
  }

  // Not needed yet
  searchUsers(query) {
    return this.users.filter(u =>
      u.name.includes(query) || u.email.includes(query)
    );
  }

  // Not needed yet
  exportToCSV() {
    // CSV export logic
  }

  // Not needed yet
  importFromCSV(csv) {
    // CSV import logic
  }
}

// Corrected: Only implement what's needed
class UserService {
  constructor() {
    this.users = [];
  }

  addUser(user) {
    this.users.push(user);
  }

  getUsers() {
    return this.users;
  }
}
```

---

### Code Organization

**File Structure**

Organize code by feature or layer. Group related files together. Use clear, descriptive names for directories and files.

```
src/
├── config/
│   ├── database.js
│   └── server.js
├── controllers/
│   ├── userController.js
│   └── postController.js
├── models/
│   ├── User.js
│   └── Post.js
├── routes/
│   ├── userRoutes.js
│   └── postRoutes.js
├── services/
│   ├── userService.js
│   └── postService.js
├── utils/
│   ├── logger.js
│   └── validator.js
├── middleware/
│   ├── auth.js
│   └── errorHandler.js
└── app.js
```

**Module Organization**

Export clear, focused interfaces. Use named exports for multiple related items, default exports for single main items.

```javascript
// Good: Clear exports
// utils/logger.js
class Logger {
  log(message) {
    console.log(`[LOG] ${message}`);
  }

  error(message) {
    console.error(`[ERROR] ${message}`);
  }
}

export default new Logger();

// utils/validator.js
export function validateEmail(email) {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}

export function validatePassword(password) {
  return password.length >= 8;
}

export function validateUsername(username) {
  return /^[a-zA-Z0-9_]{3,20}$/.test(username);
}

// Importing
import logger from './utils/logger.js';
import { validateEmail, validatePassword } from './utils/validator.js';
```

**Naming Conventions**

Use clear, descriptive names that reveal intent. Follow language conventions (camelCase for variables/functions, PascalCase for classes).

```javascript
// Bad: Unclear names
const d = new Date();
const u = users.find(x => x.id === id);
const f = (a, b) => a + b;

// Good: Descriptive names
const currentDate = new Date();
const user = users.find(user => user.id === userId);
const add = (a, b) => a + b;

// Bad: Vague function names
function process(data) {
  // What does this do?
}

// Good: Descriptive function names
function validateUserData(userData) {
  // Clear intent
}

function saveUserToDatabase(user) {
  // Clear intent
}

function sendWelcomeEmailToUser(user) {
  // Clear intent
}
```

---

### Refactoring Techniques

**Extract Method**

Break large functions into smaller, focused methods. Each method should do one thing well.

```javascript
// Before: Large function
function processOrder(order) {
  // Validate order
  if (!order.userId) {
    throw new Error('User ID required');
  }
  if (!order.items || order.items.length === 0) {
    throw new Error('Order must have items');
  }

  // Calculate total
  let total = 0;
  order.items.forEach(item => {
    total += item.price * item.quantity;
  });

  // Apply discount
  if (order.discountCode) {
    const discount = getDiscount(order.discountCode);
    total = total * (1 - discount);
  }

  // Save order
  order.total = total;
  order.status = 'pending';
  const savedOrder = database.save(order);

  // Send confirmation
  emailService.sendConfirmation(savedOrder);

  return savedOrder;
}

// After: Extracted methods
function processOrder(order) {
  validateOrder(order);
  const total = calculateOrderTotal(order);
  order.total = total;
  order.status = 'pending';
  const savedOrder = saveOrder(order);
  sendOrderConfirmation(savedOrder);
  return savedOrder;
}

function validateOrder(order) {
  if (!order.userId) {
    throw new Error('User ID required');
  }
  if (!order.items || order.items.length === 0) {
    throw new Error('Order must have items');
  }
}

function calculateOrderTotal(order) {
  let total = order.items.reduce((sum, item) => {
    return sum + item.price * item.quantity;
  }, 0);

  if (order.discountCode) {
    const discount = getDiscount(order.discountCode);
    total = total * (1 - discount);
  }

  return total;
}

function saveOrder(order) {
  return database.save(order);
}

function sendOrderConfirmation(order) {
  emailService.sendConfirmation(order);
}
```

**Replace Conditional with Polymorphism**

Replace complex conditional logic with polymorphic behavior using inheritance or strategy pattern.

```javascript
// Before: Complex conditionals
function calculateShipping(order, shippingMethod) {
  switch (shippingMethod) {
    case 'standard':
      return order.total > 50 ? 0 : 5.99;
    case 'express':
      return order.total > 100 ? 9.99 : 14.99;
    case 'overnight':
      return order.total > 200 ? 19.99 : 29.99;
    default:
      throw new Error('Unknown shipping method');
  }
}

function getDeliveryDays(shippingMethod) {
  switch (shippingMethod) {
    case 'standard':
      return 5;
    case 'express':
      return 2;
    case 'overnight':
      return 1;
    default:
      throw new Error('Unknown shipping method');
  }
}

// After: Polymorphic behavior
class ShippingStrategy {
  calculateCost(order) {
    throw new Error('Must implement calculateCost()');
  }

  getDeliveryDays() {
    throw new Error('Must implement getDeliveryDays()');
  }
}

class StandardShipping extends ShippingStrategy {
  calculateCost(order) {
    return order.total > 50 ? 0 : 5.99;
  }

  getDeliveryDays() {
    return 5;
  }
}

class ExpressShipping extends ShippingStrategy {
  calculateCost(order) {
    return order.total > 100 ? 9.99 : 14.99;
  }

  getDeliveryDays() {
    return 2;
  }
}

class OvernightShipping extends ShippingStrategy {
  calculateCost(order) {
    return order.total > 200 ? 19.99 : 29.99;
  }

  getDeliveryDays() {
    return 1;
  }
}

const shippingStrategies = {
  standard: new StandardShipping(),
  express: new ExpressShipping(),
  overnight: new OvernightShipping()
};

function calculateShipping(order, shippingMethod) {
  const strategy = shippingStrategies[shippingMethod];
  if (!strategy) {
    throw new Error('Unknown shipping method');
  }
  return strategy.calculateCost(order);
}

function getDeliveryDays(shippingMethod) {
  const strategy = shippingStrategies[shippingMethod];
  if (!strategy) {
    throw new Error('Unknown shipping method');
  }
  return strategy.getDeliveryDays();
}
```

**Decompose Conditional**

Break complex conditional expressions into named variables or functions for clarity.

```javascript
// Before: Complex conditional
if (user.age >= 18 && user.hasLicense && !user.suspended && user.insuranceValid) {
  allowRental();
}

// After: Decomposed
const isEligibleAge = user.age >= 18;
const hasValidLicense = user.hasLicense && !user.suspended;
const hasValidInsurance = user.insuranceValid;

if (isEligibleAge && hasValidLicense && hasValidInsurance) {
  allowRental();
}

// Even better: Extract function
function canRentCar(user) {
  return (
    user.age >= 18 &&
    user.hasLicense &&
    !user.suspended &&
    user.insuranceValid
  );
}

if (canRentCar(user)) {
  allowRental();
}
```

**Introduce Parameter Object**

Replace multiple parameters with a single object parameter for better readability and flexibility.

```javascript
// Before: Many parameters
function createUser(name, email, age, address, phone, preferences) {
  // ...
}

createUser('John', 'john@example.com', 30, '123 Main St', '555-1234', { theme: 'dark' });

// After: Parameter object
function createUser({ name, email, age, address, phone, preferences }) {
  // ...
}

createUser({
  name: 'John',
  email: 'john@example.com',
  age: 30,
  address: '123 Main St',
  phone: '555-1234',
  preferences: { theme: 'dark' }
});

// Even better: With default values
function createUser({
  name,
  email,
  age = 18,
  address = '',
  phone = '',
  preferences = {}
}) {
  // ...
}
```

---

## Hands-on Design Pattern Projects

### Refactor Backend Services

**Problem**

Backend services have tight coupling, making them hard to test and maintain. Services directly instantiate dependencies and have mixed responsibilities.

**Solution**

Apply Dependency Injection and Repository Pattern to decouple services from data access and external dependencies.

```javascript
// Before: Tightly coupled service
class OrderService {
  constructor() {
    this.database = new Database(); // Hard dependency
    this.emailService = new EmailService(); // Hard dependency
    this.paymentGateway = new StripeGateway(); // Hard dependency
  }

  async createOrder(orderData) {
    // Validation
    if (!orderData.userId || !orderData.items || orderData.items.length === 0) {
      throw new Error('Invalid order data');
    }

    // Calculate total
    const total = orderData.items.reduce((sum, item) => {
      return sum + item.price * item.quantity;
    }, 0);

    // Process payment
    const paymentResult = await this.paymentGateway.charge(total, orderData.paymentToken);

    if (!paymentResult.success) {
      throw new Error('Payment failed');
    }

    // Save order
    const order = {
      ...orderData,
      total,
      status: 'confirmed',
      paymentId: paymentResult.transactionId,
      createdAt: new Date()
    };

    const savedOrder = await this.database.save('orders', order);

    // Send confirmation email
    const user = await this.database.findById('users', orderData.userId);
    await this.emailService.sendOrderConfirmation(user.email, savedOrder);

    return savedOrder;
  }
}

// After: Refactored with DI and Repository pattern
// Repository interface
class OrderRepository {
  async save(order) {
    throw new Error('Must implement save()');
  }

  async findById(id) {
    throw new Error('Must implement findById()');
  }
}

class DatabaseOrderRepository extends OrderRepository {
  constructor(database) {
    super();
    this.database = database;
  }

  async save(order) {
    return await this.database.save('orders', order);
  }

  async findById(id) {
    return await this.database.findById('orders', id);
  }
}

// Refactored service
class OrderService {
  constructor(
    orderRepository,
    userRepository,
    paymentGateway,
    emailService,
    orderValidator
  ) {
    this.orderRepository = orderRepository;
    this.userRepository = userRepository;
    this.paymentGateway = paymentGateway;
    this.emailService = emailService;
    this.orderValidator = orderValidator;
  }

  async createOrder(orderData) {
    // Validate
    this.orderValidator.validate(orderData);

    // Calculate total
    const total = this.calculateTotal(orderData.items);

    // Process payment
    const paymentResult = await this.paymentGateway.charge(total, orderData.paymentToken);

    if (!paymentResult.success) {
      throw new Error('Payment failed');
    }

    // Save order
    const order = this.buildOrder(orderData, total, paymentResult.transactionId);
    const savedOrder = await this.orderRepository.save(order);

    // Send confirmation
    await this.sendConfirmation(savedOrder);

    return savedOrder;
  }

  calculateTotal(items) {
    return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
  }

  buildOrder(orderData, total, paymentId) {
    return {
      ...orderData,
      total,
      status: 'confirmed',
      paymentId,
      createdAt: new Date()
    };
  }

  async sendConfirmation(order) {
    const user = await this.userRepository.findById(order.userId);
    await this.emailService.sendOrderConfirmation(user.email, order);
  }
}

// Dependency container
class ServiceContainer {
  constructor() {
    this.database = new Database();
    this.emailService = new EmailService();
    this.paymentGateway = new StripeGateway();
    this.orderValidator = new OrderValidator();

    this.orderRepository = new DatabaseOrderRepository(this.database);
    this.userRepository = new DatabaseUserRepository(this.database);

    this.orderService = new OrderService(
      this.orderRepository,
      this.userRepository,
      this.paymentGateway,
      this.emailService,
      this.orderValidator
    );
  }

  getOrderService() {
    return this.orderService;
  }
}

// Usage
const container = new ServiceContainer();
const orderService = container.getOrderService();
const order = await orderService.createOrder(orderData);
```

---

### Build In-Memory Cache Layer

**Problem**

Frequent database queries for the same data cause performance issues and unnecessary load on the database.

**Solution**

Implement a cache layer using the Proxy and Decorator patterns to cache expensive operations.

```javascript
// Cache implementation with expiration
class Cache {
  constructor(defaultTTL = 3600) {
    this.cache = new Map();
    this.defaultTTL = defaultTTL;
  }

  set(key, value, ttl = this.defaultTTL) {
    const expiry = Date.now() + ttl * 1000;
    this.cache.set(key, { value, expiry });
  }

  get(key) {
    const item = this.cache.get(key);
    if (!item) {
      return null;
    }

    if (Date.now() > item.expiry) {
      this.cache.delete(key);
      return null;
    }

    return item.value;
  }

  has(key) {
    return this.get(key) !== null;
  }

  delete(key) {
    this.cache.delete(key);
  }

  clear() {
    this.cache.clear();
  }

  // Cleanup expired entries
  cleanup() {
    const now = Date.now();
    for (const [key, item] of this.cache.entries()) {
      if (now > item.expiry) {
        this.cache.delete(key);
      }
    }
  }
}

// Cache decorator for repositories
class CachedRepository {
  constructor(repository, cache, ttl = 3600) {
    this.repository = repository;
    this.cache = cache;
    this.ttl = ttl;
  }

  async findById(id) {
    const cacheKey = `${this.constructor.name}:findById:${id}`;
    const cached = this.cache.get(cacheKey);

    if (cached) {
      console.log(`Cache hit: ${cacheKey}`);
      return cached;
    }

    console.log(`Cache miss: ${cacheKey}`);
    const result = await this.repository.findById(id);
    if (result) {
      this.cache.set(cacheKey, result, this.ttl);
    }

    return result;
  }

  async findAll() {
    const cacheKey = `${this.constructor.name}:findAll`;
    const cached = this.cache.get(cacheKey);

    if (cached) {
      console.log(`Cache hit: ${cacheKey}`);
      return cached;
    }

    console.log(`Cache miss: ${cacheKey}`);
    const result = await this.repository.findAll();
    this.cache.set(cacheKey, result, this.ttl);

    return result;
  }

  async save(entity) {
    const result = await this.repository.save(entity);

    // Invalidate relevant cache entries
    this.cache.delete(`${this.constructor.name}:findAll`);
    if (entity.id) {
      this.cache.delete(`${this.constructor.name}:findById:${entity.id}`);
    }

    return result;
  }

  async delete(id) {
    await this.repository.delete(id);

    // Invalidate cache entries
    this.cache.delete(`${this.constructor.name}:findAll`);
    this.cache.delete(`${this.constructor.name}:findById:${id}`);
  }
}

// Usage
const cache = new Cache();
const userRepository = new DatabaseUserRepository(database);
const cachedUserRepository = new CachedRepository(userRepository, cache, 300);

// First call - cache miss
const user1 = await cachedUserRepository.findById(1);

// Second call - cache hit
const user2 = await cachedUserRepository.findById(1);

// Invalidate cache on save
await cachedUserRepository.save({ id: 1, name: 'Updated' });

// Next call - cache miss (invalidated)
const user3 = await cachedUserRepository.findById(1);

// Advanced: LRU Cache with size limit
class LRUCache {
  constructor(maxSize = 100, defaultTTL = 3600) {
    this.maxSize = maxSize;
    this.defaultTTL = defaultTTL;
    this.cache = new Map();
    this.accessOrder = new Map();
  }

  set(key, value, ttl = this.defaultTTL) {
    const expiry = Date.now() + ttl * 1000;

    // Remove oldest if at capacity
    if (this.cache.size >= this.maxSize && !this.cache.has(key)) {
      const oldestKey = this.accessOrder.keys().next().value;
      this.delete(oldestKey);
    }

    this.cache.set(key, { value, expiry });
    this.accessOrder.set(key, Date.now());
  }

  get(key) {
    const item = this.cache.get(key);
    if (!item) {
      return null;
    }

    if (Date.now() > item.expiry) {
      this.delete(key);
      return null;
    }

    // Update access order
    this.accessOrder.delete(key);
    this.accessOrder.set(key, Date.now());

    return item.value;
  }

  delete(key) {
    this.cache.delete(key);
    this.accessOrder.delete(key);
  }

  clear() {
    this.cache.clear();
    this.accessOrder.clear();
  }

  size() {
    return this.cache.size;
  }
}
```

---

### Refactor "God Object"

**Problem**

A single class handles too many responsibilities (validation, data access, business logic, UI rendering), making it difficult to maintain, test, and understand.

**Solution**

Apply Single Responsibility Principle and extract focused classes for each responsibility.

```javascript
// Before: God Object
class UserManager {
  constructor() {
    this.users = [];
    this.database = new Database();
  }

  // Data access
  async loadUsers() {
    this.users = await this.database.getAll('users');
  }

  async saveUser(user) {
    await this.database.save('users', user);
  }

  async deleteUser(id) {
    await this.database.delete('users', id);
  }

  // Validation
  validateUser(user) {
    if (!user.name || user.name.length < 2) {
      throw new Error('Invalid name');
    }
    if (!user.email || !this.isValidEmail(user.email)) {
      throw new Error('Invalid email');
    }
    if (!user.age || user.age < 18) {
      throw new Error('Invalid age');
    }
    return true;
  }

  isValidEmail(email) {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  }

  // Business logic
  createUser(userData) {
    this.validateUser(userData);
    const user = {
      id: Date.now(),
      ...userData,
      createdAt: new Date(),
      status: 'active'
    };
    this.users.push(user);
    return user;
  }

  updateUser(id, updates) {
    const index = this.users.findIndex(u => u.id === id);
    if (index === -1) {
      throw new Error('User not found');
    }
    this.validateUser({ ...this.users[index], ...updates });
    this.users[index] = { ...this.users[index], ...updates, updatedAt: new Date() };
    return this.users[index];
  }

  deactivateUser(id) {
    const user = this.users.find(u => u.id === id);
    if (!user) {
      throw new Error('User not found');
    }
    user.status = 'inactive';
    user.deactivatedAt = new Date();
    return user;
  }

  // UI/Rendering
  renderUserList() {
    return this.users.map(user => `
      <div class="user" data-id="${user.id}">
        <h3>${user.name}</h3>
        <p>${user.email}</p>
        <span class="status ${user.status}">${user.status}</span>
      </div>
    `).join('');
  }

  renderUserCard(user) {
    return `
      <div class="user-card">
        <h3>${user.name}</h3>
        <p>Email: ${user.email}</p>
        <p>Age: ${user.age}</p>
        <p>Status: ${user.status}</p>
        <button onclick="editUser(${user.id})">Edit</button>
        <button onclick="deleteUser(${user.id})">Delete</button>
      </div>
    `;
  }
}

// After: Separated concerns

// 1. Repository - Data access
class UserRepository {
  constructor(database) {
    this.database = database;
  }

  async findAll() {
    return await this.database.getAll('users');
  }

  async findById(id) {
    return await this.database.findById('users', id);
  }

  async save(user) {
    return await this.database.save('users', user);
  }

  async delete(id) {
    return await this.database.delete('users', id);
  }
}

// 2. Validator - Validation logic
class UserValidator {
  validate(user) {
    const errors = [];

    if (!user.name || user.name.length < 2) {
      errors.push('Name must be at least 2 characters');
    }

    if (!user.email || !this.isValidEmail(user.email)) {
      errors.push('Invalid email address');
    }

    if (!user.age || user.age < 18) {
      errors.push('User must be at least 18 years old');
    }

    if (errors.length > 0) {
      throw new ValidationError(errors.join(', '));
    }

    return true;
  }

  isValidEmail(email) {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  }
}

class ValidationError extends Error {
  constructor(message) {
    super(message);
    this.name = 'ValidationError';
  }
}

// 3. Service - Business logic
class UserService {
  constructor(userRepository, userValidator) {
    this.userRepository = userRepository;
    this.userValidator = userValidator;
  }

  async create(userData) {
    this.userValidator.validate(userData);

    const user = {
      id: Date.now(),
      ...userData,
      createdAt: new Date(),
      status: 'active'
    };

    return await this.userRepository.save(user);
  }

  async update(id, updates) {
    const existing = await this.userRepository.findById(id);
    if (!existing) {
      throw new Error('User not found');
    }

    const updatedUser = { ...existing, ...updates, updatedAt: new Date() };
    this.userValidator.validate(updatedUser);

    return await this.userRepository.save(updatedUser);
  }

  async deactivate(id) {
    const user = await this.userRepository.findById(id);
    if (!user) {
      throw new Error('User not found');
    }

    const deactivatedUser = {
      ...user,
      status: 'inactive',
      deactivatedAt: new Date()
    };

    return await this.userRepository.save(deactivatedUser);
  }

  async delete(id) {
    return await this.userRepository.delete(id);
  }
}

// 4. View - UI rendering
class UserView {
  renderList(users) {
    return users.map(user => `
      <div class="user" data-id="${user.id}">
        <h3>${this.escapeHtml(user.name)}</h3>
        <p>${this.escapeHtml(user.email)}</p>
        <span class="status ${user.status}">${user.status}</span>
      </div>
    `).join('');
  }

  renderCard(user) {
    return `
      <div class="user-card">
        <h3>${this.escapeHtml(user.name)}</h3>
        <p>Email: ${this.escapeHtml(user.email)}</p>
        <p>Age: ${user.age}</p>
        <p>Status: ${user.status}</p>
        <button onclick="editUser(${user.id})">Edit</button>
        <button onclick="deleteUser(${user.id})">Delete</button>
      </div>
    `;
  }

  escapeHtml(text) {
    const div = document.createElement('div');
    div.textContent = text;
    return div.innerHTML;
  }
}

// 5. Controller - Coordinates everything
class UserController {
  constructor(userService, userView) {
    this.userService = userService;
    this.userView = userView;
  }

  async createUser(userData) {
    try {
      const user = await this.userService.create(userData);
      return { success: true, user };
    } catch (error) {
      return { success: false, error: error.message };
    }
  }

  async renderUserList() {
    const users = await this.userService.findAll();
    return this.userView.renderList(users);
  }

  async renderUserCard(id) {
    const user = await this.userService.findById(id);
    return this.userView.renderCard(user);
  }
}

// Usage
const database = new Database();
const userRepository = new UserRepository(database);
const userValidator = new UserValidator();
const userService = new UserService(userRepository, userValidator);
const userView = new UserView();
const userController = new UserController(userService, userView);

// Create user
const result = await userController.createUser({
  name: 'John Doe',
  email: 'john@example.com',
  age: 30
});

// Render list
const userListHTML = await userController.renderUserList();
```

---

### Implement Event System

**Problem**

Components need to communicate without tight coupling. Direct dependencies make the system rigid and hard to test.

**Solution**

Implement an event system using the Observer and Mediator patterns to enable loose coupling between components.

```javascript
// Event emitter implementation
class EventEmitter {
  constructor() {
    this.events = new Map();
    this.onceEvents = new Map();
  }

  on(event, listener) {
    if (!this.events.has(event)) {
      this.events.set(event, []);
    }
    this.events.get(event).push(listener);
    return () => this.off(event, listener);
  }

  once(event, listener) {
    const onceWrapper = (...args) => {
      listener(...args);
      this.off(event, onceWrapper);
    };
    this.on(event, onceWrapper);
  }

  off(event, listener) {
    if (!this.events.has(event)) {
      return;
    }
    this.events.set(
      event,
      this.events.get(event).filter(l => l !== listener)
    );
  }

  emit(event, ...args) {
    if (!this.events.has(event)) {
      return false;
    }
    this.events.get(event).forEach(listener => {
      try {
        listener(...args);
      } catch (error) {
        console.error(`Error in event listener for "${event}":`, error);
      }
    });
    return true;
  }

  removeAllListeners(event) {
    if (event) {
      this.events.delete(event);
    } else {
      this.events.clear();
    }
  }

  listenerCount(event) {
    return this.events.has(event) ? this.events.get(event).length : 0;
  }
}

// Event bus for application-wide events
class EventBus {
  constructor() {
    this.emitter = new EventEmitter();
    this.eventLog = [];
    this.maxLogSize = 100;
  }

  subscribe(event, listener) {
    return this.emitter.on(event, listener);
  }

  publish(event, data) {
    this.logEvent(event, data);
    return this.emitter.emit(event, data);
  }

  unsubscribe(event, listener) {
    this.emitter.off(event, listener);
  }

  logEvent(event, data) {
    this.eventLog.push({
      event,
      data,
      timestamp: new Date()
    });

    if (this.eventLog.length > this.maxLogSize) {
      this.eventLog.shift();
    }
  }

  getEventLog() {
    return [...this.eventLog];
  }

  clearEventLog() {
    this.eventLog = [];
  }
}

// Domain events
class DomainEvents {
  static USER_CREATED = 'user.created';
  static USER_UPDATED = 'user.updated';
  static USER_DELETED = 'user.deleted';
  static ORDER_CREATED = 'order.created';
  static ORDER_PAID = 'order.paid';
  static ORDER_SHIPPED = 'order.shipped';
  static PAYMENT_FAILED = 'payment.failed';
}

// Event handlers
class UserEventHandler {
  constructor(emailService, analyticsService) {
    this.emailService = emailService;
    this.analyticsService = analyticsService;
  }

  handleUserCreated(eventBus) {
    eventBus.subscribe(DomainEvents.USER_CREATED, (user) => {
      this.emailService.sendWelcomeEmail(user.email);
      this.analyticsService.track('user_created', { userId: user.id });
    });
  }

  handleUserDeleted(eventBus) {
    eventBus.subscribe(DomainEvents.USER_DELETED, (user) => {
      this.emailService.sendGoodbyeEmail(user.email);
      this.analyticsService.track('user_deleted', { userId: user.id });
    });
  }
}

class OrderEventHandler {
  constructor(emailService, inventoryService, shippingService) {
    this.emailService = emailService;
    this.inventoryService = inventoryService;
    this.shippingService = shippingService;
  }

  handleOrderCreated(eventBus) {
    eventBus.subscribe(DomainEvents.ORDER_CREATED, (order) => {
      this.inventoryService.reserveItems(order.items);
      this.emailService.sendOrderConfirmation(order.userEmail, order);
    });
  }

  handleOrderPaid(eventBus) {
    eventBus.subscribe(DomainEvents.ORDER_PAID, (order) => {
      this.shippingService.createShipment(order);
    });
  }

  handlePaymentFailed(eventBus) {
    eventBus.subscribe(DomainEvents.PAYMENT_FAILED, (data) => {
      this.emailService.sendPaymentFailedEmail(data.userEmail, data.orderId);
      this.inventoryService.releaseReservation(data.orderId);
    });
  }
}

// Service that publishes events
class UserService {
  constructor(userRepository, eventBus) {
    this.userRepository = userRepository;
    this.eventBus = eventBus;
  }

  async create(userData) {
    const user = await this.userRepository.save(userData);
    this.eventBus.publish(DomainEvents.USER_CREATED, user);
    return user;
  }

  async update(id, updates) {
    const user = await this.userRepository.update(id, updates);
    this.eventBus.publish(DomainEvents.USER_UPDATED, user);
    return user;
  }

  async delete(id) {
    const user = await this.userRepository.delete(id);
    this.eventBus.publish(DomainEvents.USER_DELETED, user);
    return user;
  }
}

class OrderService {
  constructor(orderRepository, eventBus) {
    this.orderRepository = orderRepository;
    this.eventBus = eventBus;
  }

  async create(orderData) {
    const order = await this.orderRepository.save(orderData);
    this.eventBus.publish(DomainEvents.ORDER_CREATED, order);
    return order;
  }

  async processPayment(orderId, paymentData) {
    const order = await this.orderRepository.findById(orderId);

    try {
      const paymentResult = await this.processPayment(order, paymentData);

      order.status = 'paid';
      order.paymentId = paymentResult.transactionId;
      await this.orderRepository.save(order);

      this.eventBus.publish(DomainEvents.ORDER_PAID, order);
      return order;
    } catch (error) {
      this.eventBus.publish(DomainEvents.PAYMENT_FAILED, {
        orderId,
        userEmail: order.userEmail,
        error: error.message
      });
      throw error;
    }
  }
}

// Usage
const eventBus = new EventBus();
const userRepository = new UserRepository(database);
const orderRepository = new OrderRepository(database);

const emailService = new EmailService();
const analyticsService = new AnalyticsService();
const inventoryService = new InventoryService();
const shippingService = new ShippingService();

const userService = new UserService(userRepository, eventBus);
const orderService = new OrderService(orderRepository, eventBus);

// Register event handlers
const userEventHandler = new UserEventHandler(emailService, analyticsService);
const orderEventHandler = new OrderEventHandler(emailService, inventoryService, shippingService);

userEventHandler.handleUserCreated(eventBus);
userEventHandler.handleUserDeleted(eventBus);
orderEventHandler.handleOrderCreated(eventBus);
orderEventHandler.handleOrderPaid(eventBus);
orderEventHandler.handlePaymentFailed(eventBus);

// Events will be automatically handled
const user = await userService.create({ name: 'John', email: 'john@example.com' });
// → Welcome email sent
// → Analytics tracked

const order = await orderService.create({ userId: user.id, items: [...] });
// → Inventory reserved
// → Confirmation email sent
```

---

## Quick Reference

**Design Pattern Decision Matrix**

| Pattern | Problem | When to Use |
|---------|---------|-------------|
| Singleton | Single instance needed | Shared resources, configuration, logging |
| Factory | Object creation complexity | Runtime object creation, families of objects |
| Builder | Complex object construction | Many optional parameters, step-by-step creation |
| Prototype | Expensive object creation | Cloning objects, similar initial states |
| Adapter | Incompatible interfaces | Third-party libraries, legacy code |
| Decorator | Dynamic behavior addition | Adding responsibilities at runtime |
| Facade | Complex subsystem | Simplifying complex APIs, layering |
| Proxy | Access control needed | Lazy loading, caching, remote access |
| Observer | One-to-many notifications | Event systems, reactive programming |
| Strategy | Interchangeable algorithms | Multiple ways to accomplish task |
| Command | Encapsulate requests | Undo/redo, macro operations, queuing |
| Iterator | Traverse collections | Uniform traversal, custom iteration |
| Mediator | Complex communication | Decoupling multiple objects |
| Repository | Data access abstraction | Database operations, testing |
| MVC | Separate concerns | Web applications, UI development |
| DI | Loose coupling | Large applications, testing |

**SOLID Principles Summary**

| Principle | Description | Key Benefit |
|-----------|-------------|-------------|
| SRP | One reason to change | Focused, maintainable classes |
| OCP | Open for extension, closed for modification | Easy to add features |
| LSP | Subtypes must be substitutable | Reliable inheritance |
| ISP | Clients shouldn't depend on unused interfaces | Loose coupling |
| DIP | Depend on abstractions | Flexible, testable code |

**Clean Code Checklist**

- [ ] Functions do one thing well
- [ ] Functions are short (< 20 lines)
- [ ] Names reveal intent
- [ ] No duplicate code (DRY)
- [ ] Code is simple (KISS)
- [ ] No unnecessary features (YAGNI)
- [ ] Classes have single responsibility
- [ ] Methods are small and focused
- [ ] Comments explain "why", not "what"
- [ ] Error handling is consistent
- [ ] Tests are written for critical paths
- [ ] Code is formatted consistently

**Refactoring Techniques**

| Technique | When to Use | Benefit |
|-----------|-------------|---------|
| Extract Method | Long functions | Improved readability |
| Extract Class | Large classes | Better separation of concerns |
| Replace Conditional with Polymorphism | Complex conditionals | Open/Closed Principle |
| Decompose Conditional | Complex expressions | Better readability |
| Introduce Parameter Object | Many parameters | Cleaner interfaces |
| Replace Magic Numbers with Constants | Hardcoded values | Maintainability |
| Extract Interface | Multiple implementations | Loose coupling |

**Code Quality Metrics**

- **Cyclomatic Complexity**: Number of independent paths through code (aim for < 10)
- **Function Length**: Lines of code per function (aim for < 20)
- **Class Length**: Lines of code per class (aim for < 300)
- **Parameter Count**: Parameters per function (aim for < 4)
- **Nesting Depth**: Levels of nesting (aim for < 4)
- **Duplication**: Percentage of duplicated code (aim for < 5%)

**Common Code Smells**

- **Long Method**: Functions that do too much
- **Large Class**: Classes with too many responsibilities
- **Duplicated Code**: Same logic in multiple places
- **Long Parameter List**: Too many parameters
- **Feature Envy**: Method more interested in other class
- **Data Clumps**: Variables that appear together
- **Primitive Obsession**: Using primitives instead of objects
- **Shotgun Surgery**: Changes require many small changes
- **God Object**: Class that knows too much or does too much
- **Dead Code**: Code that's never executed
