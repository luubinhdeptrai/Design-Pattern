# Facade Pattern

## 1. Definition
The **Facade** pattern is a structural design pattern that provides a **simplified, unified interface** to a set of interfaces in a subsystem. It defines a higher-level API that makes the subsystem easier to use.

## 2. Intent (Goal of the pattern)
- Provide a **single entry point** to a complex subsystem.
- Reduce coupling between clients and subsystem implementation details.
- Improve usability by exposing only what the client needs.

## 3. Problem it solves
In real systems, functionality is often implemented across multiple classes/services (a “subsystem”).

Without a Facade:
- Client code must understand subsystem details and call many objects in the correct order.
- Client code becomes tightly coupled to subsystem classes.
- The subsystem becomes harder to change because many clients depend on its internal structure.

Facade solves this by placing a **simple wrapper** in front of the subsystem, so clients interact with **one object** instead of many.

## 4. Motivation (real-world analogy if possible)
Imagine booking a trip:
- Without a travel agent, you personally coordinate flights, hotel, car rental, insurance, and payment—many steps and vendors.
- A travel agent offers a single “bookTrip()” service. Internally they still coordinate the vendors, but you don’t need to.

The travel agent is the **Facade**.

## 5. Structure (explain the roles in the pattern)
- **Client**: The code that needs subsystem functionality.
- **Facade**: The simplified interface exposed to the client. It orchestrates calls to the subsystem.
- **Subsystem classes**: The complex set of classes that implement the actual behavior.

Key idea: the Facade does not replace subsystem classes; it **organizes access** to them.

## 6. UML diagram explanation
Typical relationships:
- Client depends on the Facade.
- Facade depends on multiple subsystem classes.
- Subsystem classes may collaborate with each other (optionally), but the client is insulated from that.

In UML terms:
- Client → Facade (association / dependency)
- Facade → SubsystemA / SubsystemB / SubsystemC (dependencies)

## 7. Implementation example (preferably in Java)
Example scenario: “watch a movie” using a home theater subsystem.

### Subsystem classes
```java
package patterns.facade;

class Amplifier {
    void on() { System.out.println("Amplifier: on"); }
    void setVolume(int level) { System.out.println("Amplifier: volume=" + level); }
    void off() { System.out.println("Amplifier: off"); }
}

class Projector {
    void on() { System.out.println("Projector: on"); }
    void wideScreenMode() { System.out.println("Projector: widescreen mode"); }
    void off() { System.out.println("Projector: off"); }
}

class StreamingPlayer {
    void on() { System.out.println("Player: on"); }
    void play(String movie) { System.out.println("Player: playing \"" + movie + "\""); }
    void stop() { System.out.println("Player: stop"); }
    void off() { System.out.println("Player: off"); }
}

class TheaterLights {
    void dim(int level) { System.out.println("Lights: dim to " + level + "%"); }
    void on() { System.out.println("Lights: on"); }
}
```

### Facade
```java
package patterns.facade;

public class HomeTheaterFacade {
    private final Amplifier amplifier;
    private final Projector projector;
    private final StreamingPlayer player;
    private final TheaterLights lights;

    public HomeTheaterFacade(Amplifier amplifier,
                             Projector projector,
                             StreamingPlayer player,
                             TheaterLights lights) {
        this.amplifier = amplifier;
        this.projector = projector;
        this.player = player;
        this.lights = lights;
    }

    public void watchMovie(String movie) {
        System.out.println("=== Get ready to watch a movie ===");
        lights.dim(10);
        projector.on();
        projector.wideScreenMode();
        amplifier.on();
        amplifier.setVolume(7);
        player.on();
        player.play(movie);
    }

    public void endMovie() {
        System.out.println("=== Shutting movie theater down ===");
        player.stop();
        player.off();
        amplifier.off();
        projector.off();
        lights.on();
    }
}
```

### Client
```java
package patterns.facade;

public class Demo {
    public static void main(String[] args) {
        Amplifier amplifier = new Amplifier();
        Projector projector = new Projector();
        StreamingPlayer player = new StreamingPlayer();
        TheaterLights lights = new TheaterLights();

        HomeTheaterFacade theater = new HomeTheaterFacade(amplifier, projector, player, lights);

        theater.watchMovie("Inception");
        System.out.println();
        theater.endMovie();
    }
}
```

## 8. Step-by-step explanation of the code
1. **Subsystem classes** (`Amplifier`, `Projector`, `StreamingPlayer`, `TheaterLights`) each provide a focused capability.
2. The **client** could call them directly, but it would need to know:
   - what order to call them,
   - which modes/volume/dim levels to set,
   - how to properly shut everything down.
3. The **Facade** (`HomeTheaterFacade`) receives the subsystem objects (via constructor injection).
4. The client calls a single method:
   - `watchMovie(movie)`
   - `endMovie()`
5. Inside those methods, the Facade orchestrates subsystem calls:
   - turns on/sets up devices in a safe, correct order,
   - hides details like “widescreen mode” and “dim to 10%”.
6. If the subsystem changes (e.g., new player type, new initialization steps), the client can often stay the same—only the Facade implementation changes.

## 9. Advantages
- **Simplifies usage**: clients call a small number of methods.
- **Reduces coupling**: clients depend on the Facade, not many subsystem classes.
- **Improves readability**: client code becomes intent-focused (`watchMovie()` reads like the business action).
- **Supports safer workflows**: the Facade can enforce ordering and handle setup/teardown consistently.

## 10. Disadvantages
- Risk of becoming a **“god object”** if the Facade grows too large.
- Can **hide important features** if it oversimplifies (clients may still need direct access to subsystem APIs).
- May add an extra layer to debug if misused (though often worth it).

## 11. When to use it
- When a subsystem is **complex** and you want an easy-to-use entry point.
- When you want to **layer** your system (e.g., UI → Facade → services → repositories).
- When you want to **reduce dependencies** between clients and internal modules.
- When you have a **legacy subsystem** and want a clean interface without rewriting internals.

## 12. When not to use it
- When the subsystem is already simple (Facade adds unnecessary indirection).
- When clients need many specialized operations that don’t fit a simplified API.
- When you actually need **fine-grained control** over subsystem behavior in the client.

## 13. Real-world examples
- **Libraries/SDKs** that provide a single class as an “easy API” over a set of lower-level APIs.
- **Application services** in layered architectures:
  - a `UserService` that coordinates repositories, password hashing, email, auditing, etc.
- **Framework convenience APIs**:
  - e.g., a single `Database` object that internally manages connections, transactions, and statements.

## 14. Related patterns
- **Adapter**: Changes one interface into another expected by the client. Facade simplifies an existing subsystem without necessarily changing interfaces.
- **Mediator**: Coordinates interactions between objects to reduce direct coupling. Facade simplifies access for clients; Mediator manages object-to-object collaboration.
- **Proxy**: Controls access to an object (lazy loading, security, remote proxy). Facade focuses on simplifying a subsystem.
- **Abstract Factory**: Often used to create subsystem objects that a Facade then uses.
